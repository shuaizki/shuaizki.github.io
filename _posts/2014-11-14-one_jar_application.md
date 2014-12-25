---
layout: post
title: One jar application
category: java
---

#One jar application

怎么把java应用依赖的库和java应用一起打包发布呢？    

###naive的做法
我一度天真的以为不借助任何IDE，只要手写就能分分钟生成这么一个东西，于是我对自己的ant脚本稍作修改。成为了下面的样子：  

    <target name="service" depends="compile">
        <jar destfile="${basedir}/XX.jar">
            <fileset dir="${basedir}/lib" includes="*.jar" />
            <fileset dir="${classes.dir}"/>
            <manifest>
                <attribute name="Main-Class" value="Service"/>
                <attribute name="Class-Path" value="${libs}"/>
            </manifest>
        </jar>
    </target>

结果生成了下面的manifest:  

    Class-Path: ../PiX/cootek/java/lib/asynchbase-1.5.0.jar
    
这当然不对了!依赖的jar是应该被打包在jar里的，这样的路径怎么可能对呢（我的心里活动），不过这难不倒我ant脚本小达人，用pathconvert轻松解决。  

    <pathconvert property="libs" pathsep=" ">
         <path refid="library.lib.classpath" />
         <flattenmapper/>
    </pathconvert>

顺利的生成心目中的class-path，运行的时候傻眼了。  

    ClassNotFound exception
    
后来google了一下才发现，原来是默认的类加载器没有能力从jar-in-jar中加载class，所以想完成任务，必须寻找合适的第三方类加载器来完成这个任务。  

###eclipse style
之前的做法是直接用eclipse的export功能，不得不吐槽一下launch configuration, 如果之前没有运行过xx.java的，这个选项是空白的，而且没有任何提示(蛋疼。。。。就不能友好一点吗)。  
Library handling的话有三种选项:  

* extract required libraries 把library中的class文件解压到新的jar里面。这种有概率导致SecurityException.  
* package required libraries 直接扔到新的jar里面。  
* copy required lib into a sub-folder 另外创建一个目录，把需要用到的库文件拷贝进去。  

三种形式都能自动生成ant脚本，第二种才是我们需要的。  

用第二种方式导出一个xml之后打开，会看到如下内容。  

    <target name="create_run_jar">
        <jar destfile="XX.jar">
            <manifest>
                <attribute name="Main-Class" value="org.eclipse.jdt.internal.jarinjarloader.JarRsrcLoader"/>
                <attribute name="Rsrc-Main-Class" value="Service"/>
                <attribute name="Class-Path" value="."/>
                <attribute name="Rsrc-Class-Path" value="./ agent-api.jar jetty-all-7.0.2.v20100331.jar/>
            </manifest>
            <zipfileset src="jar-in-jar-loader.zip"/>
            <zipfileset dir="lib" includes="*.jar"/>
        </jar>
    </target> 

manifest部分生成如下的manifest文件  

    Class-Path: .
    Rsrc-Main-Class: com.cootek.data.Xavier.search.Service
    Main-Class: org.eclipse.jdt.internal.jarinjarloader.JarRsrcLoader
    Rsrc-Class-Path: ./ agent-api.jar jetty-all-7.0.2.v20100331.jar
    
我们大致能看出，Main-Class是这个jar包的入口，其实是个eclipse自己写的class loader，这个加载器加载Rsrc-Class-Path里jar文件包含的所有class，然后运行rsrc-main-class。 
以上是eclipse生成one-jar的做法。  


###amazing intellij

一直一来我对intellij idea这个IDE都是极力宣传的态度，套个vim插件简直是IDE大杀器。在我信心满满的打开intellij之后傻眼了, 为什么我只看到了两个选项？  
Jar files from libraries

* extract to target jar
* copy to the output directory and link via manifest

在排除了我打开方式不对的因素之外，我在google上寻求帮助，发现他们好像真的没做这个feature。不得不说，真的非常amazing，就好像在我眼中世界第一好吃的外卖店突然告诉我他们其实不提供打包服务，你只能站着吃完再走一样。  


###one jar

这是我在google上找到的另外一个解决方案(感谢google)。One-jar application。[用 One-JAR 简化应用程序交付](http://www.ibm.com/developerworks/cn/java/j-onejar/),这是作者P. Simon Tuffs 在04年的一篇博客，描述了他搭建one-jar的全过程，[one-jar application](http://one-jar.sourceforge.net/index.php?page=introduction&file=intro)是项目的主页，由于作者解释的比较详尽，项目的文档比较详实，这里就做个简单的介绍。  

quickstart里面提到，one-jar主要由以下几种使用方式:  

* application generator mode 用one-jar-appgen-0.97.jar生成一个项目模板，然后你往里面填东西，如果你的项目已经完成，基本可以忽略这种方式。  
* command-line approach  需要下载 one-jar-boot-0.97.jar, 解压后会生成一些文件夹，你要做的就是把项目包扔到main文件夹里，把依赖库扔到lib文件夹里。然后修改下boot-manifest.mf，最后打包就行了，是最傻瓜式的一种方式。  
* maven approach 用maven生成
* ant taskdef approach 下载一些包和一来的xml，直接在ant脚本里生成。这个需要下载lib/one-jar-ant-task-0.96.jar，单独拿出one-jar-ant-task.xml，修改下里面的路径，然后用类似的task就能完成了。  

        <target name="service" depends="compile">
        <one-jar destfile="${basedir}/XX.jar">
            <main>
                <fileset dir="${classes.dir}"/>
            </main>
            <manifest>
                <attribute name="One-Jar-Main-Class" value="Service"/>
            </manifest>
            <lib>
                <fileset dir="${basedir}/lib">
                      <patternset refid="library.patterns"/>
                </fileset>
            </lib>
        </one-jar>
        </target>
其实生成的jar的目录结构和command-line approach是一样的。  

