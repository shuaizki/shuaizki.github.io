---
layout: post
title: How to disable output to console
---

log4j用的不熟悉，但是为了项目的log能thread-safe就改用了这个。。。  
结果在邮件里被抱怨原文打到文件里的log现在在屏幕log里也有一份  
遂在stackoverflow上找到帖子一张  
[how can I disable output to log4j.rootLogger?](http://stackoverflow.com/questions/7513463/how-can-i-disable-output-to-log4j-rootlogger)  
后来证实是additivity项的设置问题  
在log4j的文档里面有这么一段话:  

    The output of a log statement of logger C will go to all the appenders in C and its ancestors. This is the meaning of the term "appender additivity".

    However, if an ancestor of logger C, say P, has the additivity flag set to false, then C's output will be directed to all the appenders in C and its ancestors upto and including P but not the appenders in any of the ancestors of P.

    Loggers have their additivity flag set to true by default.

于是一个log被传到rootlogger的appender里了，难怪info的时候到了stderr的输出里。目前自己的项目里直接把additivity设置为false就可以了，但是best answer给出的建议是把stdout, stderr从root logger里面去掉。  
