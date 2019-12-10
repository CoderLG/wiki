# 

**anaconda长用命令**

```
conda --version

conda create --name ... python = 3.7

//查看现有环境
conda info -e

//切换环境
deactivate ...

//删除环境
conda remove -n ... 
```





### Base

```
# -*- coding: UTF-8 -*-
names = ["aa", "qq", "dd"]
j = 0
print("序号\t姓名")
for i in names:
    j += 1
    print("%d\t%s" % (j, i))
print("-----------")
for i,item in enumerate(names):
    print("%d\t%s" % (i,item))
print("-----------")
for i,item in enumerate(names,1):
    print("%d\t%s" % (i,item))
print("===字典===")
stu = {"name": "mm", "age": 18}
for k in stu.keys():
    print(k)
print("--------")
for v in stu.values():
    print(v)
print("-"*12)
for item in stu.items():
    print(item)
print("---key是否在字典中-----")
key = "name"
if key in stu:
    print("常用in")
if stu.has_key(key):
    print("you")

print("===函数===")

a = 100
def test():
    global a
    print(a)
    a += 100
test()

print(a)

```



### Init del 继承多继承

```
# -*- coding: UTF-8 -*-
class Animal:

    def __init__(self,name):
        self.name = name
        print("动物的初始化")

    def eat(self):
        print("---吃饭---")

    def sleep(self):
        print("---睡觉---")


class Dog(Animal):
    def __init__(self,name):
        print("Dog初始化了")
        self.name = name

    def shout(self):
        print("---汪汪叫---")

    def __del__(self):
        print("---del---")

# class Cat(Animal):
#     def catch(self):
#         print("===")

class ZangAo(Dog):
    def fight(self):
        print("---fight---")

dog = Dog("gg")
dog.eat()
dog.shout()

dog1 = dog
print("name = " + dog.name)
print("-"*20)
del dog
print("dog被回收")
print("内存对象没有指向的时候，执行del方法")
del dog1
print("dog1被回收")
print("="*20)
print("方法调用时，先找自己的，在找父类的")
z = ZangAo("zangAo")
print(z.name)

----------------------------------------
# -*- coding: UTF-8 -*-
class A:
    def test(self):
        print("A---test")
class B:
    def test(self):
        print("B---test")
# class C(B,A):
    def test1(self):
        print("c---test1")
c = C()
print("C中方法的优先级与他的继承顺序有关")
print(C.__mro__)
c.test()


```

### 方法重写 父类方法的调用

```
# -*- coding: UTF-8 -*-
class Animal:

    def __init__(self):
        self.color = "黄色"
        print("动物的初始化")

    def eat(self):
        print("---吃饭---")

    def sleep(self):
        print("---睡觉---")

class Dog(Animal):
    def __init__(self,name):
        print("Dog初始化了")
        super().__init__()
        self.name = name

    def eat(self):
        super().eat()
        print("---我爱吃骨头---")
    def shout(self):
        print("---汪汪叫---")

d = Dog("mm")
print(d.color)
print(d.name)
d.eat()

```

### 类的属性和多态

```
# -*- coding: UTF-8 -*-

class A:
    name = "zt"
    __password = "123"
    def eat(self):
        print("A---eat")

class B():
   name = "B"  #类属性
    def __init__(self):
        self.name = "对象属性"
    def eat(self,name):
        print("B---eat" + name)


class C(A):
    # def eat(self,name,num):
    #     print("C---eat" + name + "," + num + "个")

    def eat(self,name,num,report):
        print("C---eat2" + name +" "+ num +" "+ report)
    def setName(self,mm):
        name = mm
        print("一切皆对象，只是在这个方法中name = " + name )

c = C()
c.eat("mm","8","good")
print(C.name)
print(c.name)

# c.name = "ww"
# print(c.name)

C.name = "YY"
print(c.name)

c.setName("LG")
print(c.name)

print("="*20)

b = B()

print(b.name)
 

```

### 对象和类的区别 静态方法

```
# -*- coding: UTF-8 -*-

class A():
    name = "yy"
    def test1(self):
        print("test1 -- 对象方法")

    @classmethod
    def test2(cls):
        cls.name = "LG"
        print("test -- 类方法")
    @staticmethod
    def test3():
        A.name = "mm"
a = A ()
print(A.name)
a.test2()
print(A.name)
print(a.name)

A.test2()
print(A.name)
print(a.name)

a.test3()
print(A.name)

```

_new init str_

```
# -*- coding: UTF-8 -*-

class User:

    def __new__(cls, *args, **kwargs):
        print("对象开始构造，如果没有下面这句，则对象构造不出来")
        return object.__new__(cls)

    def __init__(self,name,age):
        self.name = name
        self.age = age

        print("对象已经构造好了之后，解释器自动回调此方法")


    def __str__(self):
        print("__str__ 方法")
        return "此方法必须有返回值，为此对象"
u = User("yy",12)
print(u)
print("name = %s age = %d"%(u.name,u.age))

```

### 单利模式

```
#-*— coding: utf-8 -*-

class User:
    __instance = None
    def __init__(self,name):
        self.name = name

    def __new__(cls, *args, **kwargs):
        if not cls.__instance:
            cls.__instance = object.__new__(cls)

        return  cls.__instance

u1 = User("ll")
u2 = User("yy")

print(u1.name, u2.name)
print(u1 == u2)
print("u1 id = %s, u2 id = %s"%(id(u1), id(u2)))

```

### 工厂模式

```
# -*- coding: utf-8 -*-

class Person:

    def __init__(self,name):
        self.name = name

    def work(self,type):
        print("person 开始工作了")
        # Axe = StoneAxe("石头")
        # Axe.cut_tree()

        Axe = Factory.create_axe(type)
        Axe.cut_tree()
class Axe:

    def __init__(self,name):
        self.name = name

    def cut_tree(self):
        print("%s斧子在砍树"%(self.name))

class StoneAxe(Axe):

    def cut_tree(self):
        print("原始%s斧子在砍树"%(self.name))

class SteelAxe(Axe):

    def cut_tree(self):
        print("现代%s斧子在砍树"%(self.name))

class Factory:

    @staticmethod
    def create_axe(type):
        if type == "stone":
            return StoneAxe("石头")

        if type == "steel":
            return SteelAxe("钢铁")

p = Person("yy")
p.work("steel")

```

### 异常处理

```
# -*- coding: utf-8 -*-
a = "123"
f = None
try:
    print("try开始")
    f = open("test.txt")
    # f.write("hello\n")
    # f.write("word %d"%a)
    # print("开始关闭文件")

    try:
        content = f.read()
        i = 1/0
        content.index("hive")

    except ValueError as ex:
        print("里面的except")
        print(ex)

except Exception as ex:
    print("外面的exception")
    print(ex)

else: #没有except的话
    print("else")

finally:  #不管有没有异常
    print("finally")
    if f:
        f.close()

```

### 列表推导

```
#encoding = utf-8

a = [(x,y) for x in range(1,3)  for y in range(1,6) ]

print(a)

b = [ i for i in range(1,101) if i%2==1]
print(b)

```

