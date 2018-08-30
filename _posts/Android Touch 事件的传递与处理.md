一、MotionEvent

MotionEvent 是 Android 系统中报告动作输入（如触摸屏幕，滚动球，鼠标）等事件的类。其中有整形数的 Action 和描述事件的各种浮点型坐标。MotionEvent 的 action 的最低一个字节，其取值我们常见的 ACTION_DOWN, MOVE, POINTER_DOWN, POINTER_UP, UP, CANCEL (后面可能简称为 DOWN 等等) 这些了。虽然我们可能会粗暴的使用下面这样的方式：

```java
switch (event.getAction()) {
    case MotionEvent.ActionDown:
        ...
        break;
    ... 
}
```

但是其实上面这样仅适用于单点触控场景，原因是 getAction() 方法返回的结果中，最低的1个字节才对应上面说的 ACTION_DOWN，而次最低一个字节则表示 PointerIndex (这个概念后面会提到)，对于单点触控场景，PointerIndex 始终为0，所以上面的代码才能跑通。更好的方式，其实是使用 getActionMasked() 方法，此处查看源码即可非常清晰。

对于一串触摸事件（指手接触到屏幕到手完全离开屏幕），用户第一个手指接触到屏幕时，触发 DOWN, 手指在屏幕上移动时，持续触发出很多 MOVE，第二个以上的手指触摸到屏幕是，触发 POINTER_DOWN，非最后一个离开屏幕的手指离开屏幕时触发 POINTER_UP, 最后一个手指离开屏幕时，触发 UP。

在一串触摸事件过程中，视图从 window detach 的话，或者 View 的 PFLAG_CANCEL_NEXT_UP_EVENT 标记位设置了时，或者 View 本来处理了一个 Down 事件但是接下来的 MOVE 等事件被 ViewGroup 拦截了，将会触发 CANCEL。当然，应该还有别的触发 CANCEL 事件的场景。下图是一串 Touch 事件的例子：

	Down → Move →...→ Move → PointerDown → Move →...→ PointerUp → Move →...→ Move → Up

MotionEvent 中有个 Pointer 的概念，Pointer 大致表示触摸到屏幕的各个手指。getPointerCount() 即为多点触控的点数。每个 pointer 都有一个 index (取值是0到getPointerCount()-1)和 id（最大是31，意味着一串触摸事件中，最多允许手指接触屏幕32次），前者在一串 Touch 事件中可能会变化，而后者保持不变，例如多点触控时，大拇指对应的 index 本来是0，后面可能变成了1了. getPointerId(int pointerIndex)和findPointerIndex(int pointerId)这两个方法是 index 和 id 间的一一映射。 那为什么要有 index 和 id 两个东西呢？

index 和 id 的区别。举个例子就清楚了。假设起始时，右手的食指，中指，无名指依次先后落在屏幕上，那么三个指头的 index 和 id 分别都是 0, 1, 2。此时抬起中指，那么食指的 index 和 id 不变，而无名指的 index 变成1，id 依然是2。即 index 总是 0, 1, 2,…, getPointerCount() - 1. 而 id 则可能大于 getPointerCount(), 另一面则是，pointer 的 id 在整个一串触摸事件中保持不变。

MotionEvent 的 getX() 与 getX(int pointerIndex), 前者等价于后者的 getX(0), getY 类似. getPointerIdBits() 返回一个类似于标志数的东西，其二进制从右向左数，第 i 位为1表示存在 id 是 i 的 pointer，此处应看源码。可以想见此结果的二进制形式中1的个数即为多点触控的点数。

