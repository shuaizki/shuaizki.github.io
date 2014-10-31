---
layout: post
title: java里的classloader
category: language_related
---
#java的classloader
##基本概念
java的classloader的作用是动态的将class载入jvm  
jvm运行的时候，用到了三个classloader:  
  
  *	bootstrap classloader  ---用来加载java最核心的class，需要加载的class在jre/lib中，bootstrap是在jvm中的，所以写java代码的时候我们是看不到的，通过-Xbootclasspath可以指定加载哪些core api    
  *	extension classloader	---加载标准拓展库的classloader, 需要加载的类在jre/lib/ext中  
  *	sys classloader	---从环境变量配置的classpath中来查找路径，设置classpath即可  

  Class.getClassLoader()会得到当前类的classloader，如果该类是String之类的class， 那么加载它的classloader就是bootstrap classloader， 如果是classpath中的包，那么会得到system classloader, ext classloader 也一样  
  ##parent delegation
  classloader 加载类的时候会采用双亲委托的模型。在当前classloader需要加载一个类的时候，先委托其parent去加载这个类，如果parent找不到这个类，自己才开始加载。这里的parent并不是superclass， 而是一个引用关系，也就是按照上面提到的那个顺序。  
  这样做的好处就是提升安全性，比如你自己实现的String就不会被加载到jvm中，因为首先会加载的是core api中的class。  
  ##class1 == class2 ?
  在比较两个class是否相等的时候，不仅比较两个class的名字是否相等，还要比较加载两个class的是否为同一classloader，只有两个条件同时满足的时候，才会认为两个class是等价的，否则就会出现转型错误  


----
  当当当当，以上是大致的概念，不过都没有提到configuration里面用到的contextclassloader  
  不过解释还是有点难懂，估计短时间内也用不着，先放着吧  
  留下一个链接，日后慢慢研究 [find a way out of the classloader maze](http://www.javaworld.com/javaworld/javaqa/2003-06/01-qa-0606-load.html)
