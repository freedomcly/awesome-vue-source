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
    
    Dep.target = null

# Watcher类

    export default class Watcher {
      vm: Component;
      expression: string;
      cb: Function;
      id: number;
      deep: boolean;
      user: boolean;
      lazy: boolean;
      sync: boolean;
      dirty: boolean;
      active: boolean;
      deps: Array<Dep>;
      newDeps: Array<Dep>;
      depIds: ISet;
      newDepIds: ISet;
      getter: Function;
      value: any;
  
      ...
  
      addDep (dep: Dep) {
        const id = dep.id
        if (!this.newDepIds.has(id)) {
          this.newDepIds.add(id)
          this.newDeps.push(dep)
          if (!this.depIds.has(id)) {
            dep.addSub(this)
          }
        }
      }
      
      update () {
        /* istanbul ignore else */
        if (this.lazy) {
          this.dirty = true
        } else if (this.sync) {
          this.run()
        } else {
          queueWatcher(this)
        }
      }
      
      run () {
        if (this.active) {
          const value = this.get()
          if (
            value !== this.value ||
            // Deep watchers and watchers on Object/Arrays should fire even
            // when the value is the same, because the value may
            // have mutated.
            isObject(value) ||
            this.deep
          ) {
            // set new value
            const oldValue = this.value
            this.value = value
            if (this.user) {
              try {
                this.cb.call(this.vm, value, oldValue)
              } catch (e) {
                handleError(e, this.vm, `callback for watcher "${this.expression}"`)
              }
            } else {
              this.cb.call(this.vm, value, oldValue)
            }
          }
        }
      }
      ...
    }

`dep.depend()`做了什么？`Dep.target.addDep(this)`，this指向当前dep实例。`Dep.target`是什么？它是一个Watcher实例，后面会解释。`dep.depend()`就是把当前的dep收集到Watcher实例Dep.target的deps和depIds中，再把watcher实例收集到Dep类的subs中。

`dep.notify()`做了什么？把所有Dep类收集到subs中的依赖全部调用update。

# 疑问

* deps和newDeps有什么区别？
