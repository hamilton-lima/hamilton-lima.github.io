---
title: 'Using Redis Publish and Subscribe'
date: Fri, 11 May 2018 01:01:24 +0000
draft: false
tags: ['java', 'junit', 'publisher', 'pubsub', 'redis', 'robolucha', 'subscriber']
---

There is this game called Robolucha where you code your robots to fight in a virtual arena. I've been working on this project for some years and last year decided to rewrite everything, mainly for the sake of technical exploration and this post share challenges using Redis Publish and subscribe.

Proving the concept
-------------------

Before I start to hit the keyboard to refactor MatchStatePublisher to send match states to Redis, I had some homework to do. So I build a POC (Proof of concept) to understand how [Jedis](https://github.com/xetorthio/jedis) client works and is possible to do with it. The first discovery is that you can't use the same connection for publishing AND subscribing, you need to separate JedisPool or even a Jedis connection if there is no pool. The POC has two separated apps one to publish data and another to subscribe to the changes, the code can be found [here](https://github.com/hamilton-lima/robolucha/tree/master/pocs/redis/java-pubsub)

How to test it
--------------

1\. start the Redis server using the default [Redis docker image](https://hub.docker.com/_/redis/)
```
docker run --name test-redis --rm -p 6379:6379 -d redis
```
2\. run the Subscriber class 3. run the Publisher class that will start 10 threads to send 5000 messages each The subscriber screen should display every single message received.

Using the lessons in the runner application
-------------------------------------------

After the POC establish the foundations for the implementation I add some extra requirements:

*   The queue names should be auto-generated based on the class name
*   Observables from [RxJava](https://github.com/ReactiveX/RxJava) should be used to implement the subscription
*   The subscription process should not block the current Thread
*   The queue message parsing from and to objects should be transparent
*   Publishing should accept [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) as parameters
*   The entire publish and subscribe process should be testable

I can say that all the requirements were fulfilled. Some with more challenges than others :) Let's go over the details of each one.

Queue names generation, Observables, and asynchronous subscription
------------------------------------------------------------------

The method that generates the queue name is very straightforward

```
private String getChannelName(Class clazz) {
    return clazz.getCanonicalName();
}
```

So when using subscribe or publish the queue names would be something like com.robolucha.models.Luchador For the use of Observables, the method subscribes returns a [BehaviorSubject](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/subjects/BehaviorSubject.html) of whatever class is sent as a parameter for the subscription.

```
public <T> BehaviorSubject subscribe(Class<T> clazzToSubscribe) {
    BehaviorSubject<T> result = BehaviorSubject.create();
    ...
    return result;
}
```

[Jedis](https://github.com/xetorthio/jedis) client is implementation blocks the current thread when the subscription happens, to workaround that there is a separated Thread to subscribe and an additional listener to interrupt this thread the BehaviorSubject completes.

```
Thread subscriber = new Thread(new Runnable() {
    public void run() {
        Jedis subscriber = subscriberPool.getResource();
        subscriber.subscribe(new JedisPubSub() {
        ...
        }, channel);
        subscriber.close();
    }
});

result.subscribe(new ThreadKiller<>(subscriber));
subscriber.start();
```

Transparent parsing, POJO
-------------------------

When publishing the object is converted to JSON and when receiving the message the JSON string is converted to the expected object type as well. making the process transparent for the consumer of the RemoteQueue implementation.

```
public void onMessage(String channel, String message) { (1)
    T data = gson.fromJson(message, clazzToSubscribe); (2)
    result.onNext(data); (3)
}
```
(1) happens when the subscription receives a message from the queue (2) convert the JSON to the object type defined by clazzToSubscribe (3) push the received value to the BehaviourSubject stream

Unit testing... er... End to End testing
----------------------------------------

The initial expectation was to build a unit test, but the final implementation is actually and End2End, and actually, a Redis server is used in the test. To have the Redis server running for the test the class [RedisDockerHelper](https://github.com/hamilton-lima/robolucha/blob/master/src/runner/src-test/com/robolucha/publisher/RedisDockerHelper.java) was created to wrap the docker commands to start and stop a Redis server, meaning that in the test the usage is only .start() and .stop(), see class [RedisDockerHelper](https://github.com/hamilton-lima/robolucha/blob/master/src/runner/src-test/com/robolucha/publisher/RedisDockerHelper.java) that run the docker commands and logs the console outputs. Here are some fragments from the test

```
try (RemoteQueue queue = new RemoteQueue(Config.getInstance())) {
```

Execute the code using [the RemoteQueue](https://github.com/hamilton-lima/robolucha/blob/master/src/runner/src/com/robolucha/publisher/RemoteQueue.java) instance as an [AutoCloseable](https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html) so when the block finishes the close() the method from the object will be called releasing the [Jedis](https://github.com/xetorthio/jedis) pools from memory.

```
assertEquals("subscribe", future.get(5, TimeUnit.SECONDS));
```

This assertion will wait for the future object ([CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)) to be fulfilled or for the timeout of 5 seconds, whatever comes first. This is an elegant way to keep the unit test waiting for the pub and subprocess to end. Note that inside the accept method the future is fulfilled with the command future.complete("subscribe"); See the complete test here: [RemoteQueueTest](https://github.com/hamilton-lima/robolucha/blob/master/src/runner/src-test/com/robolucha/publisher/RemoteQueueTest.java)

.complete() a.k.a. last words
-----------------------------

There were lots of interesting challenges in this implementation, but for me, the most interesting one was the use of generics with Observables for the subscription process, It's nice to see fragments like this:

```
public <T> BehaviorSubject subscribe(Class<T> clazzToSubscribe) {
BehaviorSubject<T> result = BehaviorSubject.create();
```

Of course, there are other ways to implement this without RxJava, or any other library, but it looks really good with the Observables.