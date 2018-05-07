`defineReactive`方法中，getter函数进行依赖收集，setter函数发出更新通知。

    export function defineReactive (
      obj: Object,
      key: string,
      val: any,
      customSetter?: ?Function,
      shallow?: boolean
    ) {
      const dep = new Dep()
      ...
  
      Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: function reactiveGetter () {
          const value = getter ? getter.call(obj) : val
          if (Dep.target) {
            dep.depend() // 依赖收集
            ...
          }
          return value
        },
        set: function reactiveSetter (newVal) {
          ...
          dep.notify() // 更新通知
        }
      })
    }
    
