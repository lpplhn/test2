# Fragment+ViewPager+BottomTab实现页面导航

原创 子林Android [子林Android](javascript:void(0);) *2024-03-10 08:02* *北京*

![img](http://mmbiz.qpic.cn/sz_mmbiz_png/p4Vib2xGTytzXuXWOLBdYctmDJpvjt1jMx7icXNC32GIhSgfyknusghGkUVFEqnSDSIxicibmxlm9w96EmEKcXiblag/300?wx_fmt=png&wxfrom=19)

**子林Android**

一个分享Android知识，教你入门Android开发的公众号，尤其面向小白零基础学习者。著有b站《手把手教你学会Android App开发》入门教程。 独木难成林，欢迎对Android感兴趣的小伙伴加入这片林子，一起学习、共同成长！

23篇原创内容



公众号

## 内容目录

\1. 目标效果2. 案例教学2.1 主界面布局2.2 准备Fragment2.3 主界面MainActivity2.4 ViewPager部分2.5 ViewPager联动底部导航按钮2.6 底部导航按钮联动ViewPager3. 完整代码

前文我们介绍了Fragment的应用之一，最简单的页面导航方式：[Fragment+BottomTab实现页面导航](http://mp.weixin.qq.com/s?__biz=MzI5OTQ2MDMwMw==&mid=2247483769&idx=1&sn=170d0fff3576e88be8310db5bb891891&chksm=ec9772d9dbe0fbcf3465dc1e88bfbc33c4919ff4aca2726a445664224786ddc9bd04b306eb43&scene=21#wechat_redirect)。这种方式虽然能实现点击底部导航按钮切换页面，但不够友好。我们更期望的是左右滑动页面也能实现页面切换，同时底部按钮的选中状态也跟着切换。本文就将介绍如何通过Fragment+ViewPager+BottomTab实现左右页面滑动切换这一效果。

# 1. 目标效果

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/p4Vib2xGTytyViaysPuMCRoxeQ5MKCSIFibesRkicykD2SWGB1zRDLTC4ibyibzWY3EcqGjsNZibQe8kwhWgzQSw8PgEQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)效果截图

# 2. 案例教学

## 2.1 主界面布局

首先，依然是主界面的布局。跟前文类似，依然是上下两部分，上面是页面主体内容区域，底部是导航按钮。只不过这次主体内容区域我们使用ViewPager这个控件来盛放多个Fragment。（还不熟悉ViewPager这个控件的小伙伴可以自行学习下它的基本用法，这里不展开讲述了）

布局文件activity_fragment_view_pager_bottom.xml：

```
 1<?xml version="1.0" encoding="utf-8"?>
 2<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 3    xmlns:app="http://schemas.android.com/apk/res-auto"
 4    xmlns:tools="http://schemas.android.com/tools"
 5    android:layout_width="match_parent"
 6    android:layout_height="match_parent"
 7    android:orientation="vertical">
 8
 9    <androidx.viewpager.widget.ViewPager
10        android:id="@+id/vp"
11        android:layout_width="match_parent"
12        android:layout_height="0dp"
13        android:layout_weight="1" />
14
15    <include layout="@layout/bottom_tab_layout" />
16
17</LinearLayout>
```

我们直接引用了前文代码的bottom_tab_layout，也就是底部导航按钮布局。这里看出将代码分开写的好处了吧，可以多处引入使用，非常方便。

## 2.2 准备Fragment

这里的Fragment我们也使用前文中的那个ExampleFragment，不再重复不必要的信息。详细介绍请看前文。

```
 1public class ExampleFragment extends Fragment {
 2    // 改为public，以便于外界能引用到
 3    public static final String ARG_PARAM1 = "param1";
 4    public static final String ARG_PARAM2 = "param2";
 5
 6    private String mParam1;
 7    private String mParam2;
 8
 9    private TextView mTvContent;
10
11    public ExampleFragment() {
12    }
13
14    public static ExampleFragment newInstance(String param1, String param2) {
15        ExampleFragment fragment = new ExampleFragment();
16        Bundle args = new Bundle();
17        args.putString(ARG_PARAM1, param1);
18        args.putString(ARG_PARAM2, param2);
19        fragment.setArguments(args);
20        return fragment;
21    }
22
23    @Override
24    public void onCreate(Bundle savedInstanceState) {
25        super.onCreate(savedInstanceState);
26        if (getArguments() != null) {
27            mParam1 = getArguments().getString(ARG_PARAM1);
28            mParam2 = getArguments().getString(ARG_PARAM2);
29        }
30    }
31
32    @Override
33    public View onCreateView(LayoutInflater inflater, ViewGroup container,
34                             Bundle savedInstanceState) {
35        return inflater.inflate(R.layout.fragment_example, container, false);
36    }
37
38    @Override
39    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
40        super.onViewCreated(view, savedInstanceState);
41        mTvContent = view.findViewById(R.id.tv_content);
42        // 如果传入的参数有值，则设置内容
43        if (!TextUtils.isEmpty(mParam1)) {
44            mTvContent.setText(mParam1);
45        }
46    }
47}
```

## 2.3 主界面MainActivity

接下来是代码部分。首先还是将控件做一些基础的声明和初始化。

```
 1/**
 2 * viewpager+fragment+bottomTab
 3 * 普通的控件拼接成的底部导航菜单
 4 */
 5public class FragmentViewPagerBottomActivity extends AppCompatActivity implements View.OnClickListener {
 6
 7    private RelativeLayout mRlHome, mRlSetting, mRlMine;
 8    private TextView mTvHome, mTvFind, mTvMine;
 9    private ImageView mIvHome, mIvFind, mIvMine;
10    private ViewPager mViewPager;
11
12    @Override
13    protected void onCreate(Bundle savedInstanceState) {
14        super.onCreate(savedInstanceState);
15        setContentView(R.layout.activity_fragment_view_pager_bottom);
16
17        initView();
18        initData();
19        initEvent();
20    }
21
22    private void initView() {
23
24        mViewPager = findViewById(R.id.vp);
25        mRlHome = findViewById(R.id.rl_home);
26        mRlSetting = findViewById(R.id.rl_find);
27        mRlMine = findViewById(R.id.rl_mine);
28
29        mTvMine = findViewById(R.id.tv_mine);
30        mTvFind = findViewById(R.id.tv_find);
31        mTvHome = findViewById(R.id.tv_home);
32
33        mIvMine = findViewById(R.id.iv_mine);
34        mIvFind = findViewById(R.id.iv_find);
35        mIvHome = findViewById(R.id.iv_home);
36    }
37
38    private void initData() {
39
40    }
41
42    private void initEvent() {
43        // 为 底部导航按钮 设置点击事件监听
44        mRlHome.setOnClickListener(this);
45        mRlSetting.setOnClickListener(this);
46        mRlMine.setOnClickListener(this);
47    }
48
49    @Override
50    public void onClick(View v) {
51        // 底部导航按钮点击后响应
52
53    }
54 }
```

## 2.4 ViewPager部分

ViewPager是一种方便多个页面滑动切换的控件，页面可以是View，也可以是Fragment，常常用来做轮播图、引导页面等。像ListView、RecyclerView一样，这种集合型的控件有个共同点：数据和View分离，中间用适配器adapter连接。

View我们已经有了，就是ViewPager，接下来准备数据。这里的数据就是多个Fragment页面

```
 1// 全局变量
 2private List<Fragment> mFragmentList;
 3
 4// 添加用来测试的Frament，这里添加三个
 5private void initData() {
 6    mFragmentList = new ArrayList<>();
 7    for (int i = 0; i < 3; i++) {
 8        ExampleFragment fragment = ExampleFragment.newInstance("fragment" + i, "");
 9        mFragmentList.add(fragment);
10    }
11}
```

最后是关键的适配器，将数据和View适配起来。MyViewPagerAdapterForFragment.java：

```
 1public class MyViewPagerAdapterForFragment extends FragmentPagerAdapter {
 2
 3    private List<Fragment> mFragmentList;
 4
 5    public MyViewPagerAdapterForFragment(@NonNull FragmentManager fm, List<Fragment> fragmentList) {
 6        super(fm);
 7        mFragmentList = fragmentList;
 8    }
 9
10    @NonNull
11    @Override
12    public Fragment getItem(int position) {
13        return mFragmentList == null ? null : mFragmentList.get(position);
14    }
15
16    @Override
17    public int getCount() {
18        return mFragmentList == null ? 0 : mFragmentList.size();
19    }
20}
```

这个适配器也比较简单，继承FragmentPagerAdapter，然后实现其中的两个方法，getItem、getCount：获取每一个Fragment，获取Fragment的总个数。数据源是mFragmentList，通过构造方法传进来。

看看在Activity中如何把这三者（ViewPager、mFragmentList、FragmentPagerAdapter）连接起来。
代码如下：

```
 1// 全局变量
 2private MyViewPagerAdapterForFragment mMyViewPagerAdapter;
 3
 4private void initEvent() {
 5    // 省略上文代码...
 6    // 创建adapter
 7    mMyViewPagerAdapter = new MyViewPagerAdapterForFragment(getSupportFragmentManager(), mFragmentList);
 8    // 为ViewPager设置adapter
 9    mViewPager.setAdapter(mMyViewPagerAdapter);
10}
```

也简单，只需要创建出adapter，再为ViewPager设置进去就行了。创建的时候传进去了数据源mFragmentList，这样就把三者连接起来了。
到这里，运行代码就已经可以看到三个页面，并且可以滑动左右切换了。但是底部导航按钮还没有变化，别急，我们接着往下看。

## 2.5 ViewPager联动底部导航按钮

上文我们做了ViewPager的滑动切换页面部分，我们期望的是滑动上面的页面，底部的按钮也跟着状态切换。那么就要求我们监听ViewPager页面滑动，然后响应方法里对按钮做状态切换。
在initEvent方法里继续添加代码：

```
 1// 为ViewPager添加页面切换监听
 2mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
 3   @Override
 4   public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
 5   }
 6
 7   @Override
 8   public void onPageSelected(int position) {   
 9       Toast.makeText(FragmentViewPagerBottomActivity.this, "当前第" + position + "页", Toast.LENGTH_SHORT).show();
10       // 页面切换后的处理
11       onPagerSelected(position);
12   }
13
14   @Override
15   public void onPageScrollStateChanged(int state) {
16   }
17});
```

主要是 onPageSelected 这个回调方法，当真正的滑到了另一个页面时会调用它，参数position表示当前第几个页面，从0开始计数。
我们在里面加了Toast，方便观察当前第几个页面。接着我们需要处理底部导航按钮状态切换的逻辑，这部分我们封装成一个方法来处理，即：onPagerSelected方法。

```
 1private void onPagerSelected(int position) {
 2    resetBottomTab();
 3    switch (position) {
 4        case 0:
 5            mIvHome.setSelected(true);
 6            mTvHome.setTextColor(getResources().getColor(R.color.green_500));
 7            break;
 8        case 1:
 9            mIvFind.setSelected(true);
10            mTvFind.setTextColor(getResources().getColor(R.color.green_500));
11            break;
12        case 2:
13            mIvMine.setSelected(true);
14            mTvMine.setTextColor(getResources().getColor(R.color.green_500));
15            break;
16        default:
17            break;
18    }
19}
20
21private void resetBottomTab() {
22     mTvHome.setTextColor(getResources().getColor(R.color.black));
23     mTvFind.setTextColor(getResources().getColor(R.color.black));
24     mTvMine.setTextColor(getResources().getColor(R.color.black));
25
26     mIvHome.setSelected(false);
27     mIvFind.setSelected(false);
28     mIvMine.setSelected(false);
29 }
```

跟前文类似，先重置所有按钮状态（恢复成都不选中），再根据当前的选中的position，设置对应的按钮为绿色。
这样就完成了ViewPager页面切换与底部按钮的联动。

这就结束了吗？
没有，不要忘了ViewPager和底部按钮的联动需要是**双向的**。上面只是页面切换 --> 底部按钮状态变换，还需要反过来：点击底部导航导航按钮 --> 页面切换。

## 2.6 底部导航按钮联动ViewPager

这一步也很简单。前文我们已经为底部按钮设置了点击监听，并且预留了onClick方法。接下来只需要在onClick方法中做页面切换的逻辑就行了，我们依然封装成一个方法来实现。

```
 1@Override
 2public void onClick(View v) {
 3    setBottomTabSelected(v.getId());
 4}
 5private void setBottomTabSelected(int tabId) {
 6    switch (tabId) {
 7        case R.id.rl_home:
 8            mViewPager.setCurrentItem(0);
 9            break;
10        case R.id.rl_find:
11            mViewPager.setCurrentItem(1);
12            break;
13        case R.id.rl_mine:
14            mViewPager.setCurrentItem(2);
15            break;
16        default:
17            break;
18    }
19}
```

可见，切换ViewPager里的页面只需要一行代码：`mViewPager.setCurrentItem(position);` ViewPager还是很方便的吧。

结束了吗？
还差最后一步，跟前文一样，最后我们还需要设置一个默认进入的页面，也就是让ViewPager默认显示哪一页。
在initEvent方法最后添加：`onPagerSelected(0);`即可。这就是封装方法的好处，体会到了吗。

# 3. 完整代码

终于，我们完成了整个案例。下面附上整个主界面FragmentViewPagerBottomActivity的完整代码。

至于其他的资源文件和前文[Fragment+BottomTab实现页面导航](http://mp.weixin.qq.com/s?__biz=MzI5OTQ2MDMwMw==&mid=2247483769&idx=1&sn=170d0fff3576e88be8310db5bb891891&chksm=ec9772d9dbe0fbcf3465dc1e88bfbc33c4919ff4aca2726a445664224786ddc9bd04b306eb43&scene=21#wechat_redirect)一样，不再复述。

```
  1/**
  2 * viewpager+fragment+bottomTab
  3 * 普通的控件拼接成的底部导航菜单
  4 */
  5public class FragmentViewPagerBottomActivity extends AppCompatActivity implements View.OnClickListener {
  6
  7    private RelativeLayout mRlHome, mRlSetting, mRlMine;
  8    private TextView mTvHome, mTvFind, mTvMine;
  9    private ImageView mIvHome, mIvFind, mIvMine;
 10    private ViewPager mViewPager;
 11
 12    private List<Fragment> mFragments;
 13    private MyViewPagerAdapterForFragment mMyViewPagerAdapter;
 14
 15
 16    @Override
 17    protected void onCreate(Bundle savedInstanceState) {
 18        super.onCreate(savedInstanceState);
 19        setContentView(R.layout.activity_fragment_view_pager_bottom);
 20
 21        initView();
 22        initData();
 23        initEvent();
 24    }
 25
 26
 27    private void initData() {
 28        mFragments = new ArrayList<>();
 29        for (int i = 0; i < 3; i++) {
 30            ExampleFragment fragment = ExampleFragment.newInstance("fragment" + i, "");
 31            mFragments.add(fragment);
 32        }
 33    }
 34
 35    private void initEvent() {
 36        mRlHome.setOnClickListener(this);
 37        mRlSetting.setOnClickListener(this);
 38        mRlMine.setOnClickListener(this);
 39
 40        mMyViewPagerAdapter = new MyViewPagerAdapterForFragment(getSupportFragmentManager(), mFragments);
 41        mViewPager.setAdapter(mMyViewPagerAdapter);
 42        mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
 43            @Override
 44            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
 45
 46            }
 47
 48            @Override
 49            public void onPageSelected(int position) {
 50                onPagerSelected(position);
 51                Toast.makeText(FragmentViewPagerBottomActivity.this, "当前第" + position + "页", Toast.LENGTH_SHORT).show();
 52            }
 53
 54            @Override
 55            public void onPageScrollStateChanged(int state) {
 56
 57            }
 58        });
 59        onPagerSelected(0);
 60    }
 61
 62    private void initView() {
 63
 64        mViewPager = findViewById(R.id.vp);
 65        mRlHome = findViewById(R.id.rl_home);
 66        mRlSetting = findViewById(R.id.rl_find);
 67        mRlMine = findViewById(R.id.rl_mine);
 68
 69        mTvMine = findViewById(R.id.tv_mine);
 70        mTvFind = findViewById(R.id.tv_find);
 71        mTvHome = findViewById(R.id.tv_home);
 72
 73        mIvMine = findViewById(R.id.iv_mine);
 74        mIvFind = findViewById(R.id.iv_find);
 75        mIvHome = findViewById(R.id.iv_home);
 76    }
 77
 78    @Override
 79    public void onClick(View v) {
 80        setBottomTabSelected(v.getId());
 81    }
 82
 83    private void resetBottomTab() {
 84        mTvHome.setTextColor(getResources().getColor(R.color.black));
 85        mTvFind.setTextColor(getResources().getColor(R.color.black));
 86        mTvMine.setTextColor(getResources().getColor(R.color.black));
 87
 88        mIvHome.setSelected(false);
 89        mIvFind.setSelected(false);
 90        mIvMine.setSelected(false);
 91    }
 92
 93    private void onPagerSelected(int position) {
 94        resetBottomTab();
 95        switch (position) {
 96            case 0:
 97                mIvHome.setSelected(true);
 98                mTvHome.setTextColor(getResources().getColor(R.color.green_500));
 99                break;
100            case 1:
101                mIvFind.setSelected(true);
102                mTvFind.setTextColor(getResources().getColor(R.color.green_500));
103                break;
104            case 2:
105                mIvMine.setSelected(true);
106                mTvMine.setTextColor(getResources().getColor(R.color.green_500));
107                break;
108            default:
109                break;
110        }
111    }
112
113    private void setBottomTabSelected(int tabId) {
114        switch (tabId) {
115            case R.id.rl_home:
116                mViewPager.setCurrentItem(0);
117                break;
118            case R.id.rl_find:
119                mViewPager.setCurrentItem(1);
120                break;
121            case R.id.rl_mine:
122                mViewPager.setCurrentItem(2);
123                break;
124            default:
125                break;
126        }
127    }
128}
```

以上就是 Fragment+ViewPager+BottomTab 实现底部导航的典型做法了。如果文章对你有用，欢迎支持，谢谢！





# Fragment+ViewPager+BottomNavigationView实现页面导航

原创 子林Android [子林Android](javascript:void(0);) *2024-03-08 08:03* *北京*

![img](http://mmbiz.qpic.cn/sz_mmbiz_png/p4Vib2xGTytzXuXWOLBdYctmDJpvjt1jMx7icXNC32GIhSgfyknusghGkUVFEqnSDSIxicibmxlm9w96EmEKcXiblag/300?wx_fmt=png&wxfrom=19)

**子林Android**

一个分享Android知识，教你入门Android开发的公众号，尤其面向小白零基础学习者。著有b站《手把手教你学会Android App开发》入门教程。 独木难成林，欢迎对Android感兴趣的小伙伴加入这片林子，一起学习、共同成长！

23篇原创内容



公众号

## 内容目录

\1. 目标效果2. 案例教学2.1 主界面布局2.2 准备Fragment2.3 主界面MainActivity2.4 ViewPager部分2.5 ViewPager联动导航按钮BottomNavigationView2.6 导航按钮BottomNavigationView联动ViewPager3. 完整代码

上篇文章[Fragment+BottomTab实现页面导航](http://mp.weixin.qq.com/s?__biz=MzI5OTQ2MDMwMw==&mid=2247483769&idx=1&sn=170d0fff3576e88be8310db5bb891891&chksm=ec9772d9dbe0fbcf3465dc1e88bfbc33c4919ff4aca2726a445664224786ddc9bd04b306eb43&scene=21#wechat_redirect)我们学习了Fragment+ViewPager+BottomTab实现页面导航，实现了左右滑动切换页面。但我们底部导航按钮的实现还比较原始，使用了大量的控件拼凑而成，相对麻烦。本文将对上节案例做进一步的优化，使用BottomNavigationView这个控件快捷的实现导航按钮。

# 1. 目标效果

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/p4Vib2xGTytw4ImjbNXMC32shAtDwYiaoLCgtD2ibFuftICsWibaJ2hSJubdc5kODvJJ0jvZXv8rzdSMxKWTfAW0AA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)效果截图

# 2. 案例教学

本节内容是在上节案例[Fragment+BottomTab实现页面导航](http://mp.weixin.qq.com/s?__biz=MzI5OTQ2MDMwMw==&mid=2247483769&idx=1&sn=170d0fff3576e88be8310db5bb891891&chksm=ec9772d9dbe0fbcf3465dc1e88bfbc33c4919ff4aca2726a445664224786ddc9bd04b306eb43&scene=21#wechat_redirect)基础上的优化，因此很多细节不再重复，详情参考上文即可。

## 2.1 主界面布局

activity_fragment_view_pager_bottom_navigation.xml

```
 1<?xml version="1.0" encoding="utf-8"?>
 2<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 3    xmlns:app="http://schemas.android.com/apk/res-auto"
 4    xmlns:tools="http://schemas.android.com/tools"
 5    android:layout_width="match_parent"
 6    android:layout_height="match_parent"
 7    android:orientation="vertical">
 8
 9    <androidx.viewpager.widget.ViewPager
10        android:id="@+id/vp"
11        android:layout_width="match_parent"
12        android:layout_height="0dp"
13        android:layout_weight="1" />
14
15    <com.google.android.material.bottomnavigation.BottomNavigationView
16        android:id="@+id/bnv"
17        android:layout_width="match_parent"
18        android:layout_height="wrap_content"
19        android:background="?android:windowBackground"
20        app:itemRippleColor="@color/green_700"
21        app:labelVisibilityMode="labeled"
22        app:menu="@menu/bottom_nav_menu" />
23
24</LinearLayout>
```

布局结构还是类似的，只不过上文的导航按钮部分`<include layout="@layout/bottom_tab_layout" />`被替换为了 BottomNavigationView。这个控件是material库里的，如果你引用不到，检查下build.gradle文件的dependencies节点里是否添加了依赖`implementation 'com.google.android.material:material:1.4.0'`。添加上即可。

我们来具体看下BottomNavigationView里面几个属性配置：

- app:menu
  声明导航按钮，它的值是一个menu文件。具体如下：

```
 1<?xml version="1.0" encoding="utf-8"?>
 2<menu xmlns:android="http://schemas.android.com/apk/res/android">
 3
 4    <item
 5        android:id="@+id/menu_home"
 6        android:icon="@drawable/selector_home"
 7        android:title="首页" />
 8
 9    <item
10        android:id="@+id/menu_find"
11        android:icon="@drawable/selector_find"
12        android:title="发现" />
13
14    <item
15        android:id="@+id/menu_mine"
16        android:icon="@drawable/selector_mine"
17        android:title="我的" />
18</menu>
```

可见，跟上文我们用控件拼接出的按钮类似，也主要包括icon、文字。

- app:labelVisibilityMode
  导航按钮的显示模式，取值有auto、selected、labeled、unlabeled

- - labeled : 几个导航按钮全部显示文字
  - unlabeled : 几个导航按钮全部不显示文字
  - selected : 只有选中的按钮显示文字
  - auto : 小于等于3个按钮时全部显示文字，即labeled；大于3个按钮时只有选中的按钮显示文字，即selected

- app:itemRippleColor
  点击导航按钮会有波纹效果，这里可以使用这个属性配置波纹的颜色

## 2.2 准备Fragment

与上文一致，直接贴代码

```
 1public class ExampleFragment extends Fragment {
 2    // 改为public，以便于外界能引用到
 3    public static final String ARG_PARAM1 = "param1";
 4    public static final String ARG_PARAM2 = "param2";
 5
 6    private String mParam1;
 7    private String mParam2;
 8
 9    private TextView mTvContent;
10
11    public ExampleFragment() {
12    }
13
14    public static ExampleFragment newInstance(String param1, String param2) {
15        ExampleFragment fragment = new ExampleFragment();
16        Bundle args = new Bundle();
17        args.putString(ARG_PARAM1, param1);
18        args.putString(ARG_PARAM2, param2);
19        fragment.setArguments(args);
20        return fragment;
21    }
22
23    @Override
24    public void onCreate(Bundle savedInstanceState) {
25        super.onCreate(savedInstanceState);
26        if (getArguments() != null) {
27            mParam1 = getArguments().getString(ARG_PARAM1);
28            mParam2 = getArguments().getString(ARG_PARAM2);
29        }
30    }
31
32    @Override
33    public View onCreateView(LayoutInflater inflater, ViewGroup container,
34                             Bundle savedInstanceState) {
35        return inflater.inflate(R.layout.fragment_example, container, false);
36    }
37
38    @Override
39    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
40        super.onViewCreated(view, savedInstanceState);
41        mTvContent = view.findViewById(R.id.tv_content);
42        // 如果传入的参数有值，则设置内容
43        if (!TextUtils.isEmpty(mParam1)) {
44            mTvContent.setText(mParam1);
45        }
46    }
47}
```

## 2.3 主界面MainActivity

首先还是控件初始化，由于使用了BottomNavigationView替代导航按钮，本节的控件就变得很少了。

```
 1public class FragmentViewPagerBottomNavigationActivity extends AppCompatActivity {
 2
 3    private ViewPager mViewPager;
 4    private BottomNavigationView mBottomNavigationView;
 5
 6    @Override
 7    protected void onCreate(Bundle savedInstanceState) {
 8        super.onCreate(savedInstanceState);
 9        setContentView(R.layout.activity_fragment_view_pager_bottom_navigation);
10
11        initView();
12        initData();
13        initEvent();
14    }
15
16    private void initView() {
17        mViewPager = findViewById(R.id.vp);
18        mBottomNavigationView = findViewById(R.id.bnv);
19    }
20
21    private void initData() {
22
23    }
24
25    private void initEvent() {
26
27    }
28
29}
```

只有ViewPager 和 BottomNavigationView两个，简单许多了吧。

## 2.4 ViewPager部分

与上文[Fragment+BottomTab实现页面导航](http://mp.weixin.qq.com/s?__biz=MzI5OTQ2MDMwMw==&mid=2247483769&idx=1&sn=170d0fff3576e88be8310db5bb891891&chksm=ec9772d9dbe0fbcf3465dc1e88bfbc33c4919ff4aca2726a445664224786ddc9bd04b306eb43&scene=21#wechat_redirect)一样，不再重复。

- 数据部分

```
1private List<Fragment> mFragmentList;
2
3 private void initData() {
4    mFragments = new ArrayList<>();
5    for (int i = 0; i < 3; i++) {
6        ExampleFragment fragment = ExampleFragment.newInstance("fragment" + i, "");
7        mFragmentList.add(fragment);
8    }
9}
```

- 适配器部分
  MyViewPagerAdapterForFragment.java ：

```
 1public class MyViewPagerAdapterForFragment extends FragmentPagerAdapter {
 2
 3    private List<Fragment> mFragmentList;
 4
 5    public MyViewPagerAdapterForFragment(@NonNull FragmentManager fm, List<Fragment> fragmentList) {
 6        super(fm);
 7        mFragmentList = fragmentList;
 8    }
 9
10    @NonNull
11    @Override
12    public Fragment getItem(int position) {
13        return mFragmentList == null ? null : mFragmentList.get(position);
14    }
15
16    @Override
17    public int getCount() {
18        return mFragmentList == null ? 0 : mFragmentList.size();
19    }
20}
```

- 数据与viewPager通过适配器绑定

```
1private void initEvent() {
2    mMyViewPagerAdapter = new MyViewPagerAdapterForFragment(getSupportFragmentManager(), mFragmentList);
3    mViewPager.setAdapter(mMyViewPagerAdapter);
4}
```

## 2.5 ViewPager联动导航按钮BottomNavigationView

为ViewPager添加页面切换的监听
在initEvent方法里继续添加代码：

```
 1// 为ViewPager添加页面切换监听
 2mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
 3   @Override
 4   public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
 5   }
 6
 7   @Override
 8   public void onPageSelected(int position) {   
 9       Toast.makeText(FragmentViewPagerBottomNavigationActivity.this, "当前第" + position + "页", Toast.LENGTH_SHORT).show();
10       // 页面切换后的处理
11       onPagerSelected(position);
12   }
13
14   @Override
15   public void onPageScrollStateChanged(int state) {
16   }
17});
```

页面切换后的处理逻辑封装到了方法onPagerSelected中：

```
 1private void onPagerSelected(int position) {
 2    switch (position) {
 3        case 0:
 4            mBottomNavigationView.setSelectedItemId(R.id.menu_home);
 5            break;
 6        case 1:
 7            mBottomNavigationView.setSelectedItemId(R.id.menu_find);
 8            break;
 9        case 2:
10            mBottomNavigationView.setSelectedItemId(R.id.menu_mine);
11            break;
12    }
13}
```

通过调用`mBottomNavigationView.setSelectedItemId`方法，传入要切换到的按钮id即可完成按钮状态的切换，包括图标颜色和文字颜色。再也不用像上文里那样，需要我们手动重置所有按钮文字颜色，然后再设置某个按钮为绿色那么麻烦了。
怎么样，BottomNavigationView 方便吧，嗯，真香～

## 2.6 导航按钮BottomNavigationView联动ViewPager

同样的，点击导航按钮ViewPager页面也要跟着切换。BottomNavigationView这种多功能控件当然也支持切换监听了。
在initEvent方法中继续添加：

```
 1mBottomNavigationView.setOnItemSelectedListener(new NavigationBarView.OnItemSelectedListener() {
 2    @Override
 3    public boolean onNavigationItemSelected(@NonNull MenuItem item) {
 4
 5        switch (item.getItemId()) {
 6            case R.id.menu_home:
 7                mViewPager.setCurrentItem(0);
 8                return true;
 9            case R.id.menu_setting:
10                mViewPager.setCurrentItem(1);
11                return true;
12            case R.id.menu_mine:
13                mViewPager.setCurrentItem(2);
14                return true;
15        }
16        return false;
17    }
18});
```

当某个按钮被选中时，回调onNavigationItemSelected，再根据menuItem的id做页面切换就行了。

最后，为用户设置默认进入的页面。只需调用`mBottomNavigationView.setSelectedItemId(R.id.menu_home);`就可设置默认选中首页这个按钮，回调后进而设置ViewPager选中首页。

# 3. 完整代码

到此，整个页面导航功能已经完成了。下面附上整个主界面FragmentViewPagerBottomNavigationActivity的完整代码。

```
 1public class FragmentViewPagerBottomNavigationActivity extends AppCompatActivity {
 2
 3    private ViewPager mViewPager;
 4    private BottomNavigationView mBottomNavigationView;
 5    private List<Fragment> mFragmentList;
 6    private MyViewPagerAdapterForFragment mMyViewPagerAdapter;
 7
 8    @Override
 9    protected void onCreate(Bundle savedInstanceState) {
10        super.onCreate(savedInstanceState);
11        setContentView(R.layout.activity_fragment_view_pager_bottom_navigation);
12
13        initView();
14        initData();
15        initEvent();
16    }
17
18    private void initView() {
19        mViewPager = findViewById(R.id.vp);
20        mBottomNavigationView = findViewById(R.id.bnv);
21    }
22
23    private void initData() {
24        mFragmentList = new ArrayList<>();
25        for (int i = 0; i < 3; i++) {
26            ExampleFragment fragment = ExampleFragment.newInstance("fragment" + i, "");
27            mFragmentList.add(fragment);
28        }
29    }
30
31    private void initEvent() {
32        mMyViewPagerAdapter = new MyViewPagerAdapterForFragment(getSupportFragmentManager(), mFragmentList);
33        mViewPager.setAdapter(mMyViewPagerAdapter);
34        mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
35            @Override
36            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
37
38            }
39
40            @Override
41            public void onPageSelected(int position) {
42                onPagerSelected(position);
43                Toast.makeText(FragmentViewPagerBottomNavigationActivity.this, "当前第" + position + "页", Toast.LENGTH_SHORT).show();
44            }
45
46            @Override
47            public void onPageScrollStateChanged(int state) {
48
49            }
50        });
51    }
52
53    private void onPagerSelected(int position) {
54        switch (position) {
55            case 0:
56                mBottomNavigationView.setSelectedItemId(R.id.menu_home);
57                break;
58            case 1:
59                mBottomNavigationView.setSelectedItemId(R.id.menu_find);
60                break;
61            case 2:
62                mBottomNavigationView.setSelectedItemId(R.id.menu_mine);
63                break;
64        }
65    }
66
67}
```

以上就是 Fragment 结合 BottomNavigationView 实现底部导航的典型做法了。如果文章对你有用，欢迎支持，谢谢！

![img](http://mmbiz.qpic.cn/sz_mmbiz_png/p4Vib2xGTytzXuXWOLBdYctmDJpvjt1jMx7icXNC32GIhSgfyknusghGkUVFEqnSDSIxicibmxlm9w96EmEKcXiblag/300?wx_fmt=png&wxfrom=19)





# Fragment+ViewPager实现多级多Tab导航页面

原创 子林Android [子林Android](javascript:void(0);) *2024-03-09 15:55* *北京*

![img](http://mmbiz.qpic.cn/sz_mmbiz_png/p4Vib2xGTytzXuXWOLBdYctmDJpvjt1jMx7icXNC32GIhSgfyknusghGkUVFEqnSDSIxicibmxlm9w96EmEKcXiblag/300?wx_fmt=png&wxfrom=19)

**子林Android**

一个分享Android知识，教你入门Android开发的公众号，尤其面向小白零基础学习者。著有b站《手把手教你学会Android App开发》入门教程。 独木难成林，欢迎对Android感兴趣的小伙伴加入这片林子，一起学习、共同成长！

23篇原创内容



公众号

## 内容目录

\1. 目标效果2. 案例教学2.1 主界面布局2.2 主界面Fragment2.3 主界面Activity2.4 多个子栏目的承载者HomeFragment2.4.1 界面布局2.4.2 代码部分3. 总结

上节我们介绍了引入BottomNavigationView实现页面导航[Fragment+ViewPager+BottomNavigationView实现页面导航](http://mp.weixin.qq.com/s?__biz=MzI5OTQ2MDMwMw==&mid=2247483776&idx=1&sn=1e08ec47495dbbee60aab5730a9e4e9d&chksm=ec977220dbe0fb363fff94d257418bc447761b10820cca2116efceda874bde2dd86197f48983&scene=21#wechat_redirect)的方法，大大简化了底部导航按钮的实现，详见：Fragment+ViewPager+BottomNavigationView实现页面导航本节我们将对Fragment+ViewPager+BottomNavigationView这种页面结构进一步丰富，引入**子Fragment**，实现外层页面导航的前提下，首页内部又有**多个栏目页面**的结构。

# 1. 目标效果

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/p4Vib2xGTytw4ImjbNXMC32shAtDwYiaoLPRNzUrmeqMMwclAUXD6HibF6vDpPEj7z3yT6nS9uD4J4v5uts0NXWOA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在这里插入图片描述

# 2. 案例教学

本节内容依然是在上节[Fragment+ViewPager+BottomNavigationView实现页面导航](http://mp.weixin.qq.com/s?__biz=MzI5OTQ2MDMwMw==&mid=2247483776&idx=1&sn=1e08ec47495dbbee60aab5730a9e4e9d&chksm=ec977220dbe0fb363fff94d257418bc447761b10820cca2116efceda874bde2dd86197f48983&scene=21#wechat_redirect)基础上的进一步完善，细节参考上节。

## 2.1 主界面布局

与上节一致，直接贴代码。

activity_fragment_view_pager_bottom_navigation2.xml:

```
 1<?xml version="1.0" encoding="utf-8"?>
 2<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 3    xmlns:app="http://schemas.android.com/apk/res-auto"
 4    xmlns:tools="http://schemas.android.com/tools"
 5    android:layout_width="match_parent"
 6    android:layout_height="match_parent"
 7    android:orientation="vertical">
 8
 9    <androidx.viewpager.widget.ViewPager
10        android:id="@+id/vp"
11        android:layout_width="match_parent"
12        android:layout_height="0dp"
13        android:layout_weight="1" />
14
15    <com.google.android.material.bottomnavigation.BottomNavigationView
16        android:id="@+id/bnv"
17        android:layout_width="match_parent"
18        android:layout_height="wrap_content"
19        android:background="?android:windowBackground"
20        app:itemRippleColor="@color/green_700"
21        app:itemTextAppearanceInactive="@color/green_500"
22        app:labelVisibilityMode="labeled"
23        app:menu="@menu/bottom_nav_menu" />
24
25</LinearLayout>
```

bottom_nav_menu.xml:

```
 1<?xml version="1.0" encoding="utf-8"?>
 2<menu xmlns:android="http://schemas.android.com/apk/res/android">
 3
 4    <item
 5        android:id="@+id/menu_home"
 6        android:icon="@drawable/selector_home"
 7        android:title="首页" />
 8
 9    <item
10        android:id="@+id/menu_find"
11        android:icon="@drawable/selector_find"
12        android:title="发现" />
13
14    <item
15        android:id="@+id/menu_mine"
16        android:icon="@drawable/selector_mine"
17        android:title="我的" />
18</menu>
```

## 2.2 主界面Fragment

```
 1public class ExampleFragment extends Fragment {
 2    // 改为public，以便于外界能引用到
 3    public static final String ARG_PARAM1 = "param1";
 4    public static final String ARG_PARAM2 = "param2";
 5
 6    private String mParam1;
 7    private String mParam2;
 8
 9    private TextView mTvContent;
10
11    public ExampleFragment() {
12    }
13
14    public static ExampleFragment newInstance(String param1, String param2) {
15        ExampleFragment fragment = new ExampleFragment();
16        Bundle args = new Bundle();
17        args.putString(ARG_PARAM1, param1);
18        args.putString(ARG_PARAM2, param2);
19        fragment.setArguments(args);
20        return fragment;
21    }
22
23    @Override
24    public void onCreate(Bundle savedInstanceState) {
25        super.onCreate(savedInstanceState);
26        if (getArguments() != null) {
27            mParam1 = getArguments().getString(ARG_PARAM1);
28            mParam2 = getArguments().getString(ARG_PARAM2);
29        }
30    }
31
32    @Override
33    public View onCreateView(LayoutInflater inflater, ViewGroup container,
34                             Bundle savedInstanceState) {
35        return inflater.inflate(R.layout.fragment_example, container, false);
36    }
37
38    @Override
39    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
40        super.onViewCreated(view, savedInstanceState);
41        mTvContent = view.findViewById(R.id.tv_content);
42        // 如果传入的参数有值，则设置内容
43        if (!TextUtils.isEmpty(mParam1)) {
44            mTvContent.setText(mParam1);
45        }
46    }
47}
```

## 2.3 主界面Activity

FragmentViewPagerBottomNavigationActivity2.java

```
 1public class FragmentViewPagerBottomNavigationActivity2 extends AppCompatActivity {
 2
 3    private ViewPager mViewPager;
 4    private List<Fragment> mFragments;
 5    private MyViewPagerAdapterForFragment mMyViewPagerAdapter;
 6    private BottomNavigationView mBottomNavigationView;
 7
 8    @Override
 9    protected void onCreate(Bundle savedInstanceState) {
10        super.onCreate(savedInstanceState);
11        setContentView(R.layout.activity_fragment_view_pager_bottom_navigation2);
12
13        initView();
14        initData();
15        initEvent();
16
17    }
18
19    private void initView() {
20        mViewPager = findViewById(R.id.vp);
21        mBottomNavigationView = findViewById(R.id.bnv);
22    }
23
24    private void initData() {
25        mFragments = new ArrayList<>();
26        // 承载多个栏目的主Fragment，把它单独用一个Fragment类来写，下文会对HomeFragment详述
27        mFragments.add(new HomeFragment());
28        // 这里是剩余两页：发现页面、我的页面
29        for (int i = 1; i < 3; i++) {
30            ExampleFragment fragment = ExampleFragment.newInstance("fragment" + i, "");
31            mFragments.add(fragment);
32        }
33    }
34
35    private void initEvent() {
36
37        mMyViewPagerAdapter = new MyViewPagerAdapterForFragment(getSupportFragmentManager(), mFragments);
38        mViewPager.setAdapter(mMyViewPagerAdapter);
39        mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
40            @Override
41            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
42
43            }
44
45            @Override
46            public void onPageSelected(int position) {
47                onPagerSelected(position);
48                Toast.makeText(FragmentViewPagerBottomNavigationActivity2.this, "当前第" + position + "页", Toast.LENGTH_SHORT).show();
49            }
50
51            @Override
52            public void onPageScrollStateChanged(int state) {
53
54            }
55        });
56
57        mBottomNavigationView.setOnItemSelectedListener(new NavigationBarView.OnItemSelectedListener() {
58            @Override
59            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
60
61                switch (item.getItemId()) {
62                    case R.id.menu_home:
63                        mViewPager.setCurrentItem(0);
64                        return true;
65                    case R.id.menu_find:
66                        mViewPager.setCurrentItem(1);
67                        return true;
68                    case R.id.menu_mine:
69                        mViewPager.setCurrentItem(2);
70                        return true;
71                }
72                return false;
73            }
74        });
75        mBottomNavigationView.setSelectedItemId(R.id.menu_home);
76
77    }
78
79    private void onPagerSelected(int position) {
80        switch (position) {
81            case 0:
82                mBottomNavigationView.setSelectedItemId(R.id.menu_home);
83                break;
84            case 1:
85                mBottomNavigationView.setSelectedItemId(R.id.menu_find);
86                break;
87            case 2:
88                mBottomNavigationView.setSelectedItemId(R.id.menu_mine);
89                break;
90        }
91    }
92}
```

与上节**不同的点就是添加Fragment数据**的部分。

本节我们想要对首页的Fragment里面再添加多个**子Fragment**，因此就不能用ExampleFragment这个仅展示一个文本的Fragment了。我们的首页Fragment里面需要有多个页面切换，以及标题，需要新的布局，新的代码。

接下来我们为它新建一个HomeFragment来具体实现。而另外两个页面Find和Mine，我们还用ExampleFragment，做个简单的显示文本页面就行了。

## 2.4 多个子栏目页面的承载者--HomeFragment

### 2.4.1 界面布局

fragment_home.xml

```
 1<?xml version="1.0" encoding="utf-8"?>
 2<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 3    xmlns:app="http://schemas.android.com/apk/res-auto"
 4    xmlns:tools="http://schemas.android.com/tools"
 5    android:layout_width="match_parent"
 6    android:layout_height="match_parent"
 7    android:orientation="vertical">
 8
 9    <com.google.android.material.tabs.TabLayout
10        android:id="@+id/home_tab_layout"
11        android:layout_width="match_parent"
12        android:layout_height="wrap_content"
13        app:tabGravity="start"
14        app:tabMode="auto" />
15
16    <androidx.viewpager.widget.ViewPager
17        android:id="@+id/home_vp"
18        android:layout_width="match_parent"
19        android:layout_height="match_parent" />
20
21</LinearLayout>
```

为了显示每个栏目的标题tab，我们引入了**TabLayout**这个控件。与上节的BottomNavigationView一样，它也是material库里面的，如果你引入不到，也需要添加`implementation 'com.google.android.material:material:1.5.0'`的依赖。
这个控件有两个属性可以了解下：

- app:tabMode
  设置tab的模式的，有这几个取值：

- - fixed：固定的，也就是标题不可滑动，有多少展示多少，平均分配长度
  - scrollable：可滑动的，小于等于5个默认靠左固定，大于五个tab后就可以滑动
  - auto：自动选择是否可滑动，小于等于5个默认居中固定，大于5个自动滑动

- app:tabGravity
  设置tab的位置

- - start：居左
  - fill：平均分配，铺满屏幕宽度
  - center：居中

### 2.4.2 代码部分

首先还是控件的初始化。
HomeFragment.java

```
 1 public class HomeFragment extends Fragment {
 2
 3    private static final String ARG_PARAM1 = "param1";
 4    private static final String ARG_PARAM2 = "param2";
 5
 6    private String mParam1;
 7    private String mParam2;
 8
 9    private ViewPager mViewPager;
10    private TabLayout mTabLayout;
11
12    public HomeFragment() {
13    }
14
15    public static HomeFragment newInstance(String param1, String param2) {
16        HomeFragment fragment = new HomeFragment();
17        Bundle args = new Bundle();
18        args.putString(ARG_PARAM1, param1);
19        args.putString(ARG_PARAM2, param2);
20        fragment.setArguments(args);
21        return fragment;
22    }
23
24    @Override
25    public void onCreate(Bundle savedInstanceState) {
26        super.onCreate(savedInstanceState);
27        if (getArguments() != null) {
28            mParam1 = getArguments().getString(ARG_PARAM1);
29            mParam2 = getArguments().getString(ARG_PARAM2);
30        }
31    }
32
33    @Override
34    public View onCreateView(LayoutInflater inflater, ViewGroup container,
35                             Bundle savedInstanceState) {
36        return inflater.inflate(R.layout.fragment_home, container, false);
37    }
38
39    @Override
40    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
41        super.onViewCreated(view, savedInstanceState);
42        initView();
43
44    }
45
46    private void initView() {
47        mViewPager = getView().findViewById(R.id.home_vp);
48        mTabLayout = getView().findViewById(R.id.home_tab_layout);
49    }
50}
```

然后是ViewPager部分

同样的，也涉及到ViewPager、Data、Adapter三个部分，与前文一样，这里快速过一下。
ViewPager我们已经声明并初始化，然后是数据部分。

- 数据部分

```
 1private List<Fragment> mFragmentList;
 2private List<String> mTitleList;
 3
 4 @Override
 5public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
 6    super.onViewCreated(view, savedInstanceState);
 7
 8    initView();
 9    initData();
10}
11
12private void initData() {
13    mTitleList = new ArrayList<>();
14    mTitleList.add("热点");
15    mTitleList.add("娱乐");
16    mTitleList.add("历史");
17    mTitleList.add("科技");
18    mTitleList.add("体育");
19    mTitleList.add("地理");
20    mTitleList.add("新奇");
21    mTitleList.add("美图");
22
23    mFragmentList = new ArrayList<>();
24    for (int i = 0; i < mTitleList.size(); i++) {
25        SubFragment subFragment = SubFragment.newInstance("fragment" + i, "");
26        mFragmentList.add(subFragment);
27    }
28}
```

这里的数据相比前文多了mTitleList，也就是那些tab标题。Fragment部分我们新建了一个SubFragment来承接。出于方便演示，SubFragment与ExampleFragment一样只有一个文本做显示，比较简单。
SubFragment.java:

```
 1public class SubFragment extends Fragment {
 2
 3    public static final String ARG_PARAM1 = "param1";
 4    public static final String ARG_PARAM2 = "param2";
 5
 6    private String mParam1;
 7    private String mParam2;
 8
 9    private TextView mTextView;
10
11    public SubFragment() {
12    }
13
14    public static SubFragment newInstance(String param1, String param2) {
15        SubFragment fragment = new SubFragment();
16        Bundle args = new Bundle();
17        args.putString(ARG_PARAM1, param1);
18        args.putString(ARG_PARAM2, param2);
19        fragment.setArguments(args);
20        return fragment;
21    }
22
23    @Override
24    public void onCreate(Bundle savedInstanceState) {
25        super.onCreate(savedInstanceState);
26        if (getArguments() != null) {
27            mParam1 = getArguments().getString(ARG_PARAM1);
28            mParam2 = getArguments().getString(ARG_PARAM2);
29        }
30    }
31
32    @Override
33    public View onCreateView(LayoutInflater inflater, ViewGroup container,
34                             Bundle savedInstanceState) {
35        return inflater.inflate(R.layout.fragment_sub, container, false);
36    }
37
38    @Override
39    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
40        super.onViewCreated(view, savedInstanceState);
41
42        mTextView = view.findViewById(R.id.sub_fragment_content);
43
44
45        if (!TextUtils.isEmpty(mParam1)) {
46            mTextView.setText(mParam1);
47        }
48    }
49}
```

- adapter部分
  也是继承自FragmentPagerAdapter，只不过这里再多重写一个获取标题的方法`getPageTitle`。也就是在前文基础上做一点改造：

MyViewPagerAdapterForFragment.java:

```
 1public class MyViewPagerAdapterForFragment extends FragmentPagerAdapter {
 2
 3    private List<Fragment> mFragmentList;
 4    private List<String> mTitleList;
 5
 6
 7    public MyViewPagerAdapterForFragment(@NonNull FragmentManager fm, List<Fragment> fragmentList) {
 8        super(fm);
 9        mFragmentList = fragmentList;
10    }
11
12    public MyViewPagerAdapterForFragment(@NonNull FragmentManager fm, List<Fragment> fragmentList, List<String> titleList) {
13        super(fm);
14        mFragmentList = fragmentList;
15        mTitleList = titleList;
16    }
17
18    @NonNull
19    @Override
20    public Fragment getItem(int position) {
21        return mFragmentList == null ? null : mFragmentList.get(position);
22    }
23
24    @Override
25    public int getCount() {
26        return mFragmentList == null ? 0 : mFragmentList.size();
27    }
28
29    @Nullable
30    @Override
31    public CharSequence getPageTitle(int position) {
32        return mTitleList == null ? super.getPageTitle(position) : mTitleList.get(position);
33    }
34}
```

我们增加了一个重载的构造方法，用来传入titleList。在getPageTitle方法中返回对应的标题。

这样三个主要的部分都准备完毕，接下来在HomeFragment中将它们连接起来即可。

HomeFragment.java完整代码：

```
 1public class HomeFragment extends Fragment {
 2
 3    private static final String ARG_PARAM1 = "param1";
 4    private static final String ARG_PARAM2 = "param2";
 5
 6    private String mParam1;
 7    private String mParam2;
 8
 9    private ViewPager mViewPager;
10    private TabLayout mTabLayout;
11    private List<Fragment> mFragmentList;
12    private List<String> mTitleList;
13    private MyViewPagerAdapterForFragment mPagerAdapterForFragment;
14
15    public HomeFragment() {
16    }
17
18    public static HomeFragment newInstance(String param1, String param2) {
19        HomeFragment fragment = new HomeFragment();
20        Bundle args = new Bundle();
21        args.putString(ARG_PARAM1, param1);
22        args.putString(ARG_PARAM2, param2);
23        fragment.setArguments(args);
24        return fragment;
25    }
26
27    @Override
28    public void onCreate(Bundle savedInstanceState) {
29        super.onCreate(savedInstanceState);
30        if (getArguments() != null) {
31            mParam1 = getArguments().getString(ARG_PARAM1);
32            mParam2 = getArguments().getString(ARG_PARAM2);
33        }
34    }
35
36    @Override
37    public View onCreateView(LayoutInflater inflater, ViewGroup container,
38                             Bundle savedInstanceState) {
39        return inflater.inflate(R.layout.fragment_home, container, false);
40    }
41
42    @Override
43    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
44        super.onViewCreated(view, savedInstanceState);
45
46        initView();
47        initData();
48        initEvent();
49    }
50
51
52    private void initView() {
53        mViewPager = getView().findViewById(R.id.home_vp);
54        mTabLayout = getView().findViewById(R.id.home_tab_layout);
55    }
56
57    private void initData() {
58        mTitleList = new ArrayList<>();
59        mTitleList.add("热点");
60        mTitleList.add("娱乐");
61        mTitleList.add("历史");
62        mTitleList.add("科技");
63        mTitleList.add("体育");
64        mTitleList.add("地理");
65        mTitleList.add("新奇");
66        mTitleList.add("美图");
67
68        mFragmentList = new ArrayList<>();
69        for (int i = 0; i < mTitleList.size(); i++) {
70            SubFragment subFragment = SubFragment.newInstance("fragment" + i, "");
71            mFragmentList.add(subFragment);
72        }
73    }
74
75    private void initEvent() {
76        // 注意, 这里传入的是getChildFragmentManager()
77        mPagerAdapterForFragment = new MyViewPagerAdapterForFragment(getChildFragmentManager(), mFragmentList, mTitleList);
78        mViewPager.setAdapter(mPagerAdapterForFragment);
79        // 设置默认选中第一页
80        mViewPager.setCurrentItem(0);
81        // 关联tabLayout与ViewPager
82        mTabLayout.setupWithViewPager(mViewPager);
83    }
84}
```

增加initEvent方法，在其中进行adapter的创建以及与ViewPager的绑定。注意, 在adapter的创建的时候需要传入FragmentManager，由于此时要创建的Fragment页面是**子Fragment**，也就是它的承载者本身就是一个Fragment，所以，这里调用的是`getChildFragmentManager()`。

- 在一个Fragment里面创建子Fragment需要获取FragmentManager，调用的是`getChildFragmentManager()`。
- 作为对比，我们再回顾下在Activity里创建Fragment需要FragmentManager，调用的是`getSupportFragmentManager()`，不带Child

另外值得注意的一点是：tabLayout与ViewPager的联动。
我们滑动ViewPager，上面的标题Tab要跟着切换。反之，点击Tab，ViewPager也要跟着切换。这该怎么做到呢？很简单，有了TabLayout这个控件，只需要一行代码`mTabLayout.setupWithViewPager(mViewPager);`即可。

这样我们就完成整个HomeFragment部分了，结合外层Activity，整个案例也完成了。

# 3. 总结

本文侧重的就是子Frament多栏目这部分，结合前文的底部导航+ViewPager，构成了最实用的、常见的Frament+ViewPager+BottomNavigationView页面结构。有了这套结构，你就可以以此为模板，快速构建App了~

如