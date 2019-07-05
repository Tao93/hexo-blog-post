---
title: Android 自定义简易 PickerView
tags: [Android]
---

最近的做的一项 feature 中，需要使用类似于 iOS 的 UIPickerView 的控件，功能是通过滚动 item 来选择其中一个，也就是下图这样的：

![](https://tao93.top/images/2019/04/07/1554640301.png)

Android SDK 倒是有一个 NumberPicker 和这个非常类似，不过 NumberPicker 设计有点问题，滚动很慢，即使是手指快速滑动，也只能滚动五六个 items，而 Github 上确实有不少优秀的实现，可以和 iOS 的 UIPickerView 非常相似，不过其代码有些复杂，以至于想要速读代码然后自己写一份也比较费时间。所以，我就想到自己基于 Android 的 ListView 写一个简单的自定义 PickerView。

既然使用 ListView，那么首先需要解决问题有如下几个：

1. 如何让所有 item 都能通过滑动而滚动到视图中间，从而表示该 item 被选中？
2. 如果让 ListView 的 idle 状态变为离散的，而非本来的任意滚动状态都可以为 idle？
3. 怎么获取当前选中的 item 的序号？

第一个问题比较好办，在 list 的首尾各填充一些空白的 item，这样就能让用户可见的所有 item 都能滑动到正中间。第二个问题，我的做法是，如果手指离开时，ListView 缓慢滑动，那么就在速度低于某个阈值时，让它滑动到恰好将一些 item 显示出来而不要有某个 item 只显示一部分；而如果手指离开时 ListView 快速滑动，那么就在 ListView 刚刚变为 idle 状态时，让它就近滑动到「恰好将一些 item 显示出来」的状态。第三个问题好办，计算一下当前显示的所有 item 的 position 就能知道最中间显示的 item 的 position。

另外还有一些细节问题：

1. 需要在中间绘制上两条灰色水平线；
2. 需要在 canvas 的最上层绘制不是正中间区域绘制半透明矩形，从而让非选中的 item 看起来是灰色的；
3. 点击非选中的 item 时，需要滚动 ListView 让该 item 滑动到正中间。

实现的代码如下：

```java

/**
 * The Adapter interface for {@link PickerView} which is like UIPickerView on iOS. 
 */
public interface PickerAdapter {
    /** get resource id of layout for each item of the picker view. */
    @LayoutRes int getItemLayoutResource();
    
    /** count of items in the picker view. */
    int getItemCount();

    /** text string for the item at the specified position. */
    String getItemAt(int pos);
}

/**
 * A view to let users to choose one item from a list, like UIPickerView on iOS.
 */
public class PickerView extends FrameLayout {
    /** the listView which is used to implement this PickerView */
    private ListView listView;
    /** the count of visible items, must be odd numbers, the middle item would be the the selected one. */
    private int visibleCount = 5; // 5 items visible by default
    private float itemHeight;

    Paint dividerLinePaint;
    Paint greyLayerPaint;
    /** adapter for this pickerView */
    PickerAdapter adapter;
    
    private float dividerThickness;
    private float dividerMarginLeft;
    private float dividerMarginRight;
    
    private float velocityThreshold;

    private VelocityTracker velocityTracker = VelocityTracker.obtain();

    public PickerView(@NonNull Context context) {
        this(context, null);
    }

    public PickerView(@NonNull Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public PickerView(@NonNull Context context, @Nullable AttributeSet attrs, final int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        setWillNotDraw(false);
        
        itemHeight = getResources().getDimension(R.dimen.picker_view_item_height);
        velocityThreshold = getResources().getDisplayMetrics().density * 260;
        
        TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.PickerView);
        dividerThickness = ta.getDimension(R.styleable.PickerView_divider_thickness, getResources().getDisplayMetrics().density);
        dividerMarginLeft = ta.getDimension(R.styleable.PickerView_divider_margin_left, 0);
        dividerMarginRight = ta.getDimension(R.styleable.PickerView_divider_margin_right, 0);
        int dividerColor = ta.getColor(R.styleable.PickerView_divider_color, Color.LTGRAY);
        ta.recycle();

        dividerLinePaint = new Paint();
        dividerLinePaint.setColor(dividerColor);
        dividerLinePaint.setStyle(Paint.Style.STROKE);
        dividerLinePaint.setStrokeWidth(dividerThickness);
        greyLayerPaint = new Paint();
        greyLayerPaint.setColor(0xb0ffffff);
    }

    public void setAdapter(PickerAdapter adapter) {
        setAdapter(adapter, 0, 0);
    }

    public void setAdapter(PickerAdapter adapter, int initPos) {
        setAdapter(adapter, 0, initPos);
    }

    /**
     * set adapter and other two parameters.
     * @param visibleCount count of visible items in the picker, must be odd numbers. 
     * @param initPos position of the initially selected item, 0 means the first item would be selected.
     */
    public void setAdapter(PickerAdapter adapter, int visibleCount, int initPos) {
        if (adapter.getItemCount() <= 0) {
            return;
        }
        if (visibleCount > 0 && visibleCount % 2 != 0) {
            this.visibleCount = visibleCount;
        }

        this.adapter = adapter;

        listView = new ListView(getContext());
        addView(listView, new LinearLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, (int) (itemHeight * this.visibleCount)));
        listView.setOverScrollMode(View.OVER_SCROLL_NEVER);  // remove over scroll effects.
        listView.setVerticalScrollBarEnabled(false);
        listView.setDivider(null);  // remove listView's dividers

        listView.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {
                if (scrollState == AbsListView.OnScrollListener.SCROLL_STATE_IDLE) {
                    // scroll to nearest place that fit the selection properly.
                    scrollToNearest();
                }
            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) { }
        });
        
        listView.setAdapter(new InnerAdapter());
        if (initPos > 0 && initPos < adapter.getItemCount()) {
            listView.setSelection(initPos);
        }
    }

    /**
     * @return the position of the currently selected item, starting from 0.
     */
    public int getSelectedPosition() {
        listView.getFirstVisiblePosition();
        View firstChild = listView.getChildAt(0);
        if (Math.abs(firstChild.getTop()) < firstChild.getHeight() / 10) {
            return listView.getFirstVisiblePosition();
        } else {
            return listView.getFirstVisiblePosition() + 1;
        }
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean superResult = super.dispatchTouchEvent(ev);
        
        velocityTracker.addMovement(ev);
        if (ev.getActionMasked() == MotionEvent.ACTION_UP) {
            velocityTracker.computeCurrentVelocity(1000);
            float yVelocity = velocityTracker.getYVelocity();
            // scroll to nearest proper place directly if the fling is very slow
            if (Math.abs(yVelocity) < velocityThreshold) {
                scrollToNearest();
            }
        }
        return superResult;
    }
    
    /** scroll the listView to fit the nearest proper position */
    private void scrollToNearest() {
        View firstChild = listView.getChildAt(0);
        if (Math.abs(firstChild.getTop()) > Math.abs(firstChild.getBottom())) {
            listView.smoothScrollToPosition(listView.getLastVisiblePosition());
        } else {
            listView.smoothScrollToPosition(listView.getFirstVisiblePosition());
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        velocityTracker.recycle();
        super.onDetachedFromWindow();
    }

    @Override
    public void draw(Canvas canvas) {
        super.draw(canvas);
        //canvas.drawRect(0, itemHeight * 2, getWidth(), itemHeight * 3, dividerLinePaint);
        canvas.drawLine(dividerMarginLeft, itemHeight * 2 - dividerThickness, 
                getWidth() - dividerMarginRight, itemHeight * 2 - dividerThickness, dividerLinePaint);
        canvas.drawLine(dividerMarginLeft, itemHeight * 3 + dividerThickness, 
                getWidth() - dividerMarginRight, itemHeight * 3 + dividerThickness, dividerLinePaint);

        // draw two transparent rectangles to make not selected items looks grey
        canvas.drawRect(0, 0, getWidth(), itemHeight * 2 - dividerThickness, greyLayerPaint);
        canvas.drawRect(0, getHeight() - 2 * itemHeight + dividerThickness, getWidth(), getBottom(), greyLayerPaint);
    }

    /**
     * the adapter for the listView
     */
    private class InnerAdapter extends BaseAdapter {

        @Override
        public int getCount() {
            // we would add visibleCount - 1 empty strings to the list View, so the count should plus visibleCount - 1.
            return adapter.getItemCount() + visibleCount - 1;
        }

        @Override
        public Object getItem(int position) {
            // there're visibleCount - 1 items with empty text, they would be equally put at the starting and ending.
            if (position < visibleCount / 2 || position >= getCount() - visibleCount / 2) {
                return "";
            }
            return adapter.getItemAt(position - visibleCount / 2);
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(final int position, View convertView, ViewGroup parent) {
            if (convertView == null) {
                convertView = View.inflate(getContext(), adapter.getItemLayoutResource(), null);
                convertView.setBackgroundColor(Color.WHITE); // to avoid the default touch effect of listView item
            }
            // select a item when it's clicked by users
            convertView.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    int cur = getSelectedPosition() + visibleCount / 2;
                    if (position > cur) {
                        listView.smoothScrollToPosition(position + visibleCount / 2);
                    } else if (position < cur) {
                        listView.smoothScrollToPosition(position - visibleCount / 2);
                    }
                }
            });
            TextView textView = convertView.findViewById(R.id.picker_text_view);
            textView.setText((String) getItem(position));
            return convertView;
        }
    }
}

```

还有一些小细节。比如 ListView 的高度需要正好是 itemView 的奇数倍，这样才能恰好只显示一些 item，并且最中间显示其中一个 item。再比如为了避免 ListView 自带的点击 item 时的视觉效果，需要给 itemView 设置纯白色背景。

至此，一个简易的 PickerView 就完成了。