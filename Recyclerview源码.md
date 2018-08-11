 ### Recyclerview
24个内部类和接口，一万两千多行代码
#### 成员变量

```java 
private final RecyclerViewDataObserver mObserver = new RecyclerViewDataObserver();
final Recycler mRecycler = new Recycler();
@VisibleForTesting LayoutManager mLayout;
AdapterHelper mAdapterHelper;//Handles adapter updates
ChildHelper mChildHelper;// Handles abstraction between LayoutManager children and RecyclerView children
```

###循环器
内部类，负责管理废弃的或是与父布局分离的条目进行重用。5330~6360
废弃的条目的意思是说这个条目仍然附属于他的父亲RecyclerView但是他已经被标记为移除或是重回用。
内部有个三级缓存，一级为mChangedScrap，mAttachedScrap，mCachedViews，二级为：mViewCacheExtension（需要开发者自己维护），三级为：mRecyclerPool（内部也是一个集合ArrayList<ViewHolder> scrapHeap ）

```java 
public final class Recycler {
    final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
    ArrayList<ViewHolder> mChangedScrap = null;
    final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();
    private final List<ViewHolder> mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap);
    private int mRequestedCacheMax = DEFAULT_CACHE_SIZE;
    int mViewCacheMax = DEFAULT_CACHE_SIZE;
    RecycledViewPool mRecyclerPool;
    private ViewCacheExtension mViewCacheExtension;
    static final int DEFAULT_CACHE_SIZE = 2;
    //清除mAttachedScrap中的数据，但是把mCachedViews中的holder放到mRecyclerPool中并清除
    public void clear() {
        mAttachedScrap.clear();
        recycleAndClearCachedViews();
    }
    //将mCachedViews中的缓存清除并且将可回收的放到pool中
    void recycleCachedViewAt(int cachedViewIndex) {
        if (DEBUG) {
            Log.d(TAG, "Recycling cached view at index " + cachedViewIndex);
        }
        ViewHolder viewHolder = mCachedViews.get(cachedViewIndex);
        if (DEBUG) {
            Log.d(TAG, "CachedViewHolder to be recycled: " + viewHolder);
        }
        addViewHolderToRecycledViewPool(viewHolder, true);
        mCachedViews.remove(cachedViewIndex);
    }
    //设置最大的缓存数这个方法通过RecyclerView中的这个方法使用
    // public void setItemViewCacheSize(int size) {
    //   mRecycler.setViewCacheSize(size);
    // } 
    public void setViewCacheSize(int viewCount) {
        mRequestedCacheMax = viewCount;
        updateViewCacheSize();
    }
    void updateViewCacheSize() {
        int extraCache = mLayout != null ? mLayout.mPrefetchMaxCountObserved : 0;
        mViewCacheMax = mRequestedCacheMax + extraCache;
        //改变完了先把超出最大范围的做回收到pool中
        // first, try the views that can be recycled
        for (int i = mCachedViews.size() - 1;
                i >= 0 && mCachedViews.size() > mViewCacheMax; i--) {
            recycleCachedViewAt(i);
        }
    }
}
```

getViewForPosition（）方法的帮助方法，检查参数中的viewholder是否可以用到对应的位置，validateViewHolderForOffsetPosition（vievholder）

```java
//RecyclerView.Recycler
/**要绑定的这个view可以是从getViewForPosition(int)中拿到的也可以是通过onCreateViewHOlder(ViewGroup,int)创建的。通常来说LayoutManager应该通过getViewForPosition(int)这个方法获得view,然后让RecyclerView处理缓存。这个方法是帮助LayoutManger处理自己的回收逻辑的。
注意：getViewForPosition(int)这个方法已经将view绑定到对应位置上了，所以你不需要自己再调用这个方法了，除非你想把这个view绑定到其他位置上。
*/
public void bindViewToPosition(View view, int position) {
...
    tryBindViewHolderByDeadline(holder, offsetPosition, position, FOREVER_NS);
...
}

private boolean tryBindViewHolderByDeadline(ViewHolder holder, int offsetPosition,int position, long deadlineNs){
    ...
    mAdapter.bindViewHolder(holder, offsetPosition);
    ...

}
/*
通过指定的位置获取到一个初始化完成的view。
这个方法再LayoutManger的实现类中被调用，用来获取一个为来展现一个adapter中的数据。
*/
public View getViewForPosition(int position) {
            return getViewForPosition(position, false);
}

View getViewForPosition(int position, boolean dryRun) {
            return tryGetViewHolderForPositionByDeadline(position, dryRun, FOREVER_NS).itemView;
}
/*
这个方法比较重要
试图为相应的位置获取一个ViewHolder，从以下几个方面获得:Recycler的丢弃的，缓存的，RecyclerViewPool，或是直接创建一个。
第一级缓存：
如果仍依赖于 RecyclerView （比如已经滑动出可视范围，但还没有被移除掉），但已经被标记移除的 ItemView 集合会被添加到 mAttachedScrap 中。然后如果 mAttachedScrap 中不再依赖时会被加入到 mCachedViews 中。 mChangedScrap 则是存储 notifXXX 方法时需要改变的 ViewHolder 。

第二级缓存：
ViewCacheExtension 是一个抽象静态类，用于充当附加的缓存池，当 RecyclerView 从第一级缓存找不到需要的 View 时，将会从 ViewCacheExtension 中找。不过这个缓存是由开发者维护的，如果没有设置它，则不会启用。通常我们也不会去设置他，系统已经预先提供了两级缓存了，除非有特殊需求，比如要在调用系统的缓存池之前，返回一个特定的视图，才会用到他。

第三级缓存：
最强大的缓存器。
*/
ViewHolder tryGetViewHolderForPositionByDeadline(int position,
            boolean dryRun, long deadlineNs) {
     // 0) If there is a changed scrap, try to find from there
           mChangedScrap;
     // 1) Find by position from scrap/hidden list/cache
           mAttachedScrap;
           mCachedViews;一级缓存
     // 2) Find from scrap/cache via stable ids, if exists
           mAttachedScrap；
           mViewCacheExtension；二级缓存
     // fallback to pool
           mRecyclerPool；三级缓存
    //自行创建一个holder
     holder = mAdapter.createViewHolder(RecyclerView.this, type);
     RecyclerView.Adapter.onCreateViewHolder(ViewGroup parent, int viewType);

}

ViewHolder getChangedScrapViewForPosition(int position) {
     // find by position
     ...
     // find by id
     ...
}

```

