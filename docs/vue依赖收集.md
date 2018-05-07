`defineReactive`方法中，每个data对应一个dep实例，getter函数进行依赖收集，setter函数发出更新通知。

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
    
# Dep类

    export default class Dep {
      static target: ?Watcher;
      id: number;
      subs: Array<Watcher>;

      constructor () {
        this.id = uid++
        this.subs = []
      }

      addSub (sub: Watcher) {
        this.subs.push(sub)
      }

      removeSub (sub: Watcher) {
        remove(this.subs, sub)
      }

      depend () {
        if (Dep.target) {
          Dep.target.addDep(this)
        }
      }

      notify () {
        // stabilize the subscriber list first
        const subs = this.subs.slice()
        for (let i = 0, l = subs.length; i < l; i++) {
          subs[i].update()
        }
      }
    }
    
