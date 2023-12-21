---


---

<p>vue2</p>
<hr>
<hr>
<h1 id="vue2">Vue2</h1>
<p>Hi! I’m your first Markdown file in <strong>StackEdit</strong>. If you want to learn about StackEdit, you can read me. If you want to play with Markdown, you can edit me. Once you have finished with me, you can create new files by opening the <strong>file explorer</strong> on the left corner of the navigation bar.</p>
<h1 id="observer-类">Observer 类</h1>
<p><img src="https://picx.zhimg.com/v2-9867b9c67f007df231f396e48a7b2844_720w.jpg?source=d16d100b" alt="Vue2源码学习笔记 - 11.响应式原理—Observer 类详解"><br>
<img src="https://pic1.zhimg.com/v2-f39fecdfde603c085c7ce6ba0c82b249_720w.jpg?source=d16d100b" alt=""></p>
<h2 id="使用场景">使用场景</h2>
<p>observer 的实例化只在 observe 函数中进行，入参为一个对象或者数组</p>
<pre class=" language-js"><code class="prism  language-js"><span class="token keyword">export</span> <span class="token keyword">function</span> <span class="token function">observe</span> <span class="token punctuation">(</span>value<span class="token punctuation">:</span> any<span class="token punctuation">,</span> asRootData<span class="token punctuation">:</span> <span class="token operator">?</span>boolean<span class="token punctuation">)</span><span class="token punctuation">:</span> Observer <span class="token operator">|</span> <span class="token keyword">void</span> <span class="token punctuation">{</span>
  <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token function">isObject</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span> <span class="token operator">||</span> value <span class="token keyword">instanceof</span> <span class="token class-name">VNode</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">return</span>
  <span class="token punctuation">}</span>
  <span class="token keyword">let</span> ob<span class="token punctuation">:</span> Observer <span class="token operator">|</span> <span class="token keyword">void</span>
  <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">hasOwn</span><span class="token punctuation">(</span>value<span class="token punctuation">,</span> <span class="token string">'__ob__'</span><span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span> value<span class="token punctuation">.</span>__ob__ <span class="token keyword">instanceof</span> <span class="token class-name">Observer</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    ob <span class="token operator">=</span> value<span class="token punctuation">.</span>__ob__
  <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token keyword">if</span> <span class="token punctuation">(</span>
    shouldObserve <span class="token operator">&amp;&amp;</span>
    <span class="token operator">!</span><span class="token function">isServerRendering</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span>
    <span class="token punctuation">(</span>Array<span class="token punctuation">.</span><span class="token function">isArray</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span> <span class="token operator">||</span> <span class="token function">isPlainObject</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span>
    Object<span class="token punctuation">.</span><span class="token function">isExtensible</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span>
    <span class="token operator">!</span>value<span class="token punctuation">.</span>_isVue
  <span class="token punctuation">)</span> <span class="token punctuation">{</span>
    ob <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Observer</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span>
  <span class="token punctuation">}</span>
  <span class="token keyword">if</span> <span class="token punctuation">(</span>asRootData <span class="token operator">&amp;&amp;</span> ob<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    ob<span class="token punctuation">.</span>vmCount<span class="token operator">++</span>
  <span class="token punctuation">}</span>
  <span class="token keyword">return</span> ob
<span class="token punctuation">}</span>

