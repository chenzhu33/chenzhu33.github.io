#Mint跨平台动态模板框架

###现有问题：

1. 布局能力较弱，与IOS不统一
2. 性能上加载模板卡顿严重
3. 不支持JS引擎，无法支持复杂应用场景
4. 目前框架无法支持布局异步化与JS引擎引入，可扩展性不够

###定位：

1. 动态化，跨平台，高性能布局框架
2. 支持多种布局模式
3. 组件化
4. 前端MVVM，数据绑定

###工作内容：

1. 设计更具扩展性的框架
2. 增强布局能力，引入Yoga布局库，统一双平台渲染效果
3. 提升模板加载与渲染速度，布局异步化，降低卡顿
4. MVVM与数据绑定
5. 模板嵌套与组件化，模板路由
6. 本地事件模块与视图组件
7. 更强的布局能力（constraintlayout）
8. 引擎动态切换
9. 异步化渲染
10. *模板开发工具*


###期望收益：
1. 动态化能力
2. 跨平台能力
3. 高性能：优于原生
4. 学习成本低，开发难度低

###收益评估方式：
1. 可以不依赖发版更新诸如搜索服务，表情商店等模块，便于A/B Test
2. android/ios共用一套布局
3. 渲染性能提升，卡顿时间降低
4. 开发难度与学习成本评估
5. 开源效益

###工作计划：

Q1：**（完成）**

1. 设计更具扩展性的框架
2. 增强布局能力，引入Yoga布局库，统一双平台渲染效果
3. 提升模板加载与渲染速度，布局异步化，降低卡顿

Q2：

4. 引入JS引擎**（完成）**
5. JS Engine支持JS层调用与创建Native object，Native层调用js对象 [3天]
6. 流程与Task重构，增加Compiler层，重构后分为四层：Parser/Compiler/Layouter/Renderer。（框架搭建）[4天]
7. JS事件定义与响应 接口定义&实现（提供上下文查询）[3天]
8. LifeCircle 定义&实现（提供生命周期）[3天]
9. DocumentModule 接口定义&实现（提供DomTree创建与渲染接口）[5天]
10. ComponentManager 接口定义&实现（提供模板嵌套与组件化能力，Html->Node）[6天]
11. RouterModule 接口定义&实现（提供路由能力）[3天]
12. Component规范定义-ListView设计与实现 [10天]
13. 模板指令编译 [4天]

-----------以上框架总体完成-----------------------

14. 增加本地事件模块（网络，I/O，DB，动画，etc.）[10天]
15. 增加若干组件支持（button, edittext, etc.）[5天]

-----------以上可以满足7.5搜索模块动态化-----------

Q3：

16. JS-Native MVVM架构
17. Duktape与V8引擎的动态切换
18. ConstraintLayout引入
19. 异步化渲染
20. 模板开发IDE
21. 注释与文档完善
22. 单元测试完善

-----------以上可以达到开源要求-----------

## 详设

### 流程

**创建流程**

- Parser [解析HTML/CSS] -> Compiler [支持IF的Compile] -> Layouter -> Renderer

**更新流程**

- 如果只更新样式: Node.setXXX() -> View.setXXX();
- 如果更新影响布局的属性: Node.setXX() -> YogaNode.setXX() -> reLayout -> reRender;
- 如果添加节点（通过JS调用Document.addNode方式添加）: Document.addNode() -> reLayout -> reRender; 
- 如果删除节点: Document.removeNode() -> reLayout -> reRender; 

**Router流程**

1. 对于Router目标页面，走一次创建流程（需要父容器的width/height,node对象）
2. 调用ParentNode.append(CreatedNode)添加 -> 其中调用ParentView.add(CreatedView)，**不需要**reLayout与reRender


### JS事件定义与响应

**Context传入**

```
nativeOnClick(View view, Script script) {
	Node rootNode = getRootNode(view);
	setNodeToJS(rootNode); // Native层往JS层塞入Native对象
	callJS(script);
}

script {
	Node node = Document.getElementById(rootNode, id);
	node.setXXX();
	node.setYYY();
	// If any layout attributes is changed
	node.requestLayout();
}
```

**事件定义**

- onClick
- onLongClick


### JS Engine支持JS层调用与创建Native object，Native层调用js对象
目前的JS Engine Framework不支持，需要实现JNI与Java接口

### LifeCircle 定义&实现（提供生命周期）[3天]

DomLifeCycle

- onCreate - Parse之前，代表开始干活了
- onDomUpdate - Compile之后DomTree构造完成，Layout之前
- onDestroy - Dom删除之前

ViewLifeCycle

- onAppear - View创建完毕，append到UI前
- onViewUpdate - View更新完毕，append到UI前
- onDisappear - View从UI上被移出之前


### DocumentModule

- createElement(node, parent, index);
- moveElement(node, parent, index);
- removeElement(node);
- updateStyle(node, styleKey, styleValue);
- addEvent(node, event);
- removeEvent(node, event);

- getElementById(htmlNode, id);


### RouterModule

**Router接口设计**

- push(htmlPath, parentNode, data);
- replace(htmlPath, parentNode, data);
- remove(index, parentNode);
- go(relativeIndex, parentNode);
- go(index, parentNode);

parentNode类型限制为<Router-View>，<Router-View>固定宽高，并且子节点Z轴排布

**Router流程**

1. 对于Router目标页面，走一次创建流程（需要父容器的width/height,node对象）
2. 调用ParentNode.append(CreatedNode)添加 -> 其中调用ParentView.add(CreatedView)，**不需要**reLayout与reRender

### Component接口规范

自定义Component需要提供

两个类：

```
YourCustomView extents BaseMintView
YourCustomViewManager extends BaseViewManager
```

YourCustomViewManager包括四种接口：

1. ComponentName, E.g. `<Image>`
2. AttributeList, E.g. `src/bind/background`
3. EventList, E.g. `onItemClick/onSwipe`
4. Interface, E.g. `bindData(List<Map<String, String>> data)`


### Component - ListView



### Component - EditText

### Component - Button

### Module接口规范

### Module - HTTP

### Module - I/O

### Module - Database

### Module - Animation


