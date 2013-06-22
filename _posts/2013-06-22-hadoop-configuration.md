---
layout: post
title: hadoop源码阅读-configuration
---
#configuration
实现了Interable<Map.Entry<String, String>>接口和Writable接口  
##私有变量
比较关键的私有变量有  
	
	ArrayList<Object> resources; 
	记录各个资源文件，configuration里总共有四种形式了，String（文件名）， URL, Path(hadoop 自己定义的), 还有inputstream, 在loadresource的时候，用instanceof进行判断然后加载
	
	Set<String> finalParameters; 记录了const的哪些配置变量（比如hdfs-site.xml里面的dfs.replication, 被设置成了final）
	
	boolean loadDefaults = true; 用于确定是否加载默认资源(默认加载哦)
	
	private static final WeakHashMap<Configuration,Object> REGISTRY = 
    new WeakHashMap<Configuration,Object>(); 各个Conf对象的一个注册表
    
    private static final CopyOnWriteArrayList<String> defaultResources =
    new CopyOnWriteArrayList<String>(); 默认资源的一个列表, CopyOnWriteArrayList是线程安全的哦
    
    private boolean storeResource; 是否要存储下面的hashmap
    
    privte HashMap<String, String> updatingResource; 存储某个配置项的来源资源文件
    
    其次是configuration里面的两个个Properties类型的变量， properties存储所有的配置项目, overlay存储所有被覆盖的配置
    还有classloader变量，是当前线程的contextclassloader, 或者是当前类的classloader（如果前者为空的话）
    
    
##方法
首先用一个静态块加载了"core-default.xml"和"core-site.xml"两个资源文件

###线程安全
由于configuration是线程安全的，因而注意代码里面的同步，有趣的是在configuration的一个构造函数Configuration(Configutaion other)中，看到了这样的代码

    this.resources = (ArrayList)other.resources.clone();
    synchronized(other) {
     if (other.properties != null) {
       this.properties = (Properties)other.properties.clone();
     }

     if (other.overlay!=null) {
       this.overlay = (Properties)other.overlay.clone();
     }
    }
   
    this.finalParameters = new HashSet<String>(other.finalParameters);
    
让人困惑的是为什么针对resources和finalParameters的操作并没有被synchronized  

###资源加载相关函数
properties并非启动或者生成对象的时候加载的，而是在需要的时候才会加载，因而当有新的资源需要载入的时候，需要一个函数清理当前的properties 和 finalParameters , 这样在需要用到的时候才能重新加载.其实这个函数很简单，设置properties == null ，clear finalParameters就行了

###get 和 set函数
####基本类型和enum
各种各样的get和set函数，五花八门，针对各个基本类型
都是针对properties进行操作的，由于Properties这个类本身是线程安全的，因而各种函数都是天然的线程安全函数---除了一个需要特别实现的函数  

	private synchronized Properties getOverlay() {
    if (overlay==null){
      overlay=new Properties();
    }
    return overlay;
  }
####IntegerRangs类和函数
将字符串转化为整数区间的类，目前还不清楚具体的作用是什么，只看到了一个isIncluded， 竟然是用遍历所有range来判断是否被包含在内的，感觉相当弱。。。。  
随着这个类提供的是一个getRange函数,获取IntegerRanges结果。
值得注意的是这里的IntegerRangs使用的是静态内部类，有关静态内部类的好处看[这里](http://book.51cto.com/art/201202/317517.htm)  

####get class
剩下的get函数就只剩下getclass系列了  
封装的很全面，不过实际上还是调用Class.forName(name, true, classloader)  
有一个不错的实现之前完全没见过，就是载入实现了某个接口的类，函数如下  

	public <U> Class<? extends U> getClass(String name, 
                                         Class<? extends U> defaultValue, 
                                         Class<U> xface) {
    try {
      Class<?> theClass = getClass(name, defaultValue);
      if (theClass != null && !xface.isAssignableFrom(theClass))
        throw new RuntimeException(theClass+" not "+xface.getName());
      else if (theClass != null)
        return theClass.asSubclass(xface);
      else
        return null;
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
    }
    运用了isAssignableFrom函数的特性, 判断当前Class是否是theclass the superclass 或者是 superinterface(两个类相等也返回true哦)
    
####get resouce
核心函数是classloader.getResource  

####loadResource
对xml的解析用的是javax.xml.parsers  
loadResource(Properties, Object, boolean)
是个递归函数，每次函数被调用，都会对 Element root进行赋值，然后判断root的tagname是否是configuration，如果不是的话会出直接退出，如果是，逐个对root的childnode进行解析  
如果子节点的tag那么是configuration，调用loadresource返回上面那个步骤。否则判断子节点的tagnode是否为property（真麻烦啊。。。不过这样强有力的保障了正确性）， blah…blah…解析子节点并放在properties和finalParmeter里面   
  
  
#####总体的感觉是封装的很标准，也有很多值得借鉴的实现。不过因为功力不够的缘故，留下了一些疑问：
#####1、某些为了保证线程安全而加的synchronized段，比如上面提到的那段
#####2、不同类型的classloader用起来有什么区别，比如configuration里涉及到的两种类型
#####3、configuration中用到hashCode的两个函数getLocalPath和getFile

