# BDLog与输入法日志使用规范

###### 基于开源项目Logger修改


##### 注意：统一使用com.baidu.input.devtool.log.BDLog打印日志，禁止在代码中使用android.util.Log、System.out.println、System.err.println。


## 1. 功能：
 - 更直观显眼的日志打印：
 
 ![image](https://bj.bcebos.com/v1/issue-luigi/log2-20161025110520-hz6hmx.png)
 
 ![image](https://bj.bcebos.com/v1/issue-luigi/log1-20161025110521-qtxv7h.png)
 
 - 支持线程信息，调用栈信息打印
 - 支持调用栈跳转
 - 支持xml数据，json数据打印
 - 支持任意实现了toString()接口的数据打印
 - 支持异常信息打印
 - 更方便的全局设置与局部设置接口


## 2. API接口：


BDLog提供v/d/i/w/e/a/printStackTrace/json/xml这几类接口。使用时只需要简单调用对应log接口即可：


```
BDLog.v("Message");
BDLog.d("Tag", "Message");
...
```


##### d(Object object)：


BDLog支持debug下打印任意实现toString()的object对象，包括list, array。如下：


```
BDLog.d(infoList);
``` 


##### xml接口


对于xml数据，BDLog使用Transformer进行数据格式化，使用方法如下：


```
BDLog.xml(xmlString);
```


##### json接口
 
对于json数据，BDLog使用json自带toString(JSON_INDENT = 2)进行格式化，使用方法如下：


```
BDLog.json(jsonString);
```


## 3. 全局设置：


全局设置会对项目中所有log产生效果，因此应尽量避免使用。


需要设置首先获取设置接口：


```
Settings settings = BDLog.init();
```


通过`Settings`类，BDLog支持的全局设置有：


- 隐藏显示线程信息，默认显示
- 打印调用栈深度，默认为2
- 日志是否打印，默认打印
- 调用栈位移，默认为0
- 自定义LogAdapter，默认为AndroidLogAdapter


目前输入法内BDLog全局设置仅设置了在非debug模式下关闭log：


```
	if (!Macro.IS_DEBUG) {
   		BDLog.init().logLevel(LogLevel.NONE);
   	}
```


## 4. 局部设置：


BDLog支持的局部设置通过`BDLog.t()`完成，支持设置包括设定TAG，设定调用栈深度两种。接口如下：


```
    public static Printer t(String tag);
    public static Printer t(int methodCount);
    public static Printer t(String tag, int methodCount);
```


使用方法如下：


```
   BDLog.t("Tag", 5).d("Message");
```


## 5. 使用建议


1. 避免使用全局设置
2. 在所有的catch块中要添加如下代码，便于以后调试：
```
if (Macro.IS_CATCH_LOG_ENABLE) {
    BDLog.printStackTrace(e);
}
```
3. 原则上，所有打印日志的代码仍然要用`Macro.IS_DEBUG`开关宏括起来，主要是防止调用BDLog方法时执行参数部分的代码，降低效率。例如：
`BDLog.d(Math.max(a,b));`虽然不会打印日志，但Math.max(a,b)仍会执行。
4. 源码位于`devtool/com/baidu/input/log/`目录下
5. 如果需要增加功能或者修改建议。随时Hi上联系我。