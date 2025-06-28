---
title: 【AndroidX】Fragment
date: 2025-06-21 22:38:14 +0800
categories: [Android]
tags: [android, fragment, jetpack, view model]
---
## 1.简介
Fragment表示应用界面中可重复使用的部分，通过将界面划分为独立的块从而实现模块化和可重用性。Fragment定义并管理自己的布局，具有自己的生命周期。但fragment不能独立存在，必须由activity或其他fragment托管。
* 官方文档：<https://developer.android.google.cn/guide/fragments>
* API文档：<https://developer.android.google.cn/reference/androidx/fragment/app/Fragment>

## 2.创建fragment
Fragment需要依赖[AndroidX Fragment库](https://developer.android.google.cn/jetpack/androidx/releases/fragment)。在build.gradle文件中添加以下依赖项：

```groovy
implementation 'androidx.fragment:fragment:1.8.8'
```

### 2.1 创建fragment类
为了创建fragment，需要扩展`androidx.fragment.app.Fragment`类，并通过超类构造器指定布局资源id。例如：

```java
import androidx.fragment.app.Fragment;

public class ExampleFragment extends Fragment {
    public ExampleFragment() {
        super(R.layout.example_fragment);
    }
}
```

也可以在`onCreateView()`方法中手动创建布局视图：

```java
public class ExampleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_example, container, false);
    }
}
```

### 2.2 向activity添加fragment
Fragment必须嵌入AndroidX `FragmentActivity`中。`FragmentActivity`是`AppCompatActivity`的超类。因此，如果已经继承了`AppCompatActivity`，则无需修改activity超类。

为了将fragment添加到activity视图中，可以直接在activity的布局文件中定义fragment；或者在布局文件中定义fragment容器，然后以编程方式添加fragment。强烈建议使用`FragmentContainerView`（而不是`FrameLayout`）作为fragment容器。

直接通过XML添加fragment的示例如下：

```xml
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="com.example.ExampleFragment" />
```

`android:name`属性指定fragment类名。

为了以编程方式添加fragment，布局应该包含`FragmentContainerView`作为fragment容器（与上面的示例相同，只是没有`android:name`属性）。在activity运行时，可以执行fragment事务，如添加、移除或替换fragment。

```java
public class ExampleActivity extends AppCompatActivity {
    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState == null) {
            getSupportFragmentManager().beginTransaction()
                .setReorderingAllowed(true)
                .add(R.id.fragment_container_view, ExampleFragment.class, null)
                .commit();
        }
    }
}
```

注意，只有在`savedInstanceState`为`null`时才会创建fragment事务，这是为了确保fragment仅在activity首次创建时添加一次。当activity重新创建时，fragment会自动从`savedInstanceState`中恢复，此时可以通过`findFragmentById()`获得fragment实例，如下所示。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    if (savedInstanceState == null) {
        ...
    } else {
        fragment = getSupportFragmentManager().findFragmentById(R.id.fragment_container_view);
    }
}
```

如果fragment需要初始数据，可以通过在`FragmentTransaction.add()`调用中提供一个`Bundle`将参数传递到fragment，如下所示：

```java
if (savedInstanceState == null) {
    Bundle bundle = new Bundle();
    bundle.putInt("some_int", 0);

    getSupportFragmentManager().beginTransaction()
        .setReorderingAllowed(true)
        .add(R.id.fragment_container_view, ExampleFragment.class, bundle)
        .commit();
}
```

然后在fragment中通过调用`requireArguments()`获取参数：

```java
public class ExampleFragment extends Fragment {
    ...

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        int someInt = requireArguments().getInt("some_int");
        ...
    }
}
```

注：
* 也可以单独创建fragment实例，这种情况下需要通过`fragment.setArguments(args)`设置参数，然后调用`add(containerViewId, fragment)`。
* 在fragment中也可以通过`getArguments()`获取参数。`requireArguments()`与该方法的区别是当参数为`null`时会抛出异常。

## 3.保存和恢复状态
各种Android系统操作都会影响fragment的状态。Android框架会自动保存和恢复fragment，而fragment中的数据需要手动保存和恢复。

### 3.1 视图状态
视图负责管理自己的状态。Android框架提供的视图都实现了`onSaveInstanceState()`和`onRestoreInstanceState()`，因此不必在fragment中管理视图状态（自定义视图也应该实现这两个方法）。

在布局中有id的视图才能保留其状态，没有id的视图无法保留状态。

### 3.2 局部变量
Fragment负责管理局部变量中的状态数据。可以使用`onSaveInstanceState(Bundle)`保存数据。当fragment被重新创建时，可以在`onCreate()`、`onCreateView()`或`onViewCreated()`方法中访问bundle中的数据。

例如：

```java
public class RandomGoodDeedFragment extends Fragment {
    private boolean isEditing;
    private String randomGoodDeed;
    private RandomGoodDeedViewModel viewModel;
    ...

