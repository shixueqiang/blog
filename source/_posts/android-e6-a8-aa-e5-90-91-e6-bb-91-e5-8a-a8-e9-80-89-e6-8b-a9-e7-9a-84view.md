---
title: android横向滑动选择的view
id: 58
categories:
  - 未分类
date: 2016-09-13 14:51:06
tags:
---

做文字编辑，从网上找来的。
<pre>HorizontalScrollSelectView：</pre>
<pre class="java">    public boolean mAlwaysOverrideTouch = true;
    protected ListAdapter mAdapter;
    private int mLeftViewIndex = -1;
    private int mRightViewIndex = 0;
    protected int mCurrentX;
    protected int mNextX;
    private int mMaxX = Integer.MAX_VALUE;
    private int mDisplayOffset = 0;
    protected Scroller mScroller;
    private GestureDetector mGesture;
    private Queue&lt;View&gt; mRemovedViewQueue = new LinkedList&lt;View&gt;();
    private OnItemSelectedListener mOnItemSelected;
    private OnItemClickListener mOnItemClicked;
    private OnItemLongClickListener mOnItemLongClicked;
    private OnScrollListener mScrollListener;
    /**
     * 选中item时图片
     */
    private Drawable mDrawable;
    private boolean mDataChanged = false;
    private Context context;
    private boolean scrollerFalg1 = false;
    private boolean scrollerFalg2 = false;
    private int position = 0x7f020000;

    public HorizontalScrollSelectView(Context context, AttributeSet attrs) {
        super(context, attrs);
        this.context = context;
        initView();
    }

    private synchronized void initView() {
        mLeftViewIndex = -1;
        mRightViewIndex = 0;
        mDisplayOffset = 0;
        mCurrentX = 0;
        mNextX = 0;
        mMaxX = Integer.MAX_VALUE;
        mScroller = new Scroller(getContext());
        mGesture = new GestureDetector(getContext(), mOnGesture);
    }

    public void setMScrollListener(OnScrollListener listener) {
        mScrollListener = listener;
    }

    @Override
    public void setOnItemSelectedListener(OnItemSelectedListener listener) {
        mOnItemSelected = listener;
    }

    @Override
    public void setOnItemClickListener(OnItemClickListener listener) {
        mOnItemClicked = listener;
    }

    @Override
    public void setOnItemLongClickListener(OnItemLongClickListener listener) {
        mOnItemLongClicked = listener;
    }

    /**
     * 设置选中状态时的图片
     *
     * @param mDrawable
     */
    public void setSelectBitmap(Drawable mDrawable) {
        this.mDrawable = mDrawable;
    }

    private DataSetObserver mDataObserver = new DataSetObserver() {

        @Override
        public void onChanged() {
            synchronized (HorizontalScrollSelectView.this) {
                mDataChanged = true;
            }
            invalidate();
            requestLayout();
        }

        @Override
        public void onInvalidated() {
            reset();
            invalidate();
            requestLayout();
        }

    };

    @Override
    public ListAdapter getAdapter() {
        return mAdapter;
    }

    @Override
    public View getSelectedView() {
        //TODO: implement
        return null;
    }

    @Override
    public void setAdapter(ListAdapter adapter) {
        if (mAdapter != null) {
            mAdapter.unregisterDataSetObserver(mDataObserver);
        }
        mAdapter = adapter;
        mAdapter.registerDataSetObserver(mDataObserver);
        reset();
    }

    private synchronized void reset() {
        initView();
        removeAllViewsInLayout();
        requestLayout();
    }

    @Override
    public void setSelection(int position) {
        //TODO: implement
        int positionX = position * this.getWidth();
        int maxWidth = this.getChildCount() * this.getWidth();
        if (positionX &lt;= 0) {
            positionX = 0;
        }
        if (positionX &gt; maxWidth) {
            positionX = maxWidth;
        }
        scrollTo(positionX);
    }

    private void addAndMeasureChild(final View child, int viewPos) {
        LayoutParams params = child.getLayoutParams();
        if (params == null) {
            params = new LayoutParams(LayoutParams.FILL_PARENT, LayoutParams.FILL_PARENT);
        }

        addViewInLayout(child, viewPos, params, true);
        child.measure(MeasureSpec.makeMeasureSpec(getWidth(), MeasureSpec.AT_MOST),
                MeasureSpec.makeMeasureSpec(getHeight(), MeasureSpec.AT_MOST));
    }

    @SuppressLint("NewApi")
    @Override
    protected synchronized void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);

        if (mAdapter == null) {
            return;
        }

        if (mDataChanged) {
            int oldCurrentX = mCurrentX;
            initView();
            removeAllViewsInLayout();
            mNextX = oldCurrentX;
            mDataChanged = false;
        }

        if (mScroller.computeScrollOffset()) {
            int scrollx = mScroller.getCurrX();
            mNextX = scrollx;
        }

