# Advanced RecyclerView

## RecyclerView.ItemDecoration

* An `ItemDecoration` allows the application to add a special drawing and layout offset to specific item views from the
  adapter's data set. This can be useful for drawing dividers between items, highlights, visual grouping boundaries and
  more.

* All `ItemDecorations` are drawn in the order they were added, before the item views in `onDraw()` and after the items
  in `onDrawOver(Canvas, RecyclerView, RecyclerView.State)`.

* **getItemOffsets**

```java
public void getItemOffsets(
    @NonNull Rect outRect,
    @NonNull View view,
    @NonNull RecyclerView parent,
    @NonNull RecyclerView.State state
)
```

* Retrieve any offsets for the given item. Each field of `outRect` specifies the number of pixels that the item view
  should be inset by, similar to padding or margin. The default implementation sets the bounds of outRect to 0 and
  returns.

* If this `ItemDecoration` does not affect the positioning of item views, it should set all four fields of `outRect`
  (left, top, right, bottom) to zero before returning.

| Parameters                          |                                                |
|-------------------------------------|------------------------------------------------|
| `@NonNull Rect` outRect             | Rect to receive the output.                    |
| `@NonNull View` view                | The child view to decorate                     |
| `@NonNull RecyclerView` parent      | RecyclerView this ItemDecoration is decorating |
| `@NonNull RecyclerView.State` state | The current state of RecyclerView              |

* **onDraw**

```java
public void onDraw(
    @NonNull Canvas c,
    @NonNull RecyclerView parent,
    @NonNull RecyclerView.State state
)
```

* Draw any appropriate decorations into the `Canvas` supplied to the `RecyclerView`. Any content drawn by this method 
will be drawn before the item views are drawn, and will thus appear underneath the views.

| Parameters                          |                                                  |
|-------------------------------------|--------------------------------------------------|
| `@NonNull Canvas` c                 | Canvas to draw into                              |
| `@NonNull RecyclerView` parent      | RecyclerView this ItemDecoration is drawing into |
| `@NonNull RecyclerView.State` state | The current state of RecyclerView                |

* **onDrawOver**

* Draw any appropriate decorations into the `Canvas` supplied to the `RecyclerView`. Any content drawn by this method 
will be drawn after the item views are drawn and will thus appear over the views.


| Parameters                          |                                                  |
|-------------------------------------|--------------------------------------------------|
| `@NonNull Canvas` c                 | Canvas to draw into                              |
| `@NonNull RecyclerView` parent      | RecyclerView this ItemDecoration is drawing into |
| `@NonNull RecyclerView.State` state | The current state of RecyclerView                |

---

## DiffUtil

* `DiffUtil` is a utility class that calculates the difference between two lists and outputs a list of update 
operations that converts the first list into the second one.

* It can be used to calculate updates for a `RecyclerView` `Adapter`. See `ListAdapter` and `AsyncListDiffer` which can 
simplify the use of `DiffUtil` on a **background thread**.

* `DiffUtil`'s algorithm does not handle items that are moved so `DiffUtil` runs a second pass on the result to detect 
items that were moved.

* Note that `DiffUtil`, `ListAdapter`, and `AsyncListDiffer` require the list to not mutate while in use. This 
generally means that both the lists themselves and their elements (or at least, the properties of elements used in 
diffing) should not be modified directly. Instead, new lists should be provided any time content changes.

* If the lists are large, this operation may take significant time so you are advised to run this on a background 
thread, get the `DiffResult` then apply it on the `RecyclerView` on the main thread.

* This algorithm is optimized for space and uses `O(N)` space to find the minimal number of addition and removal 
operations between the two lists. It has `O(N + D^2)` expected time performance where `D` is the length of the edit 
script.

* If move detection is enabled, it takes an additional `O(MN)` time where `M` is the total number of added items and 
`N` is the total number of removed items. If your lists are already sorted by the same constraint (e.g. a created 
timestamp for a list of posts), you can disable move detection to improve performance.

* **calculateDiff**

```java
public static @NonNull DiffUtil.DiffResult calculateDiff(@NonNull DiffUtil.Callback cb, boolean detectMoves)
```

* Calculates the list of update operations that can covert one list into the other one.

