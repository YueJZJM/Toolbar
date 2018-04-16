工具栏可以提供应用导航、放置菜单选项，也可以统一风格设计等，这里有个简单的例子，如图所示：

![Toolbar 示例](https://upload-images.jianshu.io/upload_images/10095597-ef4604264e97cd1a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/250)



这里我们创建工具栏菜单，分以下几点介绍：

- 在 XML 文件中定义菜单
- 让菜单显示出来
- 响应菜单的选择
- 子标题的显示
- 数据的同步

## 在 XML 文件中定义菜单

菜单及菜单选项需要用到一些字符串资源，我们首先添加字符串资源

**values/strings.xml**
```
<resources>
    <string name="new_crime">New Crime</string>
    <string name="show_subtitle">Show Subtitle</string>
    <string name="hide_subtitle">Hide Subtitle</string>
    <string name="subtitle_format">%1$d crimes</string>
</resources>
```

右键点击 res 目录， 选择 New->Android resource file 菜单项。在弹出窗口界面，选择 Menu 资源按钮，命名资源文件为 fragment_crime_list,点击 OK 确认按钮。

![创建菜单](https://upload-images.jianshu.io/upload_images/10095597-6c03d70ae5282317.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)

打开创建的 fragment_crime_list.xml 文件,添加以下代码

**res/menu/fragment_crime_list.xml**
```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/new_crime"
        android:icon="@drawable/ic_menu_add"
        android:title="@string/new_crime"
        app:showAsAction="ifRoom|withText"/>
</menu>
```
showAsAction 属性是指定菜单显示在工具栏上还是溢出菜单，这里设置的是 ifRoom 和 withText 的组合值。因此，只要空间足够，就会菜单项就会显示图标及其文字。如果空间仅够显示菜单项图标，文字描述就不会显示。如果空间大小不够显示任何项，菜单就会溢出到溢出菜单中，以三个点为表示。showAsAction 属性还有两个可选值：always 和 never。不推荐用 always，尽量使用 ifRoom。对于很少用到的菜单，可选择 never 属性。

## 让菜单显示出来
在代码中，Acrivity 类提供了管理菜单的回调函数。需要选项菜单时，Android 会调用 Activity 的 onCreateOptionsMenu(Menu) 函数。但是在本例中，与选项菜单相关的回调函数需在 fragment 而非 activity 里实现。不用担心，fragment 有一套自己的处理机制。

实例化选项菜单

**CrlmeListFragment.java**
```
public class CrimeListFragment extends Fragment {
    @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        super.onCreateOptionsMenu(menu, inflater);
        inflater.inflate(R.menu.frigment_crime_list,menu);
    }
}
```
在上面代码中，我们调用 MenuInflater.inflate(int,Menu) 方法传入菜单文件的资源 ID,将布局文件中定义的菜单项目填充到 Menu 实例中。

Fragment.onCreateOptionMenu(Menu,MenuInflater) 是由 FragmentManager 调用的。因此，当 activity 接收到操作系统的 onCreateOptionsMenu(...) 方法请求回调时，我们必须明确告诉 FragmentManager：其管理的 fragment 应该接收 onCreateOptionsMenu(...) 方法调用指令。
需调用以下方法：
> public void setHsaOptionsMenu(boolean hsaMenu)

**CrimeListFragment.java**
```
public class CrimeListFragment extends Fragment {
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setHasOptionsMenu(true);
    }
}
```
运行程序就能看到创建的菜单栏了

![显示在工具栏上的菜单栏](https://upload-images.jianshu.io/upload_images/10095597-b975b27f2dc95388.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/250)



在竖屏模式下因为空间有限，默认隐藏菜单项标题，长按可以看到。在横屏模式下就可以直接看到。


![竖屏](https://upload-images.jianshu.io/upload_images/10095597-3f5109258f9cb4ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/450)

## 响应菜单的选择

为了响应用户的菜单项，需要向 Crime 列表中添加新的 Crime。还有删除随机生成的数据。

**CrimeLab.java**
```
public void addCrime(Crime c){
    mCrimes.add(c);
}
private CrimeLab(Context context) {
    mCrimes = new ArrayList<>();
   /*for (int i = 0; i < 100; i++) {
        Crime crime = new Crime();
        crime.setTitle("Crime #" + i);
        crime.setSolved(i % 2 == 0);
        mCrimes.add(crime);
    }*/
}
```

当用户点击菜单中的菜单项时，fragment 会收到 onOptionsItemSelectde(MenuItem) 方法的回调请求。传入该方法的是一个描述用户选择的 MenuItem 实例。

**CrimeListFragment.java**
```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()){
        case R.id.new_crime:
            Crime c = new Crime();
            CrimeLab.get(getActivity()).addCrime(c);
            Intent intent = CrimePagerActivity.newIntent(getActivity(),c.getId());
            startActivity(intent);
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}

```
目前应用主要是靠后退键导航，现在向应用添加层级式导航。
**AndroidManifest.xml**
```
<activity
    android:name=".CrimePagerActivity"
    android:parentActivityName=".CrimeListActivity">
</activity>
```
现在运行就能看到向上按钮了

![CrimePagerActivity 界面的向上按钮](https://upload-images.jianshu.io/upload_images/10095597-7f82a1ab42f52647.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/250)

虽然后退键导航和向上按钮导航执行的操作是一样的，但是各自实现的机制大不相同。向上按钮时，会在回退栈中寻找指定的 activity，如果实例存在，则弹出栈内所有的其他 activity，让启动的目标 activity 出现在栈顶。

## 子标题的显示

这里我们添加一个菜单项来显示或者隐藏 CrimelistActivity 工具栏的子标题。先来添加视图的代码

**res/menu/fragment_crime_list.xml**
```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/new_crime"
        android:icon="@drawable/ic_menu_add"
        android:title="@string/new_crime"
        app:showAsAction="ifRoom|withText"/>
    <item
        android:id="@+id/show_subtitle"
        android:title="@string/show_subtitle"
        app:showAsAction="ifRoom"/>
</menu>
```
创建方法实现这个功能

**CrimeListFragment.java**

```
private void updateSubtitle(){
    CrimeLab crimeLab = CrimeLab.get(getActivity());
    int crimeCount = crimeLab.getCrimes().size();
    String subtitle = getString(R.string.subtitle_format,crimeCount);

    AppCompatActivity activity = (AppCompatActivity) getActivity();
    activity.getSupportActionBar().setSubtitle(subtitle);
}
```

getString(...) 方法接收字符串资源中的占位符的替换值,然后在 onOptionsItemSelected (...) 点击响应此方法。

响应 SHOW SHBTITLE 的点击

```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()){
        case R.id.new_crime:
            Crime c = new Crime();
            CrimeLab.get(getActivity()).addCrime(c);
            Intent intent = CrimePagerActivity.newIntent(getActivity(),c.getId());
            startActivity(intent);
            return true;
        case R.id.show_subtitle:
            updateSubtitle();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

现在点击 SHOW SHBTITLE 按钮，就可以显示出来了。但是菜单项标题依然显示为 SHOW SHBTITLE，显然，菜单项标题的切换与子标题的显示或隐藏需要联动。

调用 onOptionsItemSelected(...) 方法时，可以更新 SHOW SUBTITLE 的文字，但是设备旋转时，子标题的变化就会丢失，比较好的解决方法是在 onCreateOptionsMenu(...) 方法内更新 SHOW SUBTITLE 菜单项，并在用户点击的时候重建工具栏。

**CrimeListFragment.java**
```
public class CrimeListFragment extends Fragment {
    private boolean mSubtitleVisible;
    @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        super.onCreateOptionsMenu(menu, inflater);
        inflater.inflate(R.menu.frigment_crime_list,menu);

        MenuItem subtitleItem = menu.findItem(R.id.show_subtitle);
        if (mSubtitleVisible){
            subtitleItem.setTitle(R.string.hide_subtitle);
        }else {
            subtitleItem.setTitle(R.string.show_subtitle);
        }
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()){
            case R.id.new_crime:
                Crime c = new Crime();
                CrimeLab.get(getActivity()).addCrime(c);
                Intent intent = CrimePagerActivity.newIntent(getActivity(),c.getId());
                startActivity(intent);
                return true;
            case R.id.show_subtitle:
                mSubtitleVisible  = !mSubtitleVisible;
                getActivity().invalidateOptionsMenu();
                updateSubtitle();
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }
}
```

最后，根据 mSubtitleVisible 变量值，联动菜单项标题与子标题

**CrimeListFragment.java**
```
private void updateSubtitle(){
    CrimeLab crimeLab = CrimeLab.get(getActivity());
    int crimeCount = crimeLab.getCrimes().size();
    String subtitle = getString(R.string.subtitle_format,crimeCount);

    if (!mSubtitleVisible){
        subtitle = null;
    }
    AppCompatActivity activity = (AppCompatActivity) getActivity();
    activity.getSupportActionBar().setSubtitle(subtitle);
}

```

## 数据的同步

接下来解决数据同步的问题，新建 Crime 后，使用后退键回到 CrimeListActivity，总次数不会更新，在 onResume() 方法中调用 updateSubtitle() 就能解决这个问题。因为 onResume() 和 onCreatView() 都会调用 updateUI() 方法，那就在 updateUI() 中调用 updateSubtitle().

**CrimeListFragment.java**
```
private void updateUI() {
    CrimeLab crimeLab = CrimeLab.get(getActivity());
    List<Crime> crimes = crimeLab.getCrimes();
    if (mAdapter==null){
        mAdapter = new CrimeAdapter(crimes);
        mCrimeRecyclerView.setAdapter(mAdapter);
    }else {
        mAdapter.notifyItemChanged(mPosition);
    }
    updateSubtitle();
}
```
但是点击向上按钮，子标题被重置了，这又是什么情况呢？这是 Android 实现层级导航带来的问题：导航回退到的目标 activity 会被完全重建。既然父 activity 是全新的。实例变量值以及保存的状态显示会彻底丢失。有两种解决方案，在本例中，调用 CrimePagerActivity 的 finlsh() 方法直接回退到前一个 activity 的界面。但是这样智能回退一个层级，而实际中绝大多数都需要多层级导航。第二种是在启动 CrimePagerActivity 时，把子标题状态作为 extra 信息传给它，然后在 CrimePagerActivity 中覆盖 getParentActivityIntent() 方法，用附带的 extra 信息的 intent 重建 CrimeListActivity。
 
还有一个问题，旋转设备后，子标题会消失，只要用实例状态保存机制，就能解决问题。

**CrimeListFragment.java**
```
public class CrimeListFragment extends Fragment {
    
    private static final String SAVED_SUBTITLE_VISIBLE = "subtitle";
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_crime_list, container, false);

        mCrimeRecyclerView = (RecyclerView) view
                .findViewById(R.id.crime_recycler_view);
        mCrimeRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
        if (savedInstanceState != null){
            mSubtitleVisible = savedInstanceState.getBoolean(SAVED_SUBTITLE_VISIBLE);
        }
        updateUI();

        return view;
    }
    
    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putBoolean(SAVED_SUBTITLE_VISIBLE,mSubtitleVisible);
    }
    
}
```

[简书地址](https://www.jianshu.com/p/e9e9f34c267c)

