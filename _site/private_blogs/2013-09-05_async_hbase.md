#浏览下文档

##HBaseClient
data durability    数据有效性的设置，这个我不用管  
exception handle   多加一个callback函数就行了，输入类型是Exception  

asynchbase 用了stumbleupon的async库  
##stumbleupon Async

###Deferred
定义：a Deferred is  like a java.util.concurrent.Future with a dynamic Callback chain associated to it   
当deferred的结果可用的时候，callback chain被触发。一个callback的结果作为参数传给下一个callback。  
如果一个callback返回一个deferred的话, 那么相当于插队了，直到该deferred产生结果才会继续下一个callback。  
上面说道的exception handling就用到了，其实有两条callback链，一个正常处理数据和罗技，一个处理错误(errback)  

deferred和future的主要区别在于，deferred有一串callback等着自动被触发，但是future需要你主动的获取数据。所以并不知道什么时候去获取数据，特别是该future依赖于其他future的时候。  
然后blahblah讲了一大堆deferred的callback chain的东西，还有deferred嵌套的东西  

###一些有关deferred的注意事项
一个deferred只能接受一个初始的result  
同一时间只能有一个线程执行callback chain  
callback是按顺序发生的，所以如果没必要做同步保障（都说了是按顺序发生的）  
执行callback chain的线程把初始结果传给deferred  
callback开始执行的那一刹那，deferred就失去了对它的引用  
往deferred里添加callback在o(1)内可以完成  
一个deferred不能接受自己作为result，不然死循环了。  
也不能弄一个环，不然也是死循环  

整个代码一千七百多行就搞定了。。。shit，为毛别人那么牛逼。。。索性熬个眼研究一下怎么写的。  


###代码研读
deferred的状态就是一个有限状态机


		    ,---------------------,
		    |   ,-------,         |
		    V   v       |         |
	PENDING --> RUNNING --> DONE     PAUSED
		        |                   ^
			`-------------------'


####chain and group
两个函数做的都是同一件事儿，就是
chain把一系列的deferred串联成一个，也就是把callback串联起来。  
group做的事儿就是当所有的deferred完成的时候返回一个deferred  