回收一个detached view.将它放入到view池中，以便以后重新绑定或是重新使用。
这个view必须完全从父亲分离才可以被回收，如果他是废弃的，他将会从废弃集合中移除。
```java
public void recycleView(View view) {
            // This public recycle method tries to make view recycle-able since layout manager
            // intended to recycle this view (e.g. even if it is in scrap or change cache)
            ViewHolder holder = getChildViewHolderInt(view);
            if (holder.isTmpDetached()) {
                removeDetachedView(view, false);
            }
            if (holder.isScrap()) {
                holder.unScrap();
            } else if (holder.wasReturnedFromScrap()) {
                holder.clearReturnedFromScrapFlag();
            }
            recycleViewHolderInternal(holder);
}

 void recycleViewHolderInternal(ViewHolder holder) {
    if (forceRecycle || holder.isRecyclable()) {
        ...
        if (cachedViewSize >= mViewCacheMax && cachedViewSize > 0) {
            recycleCachedViewAt(0);
            cachedViewSize--;
        }
        ...
    }else{
        //如果一个view通过动画滚动出去的时候，他将不会被回收，他会通过ItemAnimatorRestoreListener#onAnimationFinished 这个方法回收

    }
 }
/*
将一个view标记为废弃。
一个废弃的view可以继续依附于他的父亲，但是他也可以重新绑定或是重新使用。给一个位置添加view可能重用或是重新绑定废弃view的实例。
*/
void scrapView(View view) {
    if(holder.hasAnyOfTheFlags(ViewHolder.FLAG_REMOVED | ViewHolder.FLAG_INVALID)
                || !holder.isUpdated() || canReuseUpdatedViewHolder(holder)){
        mAttachedScrap.add(holder);
    }else{
        mChangedScrap.add(holder); 
    }
}
```
###视图缓存扩展类
这是一个帮助类，他提供了额外的一层缓存，这层缓存可以有开发者控制。6361~6394
当调用Recycler.getViewForPosition(int)时，Recycler首先检查attached scrap 和一级缓存，查找是否有匹配的view，如果没有的话再调用Recycler.getViewForPositionAndType(Recycler, int, int),最后再检查RecyclerViewPool。
Recycler是不会主动向这里进行缓存的，这层缓存完全有开发者控制和维护，是否要添加，还是使用Recycler默认的缓存策略。
```java
public abstract static class ViewCacheExtension {
        /**
        返回一个可以绑定到指定位置的View.
        这个方法不能创建新的View,相反，他只能返回一个已经创建好的可以重用的view。如果这个view被标记为忽略，那再返回这个view之前要先调用LayoutManager.stopIgnoringView(View)。
        */
        public abstract View getViewForPositionAndType(Recycler recycler, int position, int type);
    }
```
### 回收视图池
你可以通过它实现多个RecyclerView之间共享视图。
如果你想通过RecyclerView来回收视图，那么你可以穿件一个RecycledViewPool实例，并通过RecyclerView#setRecycledViewPool(RecycledViewPool）来设置它。
如果你没有设置，RecyclerView会自动创建一个默认的pool。
每中type都有一个集合来缓存，每个集合又放入到一个SparseArray的map中。
```java
 public static class RecycledViewPool {
     private static final int DEFAULT_MAX_SCRAP = 5;
     static class ScrapData {
            ArrayList<ViewHolder> mScrapHeap = new ArrayList<>();
            int mMaxScrap = DEFAULT_MAX_SCRAP;
            long mCreateRunningAverageNs = 0;
            long mBindRunningAverageNs = v0;
        }
    //根据viewType缓存不同的view集合，每种type的集合默认都是5，所以这个缓存池的大小为5*type数。
    SparseArray<ScrapData> mScrap = new SparseArray<>();
    //总是返回缓存集合中的最后一个
    public ViewHolder getRecycledView(int viewType) {
                final ScrapData scrapData = mScrap.get(viewType);
                if (scrapData != null && !scrapData.mScrapHeap.isEmpty()) {
                    final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
                    return scrapHeap.remove(scrapHeap.size() - 1);
                }
                return null;
    }
 }
```
#### 绘制流程