//		if(mNextX &lt;= 0){
//			mNextX = 0;
//			mScroller.forceFinished(true);
//		}
//		if(mNextX &gt;= mMaxX) {
//			mNextX = mMaxX;
//			mScroller.forceFinished(true);
//		}

        int dx = mCurrentX - mNextX;

        removeNonVisibleItems(dx);
        fillList(dx);
        positionItems(dx);
//		Log.e("onlayout", "mLeftViewIndex"+(mLeftViewIndex+1));
        mCurrentX = mNextX;

        if (!mScroller.isFinished()) {
            post(new Runnable() {
                @Override
                public void run() {
                    requestLayout();
                }
            });

        } else {
            if (scrollerFalg1 || (!scrollerFalg1 &amp;&amp; scrollerFalg2)) {
                View chid = getChildAt(0);
                if (chid != null) {
                    mDisplayOffset = 0;
                    positionItems(0);
                    if (mDrawable != null) {
                        getChildAt(2).setBackground(mDrawable);
                    }
                    if (mScrollListener != null) {
                        int position = (int) getChildAt(2).getTag(this.position);
                        mScrollListener.onScrollSelectItem(this, position);
                    }
                }

            }
        }

    }

    /**
     * 获取屏幕宽度
     *
     * @param context
     * @return
     */
    public static int getSecreenWidth(Context context) {
        DisplayMetrics dm = new DisplayMetrics();
        dm = context.getResources().getDisplayMetrics();
        int screenWidth = dm.widthPixels;
        return screenWidth / 5;
    }

    private void fillList(final int dx) {
        int edge = 0;
        View child = getChildAt(0);
        if (child != null) {
            edge = child.getLeft();
        }
        fillListLeft(edge, dx);

        edge = 0;
        child = getChildAt(getChildCount() - 1);
        if (child != null) {
            edge = child.getRight();
        }
        fillListRight(edge, dx);

    }

    private void fillListRight(int rightEdge, final int dx) {
        //&amp;&amp; mRightViewIndex &lt; mAdapter.getCount()
        while (rightEdge + dx &lt; getWidth()) {
            if (mRightViewIndex &gt;= mAdapter.getCount()) {
                mRightViewIndex = 0;
            }
            View child = mAdapter.getView(mRightViewIndex, mRemovedViewQueue.poll(), this);
            child.setTag(position, mRightViewIndex);
            addAndMeasureChild(child, -1);
            rightEdge += child.getMeasuredWidth();

//			if(mRightViewIndex == mAdapter.getCount()-1) {
//				mMaxX = mCurrentX + rightEdge - getWidth();
//			}
//			
//			if (mMaxX &lt; 0) {
//				mMaxX = 0;
//			}
            mRightViewIndex++;
        }

    }

    private void fillListLeft(int leftEdge, final int dx) {
        //&amp;&amp; mLeftViewIndex &gt;= 0
        while (leftEdge + dx &gt; 0) {
            if (mLeftViewIndex &lt;= -1) {
                mLeftViewIndex = mAdapter.getCount() - 1;
            }
            View child = mAdapter.getView(mLeftViewIndex, mRemovedViewQueue.poll(), this);
            child.setTag(position, mLeftViewIndex);
            addAndMeasureChild(child, 0);
            leftEdge -= child.getMeasuredWidth();
            mLeftViewIndex--;
            mDisplayOffset -= child.getMeasuredWidth();
        }
    }

    private void removeNonVisibleItems(final int dx) {
        View child = getChildAt(0);
        while (child != null &amp;&amp; child.getRight() + dx &lt;= 0) {
            mDisplayOffset += child.getMeasuredWidth();
            mRemovedViewQueue.offer(child);
            removeViewInLayout(child);
            mLeftViewIndex++;
            if (mLeftViewIndex &gt;= mAdapter.getCount()) {
                mLeftViewIndex = 0;
            }
            child = getChildAt(0);

        }

        child = getChildAt(getChildCount() - 1);
        while (child != null &amp;&amp; child.getLeft() + dx &gt;= getWidth()) {
            mRemovedViewQueue.offer(child);
            removeViewInLayout(child);
            mRightViewIndex--;
            if (mRightViewIndex &lt;= -1) {
                mRightViewIndex = mAdapter.getCount() - 1;
            }
            child = getChildAt(getChildCount() - 1);
        }
    }

    private void positionItems(final int dx) {
        if (getChildCount() &gt; 0) {
            mDisplayOffset += dx;
            int left = mDisplayOffset;
            for (int i = 0; i &lt; getChildCount(); i++) {
                View child = getChildAt(i);
                getChildAt(2).setBackground(null);
                int childWidth = child.getMeasuredWidth();
                child.layout(left, 0, left + childWidth, child.getMeasuredHeight());
                left += childWidth + child.getPaddingRight();
            }
        }
    }

    public synchronized void scrollTo(int x) {
        mScroller.startScroll(mNextX, 0, x - mNextX, 0);
        requestLayout();
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean handled = super.dispatchTouchEvent(ev);
        handled |= mGesture.onTouchEvent(ev);

        switch (ev.getAction()) {
            case MotionEvent.ACTION_UP:
                scrollerFalg2 = true;
                requestLayout();
                break;
            default:
                break;
        }
        return handled;
    }

    protected boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
                              float velocityY) {
        synchronized (HorizontalScrollSelectView.this) {
            //mNextX
            mScroller.fling(mNextX, 0, (int) -velocityX, 0, Integer.MIN_VALUE, mMaxX, 0, 0);
        }
        requestLayout();
        scrollerFalg1 = true;
        return true;
    }

    protected boolean onDown(MotionEvent e) {
        mScroller.forceFinished(true);
        return true;
    }

    public interface OnScrollListener {
        /**
         * 互动过程中选中的item
         *
         * @param position
         */
        public void onScrollSelectItem(ViewGroup viewGroup, int position);

    }

    private OnGestureListener mOnGesture = new GestureDetector.SimpleOnGestureListener() {

        @Override
        public boolean onDown(MotionEvent e) {
            return HorizontalScrollSelectView.this.onDown(e);
        }

        @Override
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
                               float velocityY) {
            return HorizontalScrollSelectView.this.onFling(e1, e2, velocityX, velocityY);
        }

        @Override
        public boolean onScroll(MotionEvent e1, MotionEvent e2,
                                float distanceX, float distanceY) {

            synchronized (HorizontalScrollSelectView.this) {
                mNextX += (int) distanceX;
            }
            requestLayout();
            scrollerFalg1 = false;
            scrollerFalg2 = false;
            return true;
        }

        @Override
        public void onShowPress(MotionEvent e) {
            super.onShowPress(e);
        }

        @Override
        public boolean onSingleTapUp(MotionEvent e) {
            Log.e("onSingleTapUp", "mLeftViewIndex" + (mLeftViewIndex + 1));
//			scrollerFalg=true;
            return super.onSingleTapUp(e);
        }

        @Override
        public boolean onSingleTapConfirmed(MotionEvent e) {
            for (int i = 0; i &lt; getChildCount(); i++) {
                View child = getChildAt(i);
                if (isEventWithinView(e, child)) {
                    if (mOnItemClicked != null) {
                        mOnItemClicked.onItemClick(HorizontalScrollSelectView.this, child, mLeftViewIndex + 1 + i, mAdapter.getItemId(mLeftViewIndex + 1 + i));
                    }
                    if (mOnItemSelected != null) {
                        mOnItemSelected.onItemSelected(HorizontalScrollSelectView.this, child, mLeftViewIndex + 1 + i, mAdapter.getItemId(mLeftViewIndex + 1 + i));
                    }
                    break;
                }

            }
            return true;
        }

        @Override
        public void onLongPress(MotionEvent e) {
            int childCount = getChildCount();
            for (int i = 0; i &lt; childCount; i++) {
                View child = getChildAt(i);
                if (isEventWithinView(e, child)) {
                    if (mOnItemLongClicked != null) {
                        mOnItemLongClicked.onItemLongClick(HorizontalScrollSelectView.this, child, mLeftViewIndex + 1 + i, mAdapter.getItemId(mLeftViewIndex + 1 + i));
                    }
                    break;
                }

            }
        }

        private boolean isEventWithinView(MotionEvent e, View child) {
            Rect viewRect = new Rect();
            int[] childPosition = new int[2];
            child.getLocationOnScreen(childPosition);
            int left = childPosition[0];
            int right = left + child.getWidth();
            int top = childPosition[1];
            int bottom = top + child.getHeight();
            viewRect.set(left, top, right, bottom);
            return viewRect.contains((int) e.getRawX(), (int) e.getRawY());
        }
    };&lt;/span&gt;</pre>
<span style="font-size: large;">demo截图：</span>

<span style="font-size: large;">![](http://img.blog.csdn.net/20150901162650565?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)</span>

<span style="font-size: large;">demo地址：http://download.csdn.net/detail/s569646547/9070919</span>