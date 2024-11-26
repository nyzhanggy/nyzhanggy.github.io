---
title: Android 基础：NDK
categories: [根深柢固]
---

NDK即Native Development Kit，是Android上用来开发c/c++的开发工具包。

### NDK编译基础

NDK支持的编译方式有两种：
- CMake：NDK的默认构建工具，可在CMakeLists.txt 构建脚本中配置编译选项
- ndk-build：可在Android.mk 和 Application.mk文件中配置编译选项

在build.gradle文件中设置要使用那种方式
```java
android {
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt" // 配置 CMake
        }
        ndkBuild {
            path "Android.mk" // 配置 ndk-build
        }
    }
}
```

|特性|Android.mk + Application.mk|CMakeLists.txt|
|:-|:-|:-|
|历史背景|传统方式，老版 NDK 支持|新版方式，现代 NDK 支持|
|支持工具链|基于 ndk-build 工具|基于 CMake，跨平台通用|
|灵活性|相对较低，较依赖 Android 专用工具|更灵活，支持更多平台和工具|
|推荐使用|不再推荐，仅用于维护旧项目|推荐，现代项目主流选择|

现代 Android 项目普遍使用 CMake（通过 Gradle 配合 CMake），替代传统的 Android.mk 和 Application.mk。

### JNI基础

JNI即java native interface，是java和native代码进行交互的接口；

#### 静态注册

```kotlin
//Kotlin
class NativeDemo {
    external fun stringFromJNI(): String;
}
```

```cpp
//cpp
#include <jni.h>
#include <string>

extern "C" 
JNIEXPORT jstring JNICALL
Java_com_bc_sample_NativeDemo_stringFromJNI(
        JNIEnv* env,
        jobject obj) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

#### 动态注册

```kotlin
class NativeLib {
    // Declare the native method
    external fun dynamicHello(name: String): String

    companion object {
        // Load the native library
        init {
            System.loadLibrary("native-lib")
        }
    }
}
```

```cpp
#include <jni.h>
#include <string>

static const char *jniClassName = "com/learn/nativelib/NativeLib";

jstring nativeDynamicHello(JNIEnv *env, jobject objc, jstring name) {
    std::string message = "FromDynamicJNI";
    return env->NewStringUTF(message.c_str());
}

static JNINativeMethod methods[] = {
        {"dynamicHello", "(Ljava/lang/String;)Ljava/lang/String;", (void *) nativeDynamicHello}
};

static int registerNativeMethod(JNIEnv *env) {
    jclass clazz = env->FindClass(jniClassName);
    if (clazz == nullptr) {
        return JNI_FALSE;
    }
    jint methodSize = sizeof(methods) / sizeof(methods[0]);
    if (env->RegisterNatives(clazz, methods, methodSize) < 0) {
        return JNI_FALSE;
    }

    return JNI_TRUE;
}

JNIEXPORT jint JNICALL
JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env = nullptr;
    if (vm->GetEnv((void **) &env, JNI_VERSION_1_6) != JNI_OK) {
        return JNI_ERR;
    }
    if (!registerNativeMethod(env)) {
        return JNI_ERR;
    }
    return JNI_VERSION_1_6;
}

```

动态注册的优点：
- 灵活性：与 Java 一样，动态注册避免了静态注册复杂的函数命名规则。
- 兼容性：更容易与现有的 C/C++ 项目集成。
- 跨平台：与 CMake 或 NDK 构建工具结合，支持多种架构（如 ARM 和 x86）。