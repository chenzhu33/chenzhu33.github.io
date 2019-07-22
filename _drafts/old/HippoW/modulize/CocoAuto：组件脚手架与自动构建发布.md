# CocoAuto：组件脚手架与自动构建发布

## 1. 自动化构建

Coco输入法组件开发基于CocoAuto自动化构建发布项目，CocoAuto包含两个命令：

### 1.1. -c create

-c负责项目创建，其用法如下：

```
python CocoAuto.py -c [UserNameForIcode] [ProjectName] [ModuleName]
```

该命令会从Icode上clone脚手架工程`CocoScaffold`，并基于该工程，创建相应的开发工程。所有Coco、输入法、基础库等相关的依赖都会自动配置好，这部分开发者无需关心。

创建完成后的工程目录结构如下：

```
ProjectName // 工程名
	devapp // 开发&测试APP
	bpisdk // sdk library module
	moduleName // 需要开发的组件
```

- `devapp`是输入法项目代码，初期可以考虑用完整的输入法项目替代
- `bpisdk`是放置所有bpi文件的项目，该module会自动化生成与更新，开发者无需关心也不要修改。
- `moduleName`是真正需要开发的业务组件。

### 1.2. -d deploy

-d用于将测试完成的组件项目发布到maven仓库，其用法如下：

`python CocoAuto.py -d snapshot/release`

## 2. 输入法脚手架工程

脚手架工程是一个预配置好的工程`CocoScaffold`，如前文所介绍这个工程提供了业务组件的开发模板。脚手架工程的目的是让开发者不用关注框架，只需要关注业务本身即可。

目前包括如下功能：

1. 自动生成业务组件module
2. 自动生成开发工程module用于日常开发调试
3. 配置好的build.gradle文件：可以直接使用ImeBase，ImeCommon中的功能模块
4. 配置好的Coco组件框架：提供Coco的Sample代码，快速接入

未来将增加：

1. 组件module接入Sample：在输入法场景下，如何开发出一个业务组件
2. 可选的MVP/MVVM模板：提供MVP或MVVM模板
3. 基础库接入Sample：包括数据库，网络，图片加载