</code></pre>
<h2 id="observer">Observer</h2>
<p>observer 类会作用到每一个被观察的对象里。observer会将目标obj的属性转换为 getter/setter 的形式，这样就可以进行依赖收集和变更分发。</p>
<pre class=" language-js"><code class="prism  language-js"><span class="token operator">...</span>
<span class="token comment">/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */</span>
<span class="token keyword">export</span> <span class="token keyword">class</span> <span class="token class-name">Observer</span> <span class="token punctuation">{</span>
  <span class="token comment">// 传入实参数据 value </span>
  value<span class="token punctuation">:</span> any<span class="token punctuation">;</span>
  <span class="token comment">// Dep 对象</span>
  dep<span class="token punctuation">:</span> Dep<span class="token punctuation">;</span>
  <span class="token comment">// 把 value 当作根 $data 属性的 vm 实例个数</span>
  vmCount<span class="token punctuation">:</span> number<span class="token punctuation">;</span>

</code></pre>
<p>constructor (value: any) {<br>
// 初始化赋值<br>
this.value = value<br>
this.dep = new Dep()<br>
this.vmCount = 0<br>
// 给 vaule 值定义 <strong>ob</strong> 属性，值为本对象，这个后续有用到<br>
def(value, ‘<strong>ob</strong>’, this)<br>
if (Array.isArray(value)) {<br>
// --------------------------------------------<br>
if (hasProto) { //<br>
protoAugment(value, arrayMethods) //<br>
} else { // 数组响应式改造<br>
copyAugment(value, arrayMethods, arrayKeys)//<br>
} // ------------------------------------------<br>
// 数组则遍历元素逐个调用 observe 响应式化<br>
this.observeArray(value)<br>
} else {<br>
// 单个数据对象调用 walk 成员方法<br>
this.walk(value)<br>
}<br>
}</p>
<p>/**</p>
<ul>
<li>Walk through all properties and convert them into</li>
<li>getter/setters. This method should only be called when</li>
<li>value type is Object.<br>
*/<br>
walk (obj: Object) {<br>
const keys = Object.keys(obj)<br>
// 遍历数据对象所有键，调用 defineReactive 设置为响应式的属性<br>
for (let i = 0; i &lt; keys.length; i++) {<br>
defineReactive(obj, keys[i])<br>
}<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Observe a list of Array items.<br>
*/<br>
observeArray (items: Array) {<br>
// 遍历数组，每个元素传入 observe 响应式化<br>
for (let i = 0, l = items.length; i &lt; l; i++) {<br>
observe(items[i])<br>
}<br>
}<br>
}</li>
</ul>
<p>// helpers</p>
<p>/**</p>
<ul>
<li>Augment a target Object or Array by intercepting</li>
<li>the prototype chain using <strong>proto</strong><br>
<em>/<br>
function protoAugment (target, src: Object) {<br>
/</em> eslint-disable no-proto <em>/<br>
target.<strong>proto</strong> = src<br>
/</em> eslint-enable no-proto */<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Augment a target Object or Array by defining</li>
<li>hidden properties.<br>
<em>/<br>
/</em> istanbul ignore next */<br>
function copyAugment (target: Object, src: Object, keys: Array) {<br>
for (let i = 0, l = keys.length; i &lt; l; i++) {<br>
const key = keys[i]<br>
def(target, key, src[key])<br>
}<br>
}<br>
…</li>
</ul>
<p><strong>流程解析</strong><br>
observer 类在 new observer 实例化对象的时候传入了需要响应式处理的数据对象 value，value 是数组或者对象。在构造函数中，对 value 进行了保存，然后将 实例化的 observer 对象赋值给 value.__ob__，这样就把 value 和 observer 对象关联起来了。</p>
<p>然后根据 value 类型进行不同的处理。如果是数组，则遍历数组中元素，对每个元素调用 observe 响应式处理，执行到最后其实还是对元素执行 new Observer 增加 __ob__ 属性。如果是对象，则遍历属性值调用 defineReactive 进行响应式处理。最终的最终都是通过 defineReactive 做响应式处理，实际上就是对 value 的 setter/getter 进行改造。<br>
<img src="https://pic1.zhimg.com/v2-afe8315996e7d6536c8d9c017cd4d911_720w.jpg?source=d16d100b" alt=""></p>
<h2 id="definereactive">DefineReactive</h2>
<p><img src="https://pica.zhimg.com/v2-b38b98915477e7b1f57caf9a1504666e_720w.jpg?source=d16d100b" alt="Vue2源码学习笔记 - 8.响应式原理一def、proxy及defineReactive函数"></p>
<h1 id="dep">DEP</h1>
<p><img src="https://pic1.zhimg.com/v2-3e08ee4e0958ee3d9a7b6e1da5f6d613_720w.jpg?source=d16d100b" alt=""></p>
<p>Dep 类的实例化只有两个场景应用，一个是在 observer 类的构造函数里，建立 value.__ob__ 的 dep ，另一个是 defineReactive 里为每一个响应式数据对象添加 dep。</p>
<h2 id="observer-类中">observer 类中</h2>
<p>在构造函数里实例化一个 Dep 对象，并赋值给 observer 实例的 dep。</p>
<h2 id="definereactive-函数中">defineReactive 函数中</h2>
<p>在 object.defineProperty 的外面定义 dep = new Dep()。在 defineProperty 的reactiveGetter\reactiveSetter 中引用 dep。在 get 中使用 dep.depend() 收集依赖，在 set 中使用 dep.notify() 通知更新。<br>
每一个对象的属性都维护了一个 dep，这个 dep 中包含了该属性的依赖信息。</p>
<h2 id="dep-类">Dep 类</h2>
<pre class=" language-js"><code class="prism  language-js"><span class="token comment">/* @flow */</span>

</code></pre>
<p>import type Watcher from ‘./watcher’<br>
import { remove } from ‘…/util/index’<br>
import config from ‘…/config’</p>
<p>let uid = 0</p>
<p>/**</p>
<ul>
<li>A dep is an observable that can have multiple</li>
<li>directives subscribing to it.<br>
*/<br>
export default class Dep {<br>
// 静态变量，保存 Watcher 类型对象<br>
static target: ?Watcher;<br>
id: number; // 对象的ID<br>
subs: Array; // 订阅者数组 元素即 Watcher对象</li>
</ul>
<p>constructor () {<br>
<a href="http://this.id">this.id</a> = uid++<br>
this.subs = []<br>
}</p>
<p>// 添加订阅者<br>
addSub (sub: Watcher) {<br>
this.subs.push(sub)<br>
}<br>
// 删除订阅者<br>
removeSub (sub: Watcher) {<br>
remove(this.subs, sub)<br>
}<br>
// 依赖收集<br>
depend () {<br>
if (Dep.target) {<br>
Dep.target.addDep(this)<br>
}<br>
}<br>
// 通知订阅者 更新事件<br>
notify () {<br>
// stabilize the subscriber list first<br>
const subs = this.subs.slice()<br>
if (process.env.NODE_ENV !== ‘production’ &amp;&amp; !config.async) {<br>
// subs aren’t sorted in scheduler if not running async<br>
// we need to sort them now to make sure they fire in correct<br>
// order<br>
subs.sort((a, b) =&gt; <a href="http://a.id">a.id</a> - <a href="http://b.id">b.id</a>)<br>
}<br>
for (let i = 0, l = subs.length; i &lt; l; i++) {<br>
subs[i].update()<br>
}<br>
}<br>
}</p>
<p>// The current target watcher being evaluated.<br>
// This is globally unique because only one watcher<br>
// can be evaluated at a time.<br>
Dep.target = null // 全局静态变量初始赋值<br>
const targetStack = [] // 定义 Watcher 数组，用于实现 栈结构</p>
<p>// 入栈 Watcher 对象<br>
export function pushTarget (target: ?Watcher) {<br>
targetStack.push(target)<br>
Dep.target = target<br>
}</p>
<p>// 出栈 Watcher 对象<br>
export function popTarget () {<br>
targetStack.pop()<br>
Dep.target = targetStack[targetStack.length - 1]<br>
}</p>
<p>Dep 对象里面的 subs 收集的是订阅者 watcher，Dep 对象的依赖收集 depend 方法调用的是 Dep.target 的 啊addDep() 方法，也就是 Watcher 对象的 addDep() 方法。Dep.target 是一个 watcher ，本身 Dep 的 target 属性被定义为静态属性，也就是说在用 Dep 的时候 Dep.target 是全局唯一的，代表当前正在处理的属性的 watcher。<br>
Dep 对象里面的 addSub 、removeSub、notify 这些方法都是针对这个属性的 <strong>订阅者 watcher</strong>做的操作，notify方法也是调用的 watcher 上的 update 方法</p>
<p><img src="https://picx.zhimg.com/v2-cdb7752cddabfd01820f4863c990cd18_720w.jpg?source=d16d100b" alt=""></p>
<p>Dep 对象里面的 subs 建立了 属性对象到组件 watcher 的映射关系，收集了属性对象影响的 watcher。<br>
watcher 对象里面的 deps 建立了 watcher 对象到属性对象的映射关系，收集了所有影响自己的属性对象。</p>
<h3 id="depend方法">depend方法</h3>
<p>方法 depend 是一个用于依赖收集的方法，它在响应式属性被引用时执行，即在属性的 reactiveGetter\reactiveSetter 方法中执行。它会判断如果存在 Dep.target 全局唯一静态变量（值为 Watcher 对象）那么就调用 Dep.target 的 addDep 方法，参数为本 Dep 对象，Dep.target 就是 Watcher 对象，所以 Dep.target.addDep(this) 可以简单理解为 watcher.addDep(dep)。</p>
<h3 id="notify-方法">notify 方法</h3>
<p>它用来通知订阅的所有 Watcher 对象，即调用 addSub 时添加的对象。</p>
<h1 id="watcher-类">Watcher 类</h1>
<p>watcher 分别用在 渲染函数、计算属性、侦听属性三个地方，他们都需要在特定变量更新时，做出相应。</p>
<pre class=" language-js"><code class="prism  language-js"><span class="token comment">/**
 * A watcher parses an expression, collects dependencies,
 * and fires callback when the expression value changes.
 * This is used for both the $watch() api and directives.
 */</span>
<span class="token keyword">export</span> <span class="token keyword">default</span> <span class="token keyword">class</span> <span class="token class-name">Watcher</span> <span class="token punctuation">{</span>
  vm<span class="token punctuation">:</span> Component<span class="token punctuation">;</span>
  expression<span class="token punctuation">:</span> string<span class="token punctuation">;</span>
  cb<span class="token punctuation">:</span> Function<span class="token punctuation">;</span>
  id<span class="token punctuation">:</span> number<span class="token punctuation">;</span>
  deep<span class="token punctuation">:</span> boolean<span class="token punctuation">;</span>
  user<span class="token punctuation">:</span> boolean<span class="token punctuation">;</span>
  lazy<span class="token punctuation">:</span> boolean<span class="token punctuation">;</span>
  sync<span class="token punctuation">:</span> boolean<span class="token punctuation">;</span>
  dirty<span class="token punctuation">:</span> boolean<span class="token punctuation">;</span>
  active<span class="token punctuation">:</span> boolean<span class="token punctuation">;</span>
  deps<span class="token punctuation">:</span> Array<span class="token operator">&lt;</span>Dep<span class="token operator">&gt;</span><span class="token punctuation">;</span>
  newDeps<span class="token punctuation">:</span> Array<span class="token operator">&lt;</span>Dep<span class="token operator">&gt;</span><span class="token punctuation">;</span>
  depIds<span class="token punctuation">:</span> SimpleSet<span class="token punctuation">;</span>
  newDepIds<span class="token punctuation">:</span> SimpleSet<span class="token punctuation">;</span>
  before<span class="token punctuation">:</span> <span class="token operator">?</span>Function<span class="token punctuation">;</span>
  getter<span class="token punctuation">:</span> Function<span class="token punctuation">;</span>
  value<span class="token punctuation">:</span> any<span class="token punctuation">;</span>

</code></pre>
<p>constructor (<br>
vm: Component, // Vue类/组件 实例<br>
expOrFn: string | Function, // 字符表达式或函数<br>
cb: Function, // 回调函数，收到更新通知时执行<br>
options?: ?Object, // 其他选项<br>
isRenderWatcher?: boolean // 是否为渲染 watcher<br>
) {<br>
this.vm = vm<br>
if (isRenderWatcher) { // 如果是渲染 watcher，对象赋值给 vm._watcher<br>
vm._watcher = this<br>
}<br>
vm._watchers.push(this) // 对象放入 vm 的 _watchers 中<br>
// options<br>
if (options) {<br>
this.deep = !!options.deep<br>
this.user = !!options.user<br>
this.lazy = !!options.lazy<br>
this.sync = !!options.sync<br>
this.before = options.before<br>
} else {<br>
this.deep = this.user = this.lazy = this.sync = false<br>
}<br>
this.cb = cb // 回调函数<br>
<a href="http://this.id">this.id</a> = ++uid // uid for batching // 实例ID<br>
this.active = true<br>
// 初始化 dirty = lazy，主要用于计算属性<br>
this.dirty = this.lazy // for lazy watchers<br>
this.deps = [] // 当前观察的 dep<br>
this.newDeps = [] // 新收集的需要观察的 dep<br>
this.depIds = new Set() // deps 的ID集合<br>
this.newDepIds = new Set() // newDeps 的ID集合<br>
this.expression = process.env.NODE_ENV ! ‘production’<br>
? expOrFn.toString()<br>
: ‘’<br>
// parse expression for getter<br>
// expOrFn 转成 getter 函数<br>
if (typeof expOrFn = ‘function’) {<br>
this.getter = expOrFn<br>
} else {<br>
this.getter = parsePath(expOrFn)<br>
if (!this.getter) {<br>
this.getter = noop<br>
process.env.NODE_ENV !== ‘production’ &amp;&amp; warn(<br>
<code>Failed watching path: "&lt;/span&gt;&lt;span class="token interpolation"&gt;&lt;span class="token interpolation-punctuation punctuation"&gt;${&lt;/span&gt;expOrFn&lt;span class="token interpolation-punctuation punctuation"&gt;}&lt;/span&gt;&lt;/span&gt;&lt;span class="token string"&gt;"</code> +<br>
'Watcher only accepts simple dot-delimited paths. ’ +<br>
‘For full control, use a function instead.’,<br>
vm<br>
)<br>
}<br>
}<br>
// 如果不是 lazy 的 watcher 则立即执行 get 成员方法<br>
this.value = this.lazy<br>
? undefined<br>
: this.get()<br>
}</p>
<p>/**</p>
<ul>
<li>Evaluate the getter, and re-collect dependencies.<br>
*/<br>
// 调用 getter，并且收集依赖<br>
get () {<br>
pushTarget(this)<br>
let value<br>
const vm = this.vm<br>
try {<br>
value = this.getter.call(vm, vm)<br>
} catch (e) {<br>
if (this.user) {<br>
handleError(e, vm, <code>getter for watcher "&lt;/span&gt;&lt;span class="token interpolation"&gt;&lt;span class="token interpolation-punctuation punctuation"&gt;${&lt;/span&gt;&lt;span class="token keyword"&gt;this&lt;/span&gt;&lt;span class="token punctuation"&gt;.&lt;/span&gt;expression&lt;span class="token interpolation-punctuation punctuation"&gt;}&lt;/span&gt;&lt;/span&gt;&lt;span class="token string"&gt;"</code>)<br>
} else {<br>
throw e<br>
}<br>
} finally {<br>
// “touch” every property so they are all tracked as<br>
// dependencies for deep watching<br>
if (this.deep) {<br>
traverse(value)<br>
}<br>
popTarget()<br>
this.cleanupDeps()<br>
}<br>
return value<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Add a dependency to this directive.<br>
*/<br>
// 添加 dep<br>
addDep (dep: Dep) {<br>
const id = <a href="http://dep.id">dep.id</a><br>
if (!this.newDepIds.has(id)) {<br>
this.newDepIds.add(id)<br>
this.newDeps.push(dep)<br>
if (!this.depIds.has(id)) {<br>
dep.addSub(this)<br>
}<br>
}<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Clean up for dependency collection.<br>
*/<br>
// 清除旧的 dep，新的 dep 赋给 deps<br>
cleanupDeps () {<br>
let i = this.deps.length<br>
while (i–) {<br>
const dep = this.deps[i]<br>
if (!this.newDepIds.has(<a href="http://dep.id">dep.id</a>)) {<br>
dep.removeSub(this)<br>
}<br>
}<br>
let tmp = this.depIds<br>
this.depIds = this.newDepIds<br>
this.newDepIds = tmp<br>
this.newDepIds.clear()<br>
tmp = this.deps<br>
this.deps = this.newDeps<br>
this.newDeps = tmp<br>
this.newDeps.length = 0<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Subscriber interface.</li>
<li>Will be called when a dependency changes.<br>
<em>/<br>
// dep派发更新通知时执行<br>
update () {<br>
/</em> istanbul ignore else */<br>
if (this.lazy) {<br>
this.dirty = true<br>
} else if (this.sync) {<br>
this.run()<br>
} else {<br>
queueWatcher(this)<br>
}<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Scheduler job interface.</li>
<li>Will be called by the scheduler.<br>
*/<br>
// 调度函数<br>
run () {<br>
if (this.active) {<br>
const value = this.get()<br>
if (<br>
value !== this.value ||<br>
// Deep watchers and watchers on Object/Arrays should fire even<br>
// when the value is the same, because the value may<br>
// have mutated.<br>
isObject(value) ||<br>
this.deep<br>
) {<br>
// set new value<br>
const oldValue = this.value<br>
this.value = value<br>
if (this.user) {<br>
const info = <code>callback for watcher "&lt;/span&gt;&lt;span class="token interpolation"&gt;&lt;span class="token interpolation-punctuation punctuation"&gt;${&lt;/span&gt;&lt;span class="token keyword"&gt;this&lt;/span&gt;&lt;span class="token punctuation"&gt;.&lt;/span&gt;expression&lt;span class="token interpolation-punctuation punctuation"&gt;}&lt;/span&gt;&lt;/span&gt;&lt;span class="token string"&gt;"</code><br>
invokeWithErrorHandling(this.cb, this.vm, [value, oldValue], this.vm, info)<br>
} else {<br>
this.cb.call(this.vm, value, oldValue)<br>
}<br>
}<br>
}<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Evaluate the value of the watcher.</li>
<li>This only gets called for lazy watchers.<br>
*/<br>
// 求值方法，主要在 计算属性 中用<br>
evaluate () {<br>
this.value = this.get()<br>
this.dirty = false<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Depend on all deps collected by this watcher.<br>
*/<br>
// 调用本 watcher 拥有的所有 dep 的 depend 成员方法<br>
depend () {<br>
let i = this.deps.length<br>
while (i–) {<br>
this.deps[i].depend()<br>
}<br>
}</li>
</ul>
<p>/**</p>
<ul>
<li>Remove self from all dependencies’ subscriber list.<br>
*/<br>
// watcher 被销毁，清理相关配置<br>
teardown () {<br>
if (this.active) {<br>
// remove self from vm’s watcher list<br>
// this is a somewhat expensive operation so we skip it<br>
// if the vm is being destroyed.<br>
if (!this.vm._isBeingDestroyed) {<br>
remove(this.vm._watchers, this)<br>
}<br>
let i = this.deps.length<br>
while (i–) {<br>
this.deps[i].removeSub(this)<br>
}<br>
this.active = false<br>
}<br>
}<br>
}<br>
…</li>
</ul>
<h2 id="构造函数">构造函数</h2>
<p>首先他根据传入构造函数的第五个参数来确定 实例化的 watcher 对象是否是 <strong>渲染 watcher</strong> ，每一个组件都有一个 渲染watcher，如果是则把他赋值给 vm._watcher 并且也加入到 vm._watchers 中。<br>
然后初始化选项，user 代表 是否是 watch 侦听属性的 watcher，cb 通常是 watch 侦听属性中定义的回调函数。 sync 表示是否在接收到更新通知时同步执行调度方法。<br>
跟着把传参 expOrFn 转换为 getter 函数，如果 expOrFn 是字符串则通过调用 parsePath 函数转换成函数，它主要是按点切分为属性键用来访问对象的属性值。正如上面提到的，三处不同地方实例化对象时传入的 expOrFn 不同导致各自的 getter 各异，但是它们本质都是引用响应式变量，从而触发依赖收集。<br>
最后判断 lazy 的值，执行计算属性undefined或者直接取值。</p>
<h2 id="get-方法">get 方法</h2>
<p>调用 getter 函数，并在其执行过程中触发依赖收集。</p>
<p><img src="https://picx.zhimg.com/v2-cdb7752cddabfd01820f4863c990cd18_720w.jpg?source=d16d100b" alt=""></p>
<h3 id="depend-方法"><strong>depend 方法</strong></h3>
<p>是的，没有看错，Watcher 类里面也有一个 depend 方法，这个方法与 evaluate 同样是在计算属性的 computedGetter 中被调用的（computedGetter 代码见 evaluate 方法），当 Dep.target 不为空时则调用它。</p>
<pre class=" language-js"><code class="prism  language-js"><span class="token comment">/**
 * Depend on all deps collected by this watcher.
 */</span>
<span class="token function">depend</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">.</span>length
  <span class="token keyword">while</span> <span class="token punctuation">(</span>i<span class="token operator">--</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">.</span><span class="token function">depend</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>

</code></pre>
<p>前面文章“计算属性与侦听属性初始化浅析”中我们研究过计算属性的响应式定义，我们看看它的图简单回顾下：</p>
<p><img src="https://pica.zhimg.com/v2-dd70f2aefc6e4b44067cc583f14153f7_720w.jpg?source=d16d100b" alt=""></p>
<p>vue 计算属性初始化</p>
<p>这个 depend 中会遍历所有依赖的 Dep 并执行其 depend 方法，这有点绕。没关系，其实它的<strong>本质就是把计算属性 Watcher（computedWatcher）依赖的所有 Dep 同样也让 Dep.target 依赖之</strong>。因为在定义响应式计算属性的时候没有对应关联的 Dep 对象（不像 data 属性），所以 Dep.target 无法收集对计算属性的依赖。<strong>computedWatcher</strong>.depend() 之后，Dep.target 对于计算属性的 Dep 的依赖转移到对 <strong>computedWatcher</strong> 依赖的 Dep 上。</p>
<p><img src="https://picx.zhimg.com/v2-3f0214d5716caa8defb544e834baa663_720w.jpg?source=d16d100b" alt=""></p>
<p>vue 计算属性依赖收集</p>
<p>Dep.target 在依赖计算属性失败后，转而依赖计算属性依赖的 Dep，这样与直接依赖计算属性是一样的，而且这样还有个好处就是当计算属性依赖的变量有改变时，Dep.target 就能收到更新通知并做必要的响应。</p>
<h1 id="nexttick">nextTick</h1>
<p>在下次 DOM 更新循环结束之后执行延迟回调。在 DOM 更新之后执行。</p>
<p>nexttick 涉及两部分，一个是响应式原理，一个是浏览器事件循环</p>
<ul>
<li>
<p>响应式原理<br>
响应式原理的核心是数据劫持和依赖收集。<br>
vue 类里面会 new Observe 对象，observe 对象里面会执行数据响应式处理。<br>
通过 object.defineProperty() 对数据进行劫持，然后改写 getter/setter 方法，在里面注入依赖收集，通知更新的方法。</p>
</li>
<li>
<p>observe 类<br>
在 observe 里面通过 walk 方式，依次对对象中所有属性做响应式处理。<br>
walk 里面用 defineReactive() 方法，defineReactive() 里面 会为每一个遍历到的属性 new 一个 dep ，用于在 Object.defineProperty 里面 get 方法中 为收集依赖的 watcher  （响应式就是  dep 里面收集 watcher，watcher 里面收集 dep）。<br>
在 set 方法中用 dep.notify 通知更新</p>
</li>
<li>
<p>Dep类<br>
有一个静态变量 target 用来指示当前全局唯一的 watcher 用来对该 watcher 进行相关操作。<br>
Dep类里面有一个 subs 数组，里面存放的就是收集到的依赖，就是收集来的 各种watcher。<br>
Dep类里面还有一个 notify 方法，里面调用 subs 数组里每个 watcher 的 update 方法。</p>
</li>
<li>
<p>Watcher类<br>
watcher 里面会将 Dep.target 先指向自己，在实例化的时候让自己成为全局唯一的 watcher 来进行依赖收集等操作。<br>
依赖收集是通过调用 get 方法来触发的，也就是说要进行取值操作。所以在 watcher 里面，只需要做一个赋值操作，就可以完成 属性 的响应式处理。</p>
</li>
</ul>
<pre class=" language-js"><code class="prism  language-js"><span class="token keyword">let</span> uid<span class="token operator">=</span><span class="token number">0</span>
<span class="token keyword">class</span> <span class="token class-name">Watcher</span><span class="token punctuation">{</span>
    <span class="token comment">//vm即一个Vue对象，key要观察的属性，cb是观测到数据变化后需要做的操作，通常是指DOM变更</span>
    <span class="token function">constructor</span><span class="token punctuation">(</span>vm<span class="token punctuation">,</span>key<span class="token punctuation">,</span>cb<span class="token punctuation">)</span><span class="token punctuation">{</span>
       <span class="token keyword">this</span><span class="token punctuation">.</span>vm<span class="token operator">=</span>vm<span class="token punctuation">;</span>
       <span class="token keyword">this</span><span class="token punctuation">.</span>uid<span class="token operator">=</span>uid<span class="token operator">++</span><span class="token punctuation">;</span>
       <span class="token keyword">this</span><span class="token punctuation">.</span>cb<span class="token operator">=</span>cb<span class="token punctuation">;</span>
       <span class="token comment">//调用get触发依赖收集之前，把自身赋值给Dep.taget静态变量</span>
       Dep<span class="token punctuation">.</span>target<span class="token operator">=</span><span class="token keyword">this</span><span class="token punctuation">;</span>
       <span class="token comment">//触发对象上代理的get方法，执行get添加依赖</span>
       <span class="token keyword">this</span><span class="token punctuation">.</span>value<span class="token operator">=</span>vm<span class="token punctuation">.</span>$data<span class="token punctuation">[</span>key<span class="token punctuation">]</span><span class="token punctuation">;</span>
       <span class="token comment">//用完即清空</span>
       Dep<span class="token punctuation">.</span>target<span class="token operator">=</span><span class="token keyword">null</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token comment">//在调用set触发Dep的notify时要执行的update函数，用于响应数据变化执行run函数即dom变更</span>
    <span class="token function">update</span><span class="token punctuation">(</span>newValue<span class="token punctuation">)</span><span class="token punctuation">{</span>
        <span class="token comment">//值发生变化才变更</span>
        <span class="token keyword">if</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>value<span class="token operator">!==</span>newValue<span class="token punctuation">)</span><span class="token punctuation">{</span>
            <span class="token keyword">this</span><span class="token punctuation">.</span>value<span class="token operator">=</span>newValue<span class="token punctuation">;</span>
            <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token function">run</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    <span class="token comment">//执行DOM更新等操作</span>
    <span class="token function">run</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
        <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token function">cb</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>value<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<h2 id="为什么要用next-tick？">为什么要用next tick？</h2>
<p>频繁的更新 DOM 会造成严重的性能浪费。<br>
需要增加一种机制，可以在每次数据变化之后，不立即去更新 DOM ，而是将这些变更缓存起来，在适当的时机只执行一次更新 DOM 的操作。<br>
这个适当的时机就是指浏览器的<strong>事件循环</strong>机制。</p>
<h2 id="事件循环机制">事件循环机制</h2>
<p>JS 在浏览器中执行时，会被分成 宏任务和微任务两种。<br>
事件执行顺序：</p>
<ol>
<li>宏任务</li>
<li>本次宏任务内产生的所有微任务</li>
<li>更新视图（render）</li>
<li>下一个宏任务</li>
<li>重复2-4</li>
</ol>
<p><strong>更新视图是指进行 render 渲染，不是对 DOM 的更新操作</strong><br>
<strong>由于在每一次事件循环中，有一次视图渲染的操作，所以只需要在 render 之前进行 DOM 操作就好了。</strong></p>
<p><strong>使用Promise创建的是微任务，微任务会在本次事件循环同步代码执行结束后执行，使用setTimeout创建的是宏任务，同样会在此次同步代码执行完成后执行，区别是在setTimeout代码执行之前会穿插一次无效的视图渲染，因此我们尽量使用Promise创建微任务实现异步更新。</strong></p>
<h1 id="markdown-extensions">Markdown extensions</h1>
<p>StackEdit extends the standard Markdown syntax by adding extra <strong>Markdown extensions</strong>, providing you with some nice features.</p>
<blockquote>
<p><strong>ProTip:</strong> You can disable any <strong>Markdown extension</strong> in the <strong>File properties</strong> dialog.</p>
</blockquote>
<h2 id="smartypants">SmartyPants</h2>
<p>SmartyPants converts ASCII punctuation characters into “smart” typographic punctuation HTML entities. For example:</p>
<p>ASCII</p>
<p>HTML</p>
<p>Single backticks</p>
<p><code>'Isn't this fun?'</code></p>
<p>‘Isn’t this fun?’</p>
<p>Quotes</p>
<p><code>"Isn't this fun?"</code></p>
<p>“Isn’t this fun?”</p>
<p>Dashes</p>
<p><code>-- is en-dash, --- is em-dash</code></p>
<p>– is en-dash, — is em-dash</p>
<h2 id="katex">KaTeX</h2>
<p>You can render LaTeX mathematical expressions using <a href="https://khan.github.io/KaTeX/">KaTeX</a>:</p>
<p>The <em>Gamma function</em> satisfying Γ(n)=(n−1)!∀n∈NΓ(n)=(n−1)!∀n∈NΓ(n)=(n−1)!∀n∈NΓ(n)=(n−1)!∀n∈N\Gamma(n) = (n-1)!\quad\forall n\in\mathbb NΓ(n)=(n−1)!∀n∈NΓ(n)=(n−1)!∀n∈NΓ(n)=(n−1)!∀n∈NΓ(n)=(n−1)!∀n∈N is via the Euler integral</p>
<p>Γ(z)=∫0∞tz−1e−tdt .Γ(z)=∫0∞tz−1e−tdt .Γ(z)=∫0∞​tz−1e−tdt.Γ(z)=∫0∞tz−1e−tdt . \Gamma(z) = \int_0^\infty t<sup>{z-1}e</sup>{-t}dt\,. Γ(z)=∫0∞​tz−1e−tdt.Γ(z)=∫0∞tz−1e−tdt .Γ(z)=∫0∞​tz−1e−tdt.Γ(z)=∫0∞​tz−1e−tdt.</p>
<blockquote>
<p>You can find more information about <strong>LaTeX</strong> mathematical expressions <a href="http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference">here</a>.</p>
</blockquote>
<h2 id="uml-diagrams">UML diagrams</h2>
<p>You can render UML diagrams using <a href="https://mermaidjs.github.io/">Mermaid</a>. For example, this will produce a sequence diagram:</p>
<p>Link text</p>
<p>Square Rect</p>
<p>Circle</p>
<p>Round Rect</p>
<p>Rhombus</p>

