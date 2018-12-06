---
title: 矩阵运算在 Android 中的简单场景
tags: [Android]
mathjax: true
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

$$ T(a, b),\ R(\theta),\ S(u, v),\ SKx(\theta), SKy(\theta) $$

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
   1 & tan(\theta) & 0 \\
   0 & 1 & 0 \\
   0 & 0 & 1
\end{bmatrix}
,\ 
\begin{bmatrix}
   1 & 0 & 0 \\
   tan(\theta) & 1 & 0 \\
   0 & 0 & 1
\end{bmatrix}
$$

分别表示的含义是：

1. 向 +x 方向平移 a 且向 +y 方向平移 b；
2. 以原点为中心在 xy 平面旋转 theta 角度；
3. 以原点为 pivot，x 方向和 y 方向分别伸缩值为 u 和 v 的比例；
4. 每个点的 $x$ 坐标变为 $x + y·tan(\theta)$，而 $y$ 坐标不变，视觉表现为图形向 +y 方向倾斜，倾斜角度为 $\theta$;
5. 每个点的 $y$ 坐标变为 $x·tan(\theta) + y$，而 $x$ 坐标不变，视觉表现为图形向 +x 方向倾斜，倾斜角度为 $\theta$.

注意, $T(a, 0)$ 和 $T(0, b)$ 组合起来的变换，等价于 $T(a, b)$，且和组合顺序无关，这从矩阵乘法也可以看出来：

$$
\begin{bmatrix}
1 & 0 & a \\
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}·
\begin{bmatrix}
1 & 0 & 0 \\
0 & 1 & b \\
0 & 0 & 1
\end{bmatrix}=
\begin{bmatrix}
1 & 0 & 0 \\
0 & 1 & b \\
0 & 0 & 1
\end{bmatrix}·
\begin{bmatrix}
1 & 0 & a \\
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}=
\begin{bmatrix}
1 & 0 & a \\
0 & 1 & b \\
0 & 0 & 1
\end{bmatrix}
$$

同样的，$S(u, 0)$ 和 $S(0, v)$ 组合起来等价于 $S(u, v)$ 且和组合顺序无关。而 Skew 无此性质。事实上：

$$
\begin{bmatrix}
1 & tan(\alpha) & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}·
\begin{bmatrix}
1 & 0 & 0 \\
tan(\beta) & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}=
\begin{bmatrix}
1+tan(\alpha)tan(\beta) & tan(\alpha) & 0 \\
tan(\beta) & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

而：

$$
\begin{bmatrix}
1 & 0 & 0 \\
tan(\beta) & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}·
\begin{bmatrix}
1 & tan(\alpha) & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}=
\begin{bmatrix}
1 & tan(\alpha) & 0 \\
tan(\beta) & 1+tan(\alpha)tan(\beta) & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

