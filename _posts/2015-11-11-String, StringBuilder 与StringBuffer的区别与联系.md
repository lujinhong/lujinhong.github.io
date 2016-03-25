---
layout: post
tile:  "String, StringBuilder 与StringBuffer的区别与联系"
date:  2015-11-11 18:55:47
categories: java 
excerpt: String, StringBuilder 与StringBuffer的区别与联系
---

* content
{:toc}




##1、区别

（1）String构建的对象不能改变，每次对String进行操作时，如两个String相加，需要新建一个String对象，然后容纳最终的结果。
     而StringBuilder与StringBuffer构建的对象可以随时在修改其内容，而无需生成新的对象。一般新建一个对象是会生成16个字节的空间，之后根据需要再增加空间。
      由于一般新构建一个对象涉及分配内存空间分配、无引用对象过多时的垃圾回收等，因此，对于操作频繁的字符串需使用StringBuilder或StringBuffer。


（2）StringBuilder与StringBuffer二者之间的区别在于前者的性能更高，但非线程安全的。而StringBuffer是线程安全的。
 因此在大部分情况下，建议使用StringBuilder，除非涉及多线程情况。

 

##2、官方文档中的一些说明

（1）String

The String class represents character strings. Strings are constant; their values cannot be changed after they are created. String buffers support mutable strings. Because String objects are immutable they can be shared. 

（2）StringBuilder

A mutable sequence of characters.This class provides an API compatible with StringBuffer, but with no guarantee of synchronization. 因此，StringBuilder与StringBuffer的 API基本完全一致。

 This class is designed for use as a drop-in replacement for StringBuffer in places where the string buffer was being used by a single thread (as is generally the case). Where possible, it is recommended that this class be used in preference to StringBuffer as it will be faster under most implementations.

The principal operations on a StringBuilder are the append and insert methods, which are overloaded so as to accept data of any type. Each effectively converts a given datum to a string and then appends or inserts the characters of that string to the string builder. The append method always adds these characters at the end of the builder; the insertmethod adds the characters at a specified point.

For example, if z refers to a string builder object whose current contents are "start", then the method call z.append("le") would cause the string builder to contain "startle", whereas z.insert(4, "le") would alter the string builder to contain "starlet".

In general, if sb refers to an instance of a StringBuilder, then sb.append(x) has the same effect as sb.insert(sb.length(), x). Every string builder has a capacity. As long as the length of the character sequence contained in the string builder does not exceed the capacity, it is not necessary to allocate a new internal buffer. If the internal buffer overflows, it is automatically made larger.

Instances of StringBuilder are not safe for use by multiple threads. If such synchronization is required then it is recommended that StringBuffer be used.

 

（3）StringBuffer

A thread-safe, mutable sequence of characters. A string buffer is like a String, but can be modified. At any point in time it contains some particular sequence of characters, but the length and content of the sequence can be changed through certain method calls.

 

String buffers are safe for use by multiple threads.The methods are synchronized where necessary so that all the operations on any particular instance behave as if they occur in some serial order that is consistent with the order of the method calls made by each of the individual threads involved.

The principal operations on a StringBuffer are the append and insert methods, which are overloaded so as to accept data of any type. Each effectively converts a given datum to a string and then appends or inserts the characters of that string to the string buffer. The append method always adds these characters at the end of the buffer; the insert method adds the characters at a specified point.

For example, if z refers to a string buffer object whose current contents are "start", then the method call z.append("le") would cause the string buffer to contain "startle", whereas z.insert(4, "le") would alter the string buffer to contain "starlet".

In general, if sb refers to an instance of a StringBuffer, then sb.append(x) has the same effect as sb.insert(sb.length(), x).

Whenever an operation occurs involving a source sequence (such as appending or inserting from a source sequence) this class synchronizes only on the string buffer performing the operation, not on the source.

Every string buffer has a capacity. As long as the length of the character sequence contained in the string buffer does not exceed the capacity, it is not necessary to allocate a new internal buffer array. If the internal buffer overflows, it is automatically made larger. As of release JDK 5, this class has been supplemented with an equivalent class designed for use by a single thread, StringBuilder. The StringBuilder class should generally be used in preference to this one, as it supports all of the same operations but it is faster, as it performs no synchronization.
