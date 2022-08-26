# vue3 响应式api
## ref 
接受一个内部值，返回一个响应式的、可更改的ref 对象，此对象只有一个指向其内部值的属性.value。

ref 对象可更改的，可以为 .value 赋予新的值。它也是响应式的，即所有对 .value 的操作都将被追踪，并且写操作会触发与之相关的副作用。

如果将一个对象赋值给 ref, 那么这个对象将通过 reactive() 转为具有深层次响应式的对象。这也意味着如果对象中包含了嵌套的 ref, 他们将被深层的解包。若要避免这种深层次的转换，请使用 shallowRef() 替代。
~~~javascript
const count = ref(0);
console.log(count.value);

count.value++;
console.log(count.value);
~~~
### 为 ref 标注类型
~~~javascript
import { ref } from 'vue';
import type { Ref } from；'vue';

const year: Ref<string | number> = ref(2020);
year.value = '2020';

// 或者

const year = ref<string | number>('2020');
year.value = 2020;
~~~
如果指定了一个泛型参数但没有给出初始值，那么最后得到的就将是一个包含 undefined 的联合类型
~~~javascript
const n = ref<number>()
~~~

## computed 
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
### 为 computed 标注类型
computed 会自动从其计算函数的返回值上推导出类，可以用泛型参数显示指定类型
~~~javascript
const double = computed<number>(() => {

})
~~~
## reactive 
返回一个对象的响应式代理，响应式转换是深层的，它会影响到所有嵌套的属性。一个响应式对象也将深层的解包任何 ref 属性，同时保持响应性。

当访问到某个响应式数组或者像 Map 这样的原生集合类型中的 ref 元素时，不会执行 ref 的解包。

返回的对象以及其中嵌套的对象都会通过 es Proxy 进行包裹，不等于原对象，应该避免使用原对象。

### 创建
~~~javascript
const obj = reactive({count: 0});
obj.count++;
~~~
### ref 的解包
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
## readonly
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
## watchEffect 
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
## watchPostEffect
侦听器延迟到组件渲染后执行
## watchSyncEffect
侦听器立刻执行
## watch 
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