1. onMeasure()方法
可见在onMeasure中实际是调用mLayout的onMeasure方法去进行实际的测量。 dispatchLayoutStep1();dispatchLayoutStep2();其中这两个方法比较可以，后续再看。现在目测就是要根据测量状态更改一些状态值。
```java
@Override
protected void onMeasure(int widthSpec, int heightSpec) {
    //如果没有设置LayoutManager的时候回进行默认的measure，这就是又是忘记设置LayoutManager时界面空白的原因。因为他内部是去拿最小的宽和高，所以就没有显示出来。
    if (mLayout == null) {
        defaultOnMeasure(widthSpec, heightSpec);
        return;
    }
    if (mLayout.mAutoMeasure) {
        ...
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
        ...
        if (mState.mLayoutStep == State.STEP_START) {
            dispatchLayoutStep1();
        }
        ...
        dispatchLayoutStep2();
        // now we can get the width and height from the children.
        mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
        // if RecyclerView has non-exact width and height and if there is at least one child
        // which also has non-exact width & height, we have to re-measure.
        if (mLayout.shouldMeasureTwice()) {
            mLayout.setMeasureSpecs(
                    MeasureSpec.makeMeasureSpec(getMeasuredWidth(), MeasureSpec.EXACTLY),
                    MeasureSpec.makeMeasureSpec(getMeasuredHeight(), MeasureSpec.EXACTLY));
            mState.mIsMeasuring = true;
            dispatchLayoutStep2();
            // now we can get the width and height from the children.
            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
        }
    } else {
        if (mHasFixedSize) {
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
            return;
        }
        ...
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
        ...
    }
}
/**
* 在设置LayoutManager之前调用onMeasure时执行默认的这个方法
*/
void defaultOnMeasure(int widthSpec, int heightSpec) {
    final int width = LayoutManager.chooseSize(widthSpec,
            getPaddingLeft() + getPaddingRight(),
            ViewCompat.getMinimumWidth(this));
    final int height = LayoutManager.chooseSize(heightSpec,
            getPaddingTop() + getPaddingBottom(),
            ViewCompat.getMinimumHeight(this));
    setMeasuredDimension(width, height);
}
```
2. onLayot()方法
```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    TraceCompat.beginSection(TRACE_ON_LAYOUT_TAG);
    dispatchLayout();
    TraceCompat.endSection();
    mFirstLayoutComplete = true;
}
```
可以看到onLayout()方法中很简单，逻辑都在dispatchLayout()这个方法中呢。google很喜欢用dispatch这个单词，以后自己也学着用用。
```java 
void dispatchLayout() {
    if (mAdapter == null) {
        return;
    }
    if (mLayout == null) {
        return;
    }
    ...
    dispatchLayoutStep1();
    ...
    dispatchLayoutStep2();
    ...
    dispatchLayoutStep3();
}
```
这个dispatchLayout()方法也比较简单，但是又出现了想onMeasure()方法中的分步骤的三个函数，逻辑可定又被分散到这三个函数中了。第一步和第二部中都调用了mLayout.onLayoutChildren(mRecycler, mState);
第三步中调用了mLayout.onLayoutCompleted(mState);看来RecyclerView的layout实现也是放到了LayoutManager中去实现了。找个LinearLayoutManager看看。onLayoutChildren()方法中有fill(recycler, mLayoutState, state, false);方法发，这个方法是为了时候用Recycler回收器，来确定是否重复利用view.其中有一个 layoutChunk(recycler, state, layoutState, layoutChunkResult)方法，这里边做了实际的addView的操作。layoutChunk()方法中还调用了measureChildWithMargins(view, 0, 0)。这个用来测量子布局的宽高，包括ItemDecor的距离。

3. onDraw()方法
绘制的方法比较简单，这里除了绘制自己之外还把分割线也绘制出来了。
```java
@Override
public void onDraw(Canvas c) {
    super.onDraw(c);

    final int count = mItemDecorations.size();
    for (int i = 0; i < count; i++) {
        mItemDecorations.get(i).onDraw(c, this, mState);
    }
}
```