# Android 重复 bitmap 问题



重复 bitmap 问题，即有两条不同的 重复bitmap 引用链；

![1](https://raw.githubusercontent.com/XYScience/Blog/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/1.jpg)      
![2](https://raw.githubusercontent.com/XYScience/Blog/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/2.jpg)

#### 使用工具分析

##### 1，分析工具

* MemoryAnalyzer

  MemoryAnalyzer（MAT）是分析 hprof 文件的工具

* GIMP

  GIMP（Gnu Image Manipulation Program）可以用来还原 hprof 文件中的图片

##### 2，解析 hprof 文件

* 转换 hprof 文件

  原始 hprof 文件不能直接用 MAT 打开，需要使用命令转换：

  hprof-conv -z xxx xxx_conv.hprof （xxx为原生 hprof 文件，只是没带文件后缀）

  **注：**hprof-conv 命令行工具位于 ${Android SDK}/platform-tools/hprof-conv.exe 路径，需要配置环境变量

* MAT 打开 hprof 文件

##### 3，分析 重复bitmap 问题

* 过滤 bitmap 实例

  ![3](https://raw.githubusercontent.com/XYScience/Blog/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/3.jpg)

  对比发现存在多个 bitmap 实例 buffer 内容完全一样；

**为什么同一张图片在内存中存在两个以上的 bitmap 实例？**

**每张 bitmap 占用 4.49M，是否可以优化，如何优化？**

* 提取 重复bitmap 的 bitmap 引用链

  根据引用链分析 bitmap 的引用情况，结合源码分析生成多个 bitmap 实例的原因和优化方案；
  
  ![4](https://github.com/XYScience/Blog/raw/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/4.jpg)
  
  ![5](https://raw.githubusercontent.com/XYScience/Blog/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/5.jpg)

##### 4，手动导出图片

* 导出图片的 buffer 信息，文件名如：3383d190.data

  ![6](https://raw.githubusercontent.com/XYScience/Blog/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/6.jpg)

* 得到该 bitmap 实例对于的原始图片

  ![7](https://raw.githubusercontent.com/XYScience/Blog/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/7.jpg)

  ![8](https://github.com/XYScience/Blog/raw/master/Work/Android/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/Android%20%E9%87%8D%E5%A4%8D%20bitmap%20%E9%97%AE%E9%A2%98/images/8.jpg)

  
