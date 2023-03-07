# Advanced RecyclerView

## ItemTouchHelper

* This is a utility class to add swipe to dismiss and drag & drop support to `RecyclerView`.

* It works with a `RecyclerView` and a `Callback` class, which configures what type of interactions are enabled and
  also receives events when user performs these actions.

* Depending on which functionality you support, you should override `onMove` and / or `onSwiped`.

* This class is designed to work with any `LayoutManager` but for certain situations, it can be optimized for your
  custom `LayoutManager` by extending methods in the `ItemTouchHelper.Callback` class or implementing
  `ItemTouchHelper.ViewDropHandler` interface in your `LayoutManager`.

* By default, `ItemTouchHelper` moves the items' `translateX`/`Y` properties to reposition them. You can customize
  these behaviors by overriding `onChildDraw` or `onChildDrawOver`.

* Most of the time you only need to override `onChildDraw`.

| Nested types                                                                                                                                                                                                                                   |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `public abstract class ItemTouchHelper.Callback` <br/> This class is the contract between `ItemTouchHelper` and your application.                                                                                                              |
| `public abstract class ItemTouchHelper.SimpleCallback extends ItemTouchHelper.Callback` <br/> A simple wrapper to the default `Callback` which you can construct with drag and swipe directions and this class will handle the flag callbacks. |
| `public interface ItemTouchHelper.ViewDropHandler` <br/> An interface which can be implemented by `LayoutManager` for better integration with `ItemTouchHelper`.                                                                               |                                                                                                                                                                                                     |

| Constants                           |                                                |
|-------------------------------------|------------------------------------------------|
| static final int                    | Rect to receive the output.                    |
| `@NonNull View` view                | The child view to decorate                     |
| `@NonNull RecyclerView` parent      | RecyclerView this ItemDecoration is decorating |
| `@NonNull RecyclerView.State` state | The current state of RecyclerView              |

* **attachToRecyclerView**

```java
public void attachToRecyclerView(@Nullable RecyclerView recyclerView)
```

* Attaches the `ItemTouchHelper` to the provided `RecyclerView`. If `TouchHelper` is already attached to a 
`RecyclerView`, it will first detach from the previous one. You can call this method with null to detach it from the 
current `RecyclerView`.

* **getItemOffsets**

```java
public void getItemOffsets(
    Rect outRect,
    View view,
    RecyclerView parent,
    RecyclerView.State state
)
```

* Retrieve any offsets for the given item. Each field of `outRect` specifies the number of pixels that the item view 
should be inset by, similar to padding or margin. The default implementation sets the bounds of `outRect` to `0` and 
returns.

* If this ItemDecoration does not affect the positioning of item views, it should set all four fields of 
`outRect` (left, top, right, bottom) to zero before returning.

* **onChildViewAttachedToWindow**

```java
public void onChildViewAttachedToWindow(@NonNull View view)
```

* Called when a view is attached to the `RecyclerView`.

* **onChildViewDetachedFromWindow**

```java
public void onChildViewDetachedFromWindow(@NonNull View view)
```

* Called when a view is detached from `RecyclerView`.

* **onDraw**

```java
public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state)
```

* Draw any appropriate decorations into the `Canvas` supplied to the `RecyclerView`. Any content drawn by this method 
will be drawn before the item views are drawn, and will thus appear underneath the views.

* **onDrawOver**

```java
public void onDrawOver(
    @NonNull Canvas c,
    @NonNull RecyclerView parent,
    @NonNull RecyclerView.State state
)
```

* Draw any appropriate decorations into the `Canvas` supplied to the `RecyclerView`. Any content drawn by this method 
will be drawn after the item views are drawn and will thus appear over the views.

* **startDrag**

```java
public void startDrag(@NonNull RecyclerView.ViewHolder viewHolder)
```

* Starts dragging the provided `ViewHolder`. By default, `ItemTouchHelper` starts a drag when a `View` is long pressed. 
You can disable that behavior by overriding `isLongPressDragEnabled`.

