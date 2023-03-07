# RecyclerView

## Create dynamic lists with RecyclerView

* **RecyclerView** makes it easy to efficiently display large sets of data. You supply the data and define how each 
item looks, and the RecyclerView library dynamically creates the elements when they're needed.

* As the name implies, RecyclerView recycles those individual elements. When an item scrolls off the screen, 
RecyclerView doesn't destroy its view. Instead, RecyclerView reuses the view for new items that have scrolled onscreen. 
RecyclerView improves performance and your app's responsiveness, and it reduces power consumption.

### Key classes

* Several classes work together to build your dynamic list.

* `RecyclerView` is the `ViewGroup` that contains the views corresponding to your data. It's a view itself, so you 
add `RecyclerView` to your layout the way you would add any other UI element.

* Each individual element in the list is defined by a **view holder** object. When the view holder is created, it 
doesn't have any data associated with it. After the view holder is created, the `RecyclerView` **binds** it to its data. 
You define the view holder by extending `RecyclerView.ViewHolder`.

* The `RecyclerView` requests views, and binds the views to their data, by calling methods in the adapter. You define 
the adapter by extending `RecyclerView.Adapter`.

* The **layout manager** arranges the individual elements in your list. You can use one of the layout managers provided 
by the RecyclerView library, or you can define your own. Layout managers are all based on the library's `LayoutManager` 
abstract class.

### Steps for implementing your RecyclerView

* If you're going to use RecyclerView, there are a few things you need to do. They are explained in detail in the 
following sections. 
  1. Decide how the list or grid looks. Ordinarily, you can use one of the RecyclerView library's standard layout 
  managers. 
  2. Design how each element in the list looks and behaves. Based on this design, extend the `ViewHolder` class. Your 
  version of `ViewHolder` provides all the functionality for your list items. Your view holder is a wrapper around a 
  `View`, and that view is managed by `RecyclerView`.
  3. Define the `Adapter` that associates your data with the `ViewHolder` views.

### Plan your layout

* The items in your RecyclerView are arranged by a `LayoutManager` class. The RecyclerView library provides three 
layout managers, which handle the most common layout situations:
  * `LinearLayoutManager` arranges the items in a one-dimensional list. 
  * `GridLayoutManager` arranges the items in a two-dimensional grid:
    * If the grid is arranged vertically, `GridLayoutManager` tries to make all the elements in each row have the same 
    width and height, but different rows can have different heights. 
    * If the grid is arranged horizontally, `GridLayoutManager` tries to make all the elements in each column have the 
    same width and height, but different columns can have different widths.
  * `StaggeredGridLayoutManager` is similar to `GridLayoutManager`, but it does not require that items in a row have 
  the same height (for vertical grids) or items in the same column have the same width (for horizontal grids). The 
  result is that the items in a row or column can end up offset from each other.

### Implement your adapter and view holder

* Once you determine your layout, you need to implement your `Adapter` and `ViewHolder`. These two classes work 
together to define how your data is displayed. The `ViewHolder` is a wrapper around a `View` that contains the layout 
for an individual item in the list. The `Adapter` creates `ViewHolder` objects as needed and also sets the data for 
those views. The process of associating views to their data is called **binding**.

* When you define your adapter, you override three key methods:
  * `onCreateViewHolder()`: `RecyclerView` calls this method whenever it needs to create a new `ViewHolder`. The method 
  creates and initializes the `ViewHolder` and its associated `View`, but does not fill in the view's contents — the 
  `ViewHolder` has not yet been bound to specific data.
  * `onBindViewHolder()`: `RecyclerView` calls this method to associate a `ViewHolder` with data. The method fetches 
  the appropriate data and uses the data to fill in the view holder's layout. For example, if the `RecyclerView` 
  displays a list of names, the method might find the appropriate name in the list and fill in the view holder's 
  `TextView` widget.
  * `getItemCount()`: `RecyclerView` calls this method to get the size of the dataset. For example, in an address book 
  app, this might be the total number of addresses. RecyclerView uses this to determine when there are no more items 
  that can be displayed.

* Here's a typical example of a simple adapter with a nested `ViewHolder` that displays a list of data. The layout for 
the each view item is defined in an XML layout file, as usual.

```kotlin

class CustomAdapter(private val dataSet: Array<String>) :
        RecyclerView.Adapter<CustomAdapter.ViewHolder>() {

    /**
     * Provide a reference to the type of views that you are using
     * (custom ViewHolder)
     */
    class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val textView: TextView

        init {
            // Define click listener for the ViewHolder's View
            textView = view.findViewById(R.id.textView)
        }
    }

    // Create new views (invoked by the layout manager)
    override fun onCreateViewHolder(viewGroup: ViewGroup, viewType: Int): ViewHolder {
        // Create a new view, which defines the UI of the list item
        val view = LayoutInflater.from(viewGroup.context)
                .inflate(R.layout.text_row_item, viewGroup, false)

        return ViewHolder(view)
    }

    // Replace the contents of a view (invoked by the layout manager)
    override fun onBindViewHolder(viewHolder: ViewHolder, position: Int) {

        // Get element from your dataset at this position and replace the
        // contents of the view with that element
        viewHolder.textView.text = dataSet[position]
    }

    // Return the size of your dataset (invoked by the layout manager)
    override fun getItemCount() = dataSet.size

}

```

---

## Advanced RecyclerView customization

* You can customize the `RecyclerView` objects to meet your specific needs. The standard classes provide all the 
functionality that most developers will need; in many cases, the only customization you need to do is design the view 
for each view holder and write the code to update those views with the appropriate data. However, if your app has 
specific requirements, you can modify the standard behavior in a number of ways.

### Add item animations

* Whenever an item changes, the `RecyclerView` uses an **animator** to change its appearance. This animator is an 
object that extends the abstract `RecyclerView.ItemAnimator` class. By default, the `RecyclerView` uses 
`DefaultItemAnimator` to provide the animation. If you want to provide custom animations, you can define your own 
animator object by extending `RecyclerView.ItemAnimator`.

### Enable list-item selection

* The `recyclerview-selection` library enables users to select items in `RecyclerView` list using touch or mouse input. 
You retain control over the visual presentation of a selected item. You can also retain control over policies 
controlling selection behavior, such as items that can be eligible for selection, and how many items can be selected.

---

* Create dynamic lists with RecyclerView : https://developer.android.com/develop/ui/views/layout/recyclerview
* Advanced RecyclerView customization: https://developer.android.com/develop/ui/views/layout/recyclerview-custom
* Speeding up RecyclerView. Optimization Best Practices: https://www.youtube.com/watch?v=o8rzzQPOo2U
















