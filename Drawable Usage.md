## Andoird 中图片资源的使用

1. 同一张图片放到不同的drawable文件夹下，在加载进内存时会产生不同大小的Bitmap。
2. Bitmap尺寸越大，占用内存越大。
3. 大图片放在小文件夹下，会被拉伸成更大的Bitmap加载金内存。
4. 图片占硬盘的大小，和占内存的大小完全不同。
5. 图片在硬盘中会根据压缩规则进行压缩，而在内存中就需要把每个像素点都绘制出来，所以会变大。
6. 具体分析可以参考这篇文章：[微信公众号](https://mp.weixin.qq.com/s/9NkPFVN8WUo3ege1EPL_0Q)
