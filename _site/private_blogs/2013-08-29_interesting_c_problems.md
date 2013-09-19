#两个c语言的趣味题

挺有意思的,加深对注释和宏的好玩儿性的了解,记下来  

##写一个c程序输出hello,删除第一个字符输出world

    //*
    main() {puts("hello");}
    /*/
    main() {puts("world");}
    //*/
同时用了/\*\*/注释和//注释,搭配一下，效果就无敌了  

##写一个程序输出hello，删除最后一个字符输出world

    #ifndef D2
    #  define D2
    #  define D
    #  include __FILE__
    #  ifdef D
          main() { puts("hello");}
    #  else
         main() { puts("world");}
    #  endif
    #endif
    #undef D2

真是太贱了。。。。。
