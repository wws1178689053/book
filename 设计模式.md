# book
笔记
# 一、单例模式

## 1、懒汉式，线程不安全

```java
public class Singleton{
    private static Singleton instance;
    private Singleton(){};
    
    public static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

**描述：**这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。
这种方式 lazy loading 很明显，不要求线程安全，在多线程不能正常工作。



## 2、懒汉式，线程安全

    public class Singleton{
        private static Singleton instance;
        private Singleton(){};
        
        public static synchronized Singleton getInstance(){
            if(instance == null){
                instance = new Singleton();
            }
            return instance;
    }

}

**描述：**这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。
优点：第一次调用才初始化，避免内存浪费。
缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。



## 3、饿汉式

```java
public class Singleton{
    private static Singleton instance = new Singleton();
    private Single(){};
    
    public static Singleton getInstance(){
        return instance;
    }
}
```

**描述：**这种方式比较常用，但容易产生垃圾对象。
优点：没有加锁，执行效率会提高。
缺点：类加载时就初始化，浪费内存。
它基于 classloader 机制避免了多线程的同步问题，不过，instance 在类装载时就实例化，虽然导致类装载的原因有很多种，在单例模式中大多数都是调用 getInstance 方法， 但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化 instance 显然没有达到 lazy loading 的效果。



## 4、双检锁/双重校验锁（DCL，即 double-checked locking）

```java
public class Singleton{
    private volatile static Singleton instance;
    private Singleton(){};
    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**描述：**这种方式采用双锁机制，安全且在多线程情况下能保持高性能。
getInstance() 的性能对应用程序很关键。

### 5、登记式/静态内部类

```java
public class Singleton{
    private class SingletonHolder{
        private static final Singleton INSTANCE = new Singleton();
    }
    private Singleton(){};
    
    public static final Singleton getInstance(){
        return SingletonHolder.INSTANCE;
    }
}
```

**描述：**这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。
这种方式同样利用了 classloader 机制来保证初始化 instance 时只有一个线程，它跟第 3 种方式不同的是：第 3 种方式只要 Singleton 类被装载了，那么 instance 就会被实例化（没有达到 lazy loading 效果），而这种方式是 Singleton 类被装载了，instance 不一定被初始化。因为 SingletonHolder 类没有被主动使用，只有通过显式调用 getInstance 方法时，才会显式装载 SingletonHolder 类，从而实例化 instance。想象一下，如果实例化 instance 很消耗资源，所以想让它延迟加载，另外一方面，又不希望在 Singleton 类加载时就实例化，因为不能确保 Singleton 类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化 instance 显然是不合适的。这个时候，这种方式相比第 3 种方式就显得很合理。

## 6、枚举

```java
public enum Singleton{
    INSTANCE;
    public void method(){};
}
```

**描述：**这种实现方式还没有被广泛采用，但这是实现单例模式的最佳方法。它更简洁，自动支持序列化机制，绝对防止多次实例化。
这种方式是 Effective Java 作者 Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还自动支持序列化机制，防止反序列化重新创建新的对象，绝对防止多次实例化。不过，由于 JDK1.5 之后才加入 enum 特性，用这种方式写不免让人感觉生疏，在实际工作中，也很少用。
不能通过 reflection attack 来调用私有构造方法。



# 二、工厂模式

工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

## 介绍

**意图：**定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

**主要解决：**主要解决接口选择的问题。

**何时使用：**我们明确地计划不同条件下创建不同实例时。

**如何解决：**让其子类实现工厂接口，返回的也是一个抽象的产品。

**关键代码：**创建过程在其子类执行。

**应用实例：** 1、您需要一辆汽车，可以直接从工厂里面提货，而不用去管这辆汽车是怎么做出来的，以及这个汽车里面的具体实现。 2、Hibernate 换数据库只需换方言和驱动就可以。

**优点：** 1、一个调用者想创建一个对象，只要知道其名称就可以了。 2、扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。 3、屏蔽产品的具体实现，调用者只关心产品的接口。

**缺点：**每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。这并不是什么好事。

**使用场景：** 1、日志记录器：记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方。 2、数据库访问，当用户不知道最后系统采用哪一类数据库，以及数据库可能有变化时。 3、设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口。

**注意事项：**作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。

## 实现

我们将创建一个 *Shape* 接口和实现 *Shape* 接口的实体类。下一步是定义工厂类 *ShapeFactory*。

*FactoryPatternDemo*，我们的演示类使用 *ShapeFactory* 来获取 *Shape* 对象。它将向 *ShapeFactory* 传递信息（*CIRCLE / RECTANGLE / SQUARE*），以便获取它所需对象的类型。

![工厂模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/AB6B814A-0B09-4863-93D6-1E22D6B07FF8.jpg)

### 步骤 1

创建一个接口:

## Shape.java

```java
public interface Shape{   
    void draw(); 
}
```

### 步骤 2

创建实现接口的实体类。

## Rectangle.java

```java
public class Rectangle implements Shape {    
    @Override   
    public void draw() {      
        System.out.println("Inside Rectangle::draw() method.");   
    } 
}
```



## Square.java

```java
public class Square implements Shape {    
    @Override   
    public void draw() {      
        System.out.println("Inside Square::draw() method.");   
    } 
}
```



## Circle.java

```java
public class Circle implements Shape {    
    @Override   
    public void draw() {      
        System.out.println("Inside Circle::draw() method.");   
    } 
}
```



### 步骤 3

创建一个工厂，生成基于给定信息的实体类的对象。

## ShapeFactory.java

```java
public class ShapeFactory {       
    //使用 getShape 方法获取形状类型的对象   
    public Shape getShape(String shapeType){      
        if(shapeType == null){         
            return null;
        }              
        if(shapeType.equalsIgnoreCase("CIRCLE")){         
            return new Circle();
        } else if(shapeType.equalsIgnoreCase("RECTANGLE")){         
            return new Rectangle();      
        } else if(shapeType.equalsIgnoreCase("SQUARE")){         
            return new Square();      }      
        return null;   
    } 
}
```



### 步骤 4

使用该工厂，通过传递类型信息来获取实体类的对象。

## FactoryPatternDemo.java

```java
public class FactoryPatternDemo {    
    public static void main(String[] args) {      
        ShapeFactory shapeFactory = new ShapeFactory();       
        //获取 Circle 的对象，并调用它的 draw 方法      
        Shape shape1 = shapeFactory.getShape("CIRCLE");       
        //调用 Circle 的 draw 方法      
        shape1.draw();       
        //获取 Rectangle 的对象，并调用它的 draw 方法      
        Shape shape2 = shapeFactory.getShape("RECTANGLE");       
        //调用 Rectangle 的 draw 方法      
        shape2.draw();       
        //获取 Square 的对象，并调用它的 draw 方法      
        Shape shape3 = shapeFactory.getShape("SQUARE");       
        //调用 Square 的 draw 方法      
        shape3.draw();   
    } 
}
```

