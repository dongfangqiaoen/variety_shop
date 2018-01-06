### org.json.JSONObject 存放url

1. 一个URL字符串放入JSONObject中，url的 / 会被转义变成 \/ 。
2.  https://baidu.com -> "appurl":"https:\/\/baidu.com"。
3.  但是用json方法再取出来还是正确的url,但是将json toString之后就不正确了。
4.  obj.optString("appurl"),能得到正确的url
5.  如果想处理json toString之后的URL可以用下面的方法，得到正确的URL
6.  string.replaceAll("\\\\", "").trim();
