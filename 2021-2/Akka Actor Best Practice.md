# AKKA Actor Best Practice



如果Actor中的事件处理过程会比较耗时，或者发生阻塞，有两种方案：

1. 给Actor一个单独的dispatcher
2. 结合ask使用future

> 详见：blocking-needs-careful-management
>
> https://doc.akka.io/docs/akka/current/typed/dispatchers.html#blocking-needs-careful-management
>
> 

Actor中最好避免block，以下是方法：

>### Available solutions to blocking operations
>
>The non-exhaustive list of adequate solutions to the “blocking problem” includes the following suggestions:
>
>- Do the blocking call within a `Future`, ensuring an upper bound on the number of such calls at any point in time (submitting an unbounded number of tasks of this nature will exhaust your memory or thread limits).
>- Do the blocking call within a `Future`, providing a thread pool with an upper limit on the number of threads which is appropriate for the hardware on which the application runs, as explained in detail in this section.
>- Dedicate a single thread to manage a set of blocking resources (e.g. a NIO selector driving multiple channels) and dispatch events as they occur as actor messages.
>- Do the blocking call within an actor (or a set of actors) managed by a [router](https://doc.akka.io/docs/akka/current/routing.html), making sure to configure a thread pool which is either dedicated for this purpose or sufficiently sized.



个人比较倾向第二个方案：

对于InnerAccessActor， 

```scala
class InnerAccessActor extends Actor {

  val logActor = context.system.actorSelection("/user/logRouter")

  override def receive = {
    case request: AccessRequest => {
 
        val innerApp = InnerApp.findKey(request.innerAppId)
        val dataSourceId = ServiceInfo.findByServiceId(request.innerServiceId).get.edataSrcId.get
        val externalDataSource = ExternalDataSource.findById(dataSourceId)
        //适配器使用前，全局流水号传入edsource
        externalDataSource.get.seqNo = request.seqNo
        val adapter = externalDataSource.get.getAdapter()
        xitrum.Log.warn(s"""${request.seqNo} 外部源适配器: ${adapter.getCName()}(${adapter.getName()})-------------------------------------""")
        var returnData = adapter.access(request(Some(AccessoryAcors(logActor))), externalDataSource.get)
        val requestParamString = getRequestParamsString(request.params)
        val dlog = Dlog2(request.seqNo,startTime, DateUtil.getTimeStamp(), returnData.isCacheData.toString,
                         request.hosts,requestParamString, 1, request.innerAppId, innerApp.get.appName, "query", "yes",
                         returnData.status, xitrum.util.SeriDeseri.toJson(returnData), externalDataSource.get.id,
                         externalDataSource.get.name)
         ...
  }

```

上面的代码中，findKey， findByServiceId ， externalDataSource.get.getAdapter()等等函数，第一， 比较耗时，第二，可能会block，因此很容易发生消息堆积

如果稍作改动，考虑leader- worker模式：

 

```scala
class Leader extends Actor {

  override def receive = {
    case request: AccessRequest => {
         //worker to do the dirty task
        val longProcessRef = actorSystem actorOf Props[InnerAccessActor]
        val fut : Future[Result] = (longProcessRef ? request).mapTo[Result]
        //process future
  }
```



这样就可能会产生很多很多的InnerAccessActor， 但不用担心， 在一台32G内存的服务器上，可以容纳百万级别的Actor同时并发！

>https://doc.akka.io/docs/akka/current/typed/guide/actors-intro.html#usage-of-message-passing-avoids-locking-and-blocking
>
>- There are no locks used anywhere, and senders are not blocked. **Millions of actors can be efficiently scheduled on a dozen of threads reaching the full potential of modern CPUs. Task delegation is the natural mode of operation for actors.**
>
>



但使用的时候需要遵循一些规则，因为在akka文档的开始就提到：

>## Actor Best Practices
>
>1. Actors should be like nice co-workers: do their job efficiently without bothering everyone else needlessly and avoid hogging resources. Translated to programming this means to process events and generate responses (or more requests) in an event-driven manner. Actors should not block (i.e. passively wait while occupying a Thread) on some external entity—which might be a lock, a network socket, etc.—unless it is unavoidable; in the latter case see [`Blocking Needs Careful Management`](https://doc.akka.io/docs/akka/current/typed/dispatchers.html#blocking-management).
>2. ....