* For this method to work:
  * The provided `ViewHolder` must be a child of the `RecyclerView` to which this `ItemTouchHelper` is attached. 
  * `ItemTouchHelper.Callback` must have dragging enabled. 
  * There must be a previous touch event that was reported to the `ItemTouchHelper` through `RecyclerView`'s 
  `ItemTouchListener` mechanism. As long as no other `ItemTouchListener` grabs previous events, this should work as 
  expected.

* For example, if you would like to let your user to be able to drag an Item by touching one of its descendants, you 
may implement it as follows:

```kotlin
    viewHolder.dragButton.setOnTouchListener(new View.OnTouchListener() {
        public boolean onTouch(View v, MotionEvent event) {
            if (MotionEvent.getActionMasked(event) == MotionEvent.ACTION_DOWN) {
                mItemTouchHelper.startDrag(viewHolder);
            }
            return false;
        }
    });
```

* **startSwipe**

```java
public void startSwipe(@NonNull RecyclerView.ViewHolder viewHolder)
```

* Starts swiping the provided `ViewHolder`. By default, `ItemTouchHelper` starts swiping a `View` when user swipes 
their finger (or mouse pointer) over the `View`. You can disable this behavior by overriding `ItemTouchHelper.Callback`.

* For this method to work:
  * The provided `ViewHolder` must be a child of the `RecyclerView` to which this `ItemTouchHelper` is attached. 
  * `ItemTouchHelper.Callback` must have swiping enabled. 
  * There must be a previous touch event that was reported to the `ItemTouchHelper` through `RecyclerView`'s 
  `ItemTouchListener` mechanism. As long as no other `ItemTouchListener` grabs previous events, this should work as 
  expected.

* For example, if you would like to let your user to be able to swipe an Item by touching one of its descendants, you 
may implement it as follows:

```kotlin
    viewHolder.dragButton.setOnTouchListener(new View.OnTouchListener() {
        public boolean onTouch(View v, MotionEvent event) {
            if (MotionEvent.getActionMasked(event) == MotionEvent.ACTION_DOWN) {
                mItemTouchHelper.startSwipe(viewHolder);
            }
            return false;
        }
    });
```

### ItemTouchHelper.Callback

* This class is the contract between `ItemTouchHelper` and your application. It lets you control which touch behaviors 
are enabled per each `ViewHolder` and also receive callbacks when user performs these actions.

* To control which actions user can take on each view, you should override `getMovementFlags` and return appropriate 
set of direction flags. (`LEFT`, `RIGHT`, `START`, `END`, `UP`, `DOWN`). You can use `makeMovementFlags` to easily 
construct it. Alternatively, you can use `SimpleCallback`.

* If user drags an item, `ItemTouchHelper` will call `onMove(recyclerView, dragged, target)`. Upon receiving this 
callback, you should move the item from the old position (`dragged.getAdapterPosition()`) to new position 
(`target.getAdapterPosition()`) in your adapter and also call `notifyItemMoved`. To control where a `View` can be 
dropped, you can override `canDropOver`. When a dragging `View` overlaps multiple other views, `Callback` chooses the 
closest `View` with which dragged `View` might have changed positions. Although this approach works for many use cases, 
if you have a custom `LayoutManager`, you can override `chooseDropTarget` to select a custom drop target.

* When a `View` is swiped, `ItemTouchHelper` animates it until it goes out of bounds, then calls `onSwiped`. At this 
point, you should update your adapter (e.g. remove the item) and call related `Adapter#notify` event.

* **canDropOver**

```java
public boolean canDropOver(
    @NonNull RecyclerView recyclerView,
    @NonNull RecyclerView.ViewHolder current,
    @NonNull RecyclerView.ViewHolder target
)
```

* Return true if the current `ViewHolder` can be dropped over the the target `ViewHolder`.

