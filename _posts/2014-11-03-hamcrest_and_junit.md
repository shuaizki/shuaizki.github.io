---
layout: post
titile: Junit and hamcrest
category: java
---

#Junit and Hamcrest

###从Hamcrest开始

wiki了一下hamcrest和项目的tutorial，发现是个很不错的小框架。  

从历史说起的话，第一代单元测试框架提供了断言表达式  

    assert(x == y) 
    
但是这样对错误信息很不友好，于是第二代单元测试框架提供了一组测试语句  

    assert_equal(x , y)
    assert_not_equal(x, y)
    
这样对测试是友好了，但是测试语句的数量暴涨  

于是第三代单元测试框架就使用如hamcrest的库来支持assert_that的操作  

    assert_that(x, equals(y))
    assert_that(x, is_not(equals(y))) 
    
这样就能优美的输出错误信息。并且通过组合不同的匹配器，可以获得极大的拓展性。  

匹配器封装了一种匹配操作，可以是简单的equals操作，也可以是原来几种封装器的再封装，譬如

    Matcher a = equals(10)
    Matcher b = greatThan(0)
    Matcher c = allOf(a, b)
    
    Matcher a1 = equals(-20)
    Matcher b1 = lessThan(0)
    Matcher c1 = allOf(a1, b1)
    
    Matcher d = anyOf(c, c1)
    
a, b, a1, b1 就是比较简单的匹配器，c,c1,d就是通过其他封装器组合而成的。  

我用的junit-4.10.jar已经集成了hamcrest。至于具体是用独立的hamcrest，还是junit集成版的，后面用用看喽。  

###Junit

简单的过一遍junit的文档，记录一些自己不熟悉的东西。  


####method order

从4.11版本开始，可以指定固定的method顺序了。但是仍然不能指定test执行的顺序。目前只有两种固定顺序。 通过给测试类制定@FixMethodOrder

@FixMethodOrder(MethodSorters.JVM): 返回jvm给的方法顺序，所以每次跑的顺序可能不一样。  

@FixMethodOrder(MethodSorters.NAME_ASCENDING): Sorts the test methods by method name, in lexicographic order.  

####Rule

以前从没用过也不知道Rule这个annonation。   
Rule是用来为了给一个类中所有的测试增加某种功能的。比如说ExternalResource这个类，如果希望测试类里面的每一个测试都能申请到某数据库连接，然后在测试完之后释放这个连接的话，就可以自己实现一个ExternalResource类来完成。这样就能够不使用Before和After了。  

比较带感的是:  

TemporaryFolder 能够为测试创建临时文件或文件夹，测试函数结束后会删除  

ExternalResource 抽象类，实现了before和after之后，就能在测试函数前后执行这两个函数了  

ErrorCollector 收集throwble，然后完成测试后一次性报告所有的error  

TestWatcher  主要是测试过程记录作用，starting finished功能跟Before After相近，成功的case会进入successed，失败的case会进入failed，还有一个apply函数，暂时不知道是干什么的。  

TimeOutRule  类如其名  

还有就是自定义的了  

####Exception test

为了保证程序和预期的那样抛出异常，所以要有异常测试。  

如果是简单的小测试，可以用annotation参数来完成。  

    @Test(expected= IndexOutOfBoundsException.class) 
    public void empty() { 
         new ArrayList<Object>().get(0); 
    }

还有一种方法就是用上面提到的Rule, ExpectedException类  

####junit matcher
除了junit自带的和hamcrest自带的，还有一些第三方的  

* Excel spreadsheet matchers
* JSON matchers
* XML/XPath matchers

####Ignore a test
加上ignore annonation就行了

    @Ignore("Test is ignored as a demonstration")

####timeout
前面讲到Rule的时候有说道，有TimeOut rule，这个是对一个测试类里面的所有test适用的，还有一个对单个test作用的timeout的设置  

    @Test(timeout=1000)

####multithread / concurrent test
给出一篇文章里面的测试方法, assertConcurrent，来源在[assertConcurrent](http://www.planetgeek.ch/2009/08/25/how-to-find-a-concurrency-bug-with-java/ )，完整的实现不长，这里可以大致说说实现的方法。  

       public static void assertConcurrent(final String message, final List<? extends Runnable> runnables, final int maxTimeoutSeconds) throws InterruptedException {
          final int numThreads = runnables.size();
          final List<Throwable> exceptions = Collections.synchronizedList(new ArrayList<Throwable>());
          final ExecutorService threadPool = Executors.newFixedThreadPool(numThreads);
          try {
            final CountDownLatch allExecutorThreadsReady = new CountDownLatch(numThreads);
            final CountDownLatch afterInitBlocker = new CountDownLatch(1);
            final CountDownLatch allDone = new CountDownLatch(numThreads);
            for (final Runnable submittedTestRunnable : runnables) {
              threadPool.submit(new Runnable() {
                public void run() {
                  allExecutorThreadsReady.countDown();
                  try {
                    afterInitBlocker.await();
                    submittedTestRunnable.run();
                  } catch (final Throwable e) {
                    exceptions.add(e);
                  } finally {
                    allDone.countDown();
                  }
                }
              });
            }
            // wait until all threads are ready
            assertTrue("Timeout initializing threads! Perform long lasting initializations before passing runnables to assertConcurrent", allExecutorThreadsReady.await(runnables.size() * 10, TimeUnit.MILLISECONDS));
            // start all test runners
            afterInitBlocker.countDown();
            assertTrue(message +" timeout! More than" + maxTimeoutSeconds + "seconds", allDone.await(maxTimeoutSeconds, TimeUnit.SECONDS));
          } finally {
            threadPool.shutdownNow();
          }
          assertTrue(message + "failed with exception(s)" + exceptions, exceptions.isEmpty());
        }

大致讲解下原理，首先CountDownLatch是java concurrent包里面的一个同步工具，用一个整数初始化，初始化之后调用await()是会持续block那里的，通过另外的线程不断的调用countDown，调用输入整数个countDown之后，之前block在await的线程会继续运行。 

上面的代码建立了一个线程池，把输入的Runnable封装成Runnalbe提交到线程池里，同时保证所有的线程都不在运行，记录有多少Runnable已经被提交了，有多少Runnable已经完成了。  

然后assert所有的线程在规定时间内都提交完成了。开始启动这些线程，再最后确保这些线程在规定时间内完成。当然，运行过程中所产生的exception都被记录在一个list中，最后assert有没有产生excpetion。  

基本上完成了测试的需求，美中不足的是如果有timeout或者exeption，单从测试的结果很难定位timeout或者exception的runnalbe。有改进的空间。

####continuous test

The tests are run automatically after every change, usually in an intelligent order so that newest tests or most recently failed tests are run first。  

看起来很不错。有个叫infinitest（无尽的测试.....好冷）。可以放到intellij idea上面运行。  

至于infinitest的使用可以放到后面一个博客上面写

