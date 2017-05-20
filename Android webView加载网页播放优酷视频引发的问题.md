最近，某个项目需要加载的网页内容中，含有一个优酷视频。跑起来出现了问题，优酷视频提示错误：请允许cookie存储
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/817079-b98c96477f367fef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从这个错误提示来看，这里存在cookie存储的相关问题。是否是本地webView没有允许cookie存储？

WebView是基于webkit内核的UI控件，相当于一个浏览器客户端。它会在本地维护每次会话的cookie(保存在data/data/package_name/app_WebView/Cookies.db)。

当WebView加载URL的时候,WebView会从本地读取该URL对应的cookie，并携带该cookie与服务器进行通信。WebView通过android.webkit.[CookieManager](https://developer.android.com/reference/android/webkit/CookieManager.html)类来维护cookie。[CookieManager](https://developer.android.com/reference/android/webkit/CookieManager.html)是WebView的cookie管理类。

那么，这里到底是哪里存在问题？考虑到出现该问题的设备是出现在Android 5.0系统上，于是翻阅Android开发者文档，找到[Home>Lollipop>Android 5.0 行为变更](https://developer.android.com/about/versions/android-5.0-changes.html)这一章节（请科学上网）。找到WebView这一部分的描述如下：
> WebView
Android 5.0 更改了应用的默认行为。
**如果您的应用是面向 API 级别 21 或更高级别：**默认情况下，系统会阻止[混合内容](https://developer.mozilla.org/en-US/docs/Security/MixedContent)和第三方 Cookie。要允许混合内容和第三方 Cookie，请分别使用[setMixedContentMode()](https://developer.android.com/reference/android/webkit/WebSettings.html#setMixedContentMode(int))
和[setAcceptThirdPartyCookies()](https://developer.android.com/reference/android/webkit/CookieManager.html#setAcceptThirdPartyCookies(android.webkit.WebView, boolean))
方法。
系统现在可以智能地选择要绘制的 HTML 文档部分。这个新的默认行为有助于减少内存占用和提升性能。如果您要一次渲染整个文档，可通过调用[enableSlowWholeDocumentDraw()](https://developer.android.com/reference/android/webkit/WebView.html#enableSlowWholeDocumentDraw())
停用此优化。
**如果您的应用是面向低于 21 的 API 级别：**系统允许混合内容和第三方 Cookie，并始终一次渲染整个文档。 

于是，我们找到了问题的根源所在，原来Android 5.0体统默认不支持第三方cookie的存储，像优酷视频这样的网页组件就无法正常播放视频了。

找到解决方法，我们在加载该URL之前，加入一下代码，解决了问题。
    
    if(android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP)
        CookieManager.getInstance().setAcceptThirdPartyCookies(webView,true);