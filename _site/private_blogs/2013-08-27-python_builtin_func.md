#重新丰富一下自己的python知识
发现自己对python的标准函数库的知识异常匮乏,每天抽点时间恶补一下  

##built-in function

###classmethod(function)

    class C:
        @classmethod
        def f(cls, args...): ...

是针对类的方法。。。囧。。。还有静态方法@staticmethod。stackoverflow上找了一下，有哥们给出的解释是"类方法不是针对任何实例，但是仍然在某方面和这个类有一腿的方法。最有趣的地方就是可以被子类覆盖，这个是java的静态方法和python的模块级别的函数无法做到的。。所以可以把操纵某类A的模块函数f写成A的classmethod"。
，但是写了一个有着奇怪行为的脚本，自己还没想通，改天到stackoverflow上问问大牛们  
    
    class Tmp:
        def f_instance(self):
            print ("instance method")

        @classmethod
        def f(cls):
            print ("success")
            cls().f_instance()

        @staticmethod
        def f1():
            print ("static")
            print Tmp().f_instance()

    Tmp.f()
    Tmp.f1()

###compile
提到了ast,不懂是神马东西。。。。
