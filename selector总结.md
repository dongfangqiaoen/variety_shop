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

