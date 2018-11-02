---
title: 不一样的 RelativeLayout measure 过程
tags: [Android]
---

直接用一个简单的例子展示 RelativeLayout 的不一样之处：

![](http://tao93.top/images/2018/11/01/1541079447.png)

图中，左边是一个简单的 Android layout 例子，例子中在外侧是一个最大的 horizontal 的 LinearLayout，它有 3 个 child，分别是左部、分割线和右部。左部和右部非常相似，都是外面一个 ViewGroup 内嵌一个 TextView，且 ViewGroup 的高度都是 wrapContent 而内嵌的 TextView 的高度都是 matchParent.

因为 TextView 背景都是红色，所以从 preview 可以清晰看到，左部的 TextView 等效于 wrapContent，而右部的 TextView 则等效于 matchParent，这就是 RelativeLayout 的不一样。简而言之就是，RelativeLayout 中内嵌的 child tree 的根节点 size 如果是 matchParent，那么此 child tree 测量得到的 size 将会是最大化的，可达到 RelativeLayout size 的上限，而其余常见 ViewGroup 没有此特点，比如 LinearLayout, FrameLayout, ConstraintLayout 等。

究其原因，可以从 RelativeLayout 中的 onMeasure 方法找。

```java
	@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        for (int i = 0; i < count; i++) {
            final View child = views[i];
            if (child.getVisibility() != GONE) {
                final LayoutParams params = (LayoutParams) child.getLayoutParams();

                applyVerticalSizeRules(params, myHeight, child.getBaseline());
                measureChild(child, params, myWidth, myHeight);
                ...
    }
```

上面是 RelativeLayout#onMeasure 的片段，可知对所有 child 调用了 measureChild 方法，而这是个 RelativeLayout 的 private 方法：

```java
/**
     * Measure a child. The child should have left, top, right and bottom information
     * stored in its LayoutParams. If any of these values is VALUE_NOT_SET it means
     * that the view can extend up to the corresponding edge.
     *
     * @param child Child to measure
     * @param params LayoutParams associated with child
     * @param myWidth Width of the the RelativeLayout
     * @param myHeight Height of the RelativeLayout
     */
    private void measureChild(View child, LayoutParams params, int myWidth, int myHeight) {
        int childWidthMeasureSpec = getChildMeasureSpec(params.mLeft,
                params.mRight, params.width,
                params.leftMargin, params.rightMargin,
                mPaddingLeft, mPaddingRight,
                myWidth);
        int childHeightMeasureSpec = getChildMeasureSpec(params.mTop,
                params.mBottom, params.height,
                params.topMargin, params.bottomMargin,
                mPaddingTop, mPaddingBottom,
                myHeight);
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

从上述方法看，先获取到 child measureSpec，然后依次为参数来 measure 每个 child。再进入到 RelativeLayout#getChildMeasureSpec 这个方法中看看下面的逻辑：

```java
if (childStart != VALUE_NOT_SET && childEnd != VALUE_NOT_SET) {
    // Constraints fixed both edges, so child must be an exact size.
    childSpecMode = isUnspecified ? MeasureSpec.UNSPECIFIED : MeasureSpec.EXACTLY;
    childSpecSize = Math.max(0, maxAvailable);
} else {
    if (childSize >= 0) {
        // Child wanted an exact size. Give as much as possible.
        childSpecMode = MeasureSpec.EXACTLY;

        if (maxAvailable >= 0) {
            // We have a maximum size in this dimension.
            childSpecSize = Math.min(maxAvailable, childSize);
        } else {
            // We can grow in this dimension.
            childSpecSize = childSize;
        }
    } else if (childSize == LayoutParams.MATCH_PARENT) {
        // Child wanted to be as big as possible. Give all available
        // space.
        childSpecMode = isUnspecified ? MeasureSpec.UNSPECIFIED : MeasureSpec.EXACTLY;
        childSpecSize = Math.max(0, maxAvailable);
    } else if (childSize == LayoutParams.WRAP_CONTENT) {
        // Child wants to wrap content. Use AT_MOST to communicate
        // available space if we know our max size.
        if (maxAvailable >= 0) {
            // We have a maximum size in this dimension.
            childSpecMode = MeasureSpec.AT_MOST;
            childSpecSize = maxAvailable;
        } else {
            // We can grow in this dimension. Child can be as big as it
            // wants.
            childSpecMode = MeasureSpec.UNSPECIFIED;
            childSpecSize = 0;
        }
    }
}
```

从上面可以看到，chileSpecMode 赋值为 MeasureSpec.AT_MOST 的地方只有 27 行一个地方，也就是只要 child 声明为 match_parent，那么 child 的 specMode 不出意外就会是 AT_MOST。而 AT_MOST 意味着此 child 为根节点的 view tree 的测量结果将会是此 child 的 specSize，而这个 specSize 显然就是 RelativeLayout 能达到的最大 size (可能需要减去 padding，RelativeLayout 其它 child 占据的空间等)。

顺便再看看 LinearLayout 等等为啥不是这样的，事实上 LinearLayout 等使用的是 ViewGroup 这个抽象基类中的 getChildMeasureSpec 方法：

```java
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec);
    int specSize = MeasureSpec.getSize(spec);
    int size = Math.max(0, specSize - padding);
    int resultSize = 0;
    int resultMode = 0;
    switch (specMode) {
    // Parent has imposed an exact size on us
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            resultSize = size;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;
    ...
    //noinspection ResourceType
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

从上述代码可以看到，如果 child 声明为 matchParent 但是 RelativeLayout 的 specMode 是 AT_MOST 的话，那么 child 的 specMode 如第 35 行所示将会是 AT_MOST，这就是和 RelativeLayout 的不同之处。

总结一下，即使 RelativeLayout 的 specMode 是 AT_MOST，只要它的 child 声明 matchParent，那么 child 的 SpecMode 一般会是 EXACTLY，此 child 为根节点的 viewTree 的测量结果就是最大化的。
