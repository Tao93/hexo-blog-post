---
title: Android 绘制圆形图片
tags: [Android]
---

记录一下，以免忘记，以备使用。

方法一：

```java
public class MyView extends View {

    Paint paint = new Paint();
    Shader shader;
    Matrix mat = new Matrix();
    Bitmap bitmap;

    public MyView(Context context) {
        this(context, null);
    }

    public MyView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        // create a bitmap from the image resource what we want to draw
        bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.dossier_icon);
        // a bitmap shader, would be set into paint, here we use repeat tile mode, but since we'll
        // scale to make the bitmap fill the canvas just right, so there would be no repeat.
        shader = new BitmapShader(bitmap, Shader.TileMode.REPEAT, Shader.TileMode.REPEAT);
        // after this, the paint would contains this bitmap.
        paint.setShader(shader);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        // scale to make bitmap fill the canvas
        mat.setScale((float)getWidth() / bitmap.getWidth(), (float)getHeight() / bitmap.getHeight());
        shader.setLocalMatrix(mat);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // draw a circle with the paint.
        canvas.drawCircle(getWidth() / 2, getHeight() / 2, getWidth() / 2, paint);
    }
}
```

方法二：

```java
public class MyView2 extends View {

    private Path mPath;
    private Drawable mDrawable;

    public MyView2(Context context) {
        this(context, null);
    }

    public MyView2(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyView2(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mPath = new Path();
        mDrawable = getResources().getDrawable(R.drawable.dossier_icon);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        // make the path to be a circle whose diameter is width of the view.
        mPath.reset();
        mPath.addCircle(getWidth() / 2, getHeight() / 2, getWidth() / 2, Path.Direction.CW);
        // drawable must setBounds before draw
        mDrawable.setBounds(0, 0, getWidth(), getHeight());
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // save current (default) clip
        canvas.save();
        // make the clip to be the circle 
        canvas.clipPath(mPath);
        // draw the bitmapDrawable
        mDrawable.draw(canvas);
        // restore default clip
        canvas.restore();
    }
}
```

方法三：

```java
public class MyView3 extends View {
    Paint mPaint = new Paint();
    Paint whitePaint = new Paint();
    private Bitmap bitmap;
    private Matrix mat = new Matrix();
    private Xfermode mXfermode;

    public MyView3(Context context) {
        this(context, null);
    }

    public MyView3(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyView3(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mXfermode = new PorterDuffXfermode(PorterDuff.Mode.SRC_IN);
        whitePaint.setColor(Color.RED);
        bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.dossier_icon);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        mat.setScale((float)getWidth() / bitmap.getWidth(), (float)getHeight() / bitmap.getHeight());
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // save current layer and create a new layer
        int count = canvas.saveLayer(0, 0, getWidth(), getHeight(), mPaint);
        // draw a white circle in the new layer
        canvas.drawCircle(getWidth() / 2, getHeight() / 2, getWidth() / 2, whitePaint);
        // this xfer mode is src_in type, which could make image to be drawn in only previous white circle area.
        mPaint.setXfermode(mXfermode);
        // draw bitmap in new layer
        canvas.drawBitmap(bitmap, mat, mPaint);
        // this is necessary, or the influence would last
        mPaint.setXfermode(null);
        // back to previous layer.
        canvas.restoreToCount(count);
    }
}
```

三个方法中，方法二最简便易用，方法三最不推荐，因为新增一个 canvas layer 是开销很大的，这一点 saveLayerAlpha 方法的注释有说明。

