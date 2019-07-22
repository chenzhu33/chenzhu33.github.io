# Ime-common组件

## 1. 仓库地址

http://icode.baidu.com/repos/baidu/inputmethod/ime-common/tree/master

## 2. 支持功能

### 2.1. 图片加载ImageLoader [已支持]@陈朱伟

基于Glide4.7.1二次封装，API调用形式类似Glide，但做进一步简化。

具体可见文档：[ImageLaoder技术文档](http://agroup.baidu.com/baiduinput/md/article/355368)

### 2.2. 网络Retrofit [已支持]@查宗旬

基于Retrofit+RxJava，common组件中额外提供了一些拦截器功能。

后续会增加文件上传下载的封装。

### 2.3. 日志BDLog [已支持]@陈朱伟

替代Android Log，有更直观更丰富的日志记录。

具体可见文档：[BDLog技术文档](http://agroup.baidu.com/baiduinput/md/article/1124346)

### 2.4. RxJava [未支持]@夏涵

支持RxJava相关功能

### 2.5. 数据收集Stats [已支持]@陈朱伟

基础数据收集模块，提供基本的基于二进制的数据收集能力，一般不需要使用这个。

具体可见文档：[Stats技术文档](http://agroup.baidu.com/baiduinput/md/article/355522)

### 2.6. 存储Storage [部分支持]@陈朱伟，查宗旬，周新

存储相关所有功能，包括：

1. BDSP：支持基于SharedPreference与二进制数据的K-V形式存储模块。
2. BDFile：支持所有文件相关操作
3. BDCache：支持基于LRU的MemCache与DBCache
4. BDDatabse：基于GreenDao的数据库

### 2.7. 白名单WhiteLsit [已支持]@唐洪鉴

白名单模块，提供统一的白名单能力，开发新功能时，如果需要用到类似于`xx情况下展示，xx情况下关闭某个功能`的需求，则需要通过白名单模块进行管理。

### 2.8. 权限Permission [未支持]@李照坤

Android动态权限申请模块，当需要申请动态权限时，使用该模块接口进行申请。

### 2.9. 分享Sharer [未支持]@王康

分享模块，包括分享到QQ、微信、本地文件，支持图片，文字，GIF，Video。

### 2.10. 工具Utils [已支持]@陈朱伟，夏涵

各种工具类，包括：

1. DiskUtils：磁盘相关工具类，未来会整合进BDFile
2. FileUtils：File相关工具类，未来会整合进BDFile
3. IOUtils：文件IO，未来会整合进BDFile
4. MathUtils：数据处理工具类
5. NumberUtil：number处理工具类，未来会和MathUtils合并
6. RomUtil：判断rom型号
7. Scheme：Uri包装
8. SysIO：文件IO，未来会整合进BDFile
9. SystemPropertiesUtils：判断系统型号，未来会和RomUtil进行合并
10. ToastUtil：Toast弹窗
11. TypefaceUtils：字体相关工具类
12. ZipLoader：zip压缩/解压
13. ZipUtil：zip解压，未来会和ZipLoader进行合并


### 2.11. 事件总线Eventbus [已支持]@夏涵

基于EventBus的封装，提供事件总线能力。