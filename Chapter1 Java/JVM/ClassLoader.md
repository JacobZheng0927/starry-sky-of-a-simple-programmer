# classLoader

[TOC]

ClassLoader翻译过来就是类加载器，普通的java开发者其实用到的不多，但对于某些框架开发者来说却非常常见。理解ClassLoader的加载机制，也有利于我们编写出更高效的代码。ClassLoader的具体作用就是将class文件加载到jvm虚拟机中去，程序就可以正确运行了。但是，jvm启动的时候，并不会一次性加载所有的class文件，而是根据需要去动态加载。想想也是的，一次性加载那么多jar包那么多class，那内存不崩溃。本文的目的也是学习ClassLoader这种加载机制。。

## ClassLoader结构

![java类加载机制](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/java%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6.png)

所有类加载器的基类，它是抽象的，定义了类加载最核心的操作。所有继承与classloader的加载器，都会优先判断是否被父类加载器加载过，防止多次加载，防止加载冲突

> 二进制名称形如
>
> `java.lang.String`
>
> `javax.swing.JSpinner$DefaultEditor`
>
> `java.security.KeyStore$Builder$FileBuilder$1`
>
> `java.net.URLClassLoader$3$1`

```java
    /**
		 * 用指定的二进制名称加载类。
     *
     * @param  name
     *         The <a href="#name">binary name</a> of the class
     *
     * @return  The resulting <tt>Class</tt> object
     *
     * @throws  ClassNotFoundException
     *          If the class was not found
     */
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

		protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
    		//锁，防止多次加载，所以jvm启动巨慢
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                		//准备委派给父类加载
                    if (parent != null) {
                    		//父类存在，委派给父类
                        c = parent.loadClass(name, false);
                    } else {
                    		//父类不存在，委派给启动类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

	            if (c == null) {
	               // If still not found, then invoke findClass in order
	               // to find the class.父类加载不到，自身再加载
	               long t1 = System.nanoTime();
	               c = findClass(name);

	               // this is the defining class loader; record the stats
	               sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
	               sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
	               sun.misc.PerfCounter.getFindClasses().increment();
	           }
	       }
	       if (resolve) {
         		resolveClass(c);
         }
	        return c;
	   }
}
```

jdk 1.7为了提供并行加载class，提供ClassLoader.ParallelLoaders内部类，用来封装一组并行能力的加载器类型。这个一般是用不到的，有兴趣可以先看一下。但是需要知道ClassLoader是支持并行加载的。

### Bootstrap classLoader

位于java.lang.classload，所有的classload都要经过这个classload判断是否已经被加载过，采用native code实现，是JVM的一部分，主要加载JVM自身工作需要的类，如java.lang.、java.uti.等； 这些类位于$JAVA_HOME/jre/lib/rt.jar。Bootstrap ClassLoader不继承自ClassLoader，因为它不是一个普通的Java类，底层由C++编写，已嵌入到了JVM内核当中，当JVM启动后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器。

```java
/**
 * Returns a class loaded by the bootstrap class loader;
 * or return null if not found.
 */
private Class<?> findBootstrapClassOrNull(String name)
{
    if (!checkName(name)) return null;

    return findBootstrapClass(name);
}

// return null if not found
private native Class<?> findBootstrapClass(String name);
```

### SecureClassLoader

继承自ClassLoader，添加了关联类源码、关联系统policy权限等支持。

### URLClassLoader

继承自SecureClassLoader，支持从jar文件和文件夹中获取class，继承于classload，加载时首先去classload里判断是否由bootstrap classload加载过，1.7 新增实现closeable接口，实现在try 中自动释放资源，但扑捉不了.close()异常

```java
/**
 * This class loader is used to load classes and resources from a search
 * path of URLs referring to both JAR files and directories. Any URL that
 * ends with a '/' is assumed to refer to a directory. Otherwise, the URL
 * is assumed to refer to a JAR file which will be opened as needed.
 * <p>
 * The AccessControlContext of the thread that created the instance of
 * URLClassLoader will be used when subsequently loading classes and
 * resources.
 * <p>
 * The classes that are loaded are by default granted permission only to
 * access the URLs specified when the URLClassLoader was created.
 *
 * @author  David Connelly
 * @since   1.2
 */
public class URLClassLoader extends SecureClassLoader implements Closeable
```

### ExtClassLoader

扩展类加载器，继承自URLClassLoader继承于urlclassload，扩展的class loader，加载位于$JAVA_HOME/jre/lib/ext目录下的扩展jar。查看源码可知其查找范围为System.getProperty(“java.ext.dirs”)。

```java
public static Launcher.ExtClassLoader getExtClassLoader() throws IOException {
            //System.getProperty("java.ext.dirs");
            //在项目启动时就加载所有的ext.dirs目录下的文件，并将其初始化
            final File[] var0 = getExtDirs();

	        try {
	        //AccessController.doPrivileged特权，让程序突破当前域权限限制，临时扩大访问权限
	           return (Launcher.ExtClassLoader)AccessController.doPrivileged(new PrivilegedExceptionAction<Launcher.ExtClassLoader>() {
	               public Launcher.ExtClassLoader run() throws IOException {
	                   int var1 = var0.length;

	                   for(int var2 = 0; var2 < var1; ++var2) {
	                       MetaIndex.registerDirectory(var0[var2]);
	                   }

	                   return new Launcher.ExtClassLoader(var0);
	               }
	           });
	       } catch (PrivilegedActionException var2) {
	           throw (IOException)var2.getException();
	       }
	   }
```

### AppClassLoader

应用类加载器，继承自URLClassLoader，也叫系统类加载器（ClassLoader.getSystemClassLoader()可得到它），它负载加载应用的classpath下的类，查找范围System.getProperty(“java.class.path”)，通过-cp或-classpath指定的类都会被其加载，没有完全遵循双亲委派模型的，它重写的是loadClass方法

```java
public Class<?> loadClass(String var1, boolean var2) throws ClassNotFoundException {
    int var3 = var1.lastIndexOf(46);
    if (var3 != -1) {
        SecurityManager var4 = System.getSecurityManager();
        if (var4 != null) {
            var4.checkPackageAccess(var1.substring(0, var3));
        }
    }

  	//ucp是SharedSecrets获取的Java栈帧中存储的类信息
    if (this.ucp.knownToNotExist(var1)) {
      	//顶级类classloader加载的信息
        Class var5 = this.findLoadedClass(var1);
        if (var5 != null) {
            if (var2) {
							//link过程，Class载入必须link，link指的是把单一的Class加入到有继承关系的类树中
                this.resolveClass(var5);
            }

            return var5;
        } else {
            throw new ClassNotFoundException(var1);
        }
    } else {
        return super.loadClass(var1, var2);
    }
}
```

### Launcher

java程序入口，负责实例化相关class，ExtClassLoader和AppClassLoader都是其内部实现类

## 双亲加载机制

ClassLoader使用的是双亲委托机制来搜索加载类的，每个ClassLoader实例都有一个父类加载器的引用（不是继承的关系，是一个组合的关系），虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其它ClassLoader实例的的父类加载器。当一个ClassLoader实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。

![类加载过程](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B.png)