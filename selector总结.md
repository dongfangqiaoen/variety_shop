## 关于selector的总结

1. selector代码如下：
```
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/ic_bg_news_channel_operate_selected" android:state_selected="true" />
    <item android:drawable="@drawable/ic_bg_news_channel_operate_normal" />
</selector>
```
2. 最后一个默认item的state包含前面所有item的state值的相反值。意思就是前面有个state_selected="true" 的最后一个item就默认加上了一个state_selected="false"，前面有个state_focused=false的最后一个item就默认加上了一个state_focused=true的item。
3. 范围大的item必须写在范围小的item的后面，默认item必须写在selector的最后一行。默认item应该是对于每个state不论true还是false都存在，因为selector对于重复的state只取第一个有效，这也就解释了为什么把默认item放在最前面，后面的item不起作用了。
4. 代码中设置selector和shape
```java
 StateListDrawable selector=new StateListDrawable();
        selector.addState(new int[]{android.R.attr.state_pressed}, new ColorDrawable(Color.BLACK));//按下的item
        GradientDrawable shape=new GradientDrawable();
        shape.setShape(GradientDrawable.RECTANGLE);
        shape.setAlpha(50);
        shape.setColor(Color.GRAY);
        shape.setStroke(3,Color.YELLOW);//边界宽度和颜色
        shape.setCornerRadius(3);
        selector.addState(new int[]{},shape);//默认的item
        t11.setBackground(selector);
```
5. 如上述代码，selector对应的java类为：android.graphics.drawable.StateListDrawable，shape对应的java类为：android.graphics.drawable.GradientDrawable.

###总结:
6. 我们可以根据这些java类自定义一个通用的selector类，进行对view的背景来设置selector.可以省去大量xml
7. [参考公众号](https://mp.weixin.qq.com/s/lNRY9AcAM3bhKo5Y4z0MuQ)


