## LayoutInflater.inflate() 用法

```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) 
```
1. 当root不为null，attachToRoot为true时
表示将resource指定的布局添加到root中，添加的过程中resource所指定的的布局的根节点的各个属性都是有效的。
2. 如果root不为null，而attachToRoot为false的话
表示不将第一个参数所指定的View添加到root中，我们在开发的过程中给控件所指定的layout_width和layout_height到底是什么意思？该属性的表示一个控件在容器中的大小，就是说这个控件必须在容器中，这个属性才有意义，否则无意义。这就意味着如果我直接将linearlayout加载进来而不给它指定一个父布局，则inflate布局的根节点的layout_width和layout_height属性将会失效（因为这个时候linearlayout将不处于任何容器中，那么它的根节点的宽高自然会失效）。如果我想让linearlayout的根节点有效，又不想让其处于某一个容器中，那我就可以设置root不为null，而attachToRoot为false。这样，指定root的目的也就很明确了，即root会协助linearlayout的根节点生成布局参数，只有这一个作用。
3. 当root为null时，不论attachToRoot为true还是为false，显示效果都是一样的。当root为null表示我不需要将第一个参数所指定的布局添加到任何容器中，同时也表示没有任何容器来来协助第一个参数所指定布局的根节点生成布局参数。
4. 两个参数的inflate方法
```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root) {  
        return inflate(parser, root, root != null);  
    } 
```
这是两个参数的inflate方法，大家注意两个参数实际上最终也是调用了三个参数。
两个参数的inflate方法分为如下两种情况：
1.root为null，等同于3所述情况。
2.root不为null，等同于1所述情况。



[参考博客](http://blog.csdn.net/u012702547/article/details/52628453)
