# Flywheel

## Introduce

Online Exception Detector for android written with Kotlin.

Flywheel includes ui-block detection, crash detection, memory leak detection, network error detection. These modules are transparent to app itself, which means you don't need to write any intrusive code in your apps.

Flywheel also provides custom exception collector and real-time performance monitor to help developing.

Furthermore, all modules in flywheel are desinged to be pluggable.

## Getting Started

In your build.gradle:

```
 dependencies {
    compile 'com.baidu.flywheel:flywheel-kernal:0.3'
 }
```

To install a module:

```
module = FlyWheel.INSTANCE.install(ModuleContext, ModuleClass.class);
```

For example, you can write as below in your Application class:

```
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    
    // install this to detect ui-block and anr problem
    blockWatcher = FlyWheel.INSTANCE.install(new BlockWatcherContext(this), BlockWatcher.class);
    
    // install this to detect dalvik and native crash
    crashWatcher = FlyWheel.INSTANCE.install(new CrashWatcherContext(this), CrashWatcher.class);
    
    // install this to detect memory leak
    leakWatcher = FlyWheel.INSTANCE.install(new LeakWatcherContext(this), LeakWatcher.class);
    
    // install this to collect custom exception
    exceptionCollector = FlyWheel.INSTANCE.install(new ExceptionCollectorContext(this), ExceptionCollector.class);
    
    // install this to show runtime performance monitor
    runtimeMonitor = FlyWheel.INSTANCE.install(new RuntimeMonitorContext(this), RuntimeMonitor.class);
  }
}
```

The module won't start work immediately after being installed. To start or stop a module, call:

```
// start a module
module.start();

// stop a module
module.stop();
```

To get or delete collected data, call:

```
// get data
module.getData(LogReadListener<? extends BaseInfo>());

// delete data
module.deleteData();
```

## API Document

### 0. Basic Configuration: ModuleContext

ModuleContext provides basic configuration for modules.
You can implement the following methods to provide your own informations.

```
/**
 * provide app info
 * E.g. user id, skin info, app version, app flavor,
 */
open fun appInfo(): String {
    return ""
}
```

```
/**
 * provide network type when block happens
 * @return [String] like 2G, 3G, 4G, wifi, etc.
 */
open fun networkType(): String {
    return ""
}
```

```
/**
 * provide memory info
 * return null to use default otherwise use custom
 */
open fun memoryInfo(): String? {
    return null
}
```

```
/**
 * provide cpu info
 * return null to use default otherwise use custom
 */
open fun cpuInfo(): String? {
    return null
}
```

```
/**
 * provide phone info
 * return null to use default otherwise use custom
 */
open fun phoneInfo(): String? {
    return null
}
```

### 1. Basic API

Flywheel provides 4 common basic API for all modules, which have also been introduced in part **Getting Started**:

```
// start a module
module.start();

// stop a module
module.stop();

// get error data stored in flywheel databse
module.getData(LogReadListener<? extends BaseInfo>());

// delete data from database
module.deleteData();
```

### 1. BlockWatcher

BlockWatcher module is designed to detect ui-block or anr problem.

Except for basic configurations, it also provides following methods:

```
/**
 * Config block threshold (in millis), dispatch over this duration is regarded as a BLOCK. 
 * You may set it from performance of device.
 */
open fun blockThreshold(): Long {
    return 1000
}
```

```
/**
 * Packages that developer concern, by default it concerns all,
 * put high priority one in pre-order.
 *
 * @return null if concern all.
 */
open fun concernPackages(): List<String>? {
    return null
}
```

```
/**
 * Provide white list, entry in white list will not be shown in ui list.
 *
 * @return null if you don't need white-list filter.
 */
open fun whiteList(): List<String>? {
    return null
}
```

```
/**
 * Block interceptor, you can provide your own actions.
 *
 * @param context app context
 *
 * @param blockInfo the block information collected by flywheel
 *
 * @return true if you have already processed this block and don't want to store in flywheel database
 *         false if you want to store this block.
 */
open fun onBlock(context: Context, blockInfo: BlockInfo) : Boolean {
    return false
}
```

BlockWatcher don't extends any other APIs from the basic APIs.

### 2. CrashWatcher

CrashWatcher is designed to detect both dalvik and native crash.

It provides some more configurations as below:

```
/**
 * Crash interceptor, you can provide your own actions.
 *
 * @param isNative whether this crash is created by native code
 *
 * @param crashInfo the crash information collected by flywheel
 *
 * @return true if you have already processed this crash and don't want to store in flywheel database
 *         false if you want to store this crash.
 */
open fun onCrash(crashInfo: CrashInfo, isNative: Boolean) : Boolean {
    return false
}
```

```
/**
 * Temp directory path to store native crash
 */
open fun nativeCrashFilePath(): String {
    return applicationContext.externalCacheDir.absolutePath + "/flywheel/crashes"
}
```

CrashWatcher also doesn't extends basic APIs.

### 3. LeakWatcher

LeakWatcher is designed to detect memory leak. It's currently implemented base on open source library [LeakCanary](https://github.com/square/leakcanary).

It provides two more configurations:

```
/**
 * Max stored heap dumps of the memory leak
 */
open fun maxStoredHeapDumps() : Int {
    return 30
}
```

```
/**
 * Memory leak interceptor, you can provide your own actions.
 *
 * @param leakInfo the memory leak information collected by flywheel
 *
 * @return true if you have already processed this leak and don't want to store in flywheel database
 *         false if you want to store this leak.
 */
open fun onMemoryLeak(leakInfo : LeakInfo) : Boolean {
    return false
}
```

It provides one API:

```
/**
 * You can watch any object before released by using this method
 */
fun watch(watchObject : Any) {
    refWatcher.watch(watchObject)
}
```

Furthermore it **doesn't support** basic API of `start()` and `stop()`.

### 4. NetworkWatcher

*// TODO*

### 5. ExceptionCollector

Exception Collector is designed to collect catched exception or custom log. 

Notice that this module may be called manually by user.

Exception Collector doesn't provide more configurations.

Exception Collector provides one more API to report exceptions or logs by user:

```
/**
 * Report exceptions to flywheel and store into database
 */
fun reportException(exception : String) {
    core.reportException(exception)
}
```

Furthermore it also **doesn't support** basic API of `start()` and `stop()`.

### 6. RuntimeMonitor

*// TODO*

## License

*// TODO*

## Appendix

[Flywheel中文技术文档](todo)
