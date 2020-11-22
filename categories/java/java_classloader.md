# java ClassLoader #

----------
想要解决的问题：

1. **开发人员为什么要编写自定义的类加载器？**
2. **怎么使用类加载器去加载类？**
3. **Thread.currentThread().getClassLoader()和this.class.getClassLoader()分别在什么时候使用？区别是什么？**
4. **java类加载器的知识点**
5. **如何写一个类加载器**



----------

### 什么时候要编写自定义的类加载器 ###

java的三个类加载器加载的目录都是指定好的，所以当我们需要加载一些自定义目录下的jar文件，运行时生成的字节数组，或者从远程服务器加载类这些时，会去编写自定义的类加载器。

### 怎么使用类加载器去加载类 ###

首先我们得了解类加载器的API:

    package java.lang;

	public abstract class ClassLoader {
	
	  public Class loadClass(String name);
	  protected Class defineClass(byte[] b);
	
	  public URL getResource(String name);
	  public Enumeration getResources(String name);
	  
	  public ClassLoader getParent()
	}

到目前为止，java.class.ClassLoader最重要的方法是loadClass方法,该方法获取要加载类的标准名称返回一个Class对象。defineClass方法是用来兑现为JVM的一个类的，byte[]数组可以是从磁盘读入或者网络读入的字节数组。getResource和getResources方法返回指定资源的URL对象。我们可以将loadClass和defineClass(getResource(name).getBytes())大似等同。

    
	public class A{
		public void doSomething() {
			B b = new B();
		}
	}

这里可以等同于 B b = A.class.getClassLoader().loadClass("B");每个类都与用于加载该类的类加载器有关联。实例化classLoader()时，可以指定父类加载器。如果未明确指定父类加载器，则会使用系统默认的父加载器。


### Thread.currentThread().getClassLoader()和this.class.getClassLoader()分别在什么时候使用？区别是什么？###

打个简单的比方，你一个WEB程序，发布到Tomcat里面运行。
首先是执行Tomcat org.apache.catalina.startup.Bootstrap类，这时候的类加载器是ClassLoader.getSystemClassLoader()。
而我们后面的WEB程序，里面的jar、resources都是由Tomcat内部来加载的，所以你在代码中动态加载jar、资源文件的时候，首先应该是使用Thread.currentThread().getContextClassLoader()。如果你使用Test.class.getClassLoader()，可能会导致和当前线程所运行的类加载器不一致（因为Java天生的多线程）。


### java类加载器的知识点 ###

我觉得应该分为三个知识点。

1.