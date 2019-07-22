# Kotlin Koans

## 1. 目录

1. 介绍
	1. Hello World -- 基本语法
	2. Java到Kotlin的转换
	3. 具名实参
	4. 缺省默认参数
	5. Lambda表达式
	6. Strings
	7. Data Classes
	8. 可空类型与非空类型
	9. Smart Casts
	10. 扩展函数
	11. 对象表达式
	12. SAM转换
	13. 集合上的扩展

2. 转换
	1. comparison
	2. in
	3. range to
	4. for loop
	5. 操作符重载

## 2. 介绍

### 2.1. Hello World与基本语法

#### 包名定义与引入

和java一样，包名的定义与引入需要放在源文件头

```
package my.demo

import java.util.*

// ...
```

不同的是，kotlin不要求目录路径与包名匹配