* If your old and new lists are sorted by the same constraint and items never move (swap positions), you can disable move 
detection which takes `O(N^2)` time where `N` is the number of added, moved, removed items. In 
`calculateDiff(@NonNull DiffUtil.Callback cb)` by default is true.

| Parameters                      |                                                                       |
|---------------------------------|-----------------------------------------------------------------------|
| `@NonNull DiffUtil.Callback` cb | The callback that acts as a gateway to the backing list data          |
| boolean detectMoves             | True if `DiffUtil` should try to detect moved items, false otherwise. |


| Returns                        |                                                                                                                 |
|--------------------------------|-----------------------------------------------------------------------------------------------------------------|
| `@NonNull DiffUtil.DiffResult` | A `DiffResult` that contains the information about the edit sequence to convert the old list into the new list. |

### DiffUtil.Callback

* A `Callback` class used by `DiffUtil` while calculating the diff between two lists.

* **areContentsTheSame**

```java
public abstract boolean areContentsTheSame(int oldItemPosition, int newItemPosition)
```

* Called by the `DiffUtil` when it wants to check whether two items have the same data. `DiffUtil` uses this 
information to detect if the contents of an item has changed.

* `DiffUtil` uses this method to check equality instead of `equals` so that you can change its behavior depending on 
your UI. For example, if you are using `DiffUtil` with a `RecyclerView.Adapter`, you should return whether the items' 
visual representations are the same.

* This method is called only if `areItemsTheSame` returns `true` for these items.

| Parameters          |                                                                     |
|---------------------|---------------------------------------------------------------------|
| int oldItemPosition | The position of the item in the old list                            |
| int newItemPosition | The position of the item in the new list which replaces the oldItem |


| Returns |                                                                                |
|---------|--------------------------------------------------------------------------------|
| boolean | True if the contents of the items are the same or false if they are different. |

* **areItemsTheSame**

```java
public abstract boolean areItemsTheSame(int oldItemPosition, int newItemPosition)
```

* Called by the `DiffUtil` to decide whether two object represent the same Item.

* For example, if your items have unique ids, this method should check their id equality.

| Parameters          |                                          |
|---------------------|------------------------------------------|
| int oldItemPosition | The position of the item in the old list |
| int newItemPosition | The position of the item in the new list |

| Returns |                                                                                |
|---------|--------------------------------------------------------------------------------|
| boolean | True if the contents of the items are the same or false if they are different. |

* **getChangePayload**

```java
public @Nullable Object getChangePayload(int oldItemPosition, int newItemPosition)
```

* When `areItemsTheSame` returns `true` for two items and `areContentsTheSame` returns `false` for them, `DiffUtil` 
calls this method to get a payload about the change.

* For example, if you are using `DiffUtil` with `RecyclerView`, you can return the particular field that changed in the 
item and your `ItemAnimator` can use that information to run the correct animation.

* Default implementation returns `null`.

| Parameters          |                                          |
|---------------------|------------------------------------------|
| int oldItemPosition | The position of the item in the old list |
| int newItemPosition | The position of the item in the new list |

| Returns            |                                                                    |
|--------------------|--------------------------------------------------------------------|
| `@Nullable Object` | A payload object that represents the change between the two items. |

* **getNewListSize**

```java
public abstract int getNewListSize()
```

* Returns the size of the new list.

| Returns |                           |
|---------|---------------------------|
| int     | The size of the new list. |

* **getOldListSize**

```java
public abstract int getOldListSize()
```

* Returns the size of the old list.

| Returns |                           |
|---------|---------------------------|
| int     | The size of the old list. |

### DiffUtil.DiffResult

* This class holds the information about the result of a `calculateDiff` call.

* You can consume the updates in a `DiffResult` via `dispatchUpdatesTo` or directly stream the results into a 
`RecyclerView.Adapter` via `dispatchUpdatesTo`.

* **NO_POSITION**

```java
public static final int NO_POSITION = -1
```

* Signifies an item not present in the list.

* **convertNewPositionToOld**

```java
public int convertNewPositionToOld(@IntRange(from = 0) int newListPosition)
```

* Given a position in the new list, returns the position in the old list, or `NO_POSITION` if it was removed.


| Parameters                                |                              |
|-------------------------------------------|------------------------------|
| `@IntRange(from = 0)` int newListPosition | Position of item in new list |

