# vue3 响应式api
## 一、ref 
接受一个内部值，返回一个响应式的、可更改的ref 对象，此对象只有一个指向其内部值的属性.value。

ref 对象可更改的，可以为 .value 赋予新的值。它也是响应式的，即所有对 .value 的操作都将被追踪，并且写操作会触发与之相关的副作用。

如果将一个对象赋值给 ref, 那么这个对象将通过 reactive() 转为具有深层次响应式的对象。这也意味着如果对象中包含了嵌套的 ref, 他们将被深层的解包。若要避免这种深层次的转换，请使用 shallowRef() 替代。
~~~javascript
const count = ref(0);
console.log(count.value);

count.value++;
console.log(count.value);
~~~
如果指定了一个泛型参数但没有给出初始值，那么最后得到的就将是一个包含 undefined 的联合类型
~~~javascript
const n = ref<number>()
~~~

## 二、computed 
接收一个 getter 函数，返回一个只读的响应式 ref 对象。该 ref 通过 .value 暴露 getter 函数的返回值，它也可以接受一个带有 get 和 set 函数的对象来创建一个可写的 ref 对象。
~~~javascript
const count = ref(1);
const plusOne = computed(() => count.value + 1);
console.log(plusOne.value);
plusOne.value++；
~~~
创建一个可写的计算属性
~~~javascript
const count = ref(1);
const plusOne = computed({
    get: () => count.value + 1,
    set: (val) => {
        count.value = val + 1
    }
})

plusOne.value = 1;
console.log(count.value);
~~~
## 三、reactive 
返回一个对象的响应式代理，响应式转换是深层的，它会影响到所有嵌套的属性。一个响应式对象也将深层的解包任何 ref 属性，同时保持响应性。

当访问到某个响应式数组或者像 Map 这样的原生集合类型中的 ref 元素时，不会执行 ref 的解包。

返回的对象以及其中嵌套的对象都会通过 es Proxy 进行包裹，不等于原对象，应该避免使用原对象。

### 创建
~~~javascript
const obj = reactive({count: 0});
obj.count++;
~~~
ref 的解包
~~~javascript
const count = ref(1);
const obj = reactive({count});

console.log(obj.count === count.value); // true
count.value++;
console.log(count.value);
console.log(obj.count);

obj.count++;
console.log(obj.count);
console.log(count.value);
~~~
当访问到某个响应式数组或 Map 这样的原生集合中的 ref 元素时，不会执行 ref 的解包。
~~~javascript 
const books = reactive([ref('Vue 3 Guide')]);
console.log(books[0].value);

const map = reactive(new Map([['count', ref(0)]]));
console.log(map.get('count').value);
~~~
将 ref 赋值给一个 reactive 属性时，该 ref 会被自动解包。
~~~javascript
const count = ref(1);
const obj = reactive({});
obj.count = count;
console.log(obj.count);
console.log(obj.count === count.value);
~~~
## 四、readonly
接受一个对象或ref，返回一个原值的只读代理。该只读代理是深层的，对任何嵌套属性的访问都将是只读的，它的 ref 解包行为与 reactive 相同，但解包得到的值时只读的。

~~~javascript
const original = reactive({count: 0});
const copy = readonly(original);

watchEffect(() => {
    console.log(copy.count);
})

original.count++;
copy.count++;
~~~
## 五、watchEffect 
立即运行一个函数，同时响应式地追踪其依赖，并在依赖更改时重新执行。

第一个参数就是要运行的副作用函数，这个副作用函数的参数也是一个函数，用来注册清理回调。清理回调会在该副作用下一次执行前被调用，可以用来清理无效地副作用。例如等待中的异步请求。

第二个参数是一个可选的选项，可以用来调整副作用的刷新时机或者调试副作用的依赖。

flush: 'post' 将会使侦听器延迟到组件渲染之后再执行。在某些特殊情况下，可能有必要在响应式依赖发生改变时立即触发侦听器，这可以通过设置 flush: 'sync'实现。当使用该设置时，如果有多个属性同时更新，将导致一些性能和数据一致性的问题。

返回值是一个用来停止副作用的函数。
~~~javascript
const count = ref(0);

const stop1 = watchEffect(() => console.log(count.value));
count.value++;

cosnt stop2 = watchEffect(async (onCleanup) => {
    const { response, cancel } = doAsyncWork(id.value);
    onCleanup(cancel);
    data.value = await response;
})

// 停止侦听器
stop1();
stop2();
~~~
## 六、watchPostEffect
侦听器延迟到组件渲染后执行
## 七、watchSyncEffect
侦听器立刻执行
## 八、watch 
侦听一个或多个响应式数据源，并在数据源变化时调用所给的回调函数。
### 侦听器的源
- 函数返回一个值
- 一个 ref 对象 
- 一个 reactive 对象
- 由以上值组成的数组
### 回调
当侦听器的值发生变化时调用的回调，回调函数接受三个参数：新值、旧值、以及一个用于注册副作用的清理的回调函数。该回调函数在副作用下一次重新执行前调用，可以用来清除无效的副作用。

