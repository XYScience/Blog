# .so库使用和注意

开发Android应用时，有时候Java层的编码不能满足实现需求，就需要到C/C++实现后生成SO文件，再用`System.loadLibrary()`加载进行调用，这里成为JNI层的实现。常见的场景如：加解密算法，音视频编解码等。在生成SO文件时，需要考虑适配市面上不同手机CPU架构，而生成支持不同平台的SO文件进行兼容。

#### 一，Android支持的CPU架构
对于CPU来说，不同的架构并不意味着一定互不兼容，根据目前Android共支持七种不同类型的CPU架构，其兼容特点可总结如下：
* armeabi设备只兼容armeabi；
* armeabi-v7a设备兼容armeabi-v7a、armeabi（针对有浮点运算或高级扩展功能的）；
* arm64-v8a设备兼容arm64-v8a、armeabi-v7a、armeabi；
* X86设备兼容X86、armeabi；
* X86_64设备兼容X86_64、X86、armeabi；
* mips64设备兼容mips64、mips；
* mips只兼容mips； 

**总结：**
* armeabi的SO文件基本上可以说是万金油，它能运行在除了mips和mips64的设备上，但在非armeabi设备上运行性能还是有所损耗；
* 64位的CPU架构总能向下兼容其对应的32位指令集，如：x86_64兼容X86，arm64-v8a兼容armeabi-v7a，mips64兼容mips；

#### 二，.so库最佳选择
从目前手机端CPU市场的份额数据看，ARM架构几乎垄断，所以，除非你的用户很特殊，否则几乎可以不考虑单独编译带入X86、X86_64、mips、mips64架构SO文件。而在ARM架构中，比如高通骁龙芯片在2014年发布的410／610／808及以后的芯片都是arm64-v8a架构，华为麒麟6xx系列和930及以后也都是arm64-v8a架构，联发科？？?

![联发科：一核有难，九核围观😜😝😂](http://upload-images.jianshu.io/upload_images/760576-18dda8e6362a8b8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以：
* 就目前市场份额而言，绝大部分的设备都已经是arm64-v8a、armeabi-v7a，可以只保留armeabi-v7a架构的SO文件，这样APK体积会更小（目前 Facebook 和 Twitter 以及Google家应用等大厂都只用armeabi-v7a的.so文件）。
* 如果想要兼容性更广、性能好些（毕竟对应架构设备使用对应.so库文件才能发挥最大性能），可以在不同的目录下面放置不同的.so库文件。

#### 三，使用注意
* **虽然arm64-v8a是可以向下兼容的，但是也是兼容的有限制的。**
   * 对于arm64-v8a架构的手机，在运行app时，进入jnilibs去读取库文件时，如果有arm64-v8a文件夹，就去找相对应的.so文件，如果没有该文件夹，去找armeabi-v7a文件夹，如果没有，再去找armeabi文件夹，如果连这个文件夹也没有，就抛出异常；
  * 如果有arm64-v8a文件夹，那么就找相对应的.so文件，注意：**如果没有找到，不会再往下（armeabi-v7a文件夹）找，而是直接抛出异常。**
* **应该为每个ABI目录提供对应的.so文件**
  例如：第三方aar文件，如果这个sdk对abi的支持比较全，可能会包含armeabi、armeabi-v7a、x86、arm64-v8a、x86_64五种abi，而你应用的其它so只支持armeabi、armeabi-v7a、x86三种，直接引用sdk的aar，会自动编译出支持5种abi的包。但是应用的其它so缺少对其它两种abi的支持，那么如果应用运行于arm64-v8a、x86_64为首选abi的设备上时，就会抛出异常。

* **如果用到了第三方依赖，比如上面一个例子，可以只保留一个ABI，且应在build.gradle中加入ndk.abiFilters：**
  ```
  defaultConfig {  
    ndk {  
        abiFilters "armeabi-v7a"// 指定ndk需要兼容的ABI(这样其他依赖包里x86,armeabi之类的so会被过滤掉) 
    }  
  }  
  ```

>参考
>[Android SO文件的兼容和适配](http://blog.coderclock.com/2017/05/07/android/Android-so-files-compatibility-and-adaptation/)
>[为何 Twitter 区别于微信、淘宝，只使用了 armeabi-v7a？](https://www.diycode.cc/topics/691)
>[关于Android的.so文件你所需要知道的
>](http://www.jianshu.com/p/cb05698a1968)