![pointer_id_bits](http://tao93.top/wp-content/uploads/2018/08/pointer_id_bits.png)

二、Touch 事件的生成

Touch 事件的根源是从硬件而来，而 ViewRootImpl$ViewPostImeInputStage#processPointerEvent 方法是一个比较合适的开始追踪 Touch 事件的起点，此处应看源码。事件如果没有被消费，最终可能将在 ViewRootImpl #finishInputEvent 中回收掉(这不是本文的重点)。Touch 事件的传递堆栈 (按调用的顺序排列)：

```
ViewRootImpl$ViewPostImeInputStage#processPointerEvent
View#dispatchPointerEvent
DecorView#dispatchTouchEvent
Activity#dispatchTouchEvent
PhoneWindow#superDispatchTouchEvent
DecorView#superDispatchTouchEvent
ViewGroup#dispatchTouchEvent
```

Touch 事件传递到了 ViewGroup 的 dispatchTouchEvent 方法后，就开始是本文要关注的焦点了，即一串 Touch 事件在 View 树上是如何传递和消费的。

三、传递和处理 Touch 事件

下面是事件传递和处理中最重要的 4 个方法：

```
View#dispatchTouchEvent，非 ViewGroup 的 View 接受 Touch 事件的方法，我们一般不重载它。

View#onTouchEvent，View(含 ViewGroup)自己尝试去消费 Touch 事件，经常会被重载。

ViewGroup#dispatchTouchEvent，重载了 View 的同名方法，作为 ViewGroup 接受 Touch 事件，此方法中包含「传递 Touch 事件给 child」和「尝试自己消费 Touch 事件」这样两个分支逻辑，一般不重载它。

ViewGroup#onInterceptTouchEvent，ViewGroup 判断自己要不要拦截 Touch 事件的方法，拦截意味着自己尝试消费此事件，有时候被重载。
```

**传递:**

在 View 树中，Touch 事件是由 parent view 传递给 child View (child view 也可能是 ViewGroup)的。在一串 touch 事件中打头的总是 DOWN 事件，parent view 通过调用 child view 的 dispatchTouchEvent方法把 DOWN 事件传递给 child view，这就给了 child 一个消费此事件的机会。child 的 dispatchTouchEvent 方法返回 true 的话，就表示 child 消费了 DOWN 事件，返回 false 就表示不消费。如果 child 消费了 Down 事件，那么最简单的情况就是后续的 move 等事件都会通过调用 child 的 dispatchTouchEvent 方法直接交给此 child，不管它的 dispatchTouchEvent 方法返回什么；如果 child 没有消费 DOWN 事件，那么单点触控情形下，后续的 move 等事件与此 child 再也无缘了。

**消费：**

View(包括 ViewGroup) 消费 Touch 事件，在代码上等价于调用当前类的 onTouchEvent 方法。然后 ouTouchEvent 的返回结果作为是否消费了此事件的依据。

**拦截：**

拦截是指 ViewGroup 有机会在把 touch 事件 dispatch 给 child 前，通过调用自己的 onInterceptTouchEvent 来判断要不要把 touch 事件截下来给自己尝试消费。对于 DOWN 事件，onInterceptTouchEvent 总是会被调用；对于其他事件，child 有机会在 parent 的onInterceptTouchEvent 被调用之前，请求 parent 不要拦截，这个请求就是通过 child 调用 parent 的 requestDisallowInterceptTouchEvent 方法来实现的。

**DOWN 事件：**

DOWN 事件是一串 touch 事件的第一个事件，在 ViewGroup 的 dispatchTouchEvent 方法中，接到此事件时，首先会做一些清除工作，然后检查自己要不要拦截此事件和是不是要转为 CANCEL 事件。如果不拦截且不转为 CANCEL，那么挨个检查 child，在恰当的时候调用 child 的 dispatchTouchEvent 来看有没有 child 会消费此事件。

上面 4 段话，用伪代码来表示，大概就是下面这样的：

```java
class ViewGroup extends View {
    public boolean dispatchTouchEvent(MotionEvent event) { 
        final boolean intercepted; // 拦截即不把事件传给 child 
        boolean handled = false; // View group 是否消费了事件 
        if (is DOWN || child ever comsumed previous event) { 
            if (! disallowIntercept) { // PS: 对于 DOWN 其实 disallowIntercept 一定是 false 
                intercepted = onInterceptTouchEvent(event); 
            } else { 
                intercepted = false; 
            }
        } else { 
            intercepted = true; 
        } if (! canceled && !intercepted) {
            for (View child : all children) {
                if (! child able to consume event) {
                    continue; 
                } if (dispatchTransformedTouchEvent(event, child)) { 
                    // 做一些记录工作，然后 break 
                    break; 
                }
            } 
        } 
        // previous event 指这一串 touch 事件中在 event 前面的那些 
        if (no child ever consumed previous event) {
            handled = super.dispatchTransformedTouchEvent(event); 
            // 尝试自行消费 event 
        } else { 
            // 其实非 down 事件才能走到这个 else 分支 
            for (child : children who has consumed previous event) {
                if (child.dispatchTouchEvent(event) 
                    handled = true; 
            }
        } 
        return handled; 
    }
} 

class View {
    public boolean dispatchTouchEvent(MotionEvent event) {
        boolean result = false; 
        if (mOnTouchListener != null && isEnabled && mOnTouchListener.onTouch(this, event)) { 
            result = true; 
        } 
        // onTouch 优于 onTouchEvent 来代表 View 尝试消费 event 
        if (! result && onTouchEvent(event)) { 
            result = true; 
        } 
        return result; 
    }
    public boolean onTouchEvent(MotionEvent event) {
        if (!isEnabled()) {
            return isClickable() || isLongClickable(); 
        } if (isClickable || isLongClickable) { 
            ...
            retrun true; 
        } retrun false;
    }
}
```

各个类和方法的角色描述如下：

>ViewGroup#dispatchTouchEvent，代表 ViewGroup 接受事件，处理拦截逻辑，尝试分发事件给 child，调用 View#dispatchTouchEvent来尝试自行消费 Touch 事件。返回值表示自己这颗子树有没有消费事件。

>View#dispatchTouchEvent 代表非 ViewGroup 的 View 接受事件，调用真正尝试消费事件的代码（onTouchEvent 或者 onTouch 之类的方法）。返回值表示自己有没有消费事件。

>ViewGroup#onInterceptTouchEvent，返回值表示 ViewGroup 要不要拦截事件。

>ViewGroup#requestDisallowIntercept，被 child 来调用，表示请求 parent 不要拦截，这个请求仅在非 DOWN 事件有效，且会递归向上调用所有 parent 的同名方法。

>TouchTarget 类，此类包含消费过 Touch 事件的 child 和它消费过哪些 pointer 的事件这些信息，TouchTarget 如下图所示。在 ViewGroup 中，存有一个 TouchTarget 链表，遍历此链表，即可知道一个 pointer 是否已有事件被某个 child 处理过。

以上的介绍，其实是省略了不少信息，真正的事件分发过程，要更复杂不少，下面是我尝试画的一张流程图 (把图和源码对照着理解可能会有些帮助)：

![](http://tao93.top/wp-content/uploads/2018/08/dispatch_procedure.png)

对于上图，有一些需要解释的地方：

1. TouchTarget 持有一个 child View，和此 child View 曾消费过 Down 或者 pointerDown 事件的那些 Pointer 的 id，这些 pointer id 是以 id bits 的形式存储为一个整数的。

2. TouchTarget 链表的头结点是由 mFirstTouchTarget 引用的。在一串事件结束(处理完 UP)后，正常情况链表应该清空，在一串 Touch 事件到来前(处理 Down 前)也会清空，算是补刀。

3. 链表的意义是，存储 child 曾经消费过某些 pointer 的 Down 或者 PointerDown 事件这种信息。当一个 Pointer 结束了（手指头离开屏幕），那么所有消费过它的 Down 或者 PointerDown 事件的 TouchTarget 都需要移除掉它的 id，事实上，这一串 touch 事件中再也不会有这个 id 了；如果一个 TouchTarget 的 pointer id移除光了，那么意味着此 TouchTarget 持有的 child 没有消费过任意(现存)的 pointer 的 Down 或者 PointerDown，于是可以把此 TouchTarget 从链表移除了。

4. 每次一个 Down 或者 PointerDown 事件 ev 到来时，对于 ViewGroup 的每个 child x，若 ev 的坐标落在 x 的范围内(否则就 continue，考虑下一个 child)，进一步「如果 x 在链表中(说明 x 消费过 Down 或者 pointerDown 事件)，那么 x 就是要被分发的 child；否则如果 x.dispatchTouchEvent(ev) 返回 true 了，那么 x 同样是要被分发的 child，虽然此时分发已经结束了」。

5. 每个 move 事件 ev 到来时，链表为空的话，显然没有 child 消费过 down 或者 pointerDown，那么直接让 viewGroup 处理 ev 就好了。链表不为空的话，对于链表的每个 TouchTarget t 持有的 child x：如果转成了 cancel 事件，那么向 x 分发一个 cancel 事件，另外把 t 从链表移除(移除的原因是，例如 x 正在 detaching，所以才引发 cancel，那么当然需要把持有 x 的 t 移除)，然后 over，即 move 事件丢失了；没转成 cancel，那就检查 t 的 idBits 中有没有 ev 的任意一个 pointer 的 id，有则把 ev 交给 x，没有则 continue。

四、源码分析

下面粘贴一大段加上了我的理解作为注释的源码，代码真的很长很长，这还仅仅是 ViewGroup#dispatchTouchEvent 这一个方法的代码。关于多点触控的处理逻辑，我也没有彻底明白，sigh。。。

```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    boolean handled = false;
    // 若当前类上面覆盖了其他window, 且 ev 的flag已标记过滤「覆盖」情况下的事件，则直接跳过
    if (onFilterTouchEventForSecurity(ev)) {
        final int action = ev.getAction();
        final int actionMasked = action & MotionEvent.ACTION_MASK;
        // Handle an initial down.
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Throw away all previous state when starting a new touch gesture.
            // The framework may have dropped the up or cancel event for the previous gesture
            // due to an app switch, ANR, or some other state change.
            cancelAndClearTouchTargets(ev); //如链表非空, 向其中所有 touchTarget 持有的 child 发 cancel 事件, 并清空链表
            resetTouchState(); // reset 此 ViewGroup 的 CANCEL_NEXT_UP 和 DISALLOW_INTERCEPT
        }
        // Check for interception.
        final boolean intercepted;
        // down 事件, 或者链表不空(意思是曾有 down 事件被某个 child 消费过)时, 考虑拦截
        if (actionMasked == MotionEvent.ACTION_DOWN
                || mFirstTouchTarget != null) {
            // 前面的 resetTouchState() 使得 disallowIntercept 必定是 false
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
            if (!disallowIntercept) {
                intercepted = onInterceptTouchEvent(ev);
                ev.setAction(action); // restore action in case it was changed
            } else {
                intercepted = false;
            }
        } else { // 其他情况, 不用把事件交给任何 child, 所以直接赋值 intercepted = true
            // There are no touch targets and this action is not an initial down
            // so this view group continues to intercept touches.
            intercepted = true;
        }
        // Check for cancelation.
        // 此 ViewGroup 有 cancel_next_up 标志, 那么就转成 cancel 事件
        final boolean canceled = resetCancelNextUpFlag(this)
                || actionMasked == MotionEvent.ACTION_CANCEL;
        // Update list of touch targets for pointer down, if needed.
        // API 11 以上, split 必为 true
        final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
        TouchTarget newTouchTarget = null;
        // 标记 down/pointerDown 被某个 child 消费
        boolean alreadyDispatchedToNewTouchTarget = false;
        if (!canceled && !intercepted) {
            // down/pointerDown 才寻求 child 来接盘, 其余类型的事件都是谁消费了 down/pointerDown 就交给谁
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                // 得到 ev 当前 action 所所属的 pointer 的 index
                final int actionIndex = ev.getActionIndex(); // always 0 for down
                // idBitsToAssign 包含一个 id, 即 ev 所属 pointer 的 id
                final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                        : TouchTarget.ALL_POINTER_IDS;
                // Clean up earlier touch targets for this pointer id in case they
                // have become out of sync.
                // down/pointerDown 意味着新的 pointer, 清除一下链表中和新 pointer 的 id 之间的瓜葛
                removePointersFromTouchTargets(idBitsToAssign);
                final int childrenCount = mChildrenCount;
                if (newTouchTarget == null && childrenCount != 0) {
                    final float x = ev.getX(actionIndex);
                    final float y = ev.getY(actionIndex);
                    // Find a child that can receive the event.
                    // Scan children from front to back.
                    final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                    final boolean customOrder = preorderedList == null
                            && isChildrenDrawingOrderEnabled();
                    final View[] children = mChildren;
                    for (int i = childrenCount - 1; i >= 0; i--) {
                        final int childIndex = getAndVerifyPreorderedIndex(
                                childrenCount, i, customOrder);
                        final View child = getAndVerifyPreorderedView(
                                preorderedList, children, childIndex);
                        
                        // 不能接受或者是 ev 坐标不在 child 内部则 continue
                        if (!canViewReceivePointerEvents(child)
                                || !isTransformedTouchPointInView(x, y, child, null)) {
                            ev.setTargetAccessibilityFocus(false);
                            continue;
                        }
                        newTouchTarget = getTouchTarget(child);
                        if (newTouchTarget != null) {
                            // 说明 ev 落在 child 内部且 child 以前响应过 down/pointerDown, 那么由 child 来接盘, 故 break
                            // 如果 ev 是 Down 事件, 列表尚空, 则走不到这里来
                            // Child is already receiving touch within its bounds.
                            // Give it the new pointer in addition to the ones it is handling.
                            // child 所属 TouchTarget 可能增加一个 pointer id
                            newTouchTarget.pointerIdBits |= idBitsToAssign;
                            break;
                        }
                        resetCancelNextUpFlag(child);
                        // 尝试让 child 消费 ev
                        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                            // Child wants to receive touch within its bounds.
                            mLastTouchDownTime = ev.getDownTime();
                            if (preorderedList != null) {
                                // childIndex points into presorted list, find original index
                                for (int j = 0; j < childrenCount; j++) {
                                    if (children[childIndex] == mChildren[j]) {
                                        mLastTouchDownIndex = j;
                                        break;
                                    }
                                }
                            } else {
                                mLastTouchDownIndex = childIndex;
                            }
                            mLastTouchDownX = ev.getX();
                            mLastTouchDownY = ev.getY();
                            // 消费成功的话, 用 child 新建 TouchTarget, 插到链表头部,
                            newTouchTarget = addTouchTarget(child, idBitsToAssign);
                            alreadyDispatchedToNewTouchTarget = true;
                            break;
                        }
                        // The accessibility focus didn't handle the event, so clear
                        // the flag and do a normal dispatch to all children.
                        ev.setTargetAccessibilityFocus(false);
                    }
                    if (preorderedList != null) preorderedList.clear();
                }
                if (newTouchTarget == null && mFirstTouchTarget != null) {
                    // 没找到接盘的, 且链表不空(意思是有 Down/pointerDown 被 child View 消费过)
                    // Did not find a child to receive the event.
                    // Assign the pointer to the least recently added target.
                    // 强行让链表末尾节点持有的 child 来接盘
                    newTouchTarget = mFirstTouchTarget;
                    while (newTouchTarget.next != null) {
                        newTouchTarget = newTouchTarget.next;
                    }
                    newTouchTarget.pointerIdBits |= idBitsToAssign;
                }
            }
        }
        // Dispatch to touch targets.
        if (mFirstTouchTarget == null) { // child view 压根没有消费过事件
            // No touch targets so treat this as an ordinary view.
            // view group 尝试自行消费
            handled = dispatchTransformedTouchEvent(ev, canceled, null,
                    TouchTarget.ALL_POINTER_IDS);
        } else {
            // Dispatch to touch targets, excluding the new touch target if we already
            // dispatched to it.  Cancel touch targets if necessary.
            TouchTarget predecessor = null;
            TouchTarget target = mFirstTouchTarget;
            // 遍历链表
            while (target != null) {
                final TouchTarget next = target.next;
                if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                    // ev 是 Down 或者 pointer Down 且它被 target 持有的 child 消费了
                    handled = true;
                } else { // 对于其他的 Touch target
                    final boolean cancelChild = resetCancelNextUpFlag(target.child)
                            || intercepted;
                    // 发送 cancel 或者 ev 给 child, 或者啥也不做(target 的 idBits 没有 ev 中任意的 pointer 的 id)
                    if (dispatchTransformedTouchEvent(ev, cancelChild,
                            target.child, target.pointerIdBits)) {
                        handled = true;
                    }
                    if (cancelChild) { // 收到 cancel 后,child 就不该再收到事件了, 对应的 TouchTarget 也要移除
                        if (predecessor == null) {
                            mFirstTouchTarget = next;
                        } else {
                            predecessor.next = next;
                        }
                        target.recycle();
                        target = next;
                        continue;
                    }
                }
                predecessor = target;
                target = next;
            }
        }
        // Update list of touch targets for pointer up or cancel, if needed.
        if (canceled
                || actionMasked == MotionEvent.ACTION_UP
                || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
            resetTouchState(); // up 事件后,这一串 Touch 事件就结束了, 清理链表等
        } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
            // 某个 pointer 没了, 那么清空链表中和这个 pointer 的瓜葛
            final int actionIndex = ev.getActionIndex();
            final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
            removePointersFromTouchTargets(idBitsToRemove);
        }
    }
    if (!handled && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
    }
    return handled;
}
```

五、滑动冲突分析示例

在《安卓开发艺术探索》一书中讲 touch 事件的最后，讲了滑动冲突的解决方法。所谓滑动冲突最简单的情形就是一个水平滑动的 ScrollView 里面放一个竖直滑动的 listView，两者滑动方向不同，即为滑动冲突。对于这个情形，解决滑动冲突，其实就是在手指上下滑时吧 move 事件给 child（listView）处理，而手指左右滑动时给 parent 处理（ScrollView）即可。下面是书中的一种解决方式，使用上面讲的知识，我们可以透彻的分析这种解决方式。

下面是《安卓开发艺术探索》中提供的其中一种解决方法，我加入了比较详细的注释作为解释，在前面的基础上，这种解决方法的逻辑就很明确了。

```java
class ChildView extends android.view.View {
    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        ViewParent parent = getParent();
        if (parent != null) {
            switch (event.getActionMasked()) {
                case MotionEvent.ACTION_DOWN:
                    // 此次请求其实在 Down 事件后的首个 Move 传至 parent 中时生效
                    // 若不请求，则将会发生拦截，后续事件都和 child 无缘，所以必须请求别拦截
                    parent.requestDisallowInterceptTouchEvent(true);
                    break;
                case MotionEvent.ACTION_MOVE:
                    // 如果期望 parent 来处理，那么解除请求，则下一次 Move 事件时必定拦截，后续事件就全部交给 parent 了。
                    // 否则，什么也不做，即事件会继续源源不断的交给 child
                    if (parent should handle event){
                        parent.requestDisallowInterceptTouchEvent(false);
                    }
                    break;
                default:
                    break;
            }
        }
        return super.dispatchTouchEvent(event);
    }
}
public class ParentView extends ViewGroup {
    
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        // Down 时，必然调此方法，此时不应拦截，否则 child 永远无法处理 move 事件
        // 其他事件时，若 child 请求不拦截，那么后面的事件都交给 child 了；否则，就会
        // 调用此方法，此方法这时返回 True 即表示拦截，那么会发一个 cancel 给 child，后续的事件就和 child 无缘了
        return ev.getActionMasked() != MotionEvent.ACTION_DOWN;
    }
}
```

到此，这篇长长的、并不完美的分析也就结束了。