接下来说说 Android 中和 3×3 矩阵相关的一些类，也就是 [Matrix](https://developer.android.com/reference/android/graphics/Matrix), [Canvas](https://developer.android.com/reference/android/graphics/Canvas), [Camera](https://developer.android.com/reference/android/graphics/Camera) 这 3 个类，注意最后这个  Camera 类是 android.graphics 包中的，而不是 android.hardware 中的。

Matrix 对象表示一个 3×3 矩阵，常用于指示 x y 平面内的变换，构造函数创建的都是单位矩阵。Matrix 有一系列 set 方法，用于设置矩阵的值，这些方法对应于一些常见的变换操作，比如 setScale(int sx, int sy) 方法，其实就是把矩阵(不管原来是什么值)变成下面的值：

$$
\begin{bmatrix}
sx & 0 & 0 \\
0 & sy & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

上面的缩放是以原点为 pivot 缩放的，此外，还有 setScale(float sx, float sy, float px, float py) 这样以 (px, py) 为 pivot 进行缩放的操作，等价于把矩阵变为下面这样：

$$
\begin{bmatrix}
sx & 0 & px(1-sx) \\
0 & sy & py(1-sy) \\
0 & 0 & 1
\end{bmatrix}
$$

注意以上矩阵是可以分解的，分解为两个矩阵，且这两个矩阵交换位置后的积也是上面的矩阵。其他的 set 方法与此类似，就不多说了。

Matrix 还有一系列 post 方法和 pre 方法，比如 postRotate(int degrees) 和 preSkew(int kx, int ky) 方法，前者相当于 $M=R(degrees)·M$，后者相当于 $M=M·SK(kx,\ ky)$. 注意这里矩阵相乘的顺序取决于是 post 还是 pre.

其实 Matrix 也可以在 3 维空间中变换，比如使用它的 setPolyToPoly 方法，可以得到满足指定的映射关系的矩阵。而这种映射关系可能需要 3 为空间的变换才能满足。

Canvas 类包含一个变换矩阵，这个矩阵可以 get 也可以 set，也可以用 concact(Matrix mat) 来改变 Canvas 的变换矩阵的值，这个过程等价于 $M=M·mat$. 另外，Canvas 的 drawBitmap 方法也可以传入 Matrix 对象来控制 bitmap 被 draw 时需要的施加的变换。Canvas 的 translate 和 rotate 等方法，都是 pre 类型的，并且这种变化，可以理解为是对 canvas 的坐标系进行变化。无变换时，坐标系原点在可见区域的左上角，而 +x 向右，+y 向下。translate(10, 0) 后，canvas 原点向右移动 10，此后绘制在 (0, 0) 位置的东西，其实是位于可见区域的左上角右侧距离为 10 的地方。

以上说的可见区域，其实是 Canvas 中的 clip 概念，这是一个描述可见区域的矩形。一般 Canvas 的初始 clip 的 ltrb f分别为 0, 0, view width, view height. 当 Canvas 变换时，可理解为其坐标系变换了，那么自然 clip 也会跟着变化，比如 translate(10, 0) 后，clip 的 ltrb 就变为 -10, 0, view width - 10, view height.

Canvas 出于灵活性考虑，有 save 和 restore 两个方法用于保存和恢复变换的状态，常用做法是，在无变换时 save 一下，然后变换并做绘制，最后restore，以避免影响其它绘制工作。出了translate、rotate、Skew、Scale 这些 2 维变换，Canvas 还有一系列 clip 方法，这些方法将 Canvas 进一步剪裁，并根据剪裁后可见区域是否为空返回布尔值。剪裁操作也是可以 save 和 restore 的，但是注意剪裁操作并不变换坐标系。

Camera 对象专门用于计算变换，且包含 3 维空间中的变换(相反 Matrix 对象其实只是 xy 平面内的变换)，它的变换计算结果是一个矩阵，可通过 getMatrix 方法得到，一般会将此矩阵用作它用，可以将 Camera 理解为一个变换计算器。出于灵活性的考虑，Camera 对象中有 save 和 restore 操作，这就像是 git 中的 stash 和 pop stash 操作一样，可以把当前的变换保存，并在后面某个时候恢复出来，恢复之前的值将会丢失。不同的是，git stash 后 working tree 将是 clean 的，即已修改的内容都清空了，而 Camera 对象 save 后，它的矩阵还是 save 之前的，并不会变成单位矩阵。

需要注意的是，Camera 的坐标系，+x 是向右的，+y 是向上的，而 +z 是向屏幕里面的。所以比如 Camera 的 rotateZ(int degrees) 方法和 Matrix 的 [pre|post]Rotate(int degree) 的旋转方向是反过来的。前者等价于逆时针方向转，后者则是顺时针方向转。下面的例子可以说明这一点：

```java
Matrix mat1 = new Matrix();
Camera camera = new Camera();

camera.rotateZ(-90);
camera.getMatrix(mat1);

Matrix mat2 = new Matrix();
mat2.preRotate(90);
```

上面代码执行后 mat1 和 mat2 都是：

$$
\begin{bmatrix}
0 & -1 & 0 \\
1 & 0 & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

对于 Camera 对象， 调用它的 translate 等方法，会使得它的 matrix 变为 Camera 的坐标系中对应此变换的值，而这样的矩阵，在应用在 Canvas 上进行绘制时，又是作用在 +y 向下的坐标系中。所以比如 Camera translate(10, -20, 0) 后得到的矩阵是：

$$
\begin{bmatrix}
1 & 0 & 10 \\
0 & 1 & 20 \\
0 & 0 & 1
\end{bmatrix}
$$

需要注意的是，当 Camera 在 z 方向进行 translate 操作时，变换对象在屏幕上的投影大小发生变化，所以，实际变换效果是一个缩放变换，所以 Camera getMatrix 的结果，也是一个二维缩放变换的矩阵。Camera 的 location 也就是 3D 投影中的 camera 位置（观察者位置），此位置默认是 (0, 0, -8)，注意这个 -8 实际相当于 -576 个像素。当 z 方向 translate 距离为正时，相当于远离观察者，最终就是缩小的变换，反正是放大的变换。