* This method is used when selecting drop target for the dragged `View`. After Views are eliminated either via bounds 
check or via this method, resulting set of views will be passed to `chooseDropTarget`.

* Default implementation returns `true`.

| Parameters                                 |                                                               |
|--------------------------------------------|---------------------------------------------------------------|
| `@NonNull RecyclerView` recyclerView       | The `RecyclerView` to which `ItemTouchHelper` is attached to. |
| `@NonNull RecyclerView.ViewHolder` current | The `ViewHolder` that user is dragging.                       |
| `@NonNull RecyclerView.ViewHolder` target  | The `ViewHolder` which is below the dragged ViewHolder.       |

| Returns |                                                                                                 |
|---------|-------------------------------------------------------------------------------------------------|
| boolean | True if the dragged `ViewHolder` can be replaced with the target `ViewHolder`, false otherwise. |

* Called by `ItemTouchHelper` to select a drop target from the list of `ViewHolders` that are under the dragged `View`.

* Default implementation filters the `View` with which dragged item have changed position in the drag direction. For 
instance, if the view is dragged UP, it compares the `view.getTop()` of the two views before and after drag started. If 
that value is different, the target view passes the filter.

* Among these Views which pass the test, the one closest to the dragged view is chosen.

* This method is called on the main thread every time user moves the `View`. If you want to override it, make sure it 
does not do any expensive operations.

| Parameters                                           |                                                                                                                                                                |
|------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `@NonNull RecyclerView.ViewHolder` selected          | The `ViewHolder` being dragged by the user.                                                                                                                    |
| `@NonNull List<RecyclerView.ViewHolder>` dropTargets | The list of `ViewHolder` that are under the dragged `View` and candidate as a drop.                                                                            |
| int curX                                             | The updated left value of the dragged `View` after drag translations are applied. This value does not include margins added by `RecyclerView.ItemDecorations`. |
| int curY                                             | The updated top value of the dragged `View` after drag translations are applied. This value does not include margins added by `RecyclerView.ItemDecorations`.  |

| Returns                   |                                                                               |
|---------------------------|-------------------------------------------------------------------------------|
| `RecyclerView.ViewHolder` | A `ViewHolder` to whose position the dragged `ViewHolder` should be moved to. |

* **clearView**

```java
public void clearView(
    @NonNull RecyclerView recyclerView,
    @NonNull RecyclerView.ViewHolder viewHolder
)
```

* Called by the `ItemTouchHelper` when the user interaction with an element is over and it also completed its animation.

* This is a good place to clear all changes on the View that was done in `onSelectedChanged`, `onChildDraw` or 
`onChildDrawOver`.

| Parameters                                    |                                                                  |
|-----------------------------------------------|------------------------------------------------------------------|
| `@NonNull RecyclerView` recyclerView          | The `RecyclerView` which is controlled by the `ItemTouchHelper`. |
| `@NonNull RecyclerView.ViewHolder` viewHolder | The `View` that was interacted by the user.                      |

* **convertToAbsoluteDirection**

```java
public int convertToAbsoluteDirection(int flags, int layoutDirection)
```

* Converts a given set of flags to absolution direction which means `START` and `END` are replaced with `LEFT` and 
`RIGHT` depending on the layout direction.

| Parameters          |                                                           |
|---------------------|-----------------------------------------------------------|
| int flags           | The flag value that include any number of movement flags. |
| int layoutDirection | The layout direction of the RecyclerView.                 |

| Returns   |                                                              |
|-----------|--------------------------------------------------------------|
| int flags | Updated flags which includes only absolute direction values. |

* **convertToRelativeDirection**

```java
public static int convertToRelativeDirection(int flags, int layoutDirection)
```

* Replaces a movement direction with its relative version by taking layout direction into account.

| Parameters          |                                                                            |
|---------------------|----------------------------------------------------------------------------|
| int flags           | The flag value that include any number of movement flags.                  |
| int layoutDirection | The layout direction of the `View`. Can be obtained from `getLayoutDirection`. |