    @Override
    public void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putBoolean(IS_EDITING_KEY, isEditing);
        outState.putString(RANDOM_GOOD_DEED_KEY, randomGoodDeed);
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState != null) {
            isEditing = savedInstanceState.getBoolean(IS_EDITING_KEY, false);
            randomGoodDeed = savedInstanceState.getString(RANDOM_GOOD_DEED_KEY);
        } else {
            randomGoodDeed = viewModel.generateRandomGoodDeed();
        }
    }
}
```

注：也可以使用ViewModel来保存UI状态，这样就无需手动保存和恢复，详见[【Android】ViewModel]({% post_url 2025-06-24-android-view-model %})。

## 4.与fragment通信
为了正确响应用户事件和共享状态信息，通常需要在activity和fragment之间或者两个fragment之间进行通信。

Fragment库提供了两种通信方式：共享ViewModel和Fragment Result API。如果需要与自定义API共享持久性数据，则使用ViewModel。对于数据可放在Bundle中的一次性结果，使用Fragment Result API。

### 4.1 使用ViewModel共享数据
`ViewModel`对象用于存储和管理UI数据，详细信息参见[ViewModel概览](https://developer.android.google.cn/topic/libraries/architecture/viewmodel)。

#### 4.1.1 与宿主activity共享数据
在某些情况下，可能需要在fragment与其宿主activity之间共享数据。

例如，考虑下面的`ItemViewModel`：

```java
public class ItemViewModel extends ViewModel {
    private final MutableLiveData<Item> selectedItem = new MutableLiveData<Item>();

    public void selectItem(Item item) {
        selectedItem.setValue(item);
    }

    public LiveData<Item> getSelectedItem() {
        return selectedItem;
    }
}
```

`LiveData`是可感知生命周期的、可观察的数据容器类，详细信息参见[LiveData概览](https://developer.android.google.cn/topic/libraries/architecture/livedata)。

Fragment及其宿主activity都可以通过将activity传入`ViewModelProvider`构造器来获取ViewModel的共享实例。二者都可以观察和修改其中的数据。

```java
public class MainActivity extends AppCompatActivity {
    private ItemViewModel viewModel;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        viewModel = new ViewModelProvider(this).get(ItemViewModel.class);
        viewModel.getSelectedItem().observe(this, item -> {
            // Perform an action with the latest item data.
        });
    }
}
```

```java
public class ListFragment extends Fragment {
    private ItemViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        viewModel = new ViewModelProvider(requireActivity()).get(ItemViewModel.class);
        ...
        items.setOnClickListener(item -> {
            // Set a new item.
            viewModel.selectItem(item);
        });
    }
}
```

注：如果`ListFragment`以自身为作用域（即`new ViewModelProvider(this)`），将会得到与`MainActivity`不同的ViewModel实例。

#### 4.1.2 在fragment之间共享数据
同一activity中的两个或多个fragment通常需要相互通信。

例如，假设一个fragment显示一个列表，另一个fragment允许用户对该列表应用各种过滤器。这两个fragment可以使用其activity为作用域共享一个ViewModel来处理这种通信。这样fragment不需要相互了解，activity也不需要做任何事来辅助通信。

下面的示例展示了两个fragment如何使用共享的ViewModel进行通信。

```java
public class ListViewModel extends ViewModel {
    private final MutableLiveData<Set<Filter>> filters = new MutableLiveData<>();

    private final LiveData<List<Item>> originalList = ...;
    private final LiveData<List<Item>> filteredList = ...;

    public LiveData<List<Item>> getFilteredList() {
        return filteredList;
    }

    public LiveData<Set<Filter>> getFilters() {
        return filters;
    }

    public void addFilter(Filter filter) { ... }

