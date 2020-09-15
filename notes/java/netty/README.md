## TCP server

```java
public class NettyTcpServer implements Runnable {

    private static final Logger logger = LoggerFactory.getLogger(NettyTcpServer.class);

    private EventLoopGroup boosGroup;
    private EventLoopGroup workerGroup;
    private ServerBootstrap server;
    private int port;

    public NettyTcpServer(int port) {
        this.port = port;
        boosGroup = new NioEventLoopGroup();
        workerGroup = new NioEventLoopGroup();
        server = new ServerBootstrap();
        server.group(boosGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        
//                        ByteBuf byteBuf = Unpooled.copiedBuffer(new byte[]{(byte) 0xEB, (byte) 0x90});
//                        socketChannel.pipeline().addLast(new DelimiterBasedFrameDecoder(1024 * 1024, byteBuf));
//                        socketChannel.pipeline().addLast(new FixedLengthFrameDecoder(20));
                        socketChannel.pipeline().addLast(new LineBasedFrameDecoder(1025 * 50));
                        socketChannel.pipeline().addLast(new StringDecoder(CharsetUtil.UTF_8));
                        socketChannel.pipeline().addLast(new NettyTcpHandler());
                        socketChannel.pipeline().addLast(new StringEncoder(CharsetUtil.UTF_8));
                    }
                });
        server.option(ChannelOption.SO_BACKLOG, 128);
        server.childOption(ChannelOption.SO_KEEPALIVE, true);
    }

    @Override
    public void run() {
        try {
            ChannelFuture channelFuture = server.bind(this.port).sync();
            logger.info("TCP服务启动，监听端口：" + port);
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            logger.error("启动TCP服务，端口[" + this.port + "]发生错误", e);
        } finally {
            boosGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}!
```

handler

```java
public class NettyTcpHandler extends ChannelInboundHandlerAdapter {

    private static final Logger logger = LoggerFactory.getLogger(NettyTcpHandler.class);

    private Channel channel;
    private String str;

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        channel = ctx.channel();
        logger.info("建立连接，地址：" + channel.remoteAddress() + " ID: " + channel.id());
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
//        ByteBuf byteBuf = (ByteBuf) msg;
//        int readBytes = byteBuf.readableBytes();
//        byte[] bytes = new byte[readBytes];
//        byteBuf.readBytes(bytes);
//        String x = ByteKit.byteArrayToHexStr(bytes);
//        str = str + x;
//        System.out.println(str);

        System.out.println("..");
        channel = ctx.channel();
        String content = msg.toString();
        logger.info("接收到内容：" + content);
        channel.writeAndFlush(content + "\n");
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
        logger.error("TCP链接，服务端出现错误", cause);
        ctx.close();
    }
}
```