| Returns |                                                                |
|---------|----------------------------------------------------------------|
| int     | Position of item in old list, or `NO_POSITION` if not present. |

* **convertOldPositionToNew**

```java
public int convertOldPositionToNew(@IntRange(from = 0) int oldListPosition)
```

* Given a position in the old list, returns the position in the new list, or `NO_POSITION` if it was removed.

| Parameters                                |                              |
|-------------------------------------------|------------------------------|
| `@IntRange(from = 0)` int oldListPosition | Position of item in old list |

| Returns |                                                                |
|---------|----------------------------------------------------------------|
| int     | Position of item in new list, or `NO_POSITION` if not present. |

* **dispatchUpdatesTo**

```java
public void dispatchUpdatesTo(@NonNull ListUpdateCallback updateCallback)
```

* Dispatches update operations to the given Callback.

* These updates are atomic such that the first update call affects every update call that comes after it (the same 
as `RecyclerView`).

| Parameters                                   |                                                |
|----------------------------------------------|------------------------------------------------|
| `@NonNull ListUpdateCallback` updateCallback | The callback to receive the update operations. |

* **dispatchUpdatesTo**

```java
public void dispatchUpdatesTo(@NonNull RecyclerView.Adapter adapter)
```

* Dispatches the update events to the given adapter.

* For example, if you have an `Adapter` that is backed by a `List`, you can swap the list with the new one then call 
this method to dispatch all updates to the `RecyclerView`.

```java
List oldList = mAdapter.getData();
DiffResult result = DiffUtil.calculateDiff(new MyCallback(oldList, newList)); 
mAdapter.setData(newList); 
result.dispatchUpdatesTo(mAdapter);
```

* Note that the `RecyclerView` requires you to dispatch adapter updates immediately when you change the data (you 
cannot defer `notify*` calls). The usage above adheres to this rule because updates are sent to the adapter right after 
the backing data is changed, before `RecyclerView` tries to read it.

* On the other hand, if you have another `AdapterDataObserver` that tries to process events synchronously, this may 
confuse that observer because the list is instantly moved to its final state while the adapter updates are dispatched 
later on, one by one. If you have such an `AdapterDataObserver`, you can use `dispatchUpdatesTo` to handle each 
modification manually.

| Parameters                              |                                                                                                  |
|-----------------------------------------|--------------------------------------------------------------------------------------------------|
| `@NonNull RecyclerView.Adapter` adapter | A RecyclerView adapter which was displaying the old list and will start displaying the new list. |

### DiffUtil.ItemCallback

```java
public abstract class DiffUtil.ItemCallback<T>
```

* Callback for calculating the diff between two non-null items in a list.

* `Callback` serves two roles - list indexing, and item diffing. `ItemCallback` handles just the second of these, which 
allows separation of code that indexes into an array or List from the presentation-layer and content specific diffing 
code.

* **areContentsTheSame**

* Called to check whether two items have the same data.

* This information is used to detect if the contents of an item have changed.

* This method to check equality instead of `equals` so that you can change its behavior depending on your UI.

* For example, if you are using `DiffUtil` with a `RecyclerView.Adapter`, you should return whether the items' visual 
representations are the same.

* This method is called only if `areItemsTheSame` returns `true` for these items.

> **Note**: Two `null` items are assumed to represent the same contents. This callback will not be invoked for this case

| Parameters           |                           |
|----------------------|---------------------------|
| `@NonNull T` oldItem | The item in the old list. |
| `@NonNull T` newItem | The item in the new list. |

| Returns |                                                                                |
|---------|--------------------------------------------------------------------------------|
| boolean | True if the contents of the items are the same or false if they are different. |

* **areItemsTheSame**

```java
public abstract boolean areItemsTheSame(@NonNull T oldItem, @NonNull T newItem)
```

* Called to check whether two objects represent the same item. 

* For example, if your items have unique ids, this method should check their id equality. 

> **Note**: `null` items in the list are assumed to be the same as another `null` item and are assumed to not be the 
same as a non-`null` item. This callback will not be invoked for either of those cases.

| Parameters           |                           |
|----------------------|---------------------------|
| `@NonNull T` oldItem | The item in the old list. |
| `@NonNull T` newItem | The item in the new list. |

| Returns |                                                                                |
|---------|--------------------------------------------------------------------------------|
| boolean | True if the contents of the items are the same or false if they are different. |

