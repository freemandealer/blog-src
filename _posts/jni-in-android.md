title: 在Android中使用JNI笔记
date: 2014-11-14 21:46:15
tags: Note
---

## 为什么在Android中使用JNI ##
JNI是JAVA调用C的一个接口。在Android中使用JNI的原因很多：JNI调用C执行效率可能会提高、JNI的存在使得一些成熟的C库可以被JAVA程序调用。我这里使用JNI主要是和底层交互。JAVA可以调用C，C可以调用汇编，那么JAVA就没什么做不到的了。

<!--more-->

## 一个JNI使用实例 ##

### 在APP中的操作 ###
MainActivity是Android应用的主Activity类.在该类中新建两个native方法，并加上声明。当然也可以把JNI方法单独放置于一个类中，由主类调用。

    public native int reg(String pwd);
    public native int auth(String pwd);         
    static{
        System.loadLibrary("testJni");
    }

这里我增加了两个方法，名字分别为reg和auth，返回值是int，输入参数为String。 

### 生成头文件 ###
下面我们需要使用javah工具，根据含有native方法的类，生成一个包含C语言风格函数声明的头文件（其函数实现部分亦即后面需要被JAVA调用的代码）。

首先，确保JAVA文件已被编译，生成.class文件。在Eclipse里面就是打包运行一下工程，会发现运行不了，因为loadLibrary的那个"testJni"我们还没有构建，会报错。这里我们不用真的运行，只要产生了apk就说明.class文件已经被构建。

在项目根目录下敲入命令：

    javah -classpath bin/classes -d jni com.example.tcptlibtest.MainActivity

javah是jdk中包含的一个工具，专门根据.class文件生成对应C的头文件。通过`-classpath`参数配置.class文件的路径。`-d`指定生成的头文件保存的目录。最后写操作对象，即完整的类名，注意大小写。

命令执行完成后，会在工程根目录的jni目录中找到一个完整类名构成的头文件。打开头文件，里面包含native的方法名。如果没有，说明上述命令执行错了，可能是类名没写正确。

### 编写C代码 ###
新建一个C文件，开始处include 生成的头文件，实现头文件中的native函数。具体即复制头文件的声明部分，改写成实现。注意声明时只有类型，需要加上变量名。一个例子：

    JNIEXPORT jint JNICALL Java_com_example_tcptlibtest_MainActivity_reg
      (JNIEnv *env, jobject obj, jstring content)
    {
        const jbyte *str = (const jbyte*)(*env)->GetStringUTFChars(env,content,JNI_FALSE); //获得字符串参数
        /* 各种操作 */
        int ret;
        ret=reg_pwd(str); // 这里我们可以自由调用来自世界各地的C代码，java+jni -> java+c
        (*env)->ReleaseStringUTFChars(env, content, (const char *)str ); //释放字符串资源
        /*
         * 如果我们的返回值类型是字符串，可以通过下面的语句返回字符串给JAVA环境下的调用者
         * return (*env)->NewStringUTF(env, "Hello World! I am Native interface");
         */
        return ret;
    }

我们看到数据类型有些奇怪，字符串更是诡异。这主要是因为C和JAVA对数据类型定义不同造成（字节数，大小端，字符编码方式等），需要进行转换。调用时，参数的数据类型要从JAVA型转换为C,返回时再变换回去。更多的映射规则，请参看[这篇博客](http://blog.csdn.net/conowen/article/details/7523145)。

同时需要增加一个OnLoad函数，作为加载JNI的借口。我们可以在这个函数中实现一些JNI就被执行的代码。下面的例子仅仅是获得JNI版本号。

    jint JNI_OnLoad(JavaVM* vm, void* reserved)
    {
        void *venv;
        if ((*vm)->GetEnv(vm, (void**)&venv, JNI_VERSION_1_4) != JNI_OK) {
            return -1;
        }
        return JNI_VERSION_1_4;
    }

### 编译C代码 ###
App依然采用集成开发环境编译。但是JNI的C代码需要另外编译成库，才能被App调用，这是JNI原理决定的。在Windows中，JNI需要使用动态连接库（dll）。在Android中，库的后缀名是.so，这和Linux一致。

首先，需要一个指导构建库文件的说明书：新建Android.mk，内容如下：

    
    LOCAL_PATH:= $(call my-dir)
    include $(CLEAR_VARS)
    LOCAL_MODULE_TAGS := optional
    LOCAL_MODULE := libtestJNI   # 这里填写库的名字，就是Java文件中load的那个，前加上lib
    LOCAL_SRC_FILES := $(call all-subdir-c-files)
    LOCAL_PRELINK_MODULE := false
    LOCAL_C_INCLUDES += \
    $(JNI_H_INCLUDE)
    LOCAL_SHARED_LIBRARIES := \
    libandroid_runtime \
    libnativehelper \
    libcutils \
    libutils \
    liblog \
    libhardware
    include $(BUILD_SHARED_LIBRARY)

用NDK编译。如果下载了AOSP(Android源代码)的读者可以将整个App工程拷贝或链接进AOSP/packages/app/下，运行

    mmm packages/apps/你的App工程名字

这样你会在AOSP/out/target/product/generic/obj/lib/下找到编译好的库文件。该文件以lib打头，不用在意。

### 整合进系统 ###
现在App和库文件是分开的。也许有某种方式可以将.so文件打包进apk，但目前我的做法是将.so文件拷贝到手机`/system/lib`下，因为我发现那里存放着系统所使用的库文件。拷贝过程比平常的拷贝操作麻烦些，因为`/system/lib`挂载为只读，需要重挂。简单的方式是通过一个应用"R文件管理器"完成了这个操作。

整合完毕后，安装apk，就可以运行了。如果log出现载入库错误，那就说明库文件没有被载入。可能是文件名输入错了。库文件名除了前面有lib前缀，必须和JAVA代码中的loadLibrary参数完全一样。


## 输出log ##
在C文件头部加入:

    #include <android/log.h>
    #define LOG_TAG "cqEmbed"
    #define LOGI(...) __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)

