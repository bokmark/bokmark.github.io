---
title: '触摸事件 分发流程'
date: 2018-11-28T15:14:39+10:00
weight: 3
summary: "触摸事件"
---

# Android 的触摸事件 怎么分发的
> https://www.jianshu.com/p/e99b5e8bd67b
> https://www.cnblogs.com/aademeng/articles/10702653.html
> http://bbs.xiangxueketang.cn/article/detail/110

## 事件分为四个事件
- action down 手指触摸的时候会触发
- action move 移动的时候会触发
- action up 手指松开的时候会触发
- action cancel 事件被上层拦截的时候会触发

## 来看触摸的时间怎么分发
几个主要的类的代码内容。
- Activity 会分发到window
{{<details "Activity.java">}}
```Java
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
```
{{</details>}}

- PhoneWindow 调用Decorview的dispatch
{{<details "PhoneWindow.java">}}
```java
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }
```
{{</details>}}

- DecorView 调用了 view的superdistouch，docorview 继承自Framelayout。所以最终调用到ViewGroup
{{<details "DecorView.java">}}
```Java
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return super.dispatchTouchEvent(event);
    }
```
{{</details>}}

- ViewGroup的
{{<details "ViewGroup.java">}}
```Java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
  //..

  boolean handled = false;
  if (onFilterTouchEventForSecurity(ev)) {
      final int action = ev.getAction();
      final int actionMasked = action & MotionEvent.ACTION_MASK;

      // Handle an initial down.
      if (actionMasked == MotionEvent.ACTION_DOWN) {
          // 对touch事件重置，如果滚动会停止
          cancelAndClearTouchTargets(ev);
          resetTouchState();
      }

      // 检查拦截
      final boolean intercepted;
      // 如果当前是down事件，
      // 或者mFirstTouchTarget不为null
      if (actionMasked == MotionEvent.ACTION_DOWN
              || mFirstTouchTarget != null) {
          // 由view树结构的子节点 调用requestdisallowintercept 来设置
          final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
          // 如果子节点没有设置 不允许拦截，即允许父节点拦截
          if (!disallowIntercept) {
              // 判断 是否需要拦截
              intercepted = onInterceptTouchEvent(ev);
              // todo
              ev.setAction(action); // restore action in case it was changed
          } else {
              // 如果子节点不允许拦截，则不会进行拦截，也不会走到onintercettouchevent方法
              intercepted = false;
          }
      } else {
          // todo
          intercepted = true;
      }
      // 这里做拦截的判断,如果拦截 跳转到 anchor1 之后由viewgroup 自己处理。
      // 如果不拦截 进入语句
      // 。。。

      // Check for cancelation.
      final boolean canceled = resetCancelNextUpFlag(this)
              || actionMasked == MotionEvent.ACTION_CANCEL;

      // Update list of touch targets for pointer down, if needed.
      final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
      TouchTarget newTouchTarget = null; // 这一次分发的 touch target
      boolean alreadyDispatchedToNewTouchTarget = false;
      if (!canceled && !intercepted) {

          // 如果是down事件才会进行分发

          if (actionMasked == MotionEvent.ACTION_DOWN
                  || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                  || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
              final int actionIndex = ev.getActionIndex(); // always 0 for down
              final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                      : TouchTarget.ALL_POINTER_IDS;

              // Clean up earlier touch targets for this pointer id in case they
              // have become out of sync.
              removePointersFromTouchTargets(idBitsToAssign);

              final int childrenCount = mChildrenCount;
              if (newTouchTarget == null && childrenCount != 0) {
                  final float x = ev.getX(actionIndex);
                  final float y = ev.getY(actionIndex);
                  // 按照顺序排序好 所有的child view
                  final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                  final boolean customOrder = preorderedList == null
                          && isChildrenDrawingOrderEnabled();
                  final View[] children = mChildren;
                  for (int i = childrenCount - 1; i >= 0; i--) {
                      final int childIndex = getAndVerifyPreorderedIndex(
                              childrenCount, i, customOrder);
                      final View child = getAndVerifyPreorderedView(
                              preorderedList, children, childIndex);

                      // If there is a view that has accessibility focus we want it
                      // to get the event first and if not handled we will perform a
                      // normal dispatch. We may do a double iteration but this is
                      // safer given the timeframe.
                      if (childWithAccessibilityFocus != null) {
                          if (childWithAccessibilityFocus != child) {
                              continue;
                          }
                          childWithAccessibilityFocus = null;
                          i = childrenCount - 1;
                      }

                      // 这个child 是不是可以响应touch，或者在这个区域内
                      // 如果不是在这个范围内，则继续寻找
                      if (!child.canReceivePointerEvents()
                              || !isTransformedTouchPointInView(x, y, child, null)) {
                          ev.setTargetAccessibilityFocus(false);
                          continue;
                      }

                      // 如果走到这 就意味着找到了 能响应事件的view
                      // 第一次down mFirstTouchTarget 还是为空
                      newTouchTarget = getTouchTarget(child);
                      /**
                      private TouchTarget getTouchTarget(@NonNull View child) {
                          for (TouchTarget target = mFirstTouchTarget; target != null; target = target.next) {
                              if (target.child == child) {
                                  return target;
                              }
                          }
                          return null;
                      }
                      **/
                      if (newTouchTarget != null) {
                          // Child is already receiving touch within its bounds.
                          // Give it the new pointer in addition to the ones it is handling.
                          newTouchTarget.pointerIdBits |= idBitsToAssign;
                          break;
                      }

                      resetCancelNextUpFlag(child);
                      // 由child 是否需要消费，则寻找下一个可以响应的view
                      if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                          // 代表child 处理事件
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
                          // 赋值 mFirstTouchTarget 和 newTouchTarget
                          newTouchTarget = addTouchTarget(child, idBitsToAssign);
                          /**
                          private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
                              final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
                              target.next = mFirstTouchTarget;
                              mFirstTouchTarget = target;
                              return target;
                          }
                          **/

                          alreadyDispatchedToNewTouchTarget = true;
                          break;
                      }

                      // The accessibility focus didn't handle the event, so clear
                      // the flag and do a normal dispatch to all children.
                      ev.setTargetAccessibilityFocus(false);
                  }
                  if (preorderedList != null) preorderedList.clear();
              }
              // 如果此时 newTouchTarget 还是为空，那么就意味着没有找到可以响应的view。

              if (newTouchTarget == null && mFirstTouchTarget != null) {
                  // Did not find a child to receive the event.
                  // Assign the pointer to the least recently added target.
                  newTouchTarget = mFirstTouchTarget;
                  while (newTouchTarget.next != null) {
                      newTouchTarget = newTouchTarget.next;
                  }
                  newTouchTarget.pointerIdBits |= idBitsToAssign;
              }
          }
      }

      // anchor 1
      // down 事件触发的时候 touchtarget 一定会被清空。
      // 如果拦截了，mFirstTouchTarget 就会为空 dispatchTransformedTouchEvent 就自己处理事件
      // 如果全部不处理，那么就相当于拦截
      if (mFirstTouchTarget == null) {
          // No touch targets so treat this as an ordinary view.
          handled = dispatchTransformedTouchEvent(ev, canceled, null,
                  TouchTarget.ALL_POINTER_IDS);
      } else {
          // Dispatch to touch targets, excluding the new touch target if we already
          // dispatched to it.  Cancel touch targets if necessary.
          TouchTarget predecessor = null;
          TouchTarget target = mFirstTouchTarget;
          while (target != null) {
              final TouchTarget next = target.next;
              if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                  handled = true;
              } else {
                  final boolean cancelChild = resetCancelNextUpFlag(target.child)
                          || intercepted;
                  if (dispatchTransformedTouchEvent(ev, cancelChild,
                          target.child, target.pointerIdBits)) {
                      handled = true;
                  }
                  if (cancelChild) {
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
          resetTouchState();
      } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
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


/**
 * Transforms a motion event into the 坐标 space of a particular child view,
 * filters out irrelevant pointer ids, and overrides its action if necessary.
 * If child is null, assumes the MotionEvent will be sent to this ViewGroup instead.
 */
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
        View child, int desiredPointerIdBits) {
    final boolean handled;

    // Canceling motions is a special case.  We don't need to perform any transformations
    // or filtering.  The important part is the action, not the contents.
    final int oldAction = event.getAction();
    if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
        event.setAction(MotionEvent.ACTION_CANCEL);
        if (child == null) {
            handled = super.dispatchTouchEvent(event);
        } else {
            handled = child.dispatchTouchEvent(event);
        }
        event.setAction(oldAction);
        return handled;
    }

    // Calculate the number of pointers to deliver.
    final int oldPointerIdBits = event.getPointerIdBits();
    final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

    // If for some reason we ended up in an inconsistent state where it looks like we
    // might produce a motion event with no pointers in it, then drop the event.
    if (newPointerIdBits == 0) {
        return false;
    }

    // If the number of pointers is the same and we don't need to perform any fancy
    // irreversible transformations, then we can reuse the motion event for this
    // dispatch as long as we are careful to revert any changes we make.
    // Otherwise we need to make a copy.
    final MotionEvent transformedEvent;
    if (newPointerIdBits == oldPointerIdBits) {
        if (child == null || child.hasIdentityMatrix()) {
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                final float offsetX = mScrollX - child.mLeft;
                final float offsetY = mScrollY - child.mTop;
                event.offsetLocation(offsetX, offsetY);

                handled = child.dispatchTouchEvent(event);

                event.offsetLocation(-offsetX, -offsetY);
            }
            return handled;
        }
        transformedEvent = MotionEvent.obtain(event);
    } else {
        transformedEvent = event.split(newPointerIdBits);
    }

    // Perform any necessary transformations and dispatch.
    if (child == null) {
        handled = super.dispatchTouchEvent(transformedEvent);
    } else {
        final float offsetX = mScrollX - child.mLeft;
        final float offsetY = mScrollY - child.mTop;
        transformedEvent.offsetLocation(offsetX, offsetY);
        if (! child.hasIdentityMatrix()) {
            transformedEvent.transform(child.getInverseMatrix());
        }

        handled = child.dispatchTouchEvent(transformedEvent);
    }

    // Done.
    transformedEvent.recycle();
    return handled;
}
```
{{</details>}}


# 总结
- down：
viewgroup->dispatchTouchEvent->onInterceptTouchEvent(disallowIntercept)->
