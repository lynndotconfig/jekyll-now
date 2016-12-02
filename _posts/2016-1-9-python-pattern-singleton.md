---
layout: post
title: design-pattern-of-python
tags: [python, patterns]
category: Python
---

结构型设计模式有:

* 适配器模式
* 聚集模式
* 桥接模式
* 组合模式
* 修饰模式
* 扩充模式
* 外观模式
* 享元模式
* 代理模式

## 适配器模式

### 将某个类的接口转换成客户端期望的另一个接口表示。适配器模式可以消除由于接口不匹配所造成的兼容性问题
下面的例子中，Adapter类实现了接口的统一调用。

```python
class Dog(object):
    def __init__(self):
        self.name = "Dog"

    def bark(self):
        return "woof!"


class Cat(object):
    def __init__(self):
        self.name = "Cat"

    def meow(self):
        return "meow!"


class Human(object):
    def __init__(self):
        self.name = "Human"

    def speak(self):
        return "hello"


class Car(object):
    def __init__(self):
        self.name = "Car"

    def make_noise(self, octane_level):
        return "vroom{0}".format("!" * octane_level)


class Adapter(object):
    def __init__(self, obj, **adapted_methods):
        self.obj = obj
        self.__dict__.update(adapted_methods)

    def __getattr__(self, attr):
        return getattr(self.obj, attr)

    def original_dict(self):
        return self.obj.__dict__


def main():
    objects = []
    dog = Dog()
    print dog.__dict__
    objects.append(Adapter(dog, make_noise=dog.bark))
    print objects[0].__dict__
    print objects[0].original_dict()
    cat = Cat()
    objects.append(Adapter(cat, make_noise=cat.meow))
    human = Human()
    objects.append(Adapter(human, make_noise=human.speak))
    car = Car()
    objects.append(Adapter(car, make_noise=lambda: car.make_noise(3)))

    for obj in objects:
        print "A {0} goes {1}".format(obj.name, obj.make_noise())


if __name__ == "__main__":
    main()
```

还有一个更中规中矩的写法，从定义接口开始，这大概是所有面向对象的语言构建适配器结构的方法。

```python
class EuropeanSocketInterface:
    def voltage(self):
        pass

    def live(self):
        pass

    def neutral(self):
        pass

    def earth(self):
        pass


class Socket(EuropeanSocketInterface):
    def voltage(self):
        return 230

    def live(self):
        return 1

    def neutral(self):
        return -1

    def earth(self):
        return 0


class USASocketInterface:
    def voltage(self):
        pass

    def live(self):

        pass

    def neutral(self):
        pass


class Adapter(USASocketInterface):
    __socket = None

    def __init__(self, socket):
        self.__socket = socket

    def voltage(self):
        return 110

    def live(self):
        return self.__socket.live()

    def neutral(self):
        return self.__socket.neutral()


class ElectricKettle:
    __power = None

    def __init__(self, power):
        self.__power = power

    def boil(self):
        if self.__power.voltage() > 110:
            print "Kettle on fire!"
        else:
            if self.__power.live() == 1 and \
               self.__power.neutral() == -1:
                print "Coffee time!"
            else:
                print "No power."


def main():
    socket = Socket()
    adapter = Adapter(socket)
    kettle = ElectricKettle(adapter)

    kettle.boil()

    return 0

if __name__ == "__main__":
    main()
```

### 参考文字：

[example of adapter design pattern in python][1]
[python patterns][2]



## 外观模式

**为子系统的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用**

## 享元模式

**通过共享以便有效的支持大量小颗粒对象**

## 代理模式

**为其他对象提供一个代理以控制对这个对象的访问**


  [1]: https://gist.github.com/pazdera/1145859
  [2]: https://github.com/lynndotconfig/python-patterns/blob/master/adapter.py