* **getChangePayload**

```java
public @Nullable Object getChangePayload(@NonNull T oldItem, @NonNull T newItem)
```

* When `areItemsTheSame` returns `true` for two items and `areContentsTheSame` returns `false` for them, this method is 
called to get a payload about the change.

* For example, if you are using `DiffUtil` with `RecyclerView`, you can return the particular field that changed in the 
item and your `ItemAnimator` can use that information to run the correct animation.

* Default implementation returns `null`.

---

## AsyncListDiffer

* Helper for computing the difference between two lists via `DiffUtil` on a background thread.

* It can be connected to a `RecyclerView.Adapter`, and will signal the adapter of changes between sumbitted lists.

* For simplicity, the `ListAdapter` wrapper class can often be used instead of the `AsyncListDiffer` directly. This 
`AsyncListDiffer` can be used for complex cases, where overriding an adapter base class to support asynchronous `List` 
diffing isn't convenient.

* The `AsyncListDiffer` can consume the values from a `LiveData` of `List` and present the data simply for an adapter. 
It computes differences in list contents via `DiffUtil` on a background thread as new `Lists` are received.

* Use `getCurrentList` to access the current `List`, and present its data objects. Diff results will be dispatched to 
the `ListUpdateCallback` immediately before the current list is updated. If you're dispatching list updates directly to 
an `Adapter`, this means the `Adapter` can safely access list items and total size via `getCurrentList`.

* A complete usage pattern with Room would look like this:

```java
@Dao
interface UserDao {
    @Query("SELECT * FROM user ORDER BY lastName ASC")
    public abstract LiveData<List<User>> usersByLastName();
}

class MyViewModel extends ViewModel {
    public final LiveData<List<User>> usersList;
    public MyViewModel(UserDao userDao) {
        usersList = userDao.usersByLastName();
    }
}

class MyActivity extends AppCompatActivity {
    @Override
    public void onCreate(Bundle savedState) {
        super.onCreate(savedState);
        MyViewModel viewModel = new ViewModelProvider(this).get(MyViewModel.class);
        RecyclerView recyclerView = findViewById(R.id.user_list);
        UserAdapter adapter = new UserAdapter();
        viewModel.usersList.observe(this, list -> adapter.submitList(list));
        recyclerView.setAdapter(adapter);
    }
}

class UserAdapter extends RecyclerView.Adapter<UserViewHolder> {
    private final AsyncListDiffer<User> mDiffer = new AsyncListDiffer(this, DIFF_CALLBACK);
    @Override
    public int getItemCount() {
        return mDiffer.getCurrentList().size();
    }
    public void submitList(List<User> list) {
        mDiffer.submitList(list);
    }
    @Override
    public void onBindViewHolder(UserViewHolder holder, int position) {
        User user = mDiffer.getCurrentList().get(position);
        holder.bindTo(user);
    }
    public static final DiffUtil.ItemCallback<User> DIFF_CALLBACK
            = new DiffUtil.ItemCallback<User>() {
        @Override
        public boolean areItemsTheSame(
                @NonNull User oldUser, @NonNull User newUser) {
            // User properties may have changed if reloaded from the DB, but ID is fixed
            return oldUser.getId() == newUser.getId();
        }
        @Override
        public boolean areContentsTheSame(
                @NonNull User oldUser, @NonNull User newUser) {
            // NOTE: if you use equals, your object must properly override Object#equals()
            // Incorrectly returning false here will result in too many animations.
            return oldUser.equals(newUser);
        }
    }
}
```

* **AsyncListDiffer**

| Parameters                                       |                                                                          |
|--------------------------------------------------|--------------------------------------------------------------------------|
| `@NonNull RecyclerView.Adapter` adapter          | Adapter to dispatch position updates to.                                 |
| `@NonNull DiffUtil.ItemCallback<T>` diffCallback | ItemCallback that compares items to dispatch appropriate animations when |

* **addListListener**

```java
public void addListListener(@NonNull AsyncListDiffer.ListListener<T> listener)
```

* Add a `ListListener` to receive updates when the current `List` changes.

| Parameters                                          |                              |
|-----------------------------------------------------|------------------------------|
| `@NonNull AsyncListDiffer.ListListener<T>` listener | Listener to receive updates. |

