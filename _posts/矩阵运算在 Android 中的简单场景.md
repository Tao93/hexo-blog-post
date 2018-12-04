---
title: 矩阵运算在 Android 中的简单场景
tags: [Android]
---

让我们先从位于 android.graphics 包中的 Bitmap 类的一个方法开始说起，也就是 createBitmap(Bitmap source, int x, int y, int width, int height, Matrix m, boolean filter) 这样一个方法。这个方法中有个矩阵参数，通过传入此矩阵参数，可以将 source Bitmap 经过一定的转换再创建目标 Bitmap。那么这个矩阵是怎么起作用的呢？先看一段示例代码：

```java
Matrix matrix = new Matrix();
matrix.postRotate(90);
Bitmap bitmap2 = Bitmap.createBitmap(bitmap1, 0, 0, bitmap1.width(), bitmap1.getHeight(), matrix, true);
```

上述代码做的事情是将 bitmap1 顺时针旋转 90° 得到 bitmap2，起关键作用的就是这个 matrix。第一行代码会得到一个 3×3 的单位矩阵，而第二行代码之后，矩阵将会变成下面这样：

$$
  \begin{bmatrix}
   0 & -1 & 0 \\
   1 & 0 & 0 \\
   0 & 0 & 1
  \end{bmatrix} ·
  \begin{bmatrix}
   1 & 0 & 0 \\
   0 & 1 & 0 \\
   0 & 0 & 1
  \end{bmatrix} =
  \begin{bmatrix}
  0 & -1 & 0 \\
  1 & 0 & 0 \\
  0 & 0 & 1
  \end{bmatrix}
$$

注意在 xy 平面内顺时针 rotate 90° 的操作对应的矩阵是 

$$
\begin{bmatrix}
cos 90° & -sin 90° & 0 \\
sin 90° & cos90° & 0 \\
0 & 0 & 1
\end{bmatrix} = 
\begin{bmatrix}
   0 & -1 & 0 \\
   1 & 0 & 0 \\
   0 & 0 & 1
  \end{bmatrix}
$$

记上面的矩阵为 R90，记上面代码中的 matrix 为 M，记单位矩阵为 I，那么第 2 行的操作则是等同于下面的表达式：

$$ M = R90·M $$

也就是:

$$ M = R90·I $$

注意上面的点乘顺序是 R90 在前，I 在后（虽然这里 R90·I 和 I·R90 结果是一样，但别的场景未必如此）。原因是 代码中是 `postRotate` 方法，这里 `post` 表示是把 rotate 放在变换的最后一步。而变换矩阵作用 R90·M 作用在目标向量 V 时，将会是表达式：

$$ R90·M·V $$

这样就可以理解为是先 M·V 然后这个结果再被 R90 乘，所以是先做 M 本来的变换，然后再做 R90 的变换，也就是 rotate 90° 在最后一步。这也就是第 2 行代码的 postRotate 90° 等价于 M = R90·M 也就是 M = R90·I 的原因。

所以第 3 行代码传入的 matrix 就是：

$$\begin{bmatrix}
   0 & -1 & 0 \\
   1 & 0 & 0 \\
   0 & 0 & 1
  \end{bmatrix}
$$

现在我们进入前面说的 createBitmap 这个方法中去看看源码是怎么实现这个变换的：

```java
public static Bitmap createBitmap(Bitmap source, int x, int y, int width, int height, Matrix m, boolean filter) {
    ...
    Rect srcR = new Rect(x, y, x + width, y + height);
    RectF dstR = new RectF(0, 0, width, height);
    RectF deviceR = new RectF();
    m.mapRect(deviceR, dstR);
    neww = Math.round(deviceR.width());
    newh = Math.round(deviceR.height());
    bitmap = createBitmap(neww, newh, transformedConfig, transformed || source.hasAlpha());
    
    Canvas canvas = new Canvas(bitmap);
    canvas.translate(-deviceR.left, -deviceR.top);
    canvas.concat(m);
    canvas.drawBitmap(source, srcR, dstR, paint);
    ...
    return bitmap;
}
```

上面代码第 3 行 srcR 将是 source Bitmap 中要被转换的部分。而 dstR 是同样宽高但是左上角在 (0, 0) 的矩形。第 6 行则是将 dstR 变换后得到 deviceR，根据前面所知 的 m 的值，可知deviceR 将会是 left, top, right, bottom 分别是 -height, 0, 0, width:

![](http://tao93.top/images/2018/12/04/1543937277.png)

即 deviceR 是 dstR 绕原点顺时针旋转 90° 得到的。紧接着代码 7、8 行得到转换后的 Bitmap 的宽高。然后第 9 行以此宽高创建了新的 Bitmap。第 11 行基于此 Bitmap 创建了 canvas。

重点来了，第 12 行对此 canvas 进行平移变换，平移的目的是让 deviceR 的左上角移动到原点。从而让它位于 x 非负且 y 非负的象限。而第 13 行则将 m 矩阵的变换作用在此 canvas 是。需要注意的是，第 12、13 行的变换，都是 pre 的变换而不是 post 的。所以可以看成 13 行的变换其实是比第 12 行的平移先执行的，也就是整个变换过程可理解为先绕原点顺时针旋转 90°，然后再将左上角平移到原点，这样就成功的完成了将原 Bitmap 旋转 90° 并创建新 Bitmap 的操作，虽然这里除了 rotate，其实还利用了 translation 操作。

用 3×3 矩阵在 xy 平面内变换，可以分为 translate, rotate, scale, skew 一共 4 种，表示这四种操作的矩阵分别记作：

$$ T(a, b),\ R(\theta),\ S(u, v),\ SK(p, q) $$

那么它们分别是：

$$
\begin{bmatrix}
   1 & 0 & a \\
   0 & 1 & b \\
   0 & 0 & 1
\end{bmatrix}
,\ 
\begin{bmatrix}
   cos(\theta) & -sin(\theta) & 0 \\
   sin(\theta) & cos(\theta) & 0 \\
   0 & 0 & 1
\end{bmatrix}
,\ 
\begin{bmatrix}
   u & 0 & 0 \\
   0 & v & 0 \\
   0 & 0 & 1
\end{bmatrix}
,\ 
\begin{bmatrix}
   1 & tan(p) & 0 \\
   tan(q) & 1 & 0 \\
   0 & 0 & 1
\end{bmatrix}
$$

分别表示的含义是：

1. 向 +x 方向平移 a 且向 +y 方向平移 b；
2. 以原点为中心在 xy 平面旋转 theta 角度；
3. 以原点为 pivot，x 方向和 y 方向分别伸缩值为 u 和 v 的比例；
4. 