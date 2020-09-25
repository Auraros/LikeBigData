## Hadoop RPC 类详解

```
主要由以下三类组成：
RPC: 对外编程接口
Client: 客户端实现
Server: 服务器实现
```

### ipc.RPC类分析

ipc.RPC类中有一些内部类，为了大家对RPC类有个初步的印象，就先罗列几个我们感兴趣的分析一下吧：

- **Invocation** ：用于封装方法名和参数，作为数据传输层。
- **ClientCache** ：用于存储client对象，用socket factory作为hash key,存储结构为hashMap <SocketFactory, Client>。
- **Invoker** ：是动态代理中的调用实现类，继承了InvocationHandler.
- **Server** ：是ipc.Server的实现类。

```java
public Object invoke(Object proxy, Method method, Object[] args)
      throws Throwable {
      •••
      ObjectWritable value = (ObjectWritable)
        client.call(new Invocation(method, args), remoteId);
      •••
      return value.get();
    }
```

一般我们看到的[动态代理](http://my.oschina.net/hosee/blog/656945)的invoke()方法中总会有 method.invoke(ac, arg); 这句代码。而上面代码中却没有，这是为什么呢？其实使用 method.invoke(ac, arg); 是在本地JVM中调用；而在hadoop中，是将数据发送给服务端，服务端将处理的结果再返回给客户端，所以这里的invoke()方法必然需要进行网络通信。而网络通信就是下面的这段代码实现的：

```java
ObjectWritable value = (ObjectWritable)
client.call(new Invocation(method, args), remoteId);
```



### ipc.Client

> 提出三个问题：
>
> 1. 客户端和服务端的连接是怎样建立的？
> 2. 客户端是怎样给服务端发送数据的？
> 3. 客户端是怎样获取服务端的返回数据的？

#### 客户端和服务端的连接是怎样建立的？

```java
public Writable call(Writable param, ConnectionId remoteId)  
                       throws InterruptedException, IOException {
    Call call = new Call(param);       //将传入的数据封装成call对象
    Connection connection = getConnection(remoteId, call);   //获得一个连接
    connection.sendParam(call);     // 向服务端发送call对象
    boolean interrupted = false;
    synchronized (call) {
      while (!call.done) {
        try {
          call.wait(); // 等待结果的返回，在Call类的callComplete()方法里有notify()方法用于唤醒线程
        } catch (InterruptedException ie) {
          // 因中断异常而终止，设置标志interrupted为true
          interrupted = true;
        }
      }
      if (interrupted) {
        Thread.currentThread().interrupt();
      }

      if (call.error != null) {
        if (call.error instanceof RemoteException) {
          call.error.fillInStackTrace();
          throw call.error;
        } else { // 本地异常
          throw wrapException(remoteId.getAddress(), call.error);
        }
      } else {
        return call.value; //返回结果数据
      }
    }
  }
```

具体代码的作用我已做了注释，所以这里不再赘述。但到目前为止，你依然不知道RPC机制底层的网络连接是怎么建立的。分析代码后，我们会发现和网络通信有关的代码只会是下面的两句了：

```java
Connection connection = getConnection(remoteId, call);   //获得一个连接
connection.sendParam(call);      // 向服务端发送call对象
```

先看看是怎么获得一个到服务端的连接吧，下面贴出ipc.Client类中的getConnection()方法。

```java
private Connection getConnection(ConnectionId remoteId,
                                   Call call)
                                   throws IOException, InterruptedException {
    if (!running.get()) {
      // 如果client关闭了
      throw new IOException("The client is stopped");
    }
    Connection connection;
//如果connections连接池中有对应的连接对象，就不需重新创建了；如果没有就需重新创建一个连接对象。
//但请注意，该//连接对象只是存储了remoteId的信息，其实还并没有和服务端建立连接。
    do {
      synchronized (connections) {
        connection = connections.get(remoteId);
        if (connection == null) {
          connection = new Connection(remoteId);
          connections.put(remoteId, connection);
        }
      }
    } while (!connection.addCall(call)); //将call对象放入对应连接中的calls池，就不贴出源码了
   //这句代码才是真正的完成了和服务端建立连接哦~
    connection.setupIOstreams();
    return connection;
  }
```

下面贴出Client.Connection类中的setupIOstreams()方法：

```java
private synchronized void setupIOstreams() throws InterruptedException {
   •••
      try {
       •••
        while (true) {
          setupConnection();  //建立连接
          InputStream inStream = NetUtils.getInputStream(socket);     //获得输入流
          OutputStream outStream = NetUtils.getOutputStream(socket);  //获得输出流
          writeRpcHeader(outStream);
          •••
          this.in = new DataInputStream(new BufferedInputStream
              (new PingInputStream(inStream)));   //将输入流装饰成DataInputStream
          this.out = new DataOutputStream
          (new BufferedOutputStream(outStream));   //将输出流装饰成DataOutputStream
          writeHeader();
          // 跟新活动时间
          touch();
          //当连接建立时，启动接受线程等待服务端传回数据，注意：Connection继承了Tread
          start();
          return;
        }
      } catch (IOException e) {
        markClosed(e);
        close();
      }
    }
```

再有一步我们就知道客户端的连接是怎么建立的啦，下面贴出Client.Connection类中的setupConnection()方法：

```java
private synchronized void setupConnection() throws IOException {
      short ioFailures = 0;
      short timeoutFailures = 0;
      while (true) {
        try {
          this.socket = socketFactory.createSocket(); //终于看到创建socket的方法了
          this.socket.setTcpNoDelay(tcpNoDelay);
         •••
          // 设置连接超时为20s
          NetUtils.connect(this.socket, remoteId.getAddress(), 20000);
          this.socket.setSoTimeout(pingInterval);
          return;
        } catch (SocketTimeoutException toe) {
          /* 设置最多连接重试为45次。
           * 总共有20s*45 = 15 分钟的重试时间。
           */
          handleConnectionFailure(timeoutFailures++, 45, toe);
        } catch (IOException ie) {
          handleConnectionFailure(ioFailures++, maxRetries, ie);
        }
      }
    }
```

终于，我们知道了客户端的连接是怎样建立的了，其实就是创建一个普通的socket进行通信。



#### 客户端是怎样给服务端发送数据的？

下面贴出Client.Connection类的sendParam()方法

```java
public void sendParam(Call call) {
      if (shouldCloseConnection.get()) {
        return;
      }
      DataOutputBuffer d=null;
      try {
        synchronized (this.out) {
          if (LOG.isDebugEnabled())
            LOG.debug(getName() + " sending #" + call.id);
          //创建一个缓冲区
          d = new DataOutputBuffer();
          d.writeInt(call.id);
          call.param.write(d);
          byte[] data = d.getData();
          int dataLength = d.getLength();
          out.writeInt(dataLength);        //首先写出数据的长度
          out.write(data, 0, dataLength); //向服务端写数据
          out.flush();
        }
      } catch(IOException e) {
        markClosed(e);
      } finally {
        IOUtils.closeStream(d);
      }
    }
```

#### 客户端是怎样获取服务端的返回数据的？

```
方法一：  
  public void run() {
      •••
      while (waitForWork()) {
        receiveResponse();  //具体的处理方法
      }
      close();
     •••
}

方法二：
private void receiveResponse() {
      if (shouldCloseConnection.get()) {
        return;
      }
      touch();
      try {
        int id = in.readInt();                    // 阻塞读取id
        if (LOG.isDebugEnabled())
          LOG.debug(getName() + " got value #" + id);
          Call call = calls.get(id);    //在calls池中找到发送时的那个对象
        int state = in.readInt();     // 阻塞读取call对象的状态
        if (state == Status.SUCCESS.state) {
          Writable value = ReflectionUtils.newInstance(valueClass, conf);
          value.readFields(in);           // 读取数据
        //将读取到的值赋给call对象，同时唤醒Client等待线程，贴出setValue()代码方法三
          call.setValue(value);              
          calls.remove(id);               //删除已处理的call    
        } else if (state == Status.ERROR.state) {
        •••
        } else if (state == Status.FATAL.state) {
        •••
        }
      } catch (IOException e) {
        markClosed(e);
      }
}

方法三：
public synchronized void setValue(Writable value) {
      this.value = value;
      callComplete();   //具体实现
}
protected synchronized void callComplete() {
      this.done = true;
      notify();         // 唤醒client等待线程
}
```

完成的功能主要是：启动一个处理线程，读取从服务端传来的call对象，将call对象读取完毕后，唤醒client处理线程。就这么简单，客户端就获取了服务端返回的数据了哦~。客户端的源码分析就到这里了哦，下面我们来分析Server端的源码吧。



###  ipc.Server源码分析

> 为了让大家对ipc.Server有个初步的了解，我们先分析一下它的几个内部类吧：
>
> **Call** ：用于存储客户端发来的请求
> **Listener** ： 监听类，用于监听客户端发来的请求，同时Listener内部还有一个静态类，Listener.Reader，当监听器监听到用户请求，便让Reader读取用户请求。
> **Responder** ：响应RPC请求类，请求处理完毕，由Responder发送给请求客户端。
> **Connection** ：连接类，真正的客户端请求读取逻辑在这个类中。
> **Handler** ：请求处理类，会循环阻塞读取callQueue中的call对象，并对其进行操作。

```java
private void initialize(Configuration conf) throws IOException {
   •••
    // 创建 rpc server
    InetSocketAddress dnSocketAddr = getServiceRpcServerAddress(conf);
    if (dnSocketAddr != null) {
      int serviceHandlerCount =
        conf.getInt(DFSConfigKeys.DFS_NAMENODE_SERVICE_HANDLER_COUNT_KEY,
                    DFSConfigKeys.DFS_NAMENODE_SERVICE_HANDLER_COUNT_DEFAULT);
      //获得serviceRpcServer
      this.serviceRpcServer = RPC.getServer(this, dnSocketAddr.getHostName(), 
          dnSocketAddr.getPort(), serviceHandlerCount,
          false, conf, namesystem.getDelegationTokenSecretManager());
      this.serviceRPCAddress = this.serviceRpcServer.getListenerAddress();
      setRpcServiceServerAddress(conf);
}
//获得server
    this.server = RPC.getServer(this, socAddr.getHostName(),
        socAddr.getPort(), handlerCount, false, conf, namesystem
        .getDelegationTokenSecretManager());

   •••
    this.server.start();  //启动 RPC server   Clients只允许连接该server
    if (serviceRpcServer != null) {
      serviceRpcServer.start();  //启动 RPC serviceRpcServer 为HDFS服务的server
    }
    startTrashEmptier(conf);
  }
```

查看Namenode初始化源码得知：RPC的server对象是通过ipc.RPC类的getServer()方法获得的。下面咱们去看看ipc.RPC类中的getServer()源码

```java
public static Server getServer(final Object instance, final String bindAddress, final int port,
                                 final int numHandlers,
                                 final boolean verbose, Configuration conf,
                                 SecretManager<? extends TokenIdentifier> secretManager) 
    throws IOException {
    return new Server(instance, conf, bindAddress, port, numHandlers, verbose, secretManager);
  }
```

这时我们发现getServer()是一个创建Server对象的工厂方法，但创建的却是RPC.Server类的对象。哈哈，现在你明白了我前面说的“RPC.Server是ipc.Server的实现类，初始化Server后，Server端就运行起来了，看看ipc.Server的start()源码：

```java
/** 启动服务 */
  public synchronized void start() {
    responder.start();  //启动responder
    listener.start();   //启动listener
    handlers = new Handler[handlerCount];
    
    for (int i = 0; i < handlerCount; i++) {
      handlers[i] = new Handler(i);
      handlers[i].start();   //逐个启动Handler
    }
  }
```

分析过ipc.Client源码后，我们知道Client端的底层通信直接采用了阻塞式IO编程，当时我们曾做出猜测：Server端是不是也采用了阻塞式IO。现在我们仔细地分析一下吧，如果Server端也采用阻塞式IO，当连接进来的Client端很多时，势必会影响Server端的性能。hadoop的实现者们考虑到了这点，所以他们采用了java NIO来实现Server端，那Server端采用java NIO是怎么建立连接的呢？分析源码得知，Server端采用Listener监听客户端的连接，下面先分析一下Listener的构造函数吧：

```java
public Listener() throws IOException {
      address = new InetSocketAddress(bindAddress, port);
      // 创建ServerSocketChannel,并设置成非阻塞式
      acceptChannel = ServerSocketChannel.open();
      acceptChannel.configureBlocking(false);

      // 将server socket绑定到本地端口
      bind(acceptChannel.socket(), address, backlogLength);
      port = acceptChannel.socket().getLocalPort(); 
      // 获得一个selector
      selector= Selector.open();
      readers = new Reader[readThreads];
      readPool = Executors.newFixedThreadPool(readThreads);
      //启动多个reader线程，为了防止请求多时服务端响应延时的问题
      for (int i = 0; i < readThreads; i++) {       
        Selector readSelector = Selector.open();
        Reader reader = new Reader(readSelector);
        readers[i] = reader;
        readPool.execute(reader);
      }
      // 注册连接事件
      acceptChannel.register(selector, SelectionKey.OP_ACCEPT);
      this.setName("IPC Server listener on " + port);
      this.setDaemon(true);
    }
```

在启动Listener线程时，服务端会一直等待客户端的连接，下面贴出Server.Listener类的run()方法：

```java
public void run() {
     •••
      while (running) {
        SelectionKey key = null;
        try {
          selector.select();
          Iterator<SelectionKey> iter = selector.selectedKeys().iterator();
          while (iter.hasNext()) {
            key = iter.next();
            iter.remove();
            try {
              if (key.isValid()) {
                if (key.isAcceptable())
                  doAccept(key);     //具体的连接方法
              }
            } catch (IOException e) {
            }
            key = null;
          }
        } catch (OutOfMemoryError e) {
       •••         
    }
```

下面贴出Server.Listener类中doAccept()方法中的关键源码：

```java
void doAccept(SelectionKey key) throws IOException,  OutOfMemoryError {
      Connection c = null;
      ServerSocketChannel server = (ServerSocketChannel) key.channel();
      SocketChannel channel;
      while ((channel = server.accept()) != null) { //建立连接
        channel.configureBlocking(false);
        channel.socket().setTcpNoDelay(tcpNoDelay);
        Reader reader = getReader();  //从readers池中获得一个reader
        try {
          reader.startAdd(); // 激活readSelector，设置adding为true
          SelectionKey readKey = reader.registerChannel(channel);//将读事件设置成兴趣事件
          c = new Connection(readKey, channel, System.currentTimeMillis());//创建一个连接对象
          readKey.attach(c);   //将connection对象注入readKey
          synchronized (connectionList) {
            connectionList.add(numConnections, c);
            numConnections++;
          }
        ••• 
        } finally {
//设置adding为false，采用notify()唤醒一个reader,其实代码十三中启动的每个reader都使
//用了wait()方法等待。因篇幅有限，就不贴出源码了。
          reader.finishAdd();
        }
      }
    }
```

当reader被唤醒，reader接着执行doRead()方法。

下面贴出Server.Listener.Reader类中的doRead()方法和Server.Connection类中的readAndProcess()方法源码：

```
方法一：   
 void doRead(SelectionKey key) throws InterruptedException {
      int count = 0;
      Connection c = (Connection)key.attachment();  //获得connection对象
      if (c == null) {
        return;  
      }
      c.setLastContact(System.currentTimeMillis());
      try {
        count = c.readAndProcess();    // 接受并处理请求  
      } catch (InterruptedException ieo) {
       •••
      }
     •••    
}

方法二：
public int readAndProcess() throws IOException, InterruptedException {
      while (true) {
        •••
        if (!rpcHeaderRead) {
          if (rpcHeaderBuffer == null) {
            rpcHeaderBuffer = ByteBuffer.allocate(2);
          }
         //读取请求头
          count = channelRead(channel, rpcHeaderBuffer);
          if (count < 0 || rpcHeaderBuffer.remaining() > 0) {
            return count;
          }
        // 读取请求版本号  
          int version = rpcHeaderBuffer.get(0);
          byte[] method = new byte[] {rpcHeaderBuffer.get(1)};
        •••  
       
          data = ByteBuffer.allocate(dataLength);
        }
        // 读取请求  
        count = channelRead(channel, data);
        
        if (data.remaining() == 0) {
         •••
          if (useSasl) {
         •••
          } else {
            processOneRpc(data.array());//处理请求
          }
        •••
          }
        } 
        return count;
      }
    }
```

下面贴出Server.Connection类中的processOneRpc()方法和processData()方法的源码。

```
方法一：   
 private void processOneRpc(byte[] buf) throws IOException,
        InterruptedException {
      if (headerRead) {
        processData(buf);
      } else {
        processHeader(buf);
        headerRead = true;
        if (!authorizeConnection()) {
          throw new AccessControlException("Connection from " + this
              + " for protocol " + header.getProtocol()
              + " is unauthorized for user " + user);
        }
      }
}
方法二：
    private void processData(byte[] buf) throws  IOException, InterruptedException {
      DataInputStream dis =
        new DataInputStream(new ByteArrayInputStream(buf));
      int id = dis.readInt();      // 尝试读取id
      Writable param = ReflectionUtils.newInstance(paramClass, conf);//读取参数
      param.readFields(dis);        
        
      Call call = new Call(id, param, this);  //封装成call
      callQueue.put(call);   // 将call存入callQueue
      incRpcCount();  // 增加rpc请求的计数
    }
```