| Returns |                                                                                      |
|---------|--------------------------------------------------------------------------------------|
| int     | Updated flags which uses relative flags (`START`, `END`) instead of `LEFT`, `RIGHT`. |

* **getAnimationDuration**

```java
public long getAnimationDuration(
    @NonNull RecyclerView recyclerView,
    int animationType,
    float animateDx,
    float animateDy
)
```

* Called by the `ItemTouchHelper` when user action finished on a `ViewHolder` and now the `View` will be animated to 
its final position.

* Default implementation uses `ItemAnimator`'s duration values. If animationType is `ANIMATION_TYPE_DRAG`, it returns 
`getMoveDuration`, otherwise, it returns `getRemoveDuration`. If `RecyclerView` does not have any 
`RecyclerView.ItemAnimator` attached, this method returns `DEFAULT_DRAG_ANIMATION_DURATION` or 
`DEFAULT_SWIPE_ANIMATION_DURATION` depending on the animation type.

| Parameters                         |                                                                                                                          |
|------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| @NonNull RecyclerView recyclerView | The `RecyclerView` to which the `ItemTouchHelper` is attached to.                                                        |
| int animationType                  | The type of animation. Is one of `ANIMATION_TYPE_DRAG`, `ANIMATION_TYPE_SWIPE_CANCEL` or `ANIMATION_TYPE_SWIPE_SUCCESS`. |
| float animateDx                    | The horizontal distance that the animation will offset                                                                   |
| float animateDy                    | The vertical distance that the animation will offset                                                                     |

| Returns |                                |
|---------|--------------------------------|
| long    | The duration for the animation |

* **getBoundingBoxMargin**

```java
public int getBoundingBoxMargin()
```

* When finding views under a dragged view, by default, `ItemTouchHelper` searches for views that overlap with the 
dragged View. By overriding this method, you can extend or shrink the search box.

| Returns |                                                                    |
|---------|--------------------------------------------------------------------|
| long    | The extra margin to be added to the hit box of the dragged `View`. |

* **getDefaultUIUtil**

```java
public static @NonNull ItemTouchUIUtil getDefaultUIUtil()
```

* Returns the `ItemTouchUIUtil` that is used by the `Callback` class for visual changes on Views in response to user 
interactions. `ItemTouchUIUtil` has different implementations for different platform versions.

* By default, `Callback` applies these changes on `itemView`.

```java
    public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder){
        getDefaultUIUtil().clearView(((ItemTouchViewHolder) viewHolder).textView);
    }
    public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
        if (viewHolder != null){
            getDefaultUIUtil().onSelected(((ItemTouchViewHolder) viewHolder).textView);
        }
    }
    public void onChildDraw(Canvas c, RecyclerView recyclerView,
            RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState,
            boolean isCurrentlyActive) {
        getDefaultUIUtil().onDraw(c, recyclerView,
                ((ItemTouchViewHolder) viewHolder).textView, dX, dY,
                actionState, isCurrentlyActive);
        return true;
    }
    public void onChildDrawOver(Canvas c, RecyclerView recyclerView,
            RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState,
            boolean isCurrentlyActive) {
        getDefaultUIUtil().onDrawOver(c, recyclerView,
                ((ItemTouchViewHolder) viewHolder).textView, dX, dY,
                actionState, isCurrentlyActive);
        return true;
    }
```

* **getMoveThreshold**

```java
public float getMoveThreshold(@NonNull RecyclerView.ViewHolder viewHolder)
```

* Returns the fraction that the user should move the `View` to be considered as it is dragged. After a view is moved 
this amount, `ItemTouchHelper` starts checking for Views below it for a possible drop.

* **getMovementFlags**

```java
public abstract int getMovementFlags(
    @NonNull RecyclerView recyclerView,
    @NonNull RecyclerView.ViewHolder viewHolder
)
```

