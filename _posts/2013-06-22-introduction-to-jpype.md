---
layout: post
title: Jpype的一些事儿
category: language_related
---
#Jpype的一些事儿

Jpype提供了python访问java代码的能力，但是实现上不同于Jython语言（没错，这货是门语言），Jpype只是一个库。  
由于有一个移植小工具的需求，于是就用到了Jpype，琢磨了一番之后，深感方便。下面就简单的讲讲  
##how to use ?
首先当然是安装啦，从[官网](http://jpype.sourceforge.net)上下载， 解压，然后就。。。当当当当！可以用了！
####start a jvm  
Jpype的核心包是jpype， jpype是从jvm的native层进行调用的，首先要做的操作是打开jvm虚拟机  

    from jpype import *
    
    jvmPath = getDefaultJVMPath()
    
    startJVM(jvmPath)
    shutdownJVM()
   这里的jvmPath是能够自己设置的, getDefaultJVMPath()给出的只是当前系统下默认的jvmpath, 以os x为例， 可以看到给出的默认getDefaultJVMPath()函数是
   
    def getDefaultJVMPath() :
    # on darwin, the JVM is always in the same location it seems ...
    return '/System/Library/Frameworks/JavaVM.framework/JavaVM'
    
  没错，我要讲的就是那句贱贱的注释传达的意思，至于linux和window平台，稍微复杂一点，所以自行设置还是比较靠谱的  
  只是开关虚拟机当然没什么意思了，而且我们用Jpype的目的并不是"hello, world",所以我们直接跳过hello world的步骤，直接开始学着调用你自己的java代码  
####call your java code
   你有两种方式放置你要调用的java代码，直接编译，把所有的东西放在一个文件夹里，或者使用一个jar包，总之这个自己来控制吧。下面直接假设你所有的东西都在一个叫temp.jar的java的jar包里了  
   Jpype比较弱的一点是要你自己指定classpath, 我们假设temp.jar就在当前的文件夹下，于是classpath肯定会是这样的  
   
    ext_classpath = os.path.join(os.path.abspath('.'), 'temp.jar')
    
   然后我们就可以打开虚拟机了
    
    from Jpype import *
    
    ext_classpath = os.path.join(os.path.abspath('.'), 'temp.jar')
    startJVM(getDefaultJVMPath(), "-Djava.classpath.path="+ext_classpath)
    
   下面做一个假设
    
   这个temp.jar中有一个你想要的调用的class
    
    package trial.jpype;

    import java.io.BufferedReader;
    import java.io.BufferedWriter;
    import java.io.IOException;
    import trial.jpype.CoreDataType.NameComponents;


    import trial.on.jpype.shuaizki;

    import me.shuaizki.util.IO.MyIO;

    public class namenormalizer {

	public static void speak1()
	{
		System.out.println("blah..blah..blah");
	}
	
	public static String getNormalizedName(String str)
	{
		NameComponents nc = NameSplit.getSingleton().getNormalizedName(str);
		return ncToJson(nc);
	}
	
	public static String ncToJson(NameComponents nc)
	{
		String ret = "{";
		return ret;
	}
	
	}
	
我们想调用的是namenormalizer中的getNormalizedName和speak1，首先要获取namenormalizer这个class，有两种途径获取这个
    
    namenormalizer = JClass('trial.jpype.namenormalizer')
    利用
    ns = namenormalizer() 获取namenormlaizer 的一个对象
    利用ns.getNormalizedName(“hello”)调用或者直接利用 namenormalizer.getNormalizedName()进行调用， 这样就完成了类的获取和函数的调用
    
还有一种途径是利用Jpackage()这个函数
    
    namenormalizer = JPackage('trial').jpype.namenormalizer
   
   其实看看源码就会知道jpackage对类的访问依靠的也是jclass，只不过其提供了对包的访问。归根结底能找到类还是要依靠_jpype.findClass()这个函数，也就是C++代码里的东西，这个后面会有介绍。  
   
####运行代码的注意事项
   好了，我们最后的代码长成这个样子
    
    from Jpype import *
    
    ext_classpath = os.path.join(os.path.abspath('.'), 'temp.jar')
    startJVM(getDefaultJVMPath(), "-Djava.classpath.path="+ext_classpath)
    namenormalizer = JClass('trial.jpype.namenormalizer')
    namenormalizer.getNormalizedName("hello")
    
    shuwdownJVM()

   尝试着运行一下吧~
     
     namesplit = JClass('trial.jpype.namenormalizer')
     File "/Library/Python/2.7/site-packages/jpype/_jclass.py", line 54, in JClass
    raise _RUNTIMEEXCEPTION.PYEXC("Class %s not found" % name)
     jpype._jexception.ExceptionPyRaisable: java.lang.Exception: Class
     trial.jpype.namenormalizer not found
  纳尼！！！这是什么情况，找不到呢
  看一下我们的代码，发现没什么问题，包也在，那是为什么呢? 如果你用eclipse 的话，打开你的.classpath看看吧
    
    <classpath>
        <classpathentry kind="src" path="src"/>
        <classpathentry kind="src" path="test"/>
        <classpathentry kind="con" path="org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.launching.macosx.MacOSXType/Java SE 6 (MacOS X Default)"/>
        <classpathentry kind="lib" path="lib/protobuf-java-2.4.1.jar"/>
        <classpathentry kind="lib" path="lib/gson-2.2.4.jar"/>
        <classpathentry kind="output" path="bin"/>
    </classpath>
   如果你不用eclipse，肯定自己知道你用了一些外部包了。这些都要统统放在classpath里，不然Jpype自己是找不到的
    把这些lib里的东西都放在classpath里就行了，代码应该就能运行ok了
    
####debug建议
如果还是跑不起来的话，那就重新编译一下JPype，让他打出debug信息，自己慢慢琢磨一下吧  
打开src/native/common/include/jpype.h这个文件，第23行的
    
    //#define JPYPE_TRACING_INTERNAL
    
  被注释掉了，将它恢复正常了吧，然后回到Jpype的目录下，执行
  
    sudo python setup.py build -f
    sudo python setup.py install -f
    
  再运行你的代码的话，就会看到tracing信息了，剩下的就自己慢慢琢磨去吧
  
##原理
jpype其实用的是jni实现  
JNI就是Java Native Interface, 即可以实现Java调用本地库, 也可以实现C/C++调用Java代码, 从而实现了两种语言的互通, 可以让我们更加灵活的使用。  
之前没听过的同学可以自行学习学习  
利用c++完成对java调用的代码，做成python的库，这就是jpype的全部，实现全在代码里，大家自行参考喽  
后面我会抽时间写写关于jni的博客，顺便解析一下jpype的实现
  
