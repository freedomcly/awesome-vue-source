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

Vue中的`Object.defineProperty`响应式机制能感知到set data，不能感知什么呢？

* 不能感知对象属性的添加或删除
* 不能感知直接用索引设置数组项，例如：vm.items[indexOfItem] = newValue
* 不能感知数组长度的修改，例如：vm.items.length = newLength
* 不能感知数组的push/pop等操作

### 1.不能感知对象属性的添加或删除


    let data = {
      _ooo: {
        name: 'ooo',
        age: 30
      }
    }

    Object.defineProperty(data, 'ooo', {
      get: function() {
        return this._ooo
      },
      set: function(newValue) {
        console.log('set')
        this._ooo = newValue
      }
    })
    
    data.ooo.key = 'key' // 可以设置成功，不能感知set
    delete data.ooo.name // 可以设置成功，不能感知set
    
正如以上的demo，这种情况下，Object.defineProperty不能感知到set，Vue无法做到响应式，可以使用`Vue.set`。

### 2.不能感知直接用索引设置数组项

    var data = {
      _arr: [1, 2, 3]
    }

    Object.defineProperty(data, 'arr', {
      enumerable: true,
      configurable: true,
      get: function() {
        console.log('get arr')
        return this._arr
      },
      set: function(newValue) {
        console.log('set arr')
        this._arr = newValue
      }
    })

    Object.defineProperty(data.arr, '0', {
      enumerable: true,
      configurable: true,
      get: function() {
        console.log('get arr[0]')
        // return this[0]
      },
      set: function(newValue) {
        console.log('set arr[0]')
        // this[0] = newValue
        // data.arr[0] = newValue
      }
    })
    
    // 可以设置成功，可以感知set，set不成功。不能用数组下标的方式set
    data.arr[0] = 3 
    
把arr和arr[0]都改写成访问器属性。由于setter函数中的`this[0] = newValue`或`data.arr[0] = newValue`这种方式不能使用，所以set不能成功。使用Vue时，也可以使用`Vue.set`解决这个问题。

### 3.不能感知数组长度的修改

    // 可以设置成功，不能感知set
    data.length = 1