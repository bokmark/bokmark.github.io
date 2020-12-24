---
title: '多线程--aqs'
date: 2020-02-11T19:30:08+10:00
draft: false
weight: 3
summary: "AbstractQueuedSynchronizer、 抽象队列同步器"
---

### aqs 的简介

- aqs 为什么开发工作中没有感知到呢？

- 同步工具类都是继承自aqs。

- 模版方法设计模式。
  类似于draw（canvas） 中的 这个
  ```

  protected void dispatchDraw(Canvas canvas) {

  }

  protected void onDraw(Canvas canvas) {
  }

  public void draw(Canvas canvas) {
      final int privateFlags = mPrivateFlags;
      mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

      /*
       * Draw traversal performs several drawing steps which must be executed
       * in the appropriate order:
       *
       *      1. Draw the background
       *      2. If necessary, save the canvas' layers to prepare for fading
       *      3. Draw view's content
       *      4. Draw children
       *      5. If necessary, draw the fading edges and restore layers
       *      6. Draw decorations (scrollbars for instance)
       *      7. If necessary, draw the default focus highlight
       */

      // Step 1, draw the background, if needed
      int saveCount;

      drawBackground(canvas);

      // skip step 2 & 5 if possible (common case)
      final int viewFlags = mViewFlags;
      boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
      boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
      if (!verticalEdges && !horizontalEdges) {
          // Step 3, draw the content
          onDraw(canvas);

          // Step 4, draw the children
          dispatchDraw(canvas);

          drawAutofilledHighlight(canvas);

          // Overlay is part of the content and draws b eneath Foreground
          if (mOverlay != null && !mOverlay.isEmpty()) {
              mOverlay.getOverlayView().dispatchDraw(canvas);
          }

          // Step 6, draw decorations (foreground, scrollbars)
          onDrawForeground(canvas);

          // Step 7, draw the default focus highlight
          drawDefaultFocusHighlight(canvas);

          if (isShowingLayoutBounds()) {
              debugDrawFocus(canvas);
          }

          // we're done...
          return;
      }
  ```
- aqs中的待继承方法

### clh 队列锁
- clh 三个人的头字母
![扫描二维码](/clh.jpg)
- qnode
 - mypred
 - locked
- 线程A 需要获取锁
  new 一个qnode mypred 指向队列的尾结点 locked=true

clh 队列锁 是一种自旋锁

aqs 是按照这种思想来实现的