* Should return a composite flag which defines the enabled move directions in each state (idle, swiping, dragging).

| Parameters                                  |                                                                 |
|---------------------------------------------|-----------------------------------------------------------------|
| @NonNull RecyclerView recyclerView          | The `RecyclerView` to which the `ItemTouchHelper` is attached.  |
| @NonNull RecyclerView.ViewHolder viewHolder | The ViewHolder for which the movement information is necessary. |

| Returns |                                                                    |
|---------|--------------------------------------------------------------------|
| int     | flags specifying which movements are allowed on this `ViewHolder`. |

* **getSwipeEscapeVelocity**

```java
public float getSwipeEscapeVelocity(float defaultValue)
```

* Defines the minimum velocity which will be considered as a swipe action by the user.

| Parameters         |                                                                         |
|--------------------|-------------------------------------------------------------------------|
| float defaultValue | The default value (in pixels per second) used by the `ItemTouchHelper`. |

| Returns |                                                                                              |
|---------|----------------------------------------------------------------------------------------------|
| float   | The minimum swipe velocity. The default implementation returns the `defaultValue` parameter. |

* **getSwipeThreshold**

```java
public float getSwipeThreshold(@NonNull RecyclerView.ViewHolder viewHolder)
```

* Returns the fraction that the user should move the View to be considered as swiped. The fraction is calculated with 
respect to `RecyclerView`'s bounds.

| Parameters                                    |                                         |
|-----------------------------------------------|-----------------------------------------|
| `@NonNull RecyclerView.ViewHolder` viewHolder | The `ViewHolder` that is being dragged. |

| Returns |                                                                                  |
|---------|----------------------------------------------------------------------------------|
| float   | A float value that denotes the fraction of the View size. Default value is .5f . |

* **getSwipeVelocityThreshold**

```java
public float getSwipeVelocityThreshold(float defaultValue)
```

* Defines the maximum velocity `ItemTouchHelper` will ever calculate for pointer movements.

| Parameters         |                                                                        |
|--------------------|------------------------------------------------------------------------|
| float defaultValue | The default value(in pixels per second) used by the `ItemTouchHelper`. |

| Returns |                                                                                                          |
|---------|----------------------------------------------------------------------------------------------------------|
| float   | The velocity cap for pointer movements. The default implementation returns the `defaultValue` parameter. |

### ItemTouchHelper.SimpleCallback

* A simple wrapper to the default `Callback` which you can construct with drag and swipe directions and this class will 
handle the flag callbacks. You should still override `onMove` or `onSwiped` depending on your use case.

```java
ItemTouchHelper mIth = new ItemTouchHelper(
    new ItemTouchHelper.SimpleCallback(ItemTouchHelper.UP | ItemTouchHelper.DOWN,
        ItemTouchHelper.LEFT) {
        public boolean onMove(RecyclerView recyclerView,
            ViewHolder viewHolder, ViewHolder target) {
            final int fromPos = viewHolder.getAdapterPosition();
            final int toPos = target.getAdapterPosition();
            // move item in `fromPos` to `toPos` in adapter.
            return true;// true if moved, false otherwise
        }
        public void onSwiped(ViewHolder viewHolder, int direction) {
            // remove from adapter
        }
});
```

| Public methods |                                                                                                                                                                                                                               |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| int            | `getDragDirs( @NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder )` <br/> Returns the drag directions for the provided `ViewHolder`.                                                             |
| int            | `getMovementFlags( @NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder )` <br/> Should return a composite flag which defines the enabled move directions in each state (idle, swiping, dragging). |
| int            | `getSwipeDirs( @NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder )` <br/> Returns the swipe directions for the provided `ViewHolder`.                                                           |
| void           | `setDefaultDragDirs(int defaultDragDirs)` <br/> Updates the default drag directions.                                                                                                                                          |
| void           | `setDefaultSwipeDirs(int defaultSwipeDirs)` <br/> Updates the default swipe directions.                                                                                                                                       |






















