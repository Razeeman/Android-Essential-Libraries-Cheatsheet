# RECYCLER

- [AdapterDelegates](#adapterdelegates---)
- [FastAdapter](#fastadapter---)
- [Groupie](#groupie---)
- [Epoxy](#epoxy---)
- [Sources](#sources)



# AdapterDelegates [![Maven][adapterdelegates-mavenbadge]][adapterdelegates-maven] [![Source][adapterdelegates-sourcebadge]][adapterdelegates-source] ![adapterdelegates-starsbadge]

"Favor composition over inheritance" for RecyclerView Adapters. The idea is that you define an AdapterDelegate for each view type. This delegate is responsible for creating ViewHolder and binding ViewHolder for a certain viewtype. Then you can compose your RecyclerView Adapter by registering the AdapterDelegates that you really need.

```gradle
dependencies {
    implementation 'com.hannesdorfmann:adapterdelegates4-kotlin-dsl:4.3.0'
    implementation 'com.hannesdorfmann:adapterdelegates4-kotlin-dsl-layoutcontainer:4.3.0' // if you use synthetics
    implementation 'com.hannesdorfmann:adapterdelegates4-kotlin-dsl-viewbinding:4.3.0' // if you use ViewBinding
    
    implementation 'com.hannesdorfmann:adapterdelegates4:4.3.0' // if you don't use kotlin dsl
    implementation 'com.hannesdorfmann:adapterdelegates4-pagination:4.3.0' // for pagination
}
```

### AdapterDelegates. Basic Usage

adapterDelegate - used with findViewById().
adapterDelegateLayoutContainer - used with kotlin synthetics.
adapterDelegateViewBinding - used with data binding.

```kotlin
fun catAdapterDelegate(
    itemClickedListener : (Cat) -> Unit
) = adapterDelegateLayoutContainer<Cat, Animal>(R.layout.item_cat) {

    name.setClickListener { itemClickedListener(item) }
    bind { diffPayloads ->
        name.text = item.name
    }
}
```

You have to specify if a specific AdapterDelegate is responsible for a specific item. Per default this is done with an instanceof check like Cat instanceof Animal. You can override this if you want to handle it in a custom way by setting the on lambda and return true or false:

```kotlin
adapterDelegate<Cat, Animal> (
    layout = R.layout.item_cat,
    on = { item: Animal, items: List, position: Int ->
        if (item is Cat && position == 0)
            true // return true: this adapterDelegate handles it
        else
            false // return false
    }
){
    ...
    bind { ... }
}
```

Compose your Adapter

```kotlin
val adapter = ListDelegationAdapter<List<Animal>>(
    catAdapterDelegate(...),
    dogAdapterDelegate(),
    snakeAdapterDelegate()
)
```

### AdapterDelegates. Java Usage

Define an AdapterDelegate for each view type. An AdapterDelegate get added to an AdapterDelegatesManager. This manager is the man in the middle between RecyclerView.Adapter and each AdapterDelegate.

```java
public class CatAdapterDelegate extends AdapterDelegate<List<Animal>> {

  private LayoutInflater inflater;

  public CatAdapterDelegate(Activity activity) {
    inflater = activity.getLayoutInflater();
  }

  @Override public boolean isForViewType(@NonNull List<Animal> items, int position) {
    return items.get(position) instanceof Cat;
  }

  @NonNull @Override public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent) {
    return new CatViewHolder(inflater.inflate(R.layout.item_cat, parent, false));
  }

  @Override public void onBindViewHolder(@NonNull List<Animal> items, int position,
      @NonNull RecyclerView.ViewHolder holder, @Nullable List<Object> payloads) {

    CatViewHolder vh = (CatViewHolder) holder;
    Cat cat = (Cat) items.get(position);

    vh.name.setText(cat.getName());
  }

  static class CatViewHolder extends RecyclerView.ViewHolder {

    public TextView name;

    public CatViewHolder(View itemView) {
      super(itemView);
      name = (TextView) itemView.findViewById(R.id.name);
    }
  }
}
```

```java
public class AnimalAdapter extends RecyclerView.Adapter {

  private AdapterDelegatesManager<List<Animal>> delegatesManager;
  private List<Animal> items;

  public AnimalAdapter(Activity activity, List<Animal> items) {
    this.items = items;

    delegatesManager = new AdapterDelegatesManager<>();

    // AdapterDelegatesManager internally assigns view types integers
    delegatesManager.addDelegate(new CatAdapterDelegate(activity))
                    .addDelegate(new DogAdapterDelegate(activity))
                    .addDelegate(new GeckoAdapterDelegate(activity));

    // You can explicitly assign integer view type if you want to
    delegatesManager.addDelegate(23, new SnakeAdapterDelegate(activity));
  }

  @Override public int getItemViewType(int position) {
    return delegatesManager.getItemViewType(items, position);
  }

  @Override public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    return delegatesManager.onCreateViewHolder(parent, viewType);
  }

  @Override public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
    delegatesManager.onBindViewHolder(items, position, holder);
  }

  @Override public int getItemCount() {
    return items.size();
  }
}
```

To reduce boilerplate extend either from **ListDelegationAdapter** if the data source the adapter displays is java.util.List<?> or **AbsDelegationAdapter** which is a more general one (not limited to java.util.List).

```java
public class AnimalAdapter extends ListDelegationAdapter<List<Animal>> {

  public AnimalAdapter(Activity activity, List<Animal> items) {

    delegatesManager.addDelegate(new CatAdapterDelegate(activity))
                    .addDelegate(new DogAdapterDelegate(activity))
                    .addDelegate(new GeckoAdapterDelegate(activity))
                    .addDelegate(23, new SnakeAdapterDelegate(activity));

    setItems(items);
  }
}
```

### AdapterDelegates. Async Diff

AsyncListDifferDelegationAdapter is the equivalent to ListAdapter that uses AsyncListDiffer internally. It does calculating diff in the background thread by default and does all particular animations.

```java
public class DiffAdapter extends AsyncListDifferDelegationAdapter<Animal> {
    public DiffAdapter() {
        super(DIFF_CALLBACK) // Your diff callback for diff utils
        delegatesManager
            .addDelegate(new DogAdapterDelegate());
            .addDelegate(new CatAdapterDelegate());
    }
}
```

[Content](#recycler)
# FastAdapter [![Maven][fastadapter-mavenbadge]][fastadapter-maven] [![Source][fastadapter-sourcebadge]][fastadapter-source] ![fastadapter-starsbadge]

The bullet proof, fast and easy to use adapter library, which minimizes developing time to a fraction.

```gradle
dependencies {
    implementation "com.mikepenz:fastadapter:5.1.0"
    
    // Many helper classes available
    implementation "com.mikepenz:fastadapter-extensions-expandable:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-binding:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-diff:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-drag:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-paged:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-scroll:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-swipe:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-ui:5.1.0"
    implementation "com.mikepenz:fastadapter-extensions-utils:5.1.0"
}
```

* Core module 100% in Kotlin
* Write less code, get better results
* Highly optimized code
* Easily extensible
* Split item view and model
* Chain other Adapters
* Click / Long-Click listeners, Selection / Multi-Selection, Expandable items, Simple Drag & Drop, Headers, Footers, Filter, Endless Scroll, "Leave-Behind"-Pattern, ActionModeHelper, UndoHelper, FastScroller, Paging (via Jetpack paging lib)

### FastAdapter. Basic Usage

**Implement your item**

Create a class which extends the AbstractItem.

```kotlin
open class SimpleItem : AbstractItem<SimpleItem.ViewHolder>() {
    var name: String? = null
    var description: String? = null

    /** defines the type defining this item. must be unique. preferably an id */
    override val type: Int get() = R.id.fastadapter_sample_item_id

    /** defines the layout which will be used for this item in the list */
    override val layoutRes: Int get() = R.layout.sample_item

    override fun getViewHolder(v: View): ViewHolder = ViewHolder(v)

    class ViewHolder(view: View) : FastAdapter.ViewHolder<SimpleItem>(view) {
        var name: TextView = view.findViewById(R.id.material_drawer_name)
        var description: TextView = view.findViewById(R.id.material_drawer_description)

        override fun bindView(item: SimpleItem, payloads: List<Any>) {
            name.text = item.name
            description.text = item.name
        }

        override fun unbindView(item: SimpleItem) {
            name.text = null
            description.text = null
        }
    }
}
```

Or implement item with ViewBinding.

```kotlin
class BindingIconItem : AbstractBindingItem<IconItemBinding>() {
    var name: String? = null

    override val type: Int get() = R.id.fastadapter_icon_item_id

    override fun bindView(binding: IconItemBinding, payloads: List<Any>) {
        binding.name.text = name
    }

    override fun createBinding(inflater: LayoutInflater, parent: ViewGroup?): IconItemBinding {
        return IconItemBinding.inflate(inflater, parent, false)
    }
}
```

**Set the Adapter to the RecyclerView**

```kotlin
//create the ItemAdapter holding your Items
val itemAdapter = ItemAdapter<SimpleItem>()

//create the managing FastAdapter, by passing in the itemAdapter
val fastAdapter = FastAdapter.with(itemAdapter)

//set our adapters to the RecyclerView
recyclerView.setAdapter(fastAdapter)

//set the items to your ItemAdapter
itemAdapter.add(ITEMS)
```

### FastAdapter. Helpers

By default the FastAdapter only provides basic functionality, which comes with the abstraction of items as Item and Model. And the general functionality of adding/removing/modifying elements. To enable selections, or expandables the provided extensions need to be activated.

**SelectExtension**

```kotlin
// Gets (or creates and attaches if not yet existing) the extension from the given `FastAdapter`
val selectExtension = fastAdapter.getSelectExtension()

// configure as needed
selectExtension.isSelectable = true
selectExtension.multiSelect = true
selectExtension.selectOnLongClick = false
```

**ExpandableExtension**

```kotlin
// Gets (or creates and attaches if not yet existing) the extension.
val expandableExtension = fastAdapter.getExpandableExtension()

// configure as needed
expandableExtension.isOnlyOneExpandedItem = true
```

**Filter**

```kotlin
// Call this in onQueryTextSubmit() & onQueryTextChange() when using SearchView
itemAdapter.filter("yourSearchTerm")

itemAdapter.itemFilter.filterPredicate = { item: SimpleItem, constraint: CharSequence? ->
    item.name?.text.toString().contains(constraint.toString(), ignoreCase = true)
}
```

**Drag and drop**

```kotlin
// First, attach ItemTouchHelper to RecyclerView.
val dragCallback = SimpleDragCallback()
val touchHelper = ItemTouchHelper(dragCallback)
touchHelper.attachToRecyclerView(recyclerView)

// Implement ItemTouchCallback interface in your Activity, and override the itemTouchOnMove() method.
override fun itemTouchOnMove(oldPosition: Int, newPosition: Int): Boolean {
    DragDropUtil.onMove(fastItemAdapter.itemAdapter, oldPosition, newPosition) // change position
    return true
}
```

### FastAdapter. Different ViewHolders

```kotlin
val headerAdapter = ItemAdapter<Header>()
val itemAdapter = GenericItemAdapter()

// Order defines in which order the items will show up
val fastAdapter: GenericFastAdapter = FastAdapter.with(headerAdapter, itemAdapter) 
recyclerView.setAdapter(fastAdapter)
```

[Content](#recycler)
# Groupie [![Maven][groupie-mavenbadge]][groupie-maven] [![Source][groupie-sourcebadge]][groupie-source] ![groupie-starsbadge]

Groupie is a simple, flexible library for complex RecyclerView layouts.

```gradle
dependencies {
    implementation "com.xwray:groupie:2.8.0"
    
    implementation "com.xwray:groupie-kotlin-android-extensions:2.8.0" // for kotlin synthetics
    implementation "com.xwray:groupie-viewbinding:2.8.0" // for databinding
}
```

Groupie lets you treat your content as logical groups and handles change notifications for you - think sections with headers and footers, expandable groups, blocks of vertical columns, and much more. It makes it easy to handle asynchronous content updates and insertions and user-driven content changes. At the item level, it abstracts the boilerplate of item view types, item layouts, viewholders, and span sizes.

### Groupie. Basic Usage

Use a GroupAdapter anywhere you would normally use a RecyclerView.Adapter, and attach it to your RecyclerView as usual.

```kotlin
val adapter = GroupAdapter()
recyclerView.setAdapter(adapter)
adapter += HeaderItem()
adapter += SomeItem()
```

### Groupie. Groups

Groups are the building block of Groupie. An individual Item (the unit which an adapter inflates and recycles) is a Group of 1. You can add Groups and Items interchangeably to the adapter.

```kotlin
groupAdapter += HeaderItem()
groupAdapter += CommentItem()

val section = Section()
section.setHeader(HeaderItem())
section.addAll(bodyItems)
groupAdapter += section
```

Modifying the contents of the GroupAdapter in any way automatically sends change notifications. Adding an item calls notifyItemAdded(); adding a group calls notifyItemRangeAdded(), etc.

There are a few simple implementations of Groups within the library:

* **Section**, a list of body content with an optional header group and footer group. It supports diffing and animating moves, updates and other changes.
* **ExpandableGroup**, a single parent group with a list of body content that can be toggled hidden or shown.

### Groupie. Items

Each Item declares a view layout id, and gets a callback to bind the inflated layout.

```kotlin
class SongItem(private val song: Song) : Item() {

    override fun getLayout() = R.layout.song

    override fun bind(viewHolder: GroupieViewHolder, position: Int) {
        viewHolder.title.text = song.title
        viewHolder.artist.text = song.artist
    }
}
```

With data binding:

```java
public class SongItem extends BindableItem<SongBinding> {

    public SongItem(Song song) {
        this(song);
    }    

    @Override public void bind(SongBinding binding, int position) {
        binding.setSong(song);
    }

    @Override public int getLayout() {
        return R.layout.song;
    }
}
```

[Content](#recycler)
# Epoxy [![Maven][epoxy-mavenbadge]][epoxy-maven] [![Source][epoxy-sourcebadge]][epoxy-source] ![epoxy-starsbadge]

Library for building complex screens in a RecyclerView created by Airbnb.

```gradle
dependencies {
    implementation 'com.airbnb.android:epoxy:3.11.0'
}
```

Models are automatically generated from custom views or databinding layouts via annotation processing. These models are then used in an EpoxyController to declare what items to show in the RecyclerView.

This abstracts the boilerplate of view holders, diffing items and binding payload changes, item types, item ids, span counts, and more, in order to simplify building screens with multiple view types. Additionally, Epoxy adds support for saving view state and automatic diffing of item changes.

### Epoxy. Basic usage

There are two main components of Epoxy:

- The **EpoxyModels** that describe how your views should be displayed in the RecyclerView.
- The **EpoxyController** where the models are used to describe what items to show and with what data.

Epoxy generates models for you based on your view or layout. Generated model classes are suffixed with an underscore are used directly in your EpoxyController classes.

### Epoxy. Models

Epoxy uses EpoxyModel objects to decide which views to display and how to bind data to them. This is similar to the popular ViewModel pattern. Models also allow you to control other aspects of the view, such as the grid span size, id, and saved state. The RecyclerView concept of stable ideas is built into Epoxy, meaning each EpoxyModel should have a unique id to identify it. This allows diffing and saved state. A model's state is determined by its equals and hashCode implementations, which is based on the value of all of the model's properties. This state is used in diffing to determine when a model has changed so Epoxy can update the view. Each model is typed with the View that it supports. The model implements buildView to create a new instance of that view type. Additionally, getViewType returns an int representing that view type for use in view recycling. When a model's state changes, and the model is bound to a view on screen, Epoxy rebinds the view with only the properties that changed. This partial rebind is more efficient than rebinding the whole view.

**From custom views:**

Add the **@ModelView** annotation on a view class. Then, add a "prop" annotation on each setter method to mark it as a property for the model.

```java
@ModelView(autoLayout = Size.MATCH_WIDTH_WRAP_HEIGHT)
public class HeaderView extends LinearLayout {

  @TextProp
  public void setTitle(CharSequence text) {
    titleView.setText(text);
  }
}
```

A **HeaderViewModel_** is then generated in the same package.

**From databinding:**

Set up your xml layouts like normal:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="title" type="String" />
    </data>

    <TextView
        android:layout_width="120dp"
        android:layout_height="40dp"
        android:text="@{title}" />
</layout>
```

Then, create a **package-info.java** file in any package and add an EpoxyDataBindingLayouts annotation to declare your databinding layouts.

```java
@EpoxyDataBindingLayouts({R.layout.header_view, ... // other layouts })
package com.airbnb.epoxy.sample;

import com.airbnb.epoxy.EpoxyDataBindingLayouts;
import com.airbnb.epoxy.R;
```

From this layout name Epoxy generates a **HeaderViewBindingModel_**.

**From ViewHolders:**

If you use xml layouts without databinding you can create a model class to do the binding.

```java
@EpoxyModelClass(layout = R.layout.header_view)
public abstract class HeaderModel extends EpoxyModelWithHolder<Holder> {
  @EpoxyAttribute String title;

  @Override
  public void bind(Holder holder) {
    holder.header.setText(title);
  }

  static class Holder extends BaseEpoxyHolder {
    @BindView(R.id.text) TextView header;
  }
}
```

A **HeaderModel_** class is generated that subclasses HeaderModel and implements the model details.

### Epoxy. Controllers

A controller defines what items should be shown in the RecyclerView, by adding the corresponding models in the desired order. The EpoxyController encourages usage similar to the popular Model-View-ViewModel and Model-View-Intent patterns. Data flows in one direction: from your state, to the EpoxyModels, to the views on the RecyclerView. User input to the views triggers callbacks that may update your state and restart the cycle. EpoxyController has a second constructor that takes two Handler instances, one for running model building and one for handling diffing. By default these use the main thread, but they can be changed to allow async work for performance improvement.

The controller's **buildModels** method declares which items to show. You are responsible for calling **requestModelBuild** whenever your data changes, which triggers **buildModels** to run again. Epoxy tracks changes in the models and automatically binds and updates views.

As an example, our PhotoController shows a header, a list of photos, and a loader (if more photos are being loaded). The controller's setData(photos, loadingMore) method is called whenever photos are loaded, which triggers a call to buildModels so models representing the state of the new data can be built.

```java
public class PhotoController extends Typed2EpoxyController<List<Photo>, Boolean> {
    @AutoModel HeaderModel_ headerModel;
    @AutoModel LoaderModel_ loaderModel;

    @Override
    protected void buildModels(List<Photo> photos, Boolean loadingMore) {
      headerModel
          .title("My Photos")
          .description("My album description!")
          .addTo(this);

      for (Photo photo : photos) {
        new PhotoModel()
           .id(photo.id())
           .url(photo.url())
           .addTo(this);
      }

      loaderModel
          .addIf(loadingMore, this);
    }
  }
```

**Integrating with RecyclerView**

Get the backing adapter off the EpoxyController to set up RecyclerView:

```java
MyController controller = new MyController();
recyclerView.setAdapter(controller.getAdapter());

// Request a model build whenever your data changes
controller.requestModelBuild();

// Or if you are using a TypedEpoxyController
controller.setData(myData);
```

If you are using the EpoxyRecyclerView integration is easier.

```java
epoxyRecyclerView.setControllerAndBuildModels(new MyController());

// Request a model build on the recyclerview when data changes
epoxyRecyclerView.requestModelBuild();
```


[Content](#recycler)
# Sources

**Epoxy**

https://github.com/airbnb/epoxy
<br>
https://github.com/airbnb/epoxy/wiki/Configuration

**AdapterDelegates**

https://github.com/sockeqwe/AdapterDelegates

**FastAdapter**

https://github.com/mikepenz/FastAdapter

**Groupie**

https://github.com/lisawray/groupie



[epoxy-maven]: https://search.maven.org/artifact/com.airbnb.android/epoxy
[epoxy-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.airbnb.android/epoxy/badge.svg
[epoxy-source]: https://github.com/airbnb/epoxy
[epoxy-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[epoxy-starsbadge]: https://img.shields.io/github/stars/airbnb/epoxy

[adapterdelegates-maven]: https://search.maven.org/artifact/com.hannesdorfmann/adapterdelegates4
[adapterdelegates-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.hannesdorfmann/adapterdelegates4/badge.svg
[adapterdelegates-source]: https://github.com/sockeqwe/AdapterDelegates
[adapterdelegates-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[adapterdelegates-starsbadge]: https://img.shields.io/github/stars/sockeqwe/AdapterDelegates

[fastadapter-maven]: https://search.maven.org/artifact/com.mikepenz/fastadapter
[fastadapter-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.mikepenz/fastadapter/badge.svg
[fastadapter-source]: https://github.com/mikepenz/FastAdapter
[fastadapter-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[fastadapter-starsbadge]: https://img.shields.io/github/stars/mikepenz/FastAdapter

[groupie-maven]: https://bintray.com/lisawray/maven/groupie/_latestVersion
[groupie-mavenbadge]: https://api.bintray.com/packages/lisawray/maven/groupie/images/download.svg
[groupie-source]: https://github.com/lisawray/groupie
[groupie-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[groupie-starsbadge]: https://img.shields.io/github/stars/lisawray/groupie
