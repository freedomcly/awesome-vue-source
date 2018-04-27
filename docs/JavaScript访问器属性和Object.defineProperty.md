# JavaScript访问器属性和Object.defineProperty

## 属性（property）和特性（attribute）

JavaScript对象中的属性有两种：数据属性和访问器属性。特性可以被认为是“属性的属性”。

数据属性有4个特性：
* [[Configurable]]  能否通过delete删除、能否修改属性的特性、能否修改为访问器属性。默认true。
* [[Enumerable]]    能否通过for-in循环返回属性。默认true。
* [[Writable]]      能否修改属性的值。默认true。
* [[Value]]         属性的数据值。默认true。

访问器属性有4个特性：
* [[Configurable]]  能否通过delete删除、能否修改属性的特性、能否修改为数据属性。默认true。
* [[Enumerable]]    能否通过for-in循环返回属性。默认true。
* [[Get]]           读取属性时调用的函数。默认undefined。
* [[Set]]           写入属性时调用的函数。默认undefined。

需要注意：
* 一旦把属性[[Configurable]]设置为false，不可配置，就不能再变回可配置。
* 用对象字面量等常用方法创建的对象属性，都是数据属性。

## Object.defineProperty

ES5中的新增了几个方法。`Object.defineProperty`和`Object.defineProperties`可以配置属性的特性。`Object.getOwnPropertyDescriptor`可以查询属性的特性。

Vue响应式机制能感知到set data，但`Object.defineProperty`不能感知什么？

* 