    public void removeFilter(Filter filter) { ... }
}
```

```java
public class ListFragment extends Fragment {
    private ListViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        viewModel = new ViewModelProvider(requireActivity()).get(ListViewModel.class);
        viewModel.getFilteredList().observe(getViewLifecycleOwner(), list -> {
            // Update the list UI.
        });
    }
}
```

```java
public class FilterFragment extends Fragment {
    private ListViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        viewModel = new ViewModelProvider(requireActivity()).get(ListViewModel.class);
        viewModel.getFilters().observe(getViewLifecycleOwner(), set -> {
            // Update the selected filters UI.
        });
    }

    public void onFilterSelected(Filter filter) {
        viewModel.addFilter(filter);
    }

    public void onFilterDeselected(Filter filter) {
        viewModel.removeFilter(filter);
    }
}
```

#### 4.1.3 在父fragment与子fragment之间共享数据
为了在父fragment与子fragment之间共享数据，使用父fragment作为ViewModel作用域，如下面的示例所示：

```java
public class ListFragment extends Fragment {
    private ListViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        viewModel = new ViewModelProvider(this).get(ListViewModel.class);
        viewModel.getFilteredList().observe(getViewLifecycleOwner(), list -> {
            // Update the list UI.
        }
    }
}
```

```java
public class ChildFragment extends Fragment {
    private ListViewModel viewModel;
    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        viewModel = new ViewModelProvider(requireParentFragment()).get(ListViewModel.class);
        ...
    }
}
```

### 4.2 使用Fragment Result API获取结果
在某些情况下，可能需要在两个fragment之间或fragment与其宿主activity之间传递一次性的值。例如，一个fragment扫描二维码，并将数据传回之前的fragment。

在Fragment库1.3.0及更高版本中，`FragmentManager`类实现了`FragmentResultOwner`接口，这意味着`FragmentManager`可以充当fragment结果的集中存储。这使得fragment可以通过设置和监听结果进行通信，而无需相互引用。

#### 4.2.1 在fragment之间传递结果
为了将数据从fragment B传回fragment A，首先在fragment A（即接收结果的fragment）中设置结果监听器。对fragment A的`FragmentManager`调用`setFragmentResultListener()`，如下所示：

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    getParentFragmentManager().setFragmentResultListener("requestKey", this, new FragmentResultListener() {
        @Override
        public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle bundle) {
            // We use a String here, but any type that can be put in a Bundle is supported.
            String result = bundle.getString("bundleKey");
            // Do something with the result.
        }
    });
}
```

在fragment B（即生成结果的fragment）中，通过在同一个`FragmentManager`上调用`setFragmentResult()`使用相同的`requestKey`设置结果：

```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Bundle result = new Bundle();
        result.putString("bundleKey", "result");
        getParentFragmentManager().setFragmentResult("requestKey", result);
    }
});
```

然后，一旦fragment A处于`STARTED`状态，它就会收到结果并执行监听器回调。

![fragment之间传递结果](https://developer.android.google.cn/static/images/guide/fragments/fragment-a-to-b.png)

对于给定的键只能有一个监听器和结果。如果对同一个键多次调用`setFragmentResult()`，并且监听fragment未处于`STARTED`状态，则新结果会替换旧结果。

#### 4.2.2 在父fragment与子fragment之间传递结果
为了将结果从子fragment传递给父fragment，在父fragment中调用`setFragmentResultListener()`时使用`getChildFragmentManager()`而不是`getParentFragmentManager()`。

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Set the listener on the child fragmentManager.
    getChildFragmentManager()
        .setFragmentResultListener("requestKey", this, new FragmentResultListener() {
            @Override
            public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle bundle) {
                String result = bundle.getString("bundleKey");
                // Do something with the result.
            }
        });
}
```

子fragment使用`getParentFragmentManager()`设置结果：

```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Bundle result = new Bundle();
        result.putString("bundleKey", "result");
        // The child fragment needs to still set the result on its parent fragment manager.
        getParentFragmentManager().setFragmentResult("requestKey", result);
    }
});
```

![父fragment与子fragment之间传递结果](https://developer.android.google.cn/static/images/guide/fragments/pass-parent-child.png?hl=zh-cn)

#### 4.2.3 在宿主activity中接收结果
为了在宿主activity中接收fragment结果，使用`getSupportFragmentManager()`设置结果监听器。

```java
public class MainActivity extends AppCompatActivity {
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getSupportFragmentManager().setFragmentResultListener("requestKey", this, new FragmentResultListener() {
            @Override
            public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle bundle) {
                // We use a String here, but any type that can be put in a Bundle is supported.
                String result = bundle.getString("bundleKey");
                // Do something with the result.
            }
        });
    }
}
```

## 5.测试fragment
<https://developer.android.google.cn/guide/fragments/test>