* **getCurrentList**

```java
public @NonNull List<T> getCurrentList()
```

* Get the current `List` - any diffing to present this list has already been computed and dispatched via the 
`ListUpdateCallback`.

* If a `null` `List`, or no `List` has been submitted, an empty list will be returned.

* The returned list may not be mutated - mutations to content must be done through `submitList`.

* **removeListListener**

```java
public void removeListListener(@NonNull AsyncListDiffer.ListListener<T> listener)
```

* Remove a previously registered ListListener.

| Parameters                                          |                                 |
|-----------------------------------------------------|---------------------------------|
| `@NonNull AsyncListDiffer.ListListener<T>` listener | Previously registered listener. |

* **submitList**

```java
public void submitList(@Nullable List<T> newList, @Nullable Runnable commitCallback)
```

* Pass a new `List` to the `AdapterHelper`. `Adapter` updates will be computed on a background thread.

* If a `List` is already present, a diff will be computed asynchronously on a background thread. When the diff is 
computed, it will be applied (dispatched to the `ListUpdateCallback`), and the new `List` will be swapped in.

* The commit callback can be used to know when the `List` is committed, but note that it may not be executed. If 
`List B` is submitted immediately after `List A`, and is committed directly, the callback associated with `List A` will 
not be run.

| Parameters                          |                                                                                    |
|-------------------------------------|------------------------------------------------------------------------------------|
| `@Nullable List<T>` newList         | The new List.                                                                      |
| `@Nullable Runnable` commitCallback | Optional runnable that is executed when the List is committed, if it is committed. |

### ListAdapter

* `RecyclerView.Adapter` base class for presenting `List` data in a `RecyclerView`, including computing diffs between 
`Lists` on a background thread.

* This class is a convenience wrapper around `AsyncListDiffer` that implements `Adapter` common default behavior for 
item access and counting.

* A complete usage pattern with Room would look like this:

```java
@Dao
interface UserDao {
    @Query("SELECT * FROM user ORDER BY lastName ASC")
    public abstract LiveData<List<User>> usersByLastName();
}

class MyViewModel extends ViewModel {
    public final LiveData<List<User>> usersList;
    public MyViewModel(UserDao userDao) {
        usersList = userDao.usersByLastName();
    }
}

class MyActivity extends AppCompatActivity {
    @Override
    public void onCreate(Bundle savedState) {
        super.onCreate(savedState);
        MyViewModel viewModel = new ViewModelProvider(this).get(MyViewModel.class);
        RecyclerView recyclerView = findViewById(R.id.user_list);
        UserAdapter<User> adapter = new UserAdapter();
        viewModel.usersList.observe(this, list -> adapter.submitList(list));
        recyclerView.setAdapter(adapter);
    }
}

class UserAdapter extends ListAdapter<User, UserViewHolder> {
    public UserAdapter() {
        super(User.DIFF_CALLBACK);
    }
    @Override
    public void onBindViewHolder(UserViewHolder holder, int position) {
        holder.bindTo(getItem(position));
    }
    public static final DiffUtil.ItemCallback<User> DIFF_CALLBACK =
            new DiffUtil.ItemCallback<User>() {
        @Override
        public boolean areItemsTheSame(
                @NonNull User oldUser, @NonNull User newUser) {
            // User properties may have changed if reloaded from the DB, but ID is fixed
            return oldUser.getId() == newUser.getId();
        }
        @Override
        public boolean areContentsTheSame(
                @NonNull User oldUser, @NonNull User newUser) {
            // NOTE: if you use equals, your object must properly override Object#equals()
            // Incorrectly returning false here will result in too many animations.
            return oldUser.equals(newUser);
        }
    }
}
```

* **getCurrentList**

```java
public @NonNull List<T> getCurrentList()
```

* It returns `getCurrentList` from the `AsyncListDiffer`

* **getItemCount**

```java
public int getItemCount()
```

* Returns the total number of items in the data set held by the adapter.

* **onCurrentListChanged**

* Called when the current `List` is updated.

* If a `null` `List` is passed to `submitList`, or no `List` has been submitted, the current `List` is represented as 
an empty `List`.

* **submitList**

```java
public void submitList(@Nullable List<T> list, @Nullable Runnable commitCallback)
```

* It calls `submitList` from the `AsyncListDiffer`
