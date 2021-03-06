#### ServerBootstrap

1. EventLoopGroup维护着多个EventLoop，一个EventLoop则持有一个Selector，在一个Selector上可以维护多个Channel，这些Channel是服务端用来与客户端通信而创建的。启动服务端时也会创建一个Channel，该Channel用来与客户端连接所用，全局只有一个（即只有一个ServerChannel），而与客户端通信的会有多个。

2. ServerBootstrap的parentGroup负责与客户端的连接，由于连接所用的Channel只有一个，因此也只会用到parentGroup里面的一个EventLoop，而RocketMQ创建的parentGroup只指定一个线程也是这个原因。ServerChannel在初始化时，会在其ChannelPipeline中添加ServerBootstrapAcceptor，ServerBootstrapAcceptor主要是将与客户端通信的Channel注册到childGroup中的EventLoop，也即childGroup主要用来与客户端进行通信。

3. ChannelPipeline是双向链表，节点父类型为AbstractChannelHandlerContext，其有指向前驱和后继结点的指针。中间结点的子类型为DefaultChannelHandlerContext，其内部持有ChannelHandler，而头尾结点子类型为HeadContext和TailContext，并不持有ChannelHandler。

4. 每个与客户端通信的Channel都会注册于某个EventLoop中，该Channel对应的ChannelPipeline中的ChannelHandler都使用同一线程，除非在添加时额外指定EventLoopGroup。但是即使指定额外EventLoopGroup，ChannelHandler仍然会按照在ChannelPipeline中的顺序先后执行，并不会因为使用异步就不遵循顺序规则，因为其内部使用了队列来保证顺序。因此ChannelHandler中如果要运行耗时操作，需要在ChannelHandler中另起线程。

5. Netty中有两种写的方式，其区别如下：

   - Channel.write() : 从 tail 到 head 调用每一个outbound 的  ChannelHandlerContext.write。

   - ChannelHandlerContext.write() : 从当前的Context, 找到上一个outbound, 从后向前调用 write 。
     