当侦听对各来源时，回调函数接受两个数组，分别对应来源数组中的新值和旧值。
### 选项
第三个可选的参数是一个对象，支持以下这些选项：
- immediate: 在侦听器创建时立刻触发回调，第一次调用时旧值为 undefined。
- deep: 如果源时对象，强制深度便利，以便在层级变更时触发回调。
- flush：调整函数的刷新时机， 'post' 为组件渲染后刷新， 'sync' 为马上刷新。
- onTrack / onTrigger: 调试侦听器的依赖。

当侦听一个响应式对象时，侦听器会自动启用深层模式。
### 作用
- watch 使我们可以懒执行副作用
- 更加明确是应该用哪个状态触发侦听器重新执行
- 可以访问所侦听状态的前一个值和当前值
### 例子
~~~JavaScript 
const state = reactive({count: 0});
watch(
    () => state.count,
    (count, prevCount) => {}
);

const count = ref(0);
watch(count, {count, prevCount} => {});

watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {})
~~~
## 九、shallowRef 
ref 的浅层作用形式，只对 .value 的访问时响应式的，其他原样不变。常常用于对大型数据结构的性能优化或是与外部状态管理系统的集成。
~~~javascript
const state = shallowRef({count: 1});
// 不会触发更改
state.value.count = 2;
// 会触发更改
state.value = {count: 2};
~~~
## 十、triggerRef
强制触发依赖于一个浅层 ref 的副作用，这通常在对浅引用的内部值进行深度变更时使用。
~~~JavaScript
const shallow = shallowRef({
    greet: 'Hello, world'
});

watchEffect(() => {
    console.log(shallow.value.greet)
})

shallow.value.greet = 'Hello, universe'

triggerRef(shallow)
~~~
## 十一、customRef
创建一个自定义的 ref，显示声明对其以来追踪和更新触发的控制方式。

预期接受一个工厂函数作为参数，这个工厂函数接受 track 和 trigger 两个函数作为参数，并返回一个带有 get 和 set 方法的对象。

一般来说，track() 应该在 get() 方法中调用，而 trigger() 应该在 set 中调用，然而事实上，你可以对什么时候调用，是否应该调用他们拥有绝对的控制权。
~~~javascript
export function useDebouncedRef(value, delay = 200) {
    let timeout;
    return customRef({track, trigger} => {
        return {
            get() {
                track();
                return value;
            },
            set(newValue) {
                clearTimeout(timeout);
                timeout = setTimeout(() => {
                    value = newValue;
                    trigger();
                }, delay);
            }
        }
    })
}
~~~
在组件中使用
~~~
<script setup>
import {useDebouncedRef} from './debounceREf';
const text = useDebouncedRef('hello');
<script>
<template>
    <input v-model="text" />
</template>
~~~
## 十二、shalowReactive 
reactive 的浅层作用形式，只有根级别的属性时响应式的，其他属性的值会被原样存储和暴露，值为 ref 的属性不会被自动解包了。

浅层数据结构应该只用于组件的根级数据，将其嵌套在深层次的响应式对象中，会使创建的树由不一致的响应行为，可能很难理解和调试。
~~~javascript
const state = shallowReactive({
    foo: 1,
    nested: {
        bar: 2
    }
});

state.foo++; // 响应式
isReative(state.nested); // false
state.nest.bar++; // 非响应式 
~~~
## 十三、shallowReadonly
readonly 的浅表作用形式，只有根层级的属性变为了只读。属性的值都会被原样存储和暴露，ref 不会解包。

浅层数据结构应该只用于组件的根级数据，将其嵌套在深层次的响应式对象中，会使创建的树由不一致的响应行为，可能很难理解和调试。
~~~javascript
const state = shallowReadonly({
    foo: 1,
    nested: {
        bar: 2
    }
});

state.foo++;
isReadonly(state.nested);
state.nested.bar++;
~~~
## 十四、toRaw
返回由 reactive()、readonly()、shallowReactive() 或者 shalowReadonly()
创建的代理对应的原始对象。

这是一个可以用于临时读取而不引起代理访问/跟踪开销，或是写入而不出发更改的特殊方法。不建议保存对原始对象的持久引用。
~~~javascript
const foo = {};
const reactiveFoo = reactive(foo);
console.log(toRaw(reactiveFoo) === foo); // true
~~~
## 十五、markRow
将一个对象标记为不可转为代理，返回该对象本身。
~~~javascript 
const foo = markRaw({});
console.log(isReactive(reactive(foo))); // false

const bar = reactive({foo});
console.log(isReactive(bar.foo)); // false
~~~
## 十六、effectScope
创建一个 effect 作用域，可以捕获其中所创建的响应式副作用（即计算属性和侦听器），这样捕获到的副作用可以一起处理。
~~~javascript 
const scope = effectScope();

scope.run(() => {
    const doubled = computed(() => counter.value * 2);
    watch(doubled, () => console.log(doubled.value));
    watchEffect(() => console.log('Count: ', doubled.value));
});

scope.stop();
~~~
## 十七、getCurrentScope
获取当前活跃的 effect 作用域
## 十八、onScopeDispose 
在当前活跃的作用域上注册一个处理回调函数，当相关的 effect 作用域停止时会调用这个回调函数。

这个方法可作为复用的组合式函数中的 onMounted 的替代品，它不与组件耦合，因为每个 Vue 组件的 setup 函数也是在一个 effect 作用域内调用的。

