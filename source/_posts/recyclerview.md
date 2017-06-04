---
title: recyclerview
date: 2017-06-03 16:20:02
categories: android
tags: recyclerview
---

## 如何使用 RecyclerView

首先是recyclerview的配置问题，recyclerview是要导入android.support.v7.widget.RecyclerView的包。
还要在Gradle Scripts中添加 compile ‘com.android.support:recyclerview-v7:23.4.0’，然后同步。（23.4.0是版本号记得改成自己有的）

<!-- more -->

### 3种布局
``` bash
LinearLayoutManager ： 
LinearLayoutManager(Context context, int orientation, boolean reverseLayout)
很浅显线性布局，水平或者垂直，第三个参数的意思是：是否倒置数据，就是数据源最后面的显示在第一个，倒数第二显示在第二个

GridLayoutManager :
GridLayoutManager(Context context, int spanCount, int orientation,boolean reverseLayout)

StaggeredGridLayoutManager:
StaggeredGridLayoutManager(int spanCount, int orientation)
```

### 点击事件的实现（有两种）

``` bash
@Override
    public void onBindViewHolder(final MyViewHolder holder, int position) {
        loadThumb(position, holder);

        if (mOnItemLitener != null) {
            holder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    int pos = holder.getLayoutPosition();
                    mOnItemLitener.onItemClick(holder.itemView, pos);
                }
            });
           
            holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    int pos = holder.getLayoutPosition();
                    mOnItemLitener.onItemLongClick(holder.itemView, pos);
                    return false;
                }
            });
        }
    }
	
	 public interface OnItemLitener {
        void onItemClick(View view, int position);

        void onItemLongClick(View view, int position);
    }

    private OnItemLitener mOnItemLitener;

    public void setOnItemLitener(OnItemLitener mOnItemLitener) {
        this.mOnItemLitener = mOnItemLitener;
    }
```
另外一种通过addOnItemTouchListener自己去实现计算位置的逻辑

### ItemDecoration 间隔线 
``` bash
public class DividerItemDecoration extends RecyclerView.ItemDecoration {

    private static final int[] ATTRS = new int[]{android.R.attr.listDivider};
    private Drawable mDivider;

    public DividerItemDecoration(Context context) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        a.recycle();
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
        drawHorizontal(c, parent);
        drawVertical(c, parent);
    }

    public int getSpanCount(RecyclerView parent) {
        // 列数
        int spanCount = -1;
        LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {

            spanCount = ((GridLayoutManager) layoutManager).getSpanCount();
        } else if (layoutManager instanceof StaggeredGridLayoutManager) {
            spanCount = ((StaggeredGridLayoutManager) layoutManager)
                    .getSpanCount();
        }
        return spanCount;
    }

    public void drawHorizontal(Canvas c, RecyclerView parent) {
        int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int left = child.getLeft() - params.leftMargin;
            final int right = child.getRight() + params.rightMargin
                    + mDivider.getIntrinsicWidth();
            final int top = child.getBottom() + params.bottomMargin;
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    public void drawVertical(Canvas c, RecyclerView parent) {
        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);

            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int top = child.getTop() - params.topMargin;
            final int bottom = child.getBottom() + params.bottomMargin;
            final int left = child.getRight() + params.rightMargin;
            final int right = left + mDivider.getIntrinsicWidth();

            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    public boolean isFirstColum(RecyclerView parent, int pos, int spanCount, int childCount) { // 注意 StaggeredGridLayoutManager
        LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            if ((pos + 1) % spanCount == 1) {
                return true;
            }
        } else if (layoutManager instanceof StaggeredGridLayoutManager) {
            int orientation = ((StaggeredGridLayoutManager) layoutManager).getOrientation();
            if (orientation == StaggeredGridLayoutManager.VERTICAL) {

            } else {

            }
        }
        return false;
    }

    public boolean isLastColum(RecyclerView parent, int pos, int spanCount, int childCount) {
        LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            if ((pos + 1) % spanCount == 0)// 如果是最后一列，则不需要绘制右边
            {
                return true;
            }
        } else if (layoutManager instanceof StaggeredGridLayoutManager) {
            int orientation = ((StaggeredGridLayoutManager) layoutManager)
                    .getOrientation();
            if (orientation == StaggeredGridLayoutManager.VERTICAL) {
                if ((pos + 1) % spanCount == 0)// 如果是最后一列，则不需要绘制右边
                {
                    return true;
                }
            } else {
                childCount = childCount - childCount % spanCount;
                if (pos >= childCount)// 如果是最后一列，则不需要绘制右边
                    return true;
            }
        }
        return false;
    }

    public boolean isFirstRaw(RecyclerView parent, int pos, int spanCount, int childCount) {  // 注意 StaggeredGridLayoutManager

        LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            if (pos < spanCount) {
                return true;
            }
        } else if (layoutManager instanceof StaggeredGridLayoutManager) {
            int orientation = ((StaggeredGridLayoutManager) layoutManager).getOrientation();
            if (orientation == StaggeredGridLayoutManager.VERTICAL) {

            } else {

            }
        }
        return false;
    }

    public boolean isLastRaw(RecyclerView parent, int pos, int spanCount, int childCount) {
        LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            childCount = childCount - childCount % spanCount;
            if (pos >= childCount)// 如果是最后一行，则不需要绘制底部
                return true;
        } else if (layoutManager instanceof StaggeredGridLayoutManager) {
            int orientation = ((StaggeredGridLayoutManager) layoutManager).getOrientation();
            // StaggeredGridLayoutManager 且纵向滚动
            if (orientation == StaggeredGridLayoutManager.VERTICAL) {
                childCount = childCount - childCount % spanCount;
                // 如果是最后一行，则不需要绘制底部
                if (pos >= childCount)
                    return true;
            } else
            // StaggeredGridLayoutManager 且横向滚动
            {
                // 如果是最后一行，则不需要绘制底部
                if ((pos + 1) % spanCount == 0) {
                    return true;
                }
            }
        }
        return false;
    }

	// 主要的绘制逻辑在这里
    @Override
    public void getItemOffsets(Rect outRect, int itemPosition,
                               RecyclerView parent) {
        int spanCount = getSpanCount(parent);
        int childCount = parent.getAdapter().getItemCount();
        if (isLastRaw(parent, itemPosition, spanCount, childCount))// 如果是最后一行，则不需要绘制底部
        {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        } else if (isLastColum(parent, itemPosition, spanCount, childCount))// 如果是最后一列，则不需要绘制右边
        {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(),
                    mDivider.getIntrinsicHeight());
        }
    }
}
```

