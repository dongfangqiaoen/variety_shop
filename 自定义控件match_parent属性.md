## 自定义组合控件的match_parent属性不起效果

1. 自定义了一个继承自linearlayout的组合控件
```java
public class Custom_LinearLayout extends LinearLayout {
    public Custom_LinearLayout(Context context) {
        super(context);
        initView();
    }

    public Custom_LinearLayout(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public Custom_LinearLayout(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public Custom_LinearLayout(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
        initView();
    }

    private void initView() {

       View root= View.inflate(getContext(), R.layout.custom_linearlayout,this);
//       addView(root);
    }
}

```

2. 根布局 custom_linearlayout.xml如下:
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@color/green">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Green"/>

    <TextView android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Red"
        />
</LinearLayout>
```

3. 自定义控件添加如下:
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scroll_container"
    android:layout_width="match_parent"
    android:layout_height="2000dp"
    android:background="@android:color/holo_red_light"
    android:orientation="vertical">

    <TextView
        android:id="@+id/up_text"
        android:layout_width="wrap_content"
        android:layout_height="300dp"
        android:background="@android:color/background_light"
        android:text="hello" />

    <com.sun.webview_test.Custom_LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

    </com.sun.webview_test.Custom_LinearLayout>

    <com.sun.webview_test.BottomLayout
        android:id="@+id/bottom_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <android.support.v4.view.ViewPager
            android:id="@+id/view_pager_in_scroll"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@color/colorPrimary"></android.support.v4.view.ViewPager>
    </com.sun.webview_test.BottomLayout>
</LinearLayout>
```
### 总结 
1. 在自定义时使用如下代码，达不到让自定义控件宽度充满父布局的目的，效果还是wrap_content。
```
        View root= View.inflate(getContext(), R.layout.custom_linearlayout,null);
        addView(root);
```
2. 但使用如下代码就可以达到宽度充满父布局的目的,主要改变在于inflate()时是否制定父布局。
```
         View root= View.inflate(getContext(), R.layout.custom_linearlayout,this);
        //addView(root);

```
3. 此问题根源可以看[Inflate用法](./Inflate用法.md)，因为View.inflate()最后调用的还是LayoutInflater.inflate()。






