#今日工作小结
早上写了两行java代码之后开始专注的修改python代码  
##有关name_profile和yp_profile的读取问题  
###python解析bytearray
name_profile中的id是被hbase util的Bytes.toBytes编码成bytearray过的，直接用http来拿的话会得到一个bytearray,所以要想办法把bytearray解码成long类型。瞄了一眼toBytes(long val)的实现, 长成这样:  

    public static byte[] toBytes(long val) {
        byte [] b = new byte[8];
        for (int i = 7; i > 0; i--) {
          b[i] = (byte) val;
          val >>>= 8;
        }
        b[0] = (byte) val;
        return b;
    } 

简单的移位操作,>>>代表的是无符号右移动，>>是有符号右移.  
python里面用到的对的包是struct, 作用是转化python value和用python字符串表示的c structs, 通常用来处理二进制文件和网络数据。  
里面最常用的函数是pack和unpack  

    pack('l', 10087476)
    unpack('l', '\x00\x00\x00\x00\x00\x99\xEC4')

对于拿到的encode之后的id，只要正常的unpack然后做一次反转操作就行了。跟byte order半毛钱关系都没有，因为两种编码方式都是big endion, 所以也不用考虑这么烦的事儿了  
另外还有一些控制byte order，size和对齐的一些符号，放在最开始就能起作用。用的时候逐一查看吧。  

###hbase查询long做key的情况
之前在hbase shell里面傻不拉几的运行  

    get 'yp_profile', '\x00\x00\x00\x00\x00\x99\xEC4'
结果什么东西都没出来,结果  

    get 'yp_profile', 10087476
立刻就有结果了  
好吧，看来shell不会比我傻。。  
后来用rest发request也发现了同样的情况，后来直接id当成字符串扔了过去返回的结果才是对的，内部应该做了同样的事儿，把字符串先转成正确的类型的bytearray，然后再进行query。  
下午翻hbase文档的时候发现  

    $ curl http://<servername>:8080/testtable/%01%02%03/colfam1:col1

相当拉风啊~~

##google protobuf2json的改写
其实这个没什么好说的  
就是花了点时间去看了下google的document就行了  
