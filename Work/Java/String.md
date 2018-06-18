# String

#### 1，String s = new String("abc")
上述的语句中是创建了2个对象，第一个对象是"abc"字符串存储在常量池中，第二个对象在JAVA Heap中的 String 对象。
#### 2，String#intern
* 如果常量池中存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回

* 如果存在堆中的对象，会直接保存对象的引用，而不会重新创建对象

  

> 相关文章：
> http://tech.meituan.com/in_depth_understanding_string_intern.html
> https://juejin.im/post/58e51175a0bb9f006904bd09#comment