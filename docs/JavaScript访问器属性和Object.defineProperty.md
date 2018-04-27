# JavaScript访问器属性和Object.defineProperty

## 属性（property）和特性（attribute）

JavaScript对象中的属性有两种：数据属性和访问器属性。特性可以被认为是“属性的属性”。

数据属性有4个特性：
* [[Configurable]]  能否通过delete删除、能否修改属性的特性、能否修改为访问器属性。默认true。
* [[Enumerable]]    能否通过for-in循环返回属性。默认true。
* [[Writable]]      能否修改属性的值。默认true。
* [[Value]]         属性的数据值。默认true。
