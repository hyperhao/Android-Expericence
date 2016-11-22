# Apache Mina 
##一.首先了解下什么是Mina
Apache MINA 是一个网络应用程序框架，用来帮助用户简单地开发高性能和高可靠性的网络应用程序。它提供了一个通过 Java NIO 在不同的传输例TCP/IP 和 UDP/IP 上抽象的事件驱动的异步 API。
##二.怎么使用Mina
###Server端：
    public class MinaServer{ 
        private static final int PORT = 8901;// 定义监听端口      
        public static void main(String[] args) throws IOException  
        {  
        // 创建服务端监控线程  
        IoAcceptor acceptor = new NioSocketAcceptor();  
        // 设置日志记录器  
        acceptor.getFilterChain().addLast("logger", new LoggingFilter());  
        // 设置编码过滤器  
        acceptor.getFilterChain().addLast("codec",new ProtocolCodecFilter(new TextLineCodecFactory(Charset.forName("UTF-8"))));  
        // 指定业务逻辑处理器  
        acceptor.setHandler(new ServerHandler());  
        // 设置端口号  
        acceptor.setDefaultLocalAddress(new InetSocketAddress(PORT));  
        // 启动监听线程  
        acceptor.bind();  
     }      
    }
###Handler类：
    public class ServerHandler extends IoHandlerAdapter{ 
         // 连接创建事件 
         @Override   
         public void sessionCreated(IoSession session)  {  
             // 显示客户端的ip和端口  
            System.out.println(session.getRemoteAddress().toString());  
        }  
         //消息接收事件
         @Override  
        public void messageReceived(IoSession session, Object message) throws Exception  {  
            String str = message.toString();  
             if (str.trim().equalsIgnoreCase("quit"))  { 
                // 结束会话  
                 session.close(true);  
                 return;  
          }  
         // 返回消息字符串  
         session.write("Hi Client!");  
        // 打印客户端传来的消息内容  
        System.out.println("Message written..." + str);  
     }  
    }
###Client端：    
    public class MinaClient  {  
        public static void main(String[] args)  
        {  
            // 创建客户端连接器.  
            NioSocketConnector connector = new NioSocketConnector();  
            // 设置日志记录器  
            connector.getFilterChain().addLast("logger", new LoggingFilter());  
            // 设置编码过滤器  
            connector.getFilterChain().addLast("codec", new ProtocolCodecFilter(new TextLineCodecFactory(Charset.forName("UTF-8"))));   
            // 设置连接超时检查时间  
            connector.setConnectTimeoutCheckInterval(30);  
            // 设置事件处理器  
            connector.setHandler(new ClientHandler());  
            // 建立连接  
            ConnectFuture cf = connector.connect(new InetSocketAddress("195.2.199.170", 8901));  
            // 等待连接创建完成  
            cf.awaitUninterruptibly();  
            // 发送消息  
            cf.getSession().write("Hi Server!");  
            // 发送消息  
            cf.getSession().write("quit");  
            // 等待连接断开  
            cf.getSession().getCloseFuture().awaitUninterruptibly();  
            // 释放连接  
            connector.dispose();  
        } 
    }  
###Client Handler类：   
      public class ClientHandler extends IoHandlerAdapter {  
        @Override  
        public void messageReceived(IoSession session, Object message)  
                throws Exception {  
            System.out.println("server message:" + message.toString());// 显示接收到的消息  
        }  
    }  
     
    
