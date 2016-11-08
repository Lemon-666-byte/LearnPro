>
所谓懒加载，即用户切到对应的fragment时才加载数据(Android默认加载可见fragment和可见fragment的左右fragment即3个fragment数据,这样可能会导致界面产生卡顿）

Fragment 实现懒加载，主要涉及的方法是 setUserVisibleHint()。

下面实现基于懒加载实现一个 BaseFragment（由于项目中用到RxJava，所以继承了 RxFragment ）：

首先实现一个懒加载的 LazyFragment 基类：

```
public abstract class LazyFragment extends RxFragment {

    protected boolean isVisible;
    
    /**
     * 在这里实现Fragment数据的懒加载.
     *
     * @param isVisibleToUser
     */
    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (getUserVisibleHint()) {
            isVisible = true;
            onVisible();
        } else {
            isVisible = false;
            onInvisible();
        }
    }
    
    /**
     * fragment被设置为可见时调用
     */
    protected void onVisible() {
        lazyLoad();
    }
    
    protected abstract void lazyLoad();
    
    /**
     * fragment被设置为不可见时调用
     */
    protected void onInvisible() {
    
    }
    
}
```

然后可以再包装一层，这样用起来会方便些：

```
public abstract class BaseFragment extends LazyFragment {
  
   // 标志位，标志已经初始化完成。
    private boolean isPrepared;
    private View parentView;
    private Context mContext;
  
  @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        parentView = inflater.inflate(getLayoutResId(), container, false);
        mContext = getActivity();
        return parentView;
    }
    
    public abstract
    @LayoutRes
    int getLayoutResId();
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        //初始化控件
        initView();
        isPrepared = true;
        lazyLoad();
    }
    
    protected abstract void initView();
    
    @Override
    protected void lazyLoad() {
        if (!isPrepared || !isVisible) {
            return;
        }
        //填充数据等操作
        initData();
        //初始化操作
        initAction();
    }
    
    protected abstract void initData();
    
    protected abstract void initAction();
    
    protected View findViewById(int id) {
        return parentView.findViewById(id);
    }
}
```

在使用的时候继承 BaseFragment 即可。

如有问题，欢迎提 Issues。


