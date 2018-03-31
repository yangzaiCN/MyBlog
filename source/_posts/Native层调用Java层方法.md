---
title: Native层调用Java层方法
categories:
- java
tags:
- jni

---
接着上一篇，来学习下Native层调用Java层方法。
我们都知道，java语言是由虚拟机解释执行的，如果我们想用C语言执行java程序，那么必然要创建一个虚拟机，然后去加载指定的类，如果是静态方法，直接用类调用，如果不是静态的，那么还要创建对象，在调用方法，所以我们大概的罗列下步骤：
<!-- more -->
- 创建虚拟机
- 获得class
- 实例化对象
&emsp;&emsp;1.对于静态方法，不需要实例化对象
&emsp;&emsp;2.实例化对象
- 调用Java方法
&emsp;&emsp;1.获得方法ID
&emsp;&emsp;2.构造参数
&emsp;&emsp;3.调用方法

那么我们该如何创建虚拟机，构造参数等等呢？这就需要查jni文档了，其实我们需要的方法都罗列在jni.h的头文件里了。
我们需要用到如下几个方法：JNI_CreateJavaVM、JNIEnv.FindClass、JNIEnv.GetStaticMethodID、JNIEnv.CallStaticVoidMethod
从字面上我们很好理解方法的用途，具体的参数类型就要查文档了。
查官方文档Creating the Java Virtual Machine这一节，发现如何创建虚拟机，我简化下并抽取成方法create_vm：

```
jint create_vm(JavaVM** jvm, JNIEnv** env) {  
    JavaVMInitArgs args;  
    JavaVMOption options[1];  
    args.version = JNI_VERSION_1_6;  
    args.nOptions = 1;  
    options[0].optionString = "-Djava.class.path=./";  
    args.options = options;  
    args.ignoreUnrecognized = JNI_FALSE;  
    return JNI_CreateJavaVM(jvm, (void **)env, &args);  
}  
```

我们在定义一个Hello的java类

```
public class Hello {
	public static void main(String args[]) {
		System.out.println("Hello, world!");
	}
}
```

新建Native类c_call_java.c:

```
#include <stdio.h>  
#include <jni.h> 
//根据文档，简化一个创建jvm的方法
jint create_vm(JavaVM** jvm, JNIEnv** env) {  
    JavaVMInitArgs args;  
    JavaVMOption options[1];  
    args.version = JNI_VERSION_1_6;  
    args.nOptions = 1;  
    options[0].optionString = "-Djava.class.path=./";  
    args.options = options;  
    args.ignoreUnrecognized = JNI_FALSE;  
    return JNI_CreateJavaVM(jvm, (void **)env, &args);  
}  


int main(int argc, char **argv){
	/* 定义jvm与env的指针 */
	JavaVM* jvm;
	JNIEnv* env;

	jclass cls;
	int ret = 0;

	jmethodID mid;
		
	/* 1. 创建虚拟机 */
	if (create_vm(&jvm, &env)) {
		printf("can not create jvm\n");
		return -1;
	}
		
	/* 2. 让虚拟机加载Hello类 */
	cls = (*env)->FindClass(env, "Hello");
	if (cls == NULL) {
		printf("can not find hello class\n");
		ret = -1;
		goto destroy;
	}

	/* 3. 创建Hello对象 */

	/* 4. 调用方法
	 * 4.1 获取方法id
	 * 4.2 拼装方法参数
	 * 4.3 正在的调用方法
	 */
	//通过方法签名获取方法id，并映射到jmethodID中
	mid = (*env)->GetStaticMethodID(env, cls, "main","([Ljava/lang/String;)V");
	if (mid == NULL) {
		ret = -1;
		printf("can not get method\n");
		goto destroy;
	}
	//调用方法
	(*env)->CallStaticVoidMethod(env, cls, mid, NULL);

destroy:
	//销毁jvm
	(*jvm)->DestroyJavaVM(jvm);
	return ret;
}
```
有了虚拟机，加载了class，然后就可以调用方法了，jni约定了调用方法只需要传递相应的方法签名即可，这个签名封装为jmethodID。
例如："main","([Ljava/lang/String;)V"  就是调用参数为String数组，返回值为void的main方法，具体方法与返回值类型该怎么写
就要拆看jni的字段描述符  或者用javah来辅助生成方法签名
![](https://i.imgur.com/W973V3Z.png)
在写字段描述符的时候要注意几点：
- 用"["表示数组，比如"int []"表示为"[I"
- 对于类，要用全称"L包.子包.类名;"(前面有"L"，后面有";")，比如"Ljava/lang/String;"
- 除String类外，其他的类都用Object表示，即"Ljava/lang/Object;"

所以我们Hello.java中的main方法就可以写成这样了 "main","([Ljava/lang/String;)V" 
这里调用的是Hello的主方法，由于是静态的，不需要创建对象，而且不需要传递参数，所以CallStaticVoidMethod的最后一个参数列表为NULL。
上传Hello.java c_call_java.c文件,这里我们用到了jni.h文件还有创建虚拟机的方法，需要引入jni.h以及libjvm.so库：
用grep搜索下JNI_CreateJavaVM在哪个库中定义了
> book@book-desktop:/usr/lib/jvm$ grep "JNI_CreateJavaVM" -r .
> ./java-6-sun/jre/lib/i386/server/libjvm.so matches
发现在libjvm.so中有定义，所以链接进去。
> gcc -I/usr/lib/jvm/java-1.6.0-openjdk/include/ -o caller c_call_java.c -L /usr/lib/jvm/java-1.6.0-openjdk/jre/lib/i386/server -ljvm
除此之外还要在运行时指定libjvm.so  所以将so加入到环境变量中
LD_LIBRARY_PATH=/usr/lib/jvm/java-1.6.0-openjdk/jre/lib/i386/server 
完整编译命令是：

```
javac Hello.java
//在文件夹下查询  grep "JNI_CreateJavaVM" -r .
//-L指定库路径 -ljvm 把库链接进去  
gcc -I/usr/lib/jvm/java-1.6.0-openjdk/include/ -o caller c_call_java.c -L /usr/lib/jvm/java-1.6.0-openjdk/jre/lib/i386/server -ljvm
//设置环境变量
LD_LIBRARY_PATH=/usr/lib/jvm/java-1.6.0-openjdk/jre/lib/i386/server ./caller
```

最后输出 ：Hello，world！

### 总结： ###
Native调用Java方法主要就三步，创建虚拟机，加载类，调用方法，这里我们没有考虑方法参数问题，是最简易的demo，如果想进一步学习可以到官网查看文档。
补充说一点，我们知道Android就是C与Java实现的，所以Android内核启动Java应用层时肯定也使用到C调Java的技术，通过翻看Android的源代码，找到了Zygote进程打开Java世界就有以下代码：

```
App_main.cpp:
runtime.start("com.android.internal.os.ZygoteInit",startSystemServer);

AndroidRuntime.cpp
...
if (startVm(&mJavaVM, &env, zygote) != 0) {
        return;
}
...
jclass startClass = env->FindClass(slashClassName);
jmethodID startMeth = env->GetStaticMethodID(startClass, "main","([Ljava/lang/String;)V");
env->CallStaticVoidMethod(startClass, startMeth, strArray);

```
这样Android就通过ZygoteInit 的main方法进入Java世界，应用工程师就不用关心底层世界了。