###  ItemTouchHelper.Callback 长按拖拽功能，滑动功能 
``` bash
 public ItemDragHelperCallback callback;

    public void mAddItemDragHelperCallback() {

        callback = new ItemDragHelperCallback(mMyRecyclerViewAdapter);
        callback.setOptions(true, true);
        ItemTouchHelper helper = new ItemTouchHelper(callback);
        helper.attachToRecyclerView(mRecyclerView);

    }
````
ItemDragHelperCallback 的代码
``` bash
class ItemDragHelperCallback extends ItemTouchHelper.Callback {

    private RecyclerView.Adapter mMyRecyclerViewAdapter;

    public ItemDragHelperCallback(RecyclerView.Adapter adapter) {
        mMyRecyclerViewAdapter = adapter;
    }

	// 这里控制了拽拉拖动的实现 
    @Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        int dragFlags, swipeFlags;
        RecyclerView.LayoutManager manager = recyclerView.getLayoutManager();
        if (manager instanceof GridLayoutManager || manager instanceof StaggeredGridLayoutManager) {
            dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN | ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
            swipeFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN | ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
        } else {
            if (((LinearLayoutManager) manager).getOrientation() == LinearLayoutManager.HORIZONTAL) {
                dragFlags = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
                swipeFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
            } else {
                dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
                swipeFlags = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
            }
        }

        return makeMovementFlags(dragFlags, swipeFlags);
    }

    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
        // 不同Type之间不可移动
        if (viewHolder.getItemViewType() != target.getItemViewType()) {
            return false;
        }
        if (mMyRecyclerViewAdapter instanceof OnItemTouchListener) {
            OnItemTouchListener listener = ((OnItemTouchListener) mMyRecyclerViewAdapter);
            listener.onItemMove(viewHolder.getAdapterPosition(), target.getAdapterPosition());
        }
        return true;
    }

    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
        if (mMyRecyclerViewAdapter instanceof OnItemTouchListener) {
            OnItemTouchListener listener = ((OnItemTouchListener) mMyRecyclerViewAdapter);
            listener.onItemDelete(viewHolder.getAdapterPosition());
        }
    }

    @Override
    public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
        // 不在闲置状态
        if (actionState != ItemTouchHelper.ACTION_STATE_IDLE) {
            if (viewHolder instanceof OnDragVHListener) {
                OnDragVHListener itemViewHolder = (OnDragVHListener) viewHolder;
                itemViewHolder.onItemSelected();
            }
        }
        super.onSelectedChanged(viewHolder, actionState);
    }

    @Override
    public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        if (viewHolder instanceof OnDragVHListener) {
            OnDragVHListener itemViewHolder = (OnDragVHListener) viewHolder;
            itemViewHolder.onItemFinish();
        }
        super.clearView(recyclerView, viewHolder);
    }

    @Override
    public boolean isLongPressDragEnabled() {
        // 长按拖拽功能
        return isLongPressDragEnabled;
    }

    @Override
    public boolean isItemViewSwipeEnabled() {
        // 滑动功能
        return isItemViewSwipeEnabled;
    }

    public boolean isLongPressDragEnabled, isItemViewSwipeEnabled;

    public void setOptions(boolean isLongPressDragEnabled, boolean isItemViewSwipeEnabled) {
        this.isLongPressDragEnabled = isLongPressDragEnabled;
        this.isItemViewSwipeEnabled = isItemViewSwipeEnabled;
    }

    interface OnDragVHListener {

        // Item被选中时触发
        void onItemSelected();

        // Item在拖拽结束/滑动结束后触发
        void onItemFinish();
    }

    interface OnItemTouchListener {
        void onItemMove(int fromPosition, int toPosition);

        void onItemDelete(int position);
    }
}
```
adapter 相应的实现,要implements ItemDragHelperCallback.OnItemTouchListener，
ViewHolder 要 implements ItemDragHelperCallback.OnDragVHListener
``` bash
  @Override
    public void onItemMove(int fromPosition, int toPosition) {
        Object item = m_itemInfos.get(fromPosition);
        m_itemInfos.remove(fromPosition);
        m_itemInfos.add(toPosition, item);
        notifyItemMoved(fromPosition, toPosition);
    }

    @Override
    public void onItemDelete(int position) {
        removeItem(position);
    }
```
