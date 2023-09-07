# WolfLink - IOC
[![](https://jitpack.io/v/WolfLink-DevTeam/WolfLink-IOC.svg)](https://jitpack.io/#WolfLink-DevTeam/WolfLink-IOC)
## 介绍

本项目是一个轻量级并且线程安全的 Java IOC（控制反转）容器，整个项目只包括一个 `IOC` 类以及三个注解：`BeanProvider`，`Inject`，`Singleton`。

IOC 容器是一些大型框架的核心部分(如 Spring,Solon...)，但对于一些简单应用来说，这些框架可能会显得过于重量级。本项目旨在为这些应用提供一个简单轻量实用可靠的替代方案。

- `BeanProvider`：此注解用于标记在配置类中提供 bean 的方法。该方法的返回类型就是提供的 bean 的类型，方法不接受参数，方法名随意。
- `Inject`：此注解用于标记需要注入的字段，在通过 IOC 容器获取该字段所属类的实例时将自动为这些字段绑定到相应单例或创建新的实例。
- `Singleton`：此注解用于标记单例类。对于标记为 `Singleton` 的类，IOC 容器将只创建一个实例。

## 引入依赖

Maven
```maven
<repositories>
	<repository>
	    <id>jitpack.io</id>
		<url>https://jitpack.io</url>
	</repository>
</repositories>
<dependency>
	<groupId>com.github.WolfLink-DevTeam</groupId>
	<artifactId>WolfLink-IOC</artifactId>
	<version>建议填写当前最新版本</version>
</dependency>
```

## 快速使用

以下是如何在您的 Java 项目中使用此轻量级 IOC 容器的示例：

### 通过IOC容器获取实例
使用 @Singleton 注解标记的类是单例类，通过 IOC 容器获取其实例时会保证该类的实例不会超过1个。
不使用 @Singleton 注解标记的类是多例类，每次通过 IOC 容器获取实例都会创建一个新的实例。
```java
import org.wolflink.common.ioc.IOC;
import org.wolflink.common.ioc.Singleton;

class Test {
    Test(){}
    Test(int a){}
}

@Singleton
class SingletonTest {
}

public class Main {
    public static void main(String[] args) {
        Test test1 = IOC.getBean(Test.class);
        Test test2 = IOC.getBean(Test.class);
        // 构造器参数按照这种方式传入
        Test test3 = IOC.getBean(Test.class,114514);
        SingletonTest sTest1 = IOC.getBean(SingletonTest.class);
        SingletonTest sTest2 = IOC.getBean(SingletonTest.class);
        // 最后的结果是 test1 != test2 != test3 而 sTest1 == sTest2
    }
}
```

### 创建一个配置类并使用 `BeanProvider` 注解提供 bean
```java
package org.example;

import org.wolflink.common.ioc.BeanProvider;

public class AppConfig {

    @BeanProvider
    public Service provideService() {
        return new ServiceImpl();
    }
}
```
然后，在使用 IOC 获取该 Service 类型的 bean 之前调用以下代码注册配置类：

```java
import org.wolflink.common.ioc.IOC;

public class Main {
    public static void main(String[] args) {
        // 注册配置类
        IOC.registerBeanConfig(AppConfig.class);
        // 使用配置类中提供的 Bean，此时service实例的实际类型为 ServiceImpl
        Service service = IOC.getBean(Service.class);
    }
}
```
### 使用 `Inject` 注解在您的类中注入依赖

```java
package org.example;

import org.wolflink.common.ioc.Inject;

public class Client {

    @Inject
    private Service service;

    public void doSomething() {
        service.doWork();
    }
}
```

通过 IOC 容器获取 Bean：

```java
package org.example;

import org.wolflink.common.ioc.IOC;

public class Main {

    public static void main(String[] args) {
        // 获取bean
        Client client = IOC.getBean(Client.class);
        // 使用bean
        client.doSomething();
    }
}
```