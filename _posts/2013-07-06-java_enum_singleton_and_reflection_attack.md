---
layout: post
title: java enum singleton and related safe problems 
---

晚上在stackoverflow上闲晃，查找google的java库的相关信息的时候，看到了用enum singleton单例的写法,就顺便花时间了解了一下几个单例:  
单例要保证的是一个进程只有一个实例，一般性的写法是  

    class Singleton {
        private static Singleton singleton = new Singleton();

        private Singleton() {}

        public Singleton getSingleton()
        {
            return singleton;
        }
    }

    还有lazy implementation, 就是我常用的bob方法，其他古老的方法就不说了
    class Singleton {
        private static class SingleHolder {
            public static Singleton singleton = new Singleton();
        }
        
        private Singleton() {}; 

        public Singleton getSingleton()
        {
            return SingleHolder.singleton;
        }
    }
    只要不访问SingleHolder就不会触发singleton的构造，所以不访问getSinleton()函数是不会生成单例的.

    而enum实现单例更简单一些
    enum Singleton {
        INSTANCE;

        Singleton() {
        //your code
        };

        private xxx;
    }
    
最古老的方法不可考了也不想考了，中间了方法是我一直在稀里糊涂用着但是不知道为什么的方法，现在想想唯一的好处就是lazy implementation和不用加synchornized关键字的线程安全。至于知道enum 的这种实现方法，是看到stackoverflow上的大牛鄙视一个小菜鸟的时候顺便学到的（大牛对小菜一通鄙视，从封装到singleton，最后索性来了一句jactually everything you said is wrong, blah, blah, blah, read effective java, blah, blah，我自己也顺便中了一枪）。  

采用前两种方法会产生两个安全性问题：1、reflection attack 和 2、deserialization attack。第一种说的是可以通过反射调用似有函数，从而产生新的instance，后者说的是通过序列化和反序列化能产生新的实例，这样赤裸裸的对单例的侵犯是难以忍受的，所以衍生出了一些办法来对付这俩问题，对于前者，[这篇讲reflection attack的blog](http://initbinder.com/articles/hack_any_java_class_using_reflection_attack.html)里有详细的介绍，对于后者，readReslove里抛异常之类的奇思妙想也是比比皆是。  
但是如果用enum来实现单例的话直接就能解决这俩问题，不信你看java这个语言的设定----[enum specification](http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.9)
    
    **The final clone method in Enum ensures that enum constants can never be cloned, and the special treatment by the serialization mechanism ensures that duplicate instances are never created as a result of deserialization. Reflective instantiation of enum types is prohibited. Together, these four things ensure that no instances of an enum type exist beyond those defined by the enum constants.**

最后附上wiki里面对java singleton的[详尽介绍](http://en.wikipedia.org/wiki/Singleton_pattern)  
还有enum在java中的[具体实现](http://www.cnblogs.com/frankliiu-java/archive/2010/12/07/1898721.html)，一篇不错的中文文章.
