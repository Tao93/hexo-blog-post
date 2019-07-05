---
title: Java Comparable Contracts
tags: [Java]
---

实现 Java 中的 Comparable 接口的对象，可以用在 Collections#sort 等方法中进行排序等。通常来说，Comparable 接口的 compareTo 方法都是写起来比较简单的。不过，偶尔还是可能翻车的。比如下面的 crash log，就是翻车的现场：

```
java.lang.IllegalArgumentException: Comparison method violates its general contract! 
  at java.util.TimSort.mergeHi(TimSort.java:868) 
  at java.util.TimSort.mergeAt(TimSort.java:485) 
  at java.util.TimSort.mergeCollapse(TimSort.java:408) 
  at java.util.TimSort.sort(TimSort.java:214) 
  at java.util.TimSort.sort(TimSort.java:173) 
  at java.util.Arrays.sort(Arrays.java:659) 
  at java.util.Collections.sort(Collections.java:217) 
```

异常显示违反了 general contract，这些 contract 其实在 Comparable#compareTo 方法的注释中就讲到了，主要就是相反性、传递性、等价性三条 contract。这些 contract 可以写成下面这样的形式：

1. $sgn(x.compareTo(y)) = -sgn(y.compareTo(x)) $

2. $sgn(x.compareTo(y)) > 0$ and $sgn(y.compareTo(z)) > 0$ implies $sgn(x.compareTo(z)) > 0$

3. $sgn(x.compareTo(y)) = 0$ implies sgn(x.compareTo(z)) = sgn(y.compareTo(z))$

上面的 $sgn$ 函数就是下面这样的：

![](https://tao93.top/images/2018/12/22/1545477555.png)

所以第一条的意思是 x 与 y 的比较结果和 y 与 x 的比较结果必须相反；第二条的意思是，比较结果是具有传递性的，第 3 条是如果 x 和 y 的比较结果是相等的，那么它们与任何 z 的比较结果相同。

简单实现的 compareTo 方法通常不会违背上面的协议，不过，稍微复杂点的，就不一定了。比如下面的例子：

```java
    @Override
    public int compareTo(Object o2) {
        Data d2 = (Data) o2
        if (isSortByName()) {
            return getName().compareTo(d2.getName());
        } else if (isSortByAge()) {
            return getAge() - d2.getAge();
        } else {
            return 0;
        }
    }
```

以上面代码来实现 Comparable 接口的对象，然后进行排序，是可能存在的问题的。在一次完整的排序过程中，上面的 compareTo 方法需要调用多次，但是上面的 isSortByName 和 isSortByAge 方法的返回是可能变化的，比如多线程情况下其他线程可能修改了 isSortByName 方法所使用的变量的值。当 isSortByName 的结果变化时，意味着排序的比较标准也变了，自然非常容易违反前面说到的 contract。

而下面的代码片段，同样是有问题的：

```java
    @Override
    public int compareTo(Object o2) {
        Data d2 = (Data) o2
        boolean date1Empty = TextUtils.isEmpty(getDateStr());
        boolean date2Empty = TextUtils.isEmpty(d2.getDateStr());
        if (date1Empty && date2Empty) {
            return 0;
        }
        if (! date1Empty && ! date2Empty) {
            SimpleDateFormat format = new SimpleDateFormat("....");
            try {
                Date date1 = format.format(getDateStr());
                Date date2 = format.format(d2.getDateStr());
                return date1.compareTo(date2);
            } catch(ParseException e) {
                return 0;
            }
        }
    }
```

上面代码的问题在于，ParseException 被捕获并且直接返回 0 了。这意味着两个 date string 只要有一个是无法解析的，那么比较结果就是0，也就是相等。加入 a, b, c 三个 date string，只有 b 是无法解析的，那么 a 和 b 比较结果为 0，b 和 c 比较结果也是 0，则根据第 3 条 contract，a 和 c 的比较结果也应该是 0，可是 a 和 c 都是可以正常解析的，它们的比较结果不一定是 0。所以第 3 条 contract 会被违反。

要改正上面的代码片段，可以直接把异常抛出，以期提前将不可解析的 date string 避免掉。另一种方式更啰嗦点，就是两个不可解析的 date string 认为是相等的，然后可解析和不可解析的字符串的比较约定好大小关系，并确保符合相反性 contract。
