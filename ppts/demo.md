title: web离线缓存
speaker: 周连毅
transition: move
files: /js/demo.js,/css/demo.css,/js/zoom.js
theme: moon
usemathjax: no

[slide]

# web离线缓存
<small>演讲者：周连毅</small>
<small>部门：应用部</small>

[slide]

# 功能 {:&.flexbox.vleft}
> 让你的网页，没有网络，照样使用！

[slide]
## 特点
----
* 离线浏览 {:&.rollIn}
    用户可以在离线状态下浏览网页内容
* 速度更快
    因为数据被存储在本地，所以速度会更快
* 减轻服务器负担
    离线资源不发生改变时，不会向服务器发送请求

[slide]
## 实例
----
* 游戏：玛雅 http://192.168.2.243:3000 {:&.rollIn}
* 工具：背景图定位 http://www.spritecow.com/
* 文档：ECMAScript 5 http://yanhaijing.com/es5


[slide]
## Let's Make It!
----
* 在&lt;html&gt;上添加manifest="xxx.manifest" {:&.moveIn}
* 创建xxx.manifest
* 确保xxx.manifest的mime类型为text/cache-manifest
* 为xxx.manifest填充内容<br />
  <img src="/images/xxx.manifest.png"/>

[slide]
## 离线缓存原理
----
* &lt;html&gt;标记上没有manifest属性，正常加载文档
* &lt;html&gt;标记上有manifest
    *  如果离线缓存存在，直接使用离线缓存，如果离线缓存不存在，浏览器会加载文档 {:&.fadeIn}
    *  离线缓存不存在的时候，浏览器会获取清单中的文件，生成缓存
    *  离线缓存存在时，浏览器检查清单中的文件是否有更新（通过检查manifest的内容）
    *  如果清单文件有更新，则更新缓存，并会在下一次刷新的时候生效
    *  如果清单文件没有更新，则什么也不做
    *  只有所有需要离线的资源都获取成功，才会真正生成离线缓存

[slide data-transition="newspaper" data-incallback="removeImage"]
# 清单文件如何更新
#### 更改manifest文件中的任何一个字符，都会被认为清单进行了更新<br /><br /><br />
<img src="/images/update.png"/>

[slide data-incallback="putImage" data-outcallback="removeImage"]
## manifest说明
----
* 第一行，必须是CACHE MANIFEST {:&.zoomIn}
* 注释以#开头
* CACHE段是要缓存的文件列表
* NETWORK段说明哪些资源不被缓存
    *  不写NETWORK段，所有没被缓存的资源都不请求服务器
    *  支持通配符*，也支持单个映射
    *  通常写*，即所有未缓存的资源都请求服务器
* FALLBACK段是请求服务器失败的替代资源
    *  FALLBACK段通常用来制定404页面
    *  支持通配符/，也支持单个映射
<script src="/js/jquery.js"></script>
<script>
function putImage() {
    $('.full_img').remove();
    $('<img src="/images/full.png"/>').css({
        top:0,
        right:0,
        position:'absolute'
    }).addClass('full_img').appendTo('body');
}
function removeImage() {
    $('.full_img').remove();
}
</script>


[slide data-incallback="removeImage"]

## 坑
----
* 一旦应用了manifest特性，则该文档会被默认缓存，不需要写到manifest文件中 {:&.zoomIn}
* 一旦资源加入了离线缓存，该资源内容变化，浏览器并不会重新获取该资源
* 只有当manifest文件更新，浏览器才会重新离线缓存清单中的文件
* 浏览器重新缓存了清单中的文件，并不会立即使用，会在下一次页面加载的时候生效


[slide]

## 清除缓存
----
* 清除浏览器缓存
* 在 <span class="yellow">chrome:appcache-internals</span> 中清除
* 服务端返回404或410(下一次加载页面时生效)

[slide]

##离线缓存状态
###存在于window.applicationCache对象上
####通过window.applicationCache.status获取当前离线缓存状态
<a style="font-size:16px;" href="http://www.w3.org/TR/2011/WD-html5-20110525/offline.html">http://www.w3.org/TR/2011/WD-html5-20110525/offline.html</a>
---
关键字 | 简要含义 | 值 | 含义
:-------|:------:|-------:|--------
UNCACHED|未缓存|0|表明一个应用缓存对象还没有完全初始化
IDLE|空闲|1|应用缓存此时未处于更新过程中
CHECKING|检查|2|清单已经获取完毕并检查更新
DOWNLOADING|下载中|3|下载资源并准备加入到缓存中
UPDATEREADY|更新就绪|4|一个新版本的应用缓存可以使用
OBSOLETE|废弃|5|应用缓存现在被废弃(404或410)

[slide data-transition="newspaper"]
##JS离线事件监听
----
<pre><code class="javascript">
function onUpdateReady() {
  window.applicationCache.swapCache();
  var sure = confirm('发现新版本，是否立即使用？');
  sure && window.location.reload();
}
window.applicationCache.addEventListener('updateready', onUpdateReady);
if(window.applicationCache.status === window.applicationCache.UPDATEREADY) {
  onUpdateReady();
}
    </code></pre>

[slide]

##浏览器兼容性
---
###桌面版兼容性
桌面版 | Chrome | Firefox | IE | Opera | Safari
:-------|:------:|-------:|--------
版本号|4.0|3.5|10.0|10.6|4.0
<br />

###手机版兼容性
手机版 | Android | Firefox Mobile | IE Mobile | Opera Mobile | Safari Mobile
:-------|:------:|-------:|--------
版本号|2.1|?|未实现|11.0|3.2

[slide]

##相关资料
----
* https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache
* http://www.html5rocks.com/en/tutorials/appcache/beginner/
* http://www.w3.org/TR/2011/WD-html5-20110525/offline.html
* http://diveintohtml5.info/offline.html
* http://zhangsichu.com/html5/rocks/Slides/html5.html
* http://caniuse.com/#search=manifest
* http://html5demos.com/offlineapp

[slide]

#Question?
----





