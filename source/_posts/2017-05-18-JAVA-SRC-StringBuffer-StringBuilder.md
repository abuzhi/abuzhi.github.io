---
title: JAVA源码--StringBuffer,StringBuilder,String
author: abuzhi
date: 2021-05-18 18:32:00
categories: [Java, SourceCode]
tags: [StringBuffer, StringBuilder, String]
---

StringBuffer,StringBuilder,String 类源码实现机制详解

![](/images/2017-05-18/JAVA-SRC-StringBuffer-StringBuilder-StringBuffer.jpg)
![](/images/2017-05-18/JAVA-SRC-StringBuffer-StringBuilder-StringBuilder.jpg)




## 二者区别与联系

> 二都都继承自父类AbstractStringBuilder

> 对应方法中，StringBuffer 加入了synchronized所以说是线程安全的


### 自动扩容

> 机制源于父类AbstractStringBuilder

1. 计算最小长度

2. 按原串双倍计算，如果小，则直接加上后串的长度

```java  

abstract class AbstractStringBuilder implements Appendable, CharSequence {

        /**
         * The value is used for character storage.
         */
        char[] value;

        /**
         * The count is the number of characters used.
         */
        int count;

        ....

        public AbstractStringBuilder append(String str) {
            if (str == null)
                return appendNull();
            int len = str.length();
            ensureCapacityInternal(count + len);
            str.getChars(0, len, value, count);
            count += len;
            return this;
        }

        ....

        private void ensureCapacityInternal(int minimumCapacity) {
            // overflow-conscious code
            if (minimumCapacity - value.length > 0)
                expandCapacity(minimumCapacity);
        }    
        
        ....

        void expandCapacity(int minimumCapacity) {
            int newCapacity = value.length * 2 + 2;
            if (newCapacity - minimumCapacity < 0)
                newCapacity = minimumCapacity;
            if (newCapacity < 0) {
                if (minimumCapacity < 0) // overflow
                    throw new OutOfMemoryError();
                newCapacity = Integer.MAX_VALUE;
            }
            value = Arrays.copyOf(value, newCapacity);
        }            

}

```

## String 的不可变性 

[**String 不可变参考**][1] 

String 的不可变性在其源码中可以看出为什么不可变：

1. String是类型的，不可继承，防止被子类修改
2. 其本质为char数组，采用了private final修饰，就可以避免我们手动修改数组内的内容。

```java
/** String本质是个char数组. 而且用final关键字修饰.
    String是不可变的关键都在底层的实现，而不是一个final
*/

public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0


    ....

    ....

}

```






















 [1]: https://www.zhihu.com/question/20618891
