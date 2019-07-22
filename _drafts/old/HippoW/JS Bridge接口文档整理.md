# JS Bridge接口文档整理

### 1. 分享

### 2. 预下载的网页点击显式下载接口

```
baiduInputJSBridge.on(' onDownload', {
'click_web_download_link': 'http://gdown.baidu.com/data/wisegame/b3b4cfad62d545f7/dididache_91.apk',//点击下载的地址
'click_web_download_package': 'com.sdu.didi.psnger' // 点击下载的包名
},
function(data) {});
```

`click_web_download_package`必须与之前预下载的包名匹配，否则，将不会进行下载操作，若与预下载包名匹配，则下载进度会显式地展现在通知栏上。由于必须要和之前服务端下发的预下载包名匹配，因此这个接口目前只用于预下载。
开始支持的版本：7.0之前

### 3. 保存图片接口

```
baiduInputJSBridge.on(' saveImage', {
data: encodeURIComponent (base64)
}, function(data) {
//参数data, success为true表示保存成，false表示保存失败
//data: {"success":true/false} 
});
```
base64:为图片的base64编码，不包含头信息(即不需要data:image/xxx;base64, 开头)
开始支持的版本：7.0之前

### 4. 浏览器跳转接口
```
baiduInputJSBridge.on('openUrl', {
"browsers": ["com.baidu.searchbox", // 浏览器包名列表
"com.baidu.browser.apps", "com.baidu.input"],
"url": "http://www.baidu.com" // 要跳转的url
},
function(data) {});
```
browsers数组为空表示跳转至系统默认浏览器，否则按照数组中元素顺序遍历，找到本地已安装的浏览器并完成跳转，如遇到输入法包名则在WebView内部完成跳转。
开始支持的版本：7.0之前

### 5. 打开输入法APP化中指定tab
```
baiduInputJSBridge.on('openImeAppMain', {
tab:tab
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.2

### 6. 打开具体的皮肤列表页

```
baiduInputJSBridge.on('openSkinList', {
name:"精选布局", // 对应的皮肤列表名称
url: http://mco.ime.shahe.baidu.com:8015/v5/skin_ads/cate/smr
// 对应的地址
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.2


### 7. 打开皮肤详情页

```
baiduInputJSBridge.on('openSkinDetail', {
id:"456", // id
type: 0 // 类型，0表示皮肤，1表示主题
token:"sdfadsfsadf" // token
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本:7.2

### 8. 打开表情详情页
```
baiduInputJSBridge.on('openEmojiDetail', {
id:"1000111", // id
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.2

### 9. 打开颜文字详情页
```
baiduInputJSBridge.on('openEmojiIconDetail', {
id:"1000111", // 颜文字id
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.2

### 10. 打开词库分类
```
baiduInputJSBridge.on('openCikuSort', {
type:2, // 热词类型
id:-1 // 热词id
count:0 // 热词数量
name:"城市地理" // 热词名称
descrytion:"公交站 地名" // 热词描述
url: http://mco.ime.**** // 对应地址
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.2

### 11. 打开精品详情页
```
baiduInputJSBridge.on('openBountiqueDetail', {
id:144441, // 该精品在后台的编号
timestamp:1441414 // 最后修改的时间戳
is_store_recommend: true // 是否是应用推荐
version: 8.2.5.0 // 版本名
description: "****" // 描述
download_url:http://mco.ime.shahe.baidu.com:8015/v5/skin_ads
//下载地址
package_name:"com.baidu.baiduMap" // 应用包名
display_name:"百度地图" //应用显示的名称
store_icon_url:"****" //图标地址
thumb_left_url:"*****" // 左边详情图地址
thumb_right_url:"*****" // 右边详情图地址
size:323233 // 大小
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.2

### 12. 打开插件详情页
```
baiduInputJSBridge.on('openDiscoverDetail', {
package_name:"com.baidu.input.**", // 插件包名
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本:7.2

### 13. 获取皮肤状态
```
baiduInputJSBridge.on('getSkinState', {
type:1, // 类型，0表示皮肤，1表示主题
token:token // token
}, function(data) {
result: 0 // 是否成功
install: 0 // 是否安装
in_use: 1 // 是否在使用
share: 0 // 是否可分享
});
```
开始支持的版本：7.2

### 14. 获取表情包状态
```
baiduInputJSBridge.on('getEmojiState', {
id:1123132, // 表情包id
}, function(data) {
result: 0 // 是否成功
install: 0 // 是否安装
share: 0 // 是否可分享
});
```
开始支持的版本：7.2

### 15. 通过eventbus发送事件（目前只支持调起语音面板，分为录音态和非录音态）
```
baiduInputJSBridge.on('sendEvent', {
class:"com.baidu.input.ime.searchservice.event.DelegateShowReqEvent", // 调起语音面板对应的java类名
param:{
mDelegateType:0 // 浮层类型
mPackageName:"com.baidu.input"
//展示浮层时需要处于的包名，null表示匹配所有
mEditorId: -1 
//展示浮层时需要处于的框ID，负数表示匹配所有
mInputType:-1
//展示浮层时需要处于的框输入类型，负数表示匹配所有
mIsImmeStartVoice:true
// 是否要立刻启动录音
}
}, function(data) {
result: 0 // 是否成功
});
```
开始支持的版本：7.3

### 16. 根据webview是否进入前后台执行相应的js回调方法（目前只用于音乐播放）
```
baiduInputJSBridge.on('registerNotification', {
noti:"didBecomeActive,
// noti为didBecomeActive表示回调函数在webview进入前台执行，noti为willResignActive表示js的回调函数在webview进入后台时执行。
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.3

### 17. 在输入法进程直接下载apk文件
```
baiduInputJSBridge.on('downloadApkFile', {
url:"XXX",
md5:"XXX",
size:"12312",
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.4

### 18. 获取语音Id和用户分贝
```
baiduInputJSBridge.on('getVoiceInfo', {
// 空参数
}, function(data) {
voice_id:"XXX",
voice_avg_db:"60",
voice_max_db:"70",
});
```
开始支持的版本：7.4

### 19. 调起极简语音
```
baiduInputJSBridge.on('startTinyVoice', {
type: "tiny_voice" 或 "normal"
}, function(data) {
//参数data,这里为null
});
```
开始支持的版本：7.4

### 20. 设置webview标题
```
baiduInputJSBridge.on('setTitle', {
title: "XXX"
}, function(data) {
data:"200"(成功) 或 "0"（失败）
});
```
开始支持的版本：7.6

### 21. 拷贝礼品券，成功后，发送成功的状态码给前端
```
baiduInputJSBridge.on('copy', {
text="gift code" // 内容是要拷贝的码
}, function(data) {
data:"200"(成功) 或 "0"（失败）
});
```
开始支持的版本：7.6