---
title: Java层调用Native层函数
categories:
- java
tags:
- jni

---

先来看下java如何使用c的so库，第一步加载so、第二步找到so中调用方法，第三步调用so方法。
- 第一步：在java中我们直接使用System.loadLibrary就能加载指定的so库了，一般都放在静态代码块中，只加载一次
例如：

```
static { 	
	/* 1. load  这里要说的是，我们指定的是native字符串，但是找的是libnative.so，自动加上了lib前缀与so(linux平台)后缀。 */
	System.loadLibrary("native"); /* libnative.so */
}
```
<!-- more -->

- 第二步：想在java代码中调用so的函数，首先要在java类中声明是本地方法：

```
public native void hello();
```
所以java类完整代码是：

```
public class JNIDemo {
	static { 		
		System.loadLibrary("native"); /* libnative.so */
 	}
	public native void hello();
	public static void main (String args[]) {
		JNIDemo d = new JNIDemo();		
		/* 2. map java hello <-->c c_hello */

		/* 3. call */
		d.hello();
	}
}
```
接下来就是如何让我们的native方法与so中的方法映射,JNI提供了两种方式，显式与隐式。

### 显式调用 ###

- 显式调用(动态注册)：就是我们自己将native函数注册到java类中
  原理：JNI在加载so库的使用会先执行一个固定的方法JNI_OnLoad函数，所以我们可以在这里建立我们的映射关系。
  JNI中已经提供里映射的描述：JNINativeMethod结构体

```
typedef struct {
	char *name;  /* java里嗲用的方法名
	char *signature; /* JNI字段描述符，用来表示Java里调用的方法的签名*/
	void *fnPtr; /* C语言中实现的本地函数 */
} JNINativeMethod;
```
通过这个结构体，我们就能将java中的native方法与c中的函数关联了。但是我们怎样建立关联呢？JNI提供了一个注册native方法的函数：
RegisterNatives,它负责将native方法通过虚拟机注册到java的类中，这样就完成了java调用native函数了。本地native.c完整代码：

```
#include <jni.h>  /* /usr/lib/jvm/java-1.6.0-openjdk/include/ */
#include <stdio.h>

#if 0
typedef struct {
	char *name;  /* java里嗲用的方法名
	char *signature; /* JNI字段描述符，用来表示Java里调用的方法的签名*/
	void *fnPtr; /* C语言中实现的本地函数 */
} JNINativeMethod;
#endif

void c_hello(){
	printf("hello world!\n");
}


/* 这个数组用来映射多个方法 */
static const JNINativeMethod methods[] = {
	{"hello","()v",(void *)c_hello},
};


JNI_OnLoad(JavaVM *jvm, void *reserved){
	JNIEnv *env;//java程序的运行环境
	jclass cls;
	
	/* 获得一个运行时环境 */
	if ((*jvm)->GetEnv(jvm, (void **)&env, JNI_VERSION_1_4)) {
		return JNI_ERR; /* JNI version not supported */
	}
	/* 查找java中的JNIDemo类 */
	cls = (*env)->FindClass(env,"JNIDemo");
	if (cls == NULL) {
		return JNI_ERR;
	}
	
	/**
	 *使用运行时环境，建立映射 java hello <--> c c_hello 
	 *jint RegisterNatives(JNIEnv *env, jclass clazz,
	 *const JNINativeMethod *methods, jint nMethods);
	 */
	if((*env) -> RegisterNatives(env,cls,methods,1) < 0)
		return JNI_ERR;
	
	return JNI_VERSION_1_4;
}
```
这里简单介绍下几个指针：
\*jvm 代表java虚拟机，每个进程都会有一个
\*env 代表每个线程的上下文环境，它提供了很多方法来操作java的类，对象等，如RegisterNatives、FindClass。
如果想查看这些方法的完整签名可以在jni.h中查找。
JNI_VERSION_1_4指定我们JNI的版本


我们先生成动态库libnative.so
> gcc -I/usr/lib/jvm/java-1.6.0-openjdk/include/ -shared -o libnative.so native.c
 
这里的gcc -I是用来指定jni.h的位置，-shared -o libnative.so是生成so库的命令。
再使用javac JNIDemo.java命令编译java文件
在执行java JNIDemo前，我们还有告诉虚拟机到哪加载我们的so文件，export LD_LIBRARY_PATH=.  这代表到当前路径找so文件。
完整的命令时：
> javac JNIDemo.java
//添加头文件的路径
gcc -I/usr/lib/jvm/java-1.6.0-openjdk/include/ -shared -o libnative.so native.c
//表示去哪个目录去搜索这个库
export LD_LIBRARY_PATH=.
java JNIDemo 

输出：
> book@book-desktop:~/tmp$ java JNIDemo 
hello world




### 隐私调用 ###
- 隐式调用(静态注册)：我们只需要遵循JNI的书写规范，jvm会自动帮我们映射java中的native方法与c中的函数
- 原理 ：使用javah -jni 可以生成一个.h的头文件，我们只要写一个c文件实现该.h函数就可以了。
我们使用 javah -jni JNIDemo 生成的JNIDemo.h文件内容：

```
#include <jni.h>
/* Header for class JNIDemo */
#ifndef _Included_JNIDemo
#define _Included_JNIDemo
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     JNIDemo
 * Method:    hello
 * Signature: ()V
 */
JNIEXPORT void JNICALL Java_JNIDemo_hello
  (JNIEnv *, jobject);
#ifdef __cplusplus
}
#endif
#endif

``` 

这里最重要的是JNIEXPORT void JNICALL Java\_JNIDemo\_hello(JNIEnv \*, jobject);这个是我们hello方法的全路径名
隐式调用就是在我们调用hello方法的时候，虚拟机先到对应的jni库中寻找该方法对应的全路径名本地方法，如果有那么就建立映射关系，
否则报错。javah生成的方法名规则是：Java开头 + 包名+类名+方法名，如果遇到“\_”则用“_l”代替。如果我们熟悉了JNI的字段描述符，
自己也能写出方法的全路径名。
我们新建一个native2 实现JNIDemo.h中的方法：



> #include <jni.h> 
> \#include <stdio.h>
> JNIEXPORT void JNICALL Java\_JNIDemo\_hello(JNIEnv \*env, jobject cls){
	printf("hello world!\n");
}


然后我们生成so，执行Java JNIDemo
>book@book-desktop:~/tmp$ ls
JNIDemo.java  native2.c
book@book-desktop:~/tmp$ gcc -I/usr/lib/jvm/java-1.6.0-openjdk/include/ -shared -o libnative.so native2.c
book@book-desktop:~/tmp$ ls
JNIDemo.java  libnative.so  native2.c
book@book-desktop:~/tmp$ javac JNIDemo.java 
book@book-desktop:~/tmp$ java JNIDemo 
hello world
book@book-desktop:~/tmp$ 

### 总结： ###
以上两种方式都是java层调用native层函数，相比起来，显示调用代码比较多，但是能很好的控制注册时机，而且自己映射方法，比较灵活
，隐式调用就显得比较死板了，方法名必须按照一定的格式书写，一旦java层包名变化，so层的函数调用就会失败，但优点就是代码量少。
大多数都是使用显示调用的。

