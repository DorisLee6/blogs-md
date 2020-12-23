vue3.0 组合式API入门(一)

vue3.0新特性组合式API的基本使用
# vue3.0组合composition Api介绍

组合式API:
  setup函数: setup函数是vue3的一个新的组件选项,用于组合式API的入口
  参数: setup函数接收两个参数 --- 1. props (接收父组件传递的参数值) ; 2.context (包含attrs, slots, emit的普通JavaScript对象)

## setup函数详解

setup作为一个新的组件选项,接下来将要介绍关于它的执行时机,合并策略,以及在setup函数中访问组件实例信息的方式

**1.setup函数执行时机**

setup 是围绕 beforeCreate 和 created 生命周期钩子运行的,熟悉vue2.0声明周期的同学应该知道,vue2.0的各个声明周期勾子函数,这里简单介绍一下,不清楚的同学可以看下 [官方文档](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)

  -- beforeCreate(实例创建前执行,data 未初始化和 event/watcher 未配置) 都不可访问
  -- created(实例创建完成后执行, data和event可访问, 组件未挂载到真实dom中, $el不可用)
  -- beforeMount(组件挂载前, $el不可用)
  -- mounted(组件挂载完成, $el可用)
  -- beforeUpdate(组件数据更新前执行)
  -- updated(数据更新完, 虚拟dom重新渲染后调用)
  -- beforeDestroy(组件实例销毁之前调用, 实例的相关信息仍然可用)
  -- destroyed(组件实例销毁后调用, 组件相关信息都被销毁)
  [参考声明周期图示](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)
**2.setup中的this**

借用官方文档的话,在 setup() 内部，**this 不会是该活跃实例的引用**，因为 setup() 是在解析其它组件选项之前被调用的，所以 setup() 内部的 this 的行为与其它选项中的 this 完全不同。这在和其它选项式 API 一起使用 setup() 时可能会导致混淆。**所以在setup函数内部使用this会是undefined**

**3.setup中使用其他组件选项**

- **声明周期函数**

```js
  import { onMounted, onUpdated, onUnmounted } from 'vue';

  export default {
    setup(props, context) {
      // 使用声明周期hook
      onMounted(() => {});
      onUpdated(() => {});
      onUnmounted(() => {});
      // ...
    }
  }
```

在setup函数内部使用时需要引入

**选项 API 生命周期选项和组合式 API 之间的映射**

- vue3.0声明周期hook -> setup函数内部使用
- beforeCreate -> setup()
- created -> setup()
- beforeMount -> onBeforeMount
- mounted -> onMounted
- beforeUpdate -> onBeforeUpdate
- updated -> onUpdated
- beforeUnmount(vue2中beforeDestroy) -> onBeforeUnmount
- unmounted(vue2中destroyed) -> onUnmounted
- errorCaptured(vue3.0新增) -> onErrorCaptured
- renderTracked(vue3.0新增) -> onRenderTracked
- renderTriggered(vue3.0新增) -> onRenderTriggered

[vue3.0声明周期hook参考]:(https://www.vue3js.cn/docs/zh/api/options-lifecycle-hooks.html)

- **获取组件实例**
使用 getCurrentInstance 方法可以获取到组件实例信息

```js
import { getCurrentInstance } from "vue";
export default {
  setup(props, context) {
    let ctx = getCurrentInstance();
  }
}
```

- **setup中使用computed与watch**

```js
import { computed, watch } from "vue";
export default {
  setup(props, context) {
    /**
     * @des setup函数中使用computed
     * */
    // 申明响应式数据
    let count = ref(1);
    // 计算属性生成新值
    let newCount = computed(() => count.value + 1);
    // 使用get set ==> 这样就可以双向数据绑定
    let newCount1 = computed({
      get: () => count.value + 1,
      set: val => count.value = val
    }) 
    /**
     *  @des setup函数中使用watch
     **/
    // 监听ref申明的data
    watch(count, (count, prevCount) => {})
    // 监听一个getter
    let state = reactive({ count: 1 })
    watch(
      () => state.count,
      (count, prevCount) => {}
    )
    // return 出去供组件使用
    return {
      count,
      newCount,
      newCount1
    }
  }
}
```

**4.setup响应式data**

```js
import { ref, reactive } from "vue";
export default {
  setup(props, context) {
    /**
     * @des 申明基本类型的数据
     * @ref 使用ref申明基本类型的data,其返回值是一个Ref对象
     * @注意 使用ref申明的数据在setup函数中访问时需要加上.value
     * 
    */
    let name = ref("name")
    // 修改name值
    name.value = "name-new";
    /**
     * @des 申明对象
     * @reactive 使用reactive来声明对象类型的data
     */
    let myDescription = reactive({
      name: "dorislee",
      nation: "汉"
    })
    // 修改myDescription对象的值
    myDescription.name = "new_name";
    /** @des 将声明的data return出去,组件实例才可以访问到该响应式数据 **/
    return {
      name,
      myDescription
    }
  }
}
```

以上我们就可以通过setup函数来申明一些data,类似于在data中声明初始化数据,并在setup函数中使用申明周期hook,及computed,watch,本文只从几个方面介绍了vue3.0 setup函数的基本使用,后续我会从项目的角度更深度的讲述vue3.0的新特性,以及vue3.0的hooks写法对比mixins的优缺点,敬请关注本博客.
