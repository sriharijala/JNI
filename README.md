# Creating JNI apps

## From Java to call functions in dll in windows or library (lo) on unix/linux that are created in C or C++ follow the instructions.

```[java]
// set the JAVA_HOME and make sure you installed gcc to compile c or c++
// on Windows DOS prompt type following commands

set JAVA_HOME={jdk-installed-directory}
gcc -Wl,--add-stdcall-alias -I"%JAVA_HOME%\include" -I"%JAVA_HOME%\include\win32" -shared -o myjni.dll TestJNIPrimitive.c

```

## 1  .  Passing Arguments and Result between Java & Native Programs
### 1.1  Passing Primitives
Passing Java primitives is straight forward. A jxxx type is defined in the native system, i.e,. jint, jbyte, jshort, jlong, jfloat, jdouble, jchar and jboolean for each of the Java's primitives int, byte, short, long, float, double, char and boolean, respectively.
Java JNI Program: TestJNIPrimitive.java

```[java]
public class TestJNIPrimitive {
   static {
      System.loadLibrary("myjni"); // myjni.dll (Windows) or libmyjni.so (Unixes)
   }
 
   // Declare a native method average() that receives two ints and return a double containing the average
   private native double average(int n1, int n2);
 
   // Test Driver
   public static void main(String args[]) {
      System.out.println("In Java, the average is " + new TestJNIPrimitive().average(3, 2));
   }
}
```

This JNI program loads a shared library myjni.dll (Windows) or libmyjni.so (Unixes). It declares a native method average() that receives two int's and returns a double containing the average value of the two int's. The main() method invoke the average().
Compile the Java program into "TestJNIPrimitive.class" and generate the C/C++ header file "TestJNIPrimitive.h":

```[java]
 javac TestJNIPrimitive.java
 javah TestJNIPrimitive       // Output is TestJNIPrimitive.h
```

### 1.2 C Implementation - TestJNIPrimitive.c
The header file TestJNIPrimitive.h contains a function declaration Java_TestJNIPrimitive_average() which takes a JNIEnv* (for accessing JNI environment interface), a jobject (for referencing this object), two jint's (Java native method's two arguments) and returns a jouble (Java native method's return-type).

```[java]
JNIEXPORT jdouble JNICALL Java_TestJNIPrimitive_average(JNIEnv *, jobject, jint, jint);
```

The JNI types jint and jdouble correspond to Java's type int and double, respectively.
The "jni.h" and "win32\jni_mh.h" (which is platform dependent) contains these typedef statements for the eight JNI primitives and an additional jsize.
It is interesting to note that jint is mapped to C's long (which is at least 32 bits), instead of of C's int (which could be 16 bits). Hence, it is important to use jint in the C program, instead of simply using int. Cygwin does not support __int64.

```[java]
// In "win\jni_mh.h" - machine header which is machine dependent
typedef long            jint;
typedef __int64         jlong;
typedef signed char     jbyte;
 
// In "jni.h"
typedef unsigned char   jboolean;
typedef unsigned short  jchar;
typedef short           jshort;
typedef float           jfloat;
typedef double          jdouble;
typedef jint            jsize;
```

### 1.3 The implementation TestJNIPrimitive.c is as follows:

```[java]
#include <jni.h>
#include <stdio.h>
#include "TestJNIPrimitive.h"
 
JNIEXPORT jdouble JNICALL Java_TestJNIPrimitive_average
          (JNIEnv *env, jobject thisObj, jint n1, jint n2) {
   jdouble result;
   printf("In C, the numbers are %d and %d\n", n1, n2);
   result = ((jdouble)n1 + n2) / 2.0;
   // jint is mapped to int, jdouble is mapped to double
   return result;
}
```

### 1.4 Compile the C program into shared library (jni.dll).

```[java]
// MinGW GCC under Windows
> set JAVA_HOME={jdk-installed-directory}
> gcc -Wl,--add-stdcall-alias -I"%JAVA_HOME%\include" -I"%JAVA_HOME%\include\win32" -shared -o myjni.dll TestJNIPrimitive.c
```

### 1.5 Now, run the Java Program:
```[java]
> java TestJNIPrimitive
```

### 1.6 C++ Implementation - TestJNIPrimitive.cpp

```[java]
#include <jni.h>
#include <iostream>
#include "TestJNIPrimitive.h"
using namespace std;
 
JNIEXPORT jdouble JNICALL Java_TestJNIPrimitive_average
          (JNIEnv *env, jobject obj, jint n1, jint n2) {
   jdouble result;
   cout << "In C++, the numbers are " << n1 << " and " << n2 << endl;
   result = ((jdouble)n1 + n2) / 2.0;
   // jint is mapped to int, jdouble is mapped to double
   return result;
}
```

Use g++ (instead of gcc) to compile the C++ program:

```[java]
// MinGW GCC under Windows
> g++ -Wl,--add-stdcall-alias -I"%JAVA_HOME%\include" -I"%JAVA_HOME%\include\win32" -shared -o myjni.dll TestJNIPrimitive.cpp
```

Reference : https://www3.ntu.edu.sg/home/ehchua/programming/java/JavaNativeInterface.html




