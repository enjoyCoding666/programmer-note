
学习 Spring 的源码，也可以通过 SpringBoot  搭环境。

不管是什么源码，最好写个 demo，跑起来，然后从常用的类和方法入手，跟踪调试。

### 配置对象

新建一个 SpringBoot 的项目， 详情见： https://blog.csdn.net/sinat_32502451/article/details/133039001

接着在 com.example.demo.model 路径建一个类 Person，属性包括 name和 age (也可以是其它的路径，与以下的class路径一致就行) 。

```
public class Person {


    private int age;


    private String name;
    
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}    
```



然后在 resources 文件夹下，添加一个  mySpring.xml 的文件，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class= "com.example.demo.model.Person">
        <property name="name" value="Tom"/>
        <property name="age" value="1"/>
    </bean>


</beans>
```



### 查看配置的对象值：

创建一个类，查看 xml 配置的对象的值：

```
public class MySpring {

    public static void main(String[] args) {
    	//可以在以下这段代码打个断点，跟踪调试
        ApplicationContext context =
                new ClassPathXmlApplicationContext("classpath*:mySpring.xml");
         //在 getBean 这一行，打个断点
        Person person = context.getBean("person", Person.class);
        System.out.println("=======>"+ person.getName());

    }

}
```

在 ApplicationContext 开头的这段代码，打个断点，
在 context.getBean 这一行， 也打个断点。
然后 跟踪调试。

### ClassPathXmlApplicationContext

从 xml 加载定义的 bean 对象，并且会通过 refresh 刷新 context 上下文 。

```
	/**
	 * Create a new ClassPathXmlApplicationContext with the given parent,
	 * loading the definitions from the given XML files.
	 * @param configLocations array of resource locations
	 * @param refresh whether to automatically refresh the context,
	 * loading all bean definitions and creating all singletons.
	 * Alternatively, call refresh manually after further configuring the context.
	 * @param parent the parent context
	 * @throws BeansException if context creation failed
	 * @see #refresh()
	 */
	public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		//设置配置文件的路径
		setConfigLocations(configLocations);
		if (refresh) {
			//refresh()方法，负责初始化 ApplicationContext 容器。可以重点看看。
			refresh();
		}
	}
```



### refresh() 源码

详情见： https://blog.csdn.net/sinat_32502451/article/details/140335613


### 必备插件：
* Translation ：(超好用)Intellij Idea 的翻译插件，右键选择Translate即可。或者用快捷键  ctrl+shift+ Y。
  开源代码看不懂，就用 Translation 翻译。

* UML。IDEA UML类图插件，可以展示Diagram，也就是类的继承和实现关系。

### 调试方法：
* 要带着问题去看源码，不需要所有的方法都看，陷入琐碎的支节。

* 重点看核心的方法。其实核心的方法，往往只有几个。
  如果不清楚哪些是核心的方法，可以多搜索一下。

* 可以通过 Step Into 进入方法，也就是 Intellij Idea 的 F7 快捷键，继续跟踪。

* Step Over ，也就是 Intellij Idea 的 F8 快捷键，单步调试，逐行调试。

* 不清楚变量是如何变化的，可以用 Intellij Idea 的 watch 监控变量，Alt + F8  快捷键。

* 遇到调用接口，如果不清楚是哪个接口实现类，可以直接在 Intellij Idea 的 接口上打断点，调试时会自动跳转到对应的接口实现类。

* 调试过程中，遇到不懂的，也可以百度搜索下。
