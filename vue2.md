---


---

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

  <span class="token function">constructor</span> <span class="token punctuation">(</span>value<span class="token punctuation">:</span> any<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment">// 初始化赋值</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>value <span class="token operator">=</span> value
    <span class="token keyword">this</span><span class="token punctuation">.</span>dep <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Dep</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>vmCount <span class="token operator">=</span> <span class="token number">0</span>
    <span class="token comment">// 给 vaule 值定义 __ob__ 属性，值为本对象，这个后续有用到</span>
    <span class="token function">def</span><span class="token punctuation">(</span>value<span class="token punctuation">,</span> <span class="token string">'__ob__'</span><span class="token punctuation">,</span> <span class="token keyword">this</span><span class="token punctuation">)</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>Array<span class="token punctuation">.</span><span class="token function">isArray</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment">// --------------------------------------------</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span>hasProto<span class="token punctuation">)</span> <span class="token punctuation">{</span>                              <span class="token comment">//</span>
        <span class="token function">protoAugment</span><span class="token punctuation">(</span>value<span class="token punctuation">,</span> arrayMethods<span class="token punctuation">)</span>          <span class="token comment">//</span>
      <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>                                     <span class="token comment">// 数组响应式改造</span>
        <span class="token function">copyAugment</span><span class="token punctuation">(</span>value<span class="token punctuation">,</span> arrayMethods<span class="token punctuation">,</span> arrayKeys<span class="token punctuation">)</span><span class="token comment">//</span>
      <span class="token punctuation">}</span> <span class="token comment">// ------------------------------------------</span>
      <span class="token comment">// 数组则遍历元素逐个调用 observe 响应式化</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token function">observeArray</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span>
    <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
      <span class="token comment">// 单个数据对象调用 walk 成员方法</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token function">walk</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */</span>
  <span class="token function">walk</span> <span class="token punctuation">(</span>obj<span class="token punctuation">:</span> Object<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">const</span> keys <span class="token operator">=</span> Object<span class="token punctuation">.</span><span class="token function">keys</span><span class="token punctuation">(</span>obj<span class="token punctuation">)</span>
    <span class="token comment">// 遍历数据对象所有键，调用 defineReactive 设置为响应式的属性</span>
    <span class="token keyword">for</span> <span class="token punctuation">(</span><span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> i <span class="token operator">&lt;</span> keys<span class="token punctuation">.</span>length<span class="token punctuation">;</span> i<span class="token operator">++</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token function">defineReactive</span><span class="token punctuation">(</span>obj<span class="token punctuation">,</span> keys<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Observe a list of Array items.
   */</span>
  <span class="token function">observeArray</span> <span class="token punctuation">(</span>items<span class="token punctuation">:</span> Array<span class="token operator">&lt;</span>any<span class="token operator">&gt;</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment">// 遍历数组，每个元素传入 observe 响应式化</span>
    <span class="token keyword">for</span> <span class="token punctuation">(</span><span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">,</span> l <span class="token operator">=</span> items<span class="token punctuation">.</span>length<span class="token punctuation">;</span> i <span class="token operator">&lt;</span> l<span class="token punctuation">;</span> i<span class="token operator">++</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token function">observe</span><span class="token punctuation">(</span>items<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>

<span class="token comment">// helpers</span>

<span class="token comment">/**
 * Augment a target Object or Array by intercepting
 * the prototype chain using __proto__
 */</span>
<span class="token keyword">function</span> <span class="token function">protoAugment</span> <span class="token punctuation">(</span>target<span class="token punctuation">,</span> src<span class="token punctuation">:</span> Object<span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token comment">/* eslint-disable no-proto */</span>
  target<span class="token punctuation">.</span>__proto__ <span class="token operator">=</span> src
  <span class="token comment">/* eslint-enable no-proto */</span>
<span class="token punctuation">}</span>

<span class="token comment">/**
 * Augment a target Object or Array by defining
 * hidden properties.
 */</span>
<span class="token comment">/* istanbul ignore next */</span>
<span class="token keyword">function</span> <span class="token function">copyAugment</span> <span class="token punctuation">(</span>target<span class="token punctuation">:</span> Object<span class="token punctuation">,</span> src<span class="token punctuation">:</span> Object<span class="token punctuation">,</span> keys<span class="token punctuation">:</span> Array<span class="token operator">&lt;</span>string<span class="token operator">&gt;</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">for</span> <span class="token punctuation">(</span><span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">,</span> l <span class="token operator">=</span> keys<span class="token punctuation">.</span>length<span class="token punctuation">;</span> i <span class="token operator">&lt;</span> l<span class="token punctuation">;</span> i<span class="token operator">++</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">const</span> key <span class="token operator">=</span> keys<span class="token punctuation">[</span>i<span class="token punctuation">]</span>
    <span class="token function">def</span><span class="token punctuation">(</span>target<span class="token punctuation">,</span> key<span class="token punctuation">,</span> src<span class="token punctuation">[</span>key<span class="token punctuation">]</span><span class="token punctuation">)</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
<span class="token operator">...</span>

</code></pre>
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

<span class="token keyword">import</span> type Watcher <span class="token keyword">from</span> <span class="token string">'./watcher'</span>
<span class="token keyword">import</span> <span class="token punctuation">{</span> remove <span class="token punctuation">}</span> <span class="token keyword">from</span> <span class="token string">'../util/index'</span>
<span class="token keyword">import</span> config <span class="token keyword">from</span> <span class="token string">'../config'</span>

<span class="token keyword">let</span> uid <span class="token operator">=</span> <span class="token number">0</span>

<span class="token comment">/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */</span>
<span class="token keyword">export</span> <span class="token keyword">default</span> <span class="token keyword">class</span> <span class="token class-name">Dep</span> <span class="token punctuation">{</span>
  <span class="token comment">// 静态变量，保存 Watcher 类型对象</span>
  <span class="token keyword">static</span> target<span class="token punctuation">:</span> <span class="token operator">?</span>Watcher<span class="token punctuation">;</span>
  id<span class="token punctuation">:</span> number<span class="token punctuation">;</span>               <span class="token comment">// 对象的ID</span>
  subs<span class="token punctuation">:</span> Array<span class="token operator">&lt;</span>Watcher<span class="token operator">&gt;</span><span class="token punctuation">;</span>     <span class="token comment">// 订阅者数组 元素即 Watcher对象</span>

  <span class="token function">constructor</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>id <span class="token operator">=</span> uid<span class="token operator">++</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>subs <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span>
  <span class="token punctuation">}</span>

  <span class="token comment">// 添加订阅者</span>
  <span class="token function">addSub</span> <span class="token punctuation">(</span>sub<span class="token punctuation">:</span> Watcher<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>subs<span class="token punctuation">.</span><span class="token function">push</span><span class="token punctuation">(</span>sub<span class="token punctuation">)</span>
  <span class="token punctuation">}</span>
  <span class="token comment">// 删除订阅者</span>
  <span class="token function">removeSub</span> <span class="token punctuation">(</span>sub<span class="token punctuation">:</span> Watcher<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">remove</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>subs<span class="token punctuation">,</span> sub<span class="token punctuation">)</span>
  <span class="token punctuation">}</span>
  <span class="token comment">// 依赖收集</span>
  <span class="token function">depend</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>Dep<span class="token punctuation">.</span>target<span class="token punctuation">)</span> <span class="token punctuation">{</span>
      Dep<span class="token punctuation">.</span>target<span class="token punctuation">.</span><span class="token function">addDep</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
  <span class="token comment">// 通知订阅者 更新事件</span>
  <span class="token function">notify</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment">// stabilize the subscriber list first</span>
    <span class="token keyword">const</span> subs <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>subs<span class="token punctuation">.</span><span class="token function">slice</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>process<span class="token punctuation">.</span>env<span class="token punctuation">.</span>NODE_ENV <span class="token operator">!==</span> <span class="token string">'production'</span> <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span>config<span class="token punctuation">.</span><span class="token keyword">async</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment">// subs aren't sorted in scheduler if not running async</span>
      <span class="token comment">// we need to sort them now to make sure they fire in correct</span>
      <span class="token comment">// order</span>
      subs<span class="token punctuation">.</span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">(</span>a<span class="token punctuation">,</span> b<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> a<span class="token punctuation">.</span>id <span class="token operator">-</span> b<span class="token punctuation">.</span>id<span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">for</span> <span class="token punctuation">(</span><span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">,</span> l <span class="token operator">=</span> subs<span class="token punctuation">.</span>length<span class="token punctuation">;</span> i <span class="token operator">&lt;</span> l<span class="token punctuation">;</span> i<span class="token operator">++</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      subs<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">.</span><span class="token function">update</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>

<span class="token comment">// The current target watcher being evaluated.</span>
<span class="token comment">// This is globally unique because only one watcher</span>
<span class="token comment">// can be evaluated at a time.</span>
Dep<span class="token punctuation">.</span>target <span class="token operator">=</span> <span class="token keyword">null</span>       <span class="token comment">// 全局静态变量初始赋值</span>
<span class="token keyword">const</span> targetStack <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span>  <span class="token comment">// 定义 Watcher 数组，用于实现 栈结构</span>

<span class="token comment">// 入栈 Watcher 对象</span>
<span class="token keyword">export</span> <span class="token keyword">function</span> <span class="token function">pushTarget</span> <span class="token punctuation">(</span>target<span class="token punctuation">:</span> <span class="token operator">?</span>Watcher<span class="token punctuation">)</span> <span class="token punctuation">{</span>
  targetStack<span class="token punctuation">.</span><span class="token function">push</span><span class="token punctuation">(</span>target<span class="token punctuation">)</span>
  Dep<span class="token punctuation">.</span>target <span class="token operator">=</span> target
<span class="token punctuation">}</span>

<span class="token comment">// 出栈 Watcher 对象</span>
<span class="token keyword">export</span> <span class="token keyword">function</span> <span class="token function">popTarget</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  targetStack<span class="token punctuation">.</span><span class="token function">pop</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  Dep<span class="token punctuation">.</span>target <span class="token operator">=</span> targetStack<span class="token punctuation">[</span>targetStack<span class="token punctuation">.</span>length <span class="token operator">-</span> <span class="token number">1</span><span class="token punctuation">]</span>
<span class="token punctuation">}</span>

</code></pre>
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
<p>作者：小问<br>
链接：<a href="https://zhuanlan.zhihu.com/p/547538212">https://zhuanlan.zhihu.com/p/547538212</a><br>
来源：知乎<br>
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。</p>
<pre class=" language-js"><code class="prism  language-js"><span class="token operator">...</span>
<span class="token comment">/**
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

  <span class="token function">constructor</span> <span class="token punctuation">(</span>
    vm<span class="token punctuation">:</span> Component<span class="token punctuation">,</span>               <span class="token comment">// Vue类/组件 实例</span>
    expOrFn<span class="token punctuation">:</span> string <span class="token operator">|</span> Function<span class="token punctuation">,</span>  <span class="token comment">// 字符表达式或函数</span>
    cb<span class="token punctuation">:</span> Function<span class="token punctuation">,</span>                <span class="token comment">// 回调函数，收到更新通知时执行</span>
    options<span class="token operator">?</span><span class="token punctuation">:</span> <span class="token operator">?</span>Object<span class="token punctuation">,</span>           <span class="token comment">// 其他选项</span>
    isRenderWatcher<span class="token operator">?</span><span class="token punctuation">:</span> boolean    <span class="token comment">// 是否为渲染 watcher</span>
  <span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>vm <span class="token operator">=</span> vm
    <span class="token keyword">if</span> <span class="token punctuation">(</span>isRenderWatcher<span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token comment">// 如果是渲染 watcher，对象赋值给 vm._watcher</span>
      vm<span class="token punctuation">.</span>_watcher <span class="token operator">=</span> <span class="token keyword">this</span>
    <span class="token punctuation">}</span>
    vm<span class="token punctuation">.</span>_watchers<span class="token punctuation">.</span><span class="token function">push</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span> <span class="token comment">// 对象放入 vm 的 _watchers 中</span>
    <span class="token comment">// options</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>options<span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>deep <span class="token operator">=</span> <span class="token operator">!</span><span class="token operator">!</span>options<span class="token punctuation">.</span>deep
      <span class="token keyword">this</span><span class="token punctuation">.</span>user <span class="token operator">=</span> <span class="token operator">!</span><span class="token operator">!</span>options<span class="token punctuation">.</span>user
      <span class="token keyword">this</span><span class="token punctuation">.</span>lazy <span class="token operator">=</span> <span class="token operator">!</span><span class="token operator">!</span>options<span class="token punctuation">.</span>lazy
      <span class="token keyword">this</span><span class="token punctuation">.</span>sync <span class="token operator">=</span> <span class="token operator">!</span><span class="token operator">!</span>options<span class="token punctuation">.</span>sync
      <span class="token keyword">this</span><span class="token punctuation">.</span>before <span class="token operator">=</span> options<span class="token punctuation">.</span>before
    <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>deep <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>user <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>lazy <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>sync <span class="token operator">=</span> <span class="token boolean">false</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>cb <span class="token operator">=</span> cb <span class="token comment">// 回调函数</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>id <span class="token operator">=</span> <span class="token operator">++</span>uid <span class="token comment">// uid for batching // 实例ID</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>active <span class="token operator">=</span> <span class="token boolean">true</span>
    <span class="token comment">// 初始化 dirty = lazy，主要用于计算属性</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>dirty <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>lazy <span class="token comment">// for lazy watchers</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>deps <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span>     <span class="token comment">// 当前观察的 dep</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>newDeps <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span>  <span class="token comment">// 新收集的需要观察的 dep</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>depIds <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Set</span><span class="token punctuation">(</span><span class="token punctuation">)</span>     <span class="token comment">// deps 的ID集合</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>newDepIds <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Set</span><span class="token punctuation">(</span><span class="token punctuation">)</span>  <span class="token comment">// newDeps 的ID集合</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>expression <span class="token operator">=</span> process<span class="token punctuation">.</span>env<span class="token punctuation">.</span>NODE_ENV <span class="token operator">!==</span> <span class="token string">'production'</span>
      <span class="token operator">?</span> expOrFn<span class="token punctuation">.</span><span class="token function">toString</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
      <span class="token punctuation">:</span> <span class="token string">''</span>
    <span class="token comment">// parse expression for getter</span>
    <span class="token comment">// expOrFn 转成 getter 函数</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">typeof</span> expOrFn <span class="token operator">===</span> <span class="token string">'function'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>getter <span class="token operator">=</span> expOrFn
    <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>getter <span class="token operator">=</span> <span class="token function">parsePath</span><span class="token punctuation">(</span>expOrFn<span class="token punctuation">)</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token keyword">this</span><span class="token punctuation">.</span>getter<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword">this</span><span class="token punctuation">.</span>getter <span class="token operator">=</span> noop
        process<span class="token punctuation">.</span>env<span class="token punctuation">.</span>NODE_ENV <span class="token operator">!==</span> <span class="token string">'production'</span> <span class="token operator">&amp;&amp;</span> <span class="token function">warn</span><span class="token punctuation">(</span>
          <span class="token template-string"><span class="token string">`Failed watching path: "</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>expOrFn<span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">" `</span></span> <span class="token operator">+</span>
          <span class="token string">'Watcher only accepts simple dot-delimited paths. '</span> <span class="token operator">+</span>
          <span class="token string">'For full control, use a function instead.'</span><span class="token punctuation">,</span>
          vm
        <span class="token punctuation">)</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    <span class="token comment">// 如果不是 lazy 的 watcher 则立即执行 get 成员方法</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>value <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>lazy
      <span class="token operator">?</span> undefined
      <span class="token punctuation">:</span> <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Evaluate the getter, and re-collect dependencies.
   */</span>
   <span class="token comment">// 调用 getter，并且收集依赖</span>
  <span class="token keyword">get</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">pushTarget</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span>
    <span class="token keyword">let</span> value
    <span class="token keyword">const</span> vm <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>vm
    <span class="token keyword">try</span> <span class="token punctuation">{</span>
      value <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>getter<span class="token punctuation">.</span><span class="token function">call</span><span class="token punctuation">(</span>vm<span class="token punctuation">,</span> vm<span class="token punctuation">)</span>
    <span class="token punctuation">}</span> <span class="token keyword">catch</span> <span class="token punctuation">(</span><span class="token class-name">e</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>user<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token function">handleError</span><span class="token punctuation">(</span>e<span class="token punctuation">,</span> vm<span class="token punctuation">,</span> <span class="token template-string"><span class="token string">`getter for watcher "</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span><span class="token keyword">this</span><span class="token punctuation">.</span>expression<span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">"`</span></span><span class="token punctuation">)</span>
      <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
        <span class="token keyword">throw</span> e
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span> <span class="token keyword">finally</span> <span class="token punctuation">{</span>
      <span class="token comment">// "touch" every property so they are all tracked as</span>
      <span class="token comment">// dependencies for deep watching</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>deep<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token function">traverse</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span>
      <span class="token punctuation">}</span>
      <span class="token function">popTarget</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token function">cleanupDeps</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">return</span> value
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Add a dependency to this directive.
   */</span>
   <span class="token comment">// 添加 dep</span>
  <span class="token function">addDep</span> <span class="token punctuation">(</span>dep<span class="token punctuation">:</span> Dep<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">const</span> id <span class="token operator">=</span> dep<span class="token punctuation">.</span>id
    <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token keyword">this</span><span class="token punctuation">.</span>newDepIds<span class="token punctuation">.</span><span class="token function">has</span><span class="token punctuation">(</span>id<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>newDepIds<span class="token punctuation">.</span><span class="token function">add</span><span class="token punctuation">(</span>id<span class="token punctuation">)</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>newDeps<span class="token punctuation">.</span><span class="token function">push</span><span class="token punctuation">(</span>dep<span class="token punctuation">)</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token keyword">this</span><span class="token punctuation">.</span>depIds<span class="token punctuation">.</span><span class="token function">has</span><span class="token punctuation">(</span>id<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        dep<span class="token punctuation">.</span><span class="token function">addSub</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Clean up for dependency collection.
   */</span>
   <span class="token comment">// 清除旧的 dep，新的 dep 赋给 deps</span>
  <span class="token function">cleanupDeps</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">.</span>length
    <span class="token keyword">while</span> <span class="token punctuation">(</span>i<span class="token operator">--</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">const</span> dep <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">[</span>i<span class="token punctuation">]</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token keyword">this</span><span class="token punctuation">.</span>newDepIds<span class="token punctuation">.</span><span class="token function">has</span><span class="token punctuation">(</span>dep<span class="token punctuation">.</span>id<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        dep<span class="token punctuation">.</span><span class="token function">removeSub</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">let</span> tmp <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>depIds
    <span class="token keyword">this</span><span class="token punctuation">.</span>depIds <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>newDepIds
    <span class="token keyword">this</span><span class="token punctuation">.</span>newDepIds <span class="token operator">=</span> tmp
    <span class="token keyword">this</span><span class="token punctuation">.</span>newDepIds<span class="token punctuation">.</span><span class="token function">clear</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    tmp <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>deps
    <span class="token keyword">this</span><span class="token punctuation">.</span>deps <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>newDeps
    <span class="token keyword">this</span><span class="token punctuation">.</span>newDeps <span class="token operator">=</span> tmp
    <span class="token keyword">this</span><span class="token punctuation">.</span>newDeps<span class="token punctuation">.</span>length <span class="token operator">=</span> <span class="token number">0</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */</span>
   <span class="token comment">// dep派发更新通知时执行</span>
  <span class="token function">update</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment">/* istanbul ignore else */</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>lazy<span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>dirty <span class="token operator">=</span> <span class="token boolean">true</span>
    <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>sync<span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token function">run</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
      <span class="token function">queueWatcher</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */</span>
   <span class="token comment">// 调度函数</span>
  <span class="token function">run</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>active<span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">const</span> value <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span>
        value <span class="token operator">!==</span> <span class="token keyword">this</span><span class="token punctuation">.</span>value <span class="token operator">||</span>
        <span class="token comment">// Deep watchers and watchers on Object/Arrays should fire even</span>
        <span class="token comment">// when the value is the same, because the value may</span>
        <span class="token comment">// have mutated.</span>
        <span class="token function">isObject</span><span class="token punctuation">(</span>value<span class="token punctuation">)</span> <span class="token operator">||</span>
        <span class="token keyword">this</span><span class="token punctuation">.</span>deep
      <span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment">// set new value</span>
        <span class="token keyword">const</span> oldValue <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>value
        <span class="token keyword">this</span><span class="token punctuation">.</span>value <span class="token operator">=</span> value
        <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>user<span class="token punctuation">)</span> <span class="token punctuation">{</span>
          <span class="token keyword">const</span> info <span class="token operator">=</span> <span class="token template-string"><span class="token string">`callback for watcher "</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span><span class="token keyword">this</span><span class="token punctuation">.</span>expression<span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">"`</span></span>
          <span class="token function">invokeWithErrorHandling</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>cb<span class="token punctuation">,</span> <span class="token keyword">this</span><span class="token punctuation">.</span>vm<span class="token punctuation">,</span> <span class="token punctuation">[</span>value<span class="token punctuation">,</span> oldValue<span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token keyword">this</span><span class="token punctuation">.</span>vm<span class="token punctuation">,</span> info<span class="token punctuation">)</span>
        <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
          <span class="token keyword">this</span><span class="token punctuation">.</span>cb<span class="token punctuation">.</span><span class="token function">call</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>vm<span class="token punctuation">,</span> value<span class="token punctuation">,</span> oldValue<span class="token punctuation">)</span>
        <span class="token punctuation">}</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */</span>
   <span class="token comment">// 求值方法，主要在 计算属性 中用</span>
  <span class="token function">evaluate</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>value <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span><span class="token keyword">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token keyword">this</span><span class="token punctuation">.</span>dirty <span class="token operator">=</span> <span class="token boolean">false</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Depend on all deps collected by this watcher.
   */</span>
   <span class="token comment">// 调用本 watcher 拥有的所有 dep 的 depend 成员方法</span>
  <span class="token function">depend</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">.</span>length
    <span class="token keyword">while</span> <span class="token punctuation">(</span>i<span class="token operator">--</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">.</span><span class="token function">depend</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token comment">/**
   * Remove self from all dependencies' subscriber list.
   */</span>
   <span class="token comment">// watcher 被销毁，清理相关配置</span>
  <span class="token function">teardown</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>active<span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment">// remove self from vm's watcher list</span>
      <span class="token comment">// this is a somewhat expensive operation so we skip it</span>
      <span class="token comment">// if the vm is being destroyed.</span>
      <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token keyword">this</span><span class="token punctuation">.</span>vm<span class="token punctuation">.</span>_isBeingDestroyed<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token function">remove</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">.</span>vm<span class="token punctuation">.</span>_watchers<span class="token punctuation">,</span> <span class="token keyword">this</span><span class="token punctuation">)</span>
      <span class="token punctuation">}</span>
      <span class="token keyword">let</span> i <span class="token operator">=</span> <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">.</span>length
      <span class="token keyword">while</span> <span class="token punctuation">(</span>i<span class="token operator">--</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword">this</span><span class="token punctuation">.</span>deps<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">.</span><span class="token function">removeSub</span><span class="token punctuation">(</span><span class="token keyword">this</span><span class="token punctuation">)</span>
      <span class="token punctuation">}</span>
      <span class="token keyword">this</span><span class="token punctuation">.</span>active <span class="token operator">=</span> <span class="token boolean">false</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>

</code></pre>
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
<h1 id="markdown-extensions">Markdown extensions</h1>
<p>StackEdit extends the standard Markdown syntax by adding extra <strong>Markdown extensions</strong>, providing you with some nice features.</p>
<blockquote>
<p><strong>ProTip:</strong> You can disable any <strong>Markdown extension</strong> in the <strong>File properties</strong> dialog.</p>
</blockquote>
<h2 id="smartypants">SmartyPants</h2>
<p>SmartyPants converts ASCII punctuation characters into “smart” typographic punctuation HTML entities. For example:</p>

<table>
<thead>
<tr>
<th></th>
<th>ASCII</th>
<th>HTML</th>
</tr>
</thead>
<tbody>
<tr>
<td>Single backticks</td>
<td><code>'Isn't this fun?'</code></td>
<td>‘Isn’t this fun?’</td>
</tr>
<tr>
<td>Quotes</td>
<td><code>"Isn't this fun?"</code></td>
<td>“Isn’t this fun?”</td>
</tr>
<tr>
<td>Dashes</td>
<td><code>-- is en-dash, --- is em-dash</code></td>
<td>– is en-dash, — is em-dash</td>
</tr>
</tbody>
</table><h2 id="katex">KaTeX</h2>
<p>You can render LaTeX mathematical expressions using <a href="https://khan.github.io/KaTeX/">KaTeX</a>:</p>
<p>The <em>Gamma function</em> satisfying <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML"><semantics><mrow><mi mathvariant="normal">Γ</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo><mo>=</mo><mo stretchy="false">(</mo><mi>n</mi><mo>−</mo><mn>1</mn><mo stretchy="false">)</mo><mo stretchy="false">!</mo><mspace width="1em"></mspace><mi mathvariant="normal">∀</mi><mi>n</mi><mo>∈</mo><mi mathvariant="double-struck">N</mi></mrow><annotation encoding="application/x-tex">\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord">Γ</span><span class="mopen">(</span><span class="mord mathnormal">n</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mopen">(</span><span class="mord mathnormal">n</span><span class="mspace" style="margin-right: 0.222222em;"></span><span class="mbin">−</span><span class="mspace" style="margin-right: 0.222222em;"></span></span><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord">1</span><span class="mclose">)!</span><span class="mspace" style="margin-right: 1em;"></span><span class="mord">∀</span><span class="mord mathnormal">n</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">∈</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 0.68889em; vertical-align: 0em;"></span><span class="mord mathbb">N</span></span></span></span></span> is via the Euler integral</p>
<p><span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><semantics><mrow><mi mathvariant="normal">Γ</mi><mo stretchy="false">(</mo><mi>z</mi><mo stretchy="false">)</mo><mo>=</mo><msubsup><mo>∫</mo><mn>0</mn><mi mathvariant="normal">∞</mi></msubsup><msup><mi>t</mi><mrow><mi>z</mi><mo>−</mo><mn>1</mn></mrow></msup><msup><mi>e</mi><mrow><mo>−</mo><mi>t</mi></mrow></msup><mi>d</mi><mi>t</mi> <mi mathvariant="normal">.</mi></mrow><annotation encoding="application/x-tex">
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord">Γ</span><span class="mopen">(</span><span class="mord mathnormal" style="margin-right: 0.04398em;">z</span><span class="mclose">)</span><span class="mspace" style="margin-right: 0.277778em;"></span><span class="mrel">=</span><span class="mspace" style="margin-right: 0.277778em;"></span></span><span class="base"><span class="strut" style="height: 2.32624em; vertical-align: -0.91195em;"></span><span class="mop"><span class="mop op-symbol large-op" style="margin-right: 0.44445em; position: relative; top: -0.001125em;">∫</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.41429em;"><span class="" style="top: -1.78805em; margin-left: -0.44445em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">0</span></span></span><span class="" style="top: -3.8129em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight">∞</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.91195em;"><span class=""></span></span></span></span></span></span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord"><span class="mord mathnormal">t</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.864108em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathnormal mtight" style="margin-right: 0.04398em;">z</span><span class="mbin mtight">−</span><span class="mord mtight">1</span></span></span></span></span></span></span></span></span><span class="mord"><span class="mord mathnormal">e</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.843556em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mtight">−</span><span class="mord mathnormal mtight">t</span></span></span></span></span></span></span></span></span><span class="mord mathnormal">d</span><span class="mord mathnormal">t</span><span class="mspace" style="margin-right: 0.166667em;"></span><span class="mord">.</span></span></span></span></span></span></p>
<blockquote>
<p>You can find more information about <strong>LaTeX</strong> mathematical expressions <a href="http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference">here</a>.</p>
</blockquote>
<h2 id="uml-diagrams">UML diagrams</h2>
<p>You can render UML diagrams using <a href="https://mermaidjs.github.io/">Mermaid</a>. For example, this will produce a sequence diagram:</p>
<pre class=" language-mermaid"><svg id="mermaid-svg-RUcpa7jdEfNttwZm" width="100%" xmlns="http://www.w3.org/2000/svg" height="567" style="max-width: 843px;" viewBox="-50 -10 843 567"><style>#mermaid-svg-RUcpa7jdEfNttwZm{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#000000;}#mermaid-svg-RUcpa7jdEfNttwZm .error-icon{fill:#552222;}#mermaid-svg-RUcpa7jdEfNttwZm .error-text{fill:#552222;stroke:#552222;}#mermaid-svg-RUcpa7jdEfNttwZm .edge-thickness-normal{stroke-width:2px;}#mermaid-svg-RUcpa7jdEfNttwZm .edge-thickness-thick{stroke-width:3.5px;}#mermaid-svg-RUcpa7jdEfNttwZm .edge-pattern-solid{stroke-dasharray:0;}#mermaid-svg-RUcpa7jdEfNttwZm .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-svg-RUcpa7jdEfNttwZm .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-svg-RUcpa7jdEfNttwZm .marker{fill:#666;stroke:#666;}#mermaid-svg-RUcpa7jdEfNttwZm .marker.cross{stroke:#666;}#mermaid-svg-RUcpa7jdEfNttwZm svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-svg-RUcpa7jdEfNttwZm .actor{stroke:hsl(0,0%,83%);fill:#eee;}#mermaid-svg-RUcpa7jdEfNttwZm text.actor > tspan{fill:#333;stroke:none;}#mermaid-svg-RUcpa7jdEfNttwZm .actor-line{stroke:#666;}#mermaid-svg-RUcpa7jdEfNttwZm .messageLine0{stroke-width:1.5;stroke-dasharray:none;stroke:#333;}#mermaid-svg-RUcpa7jdEfNttwZm .messageLine1{stroke-width:1.5;stroke-dasharray:2,2;stroke:#333;}#mermaid-svg-RUcpa7jdEfNttwZm #arrowhead path{fill:#333;stroke:#333;}#mermaid-svg-RUcpa7jdEfNttwZm .sequenceNumber{fill:white;}#mermaid-svg-RUcpa7jdEfNttwZm #sequencenumber{fill:#333;}#mermaid-svg-RUcpa7jdEfNttwZm #crosshead path{fill:#333;stroke:#333;}#mermaid-svg-RUcpa7jdEfNttwZm .messageText{fill:#333;stroke:#333;}#mermaid-svg-RUcpa7jdEfNttwZm .labelBox{stroke:hsl(0,0%,83%);fill:#eee;}#mermaid-svg-RUcpa7jdEfNttwZm .labelText,#mermaid-svg-RUcpa7jdEfNttwZm .labelText > tspan{fill:#333;stroke:none;}#mermaid-svg-RUcpa7jdEfNttwZm .loopText,#mermaid-svg-RUcpa7jdEfNttwZm .loopText > tspan{fill:#333;stroke:none;}#mermaid-svg-RUcpa7jdEfNttwZm .loopLine{stroke-width:2px;stroke-dasharray:2,2;stroke:hsl(0,0%,83%);fill:hsl(0,0%,83%);}#mermaid-svg-RUcpa7jdEfNttwZm .note{stroke:hsl(60,100%,23.3333333333%);fill:#ffa;}#mermaid-svg-RUcpa7jdEfNttwZm .noteText,#mermaid-svg-RUcpa7jdEfNttwZm .noteText > tspan{fill:#333;stroke:none;}#mermaid-svg-RUcpa7jdEfNttwZm .activation0{fill:#f4f4f4;stroke:#666;}#mermaid-svg-RUcpa7jdEfNttwZm .activation1{fill:#f4f4f4;stroke:#666;}#mermaid-svg-RUcpa7jdEfNttwZm .activation2{fill:#f4f4f4;stroke:#666;}#mermaid-svg-RUcpa7jdEfNttwZm:root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}#mermaid-svg-RUcpa7jdEfNttwZm sequence{fill:apa;}</style><g></g><g><line id="actor3" x1="75" y1="5" x2="75" y2="556" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: Open-Sans, &quot;sans-serif&quot;;"><tspan x="75" dy="0">Alice</tspan></text></g><g><line id="actor4" x1="331" y1="5" x2="331" y2="556" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="256" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="331" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: Open-Sans, &quot;sans-serif&quot;;"><tspan x="331" dy="0">Bob</tspan></text></g><g><line id="actor5" x1="568" y1="5" x2="568" y2="556" class="actor-line" stroke-width="0.5px" stroke="#999"></line><rect x="493" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="568" y="32.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: Open-Sans, &quot;sans-serif&quot;;"><tspan x="568" dy="0">John</tspan></text></g><defs><marker id="arrowhead" refX="9" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z"></path></marker></defs><defs><marker id="crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="16" refY="4"><path fill="black" stroke="#000000" stroke-width="1px" d="M 9,2 V 6 L16,4 Z" style="stroke-dasharray: 0, 0;"></path><path fill="none" stroke="#000000" stroke-width="1px" d="M 0,1 L 6,7 M 6,1 L 0,7" style="stroke-dasharray: 0, 0;"></path></marker></defs><defs><marker id="filled-head" refX="18" refY="7" markerWidth="20" markerHeight="28" orient="auto"><path d="M 18,7 L9,13 L14,7 L9,1 Z"></path></marker></defs><defs><marker id="sequencenumber" refX="15" refY="15" markerWidth="60" markerHeight="40" orient="auto"><circle cx="15" cy="15" r="6"></circle></marker></defs><text x="203" y="80" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">Hello Bob, how are you?</text><line x1="75" y1="117" x2="331" y2="117" class="messageLine0" stroke-width="2" stroke="none" marker-end="url(#arrowhead)" style="fill: none;"></line><text x="450" y="132" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">How about you John?</text><line x1="331" y1="169" x2="568" y2="169" class="messageLine1" stroke-width="2" stroke="none" marker-end="url(#arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line><text x="203" y="184" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">I am good thanks!</text><line x1="331" y1="221" x2="75" y2="221" class="messageLine1" stroke-width="2" stroke="none" marker-end="url(#crosshead)" style="stroke-dasharray: 3, 3; fill: none;"></line><text x="450" y="236" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">I am good thanks!</text><line x1="331" y1="273" x2="568" y2="273" class="messageLine0" stroke-width="2" stroke="none" marker-end="url(#crosshead)" style="fill: none;"></line><g><rect x="593" y="283" fill="#EDF2AE" stroke="#666" width="150" height="84" rx="0" ry="0" class="note"></rect><text x="668" y="288" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="668">Bob thinks a long</tspan></text><text x="668" y="304" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="668">long time, so long</tspan></text><text x="668" y="320" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="668">that the text does</tspan></text><text x="668" y="336" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="noteText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 14px; font-weight: 400;"><tspan x="668">not fit on a row.</tspan></text></g><text x="203" y="382" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">Checking with John...</text><line x1="331" y1="419" x2="75" y2="419" class="messageLine1" stroke-width="2" stroke="none" style="stroke-dasharray: 3, 3; fill: none;"></line><text x="322" y="434" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" class="messageText" dy="1em" style="font-family: &quot;trebuchet ms&quot;, verdana, arial, sans-serif; font-size: 16px; font-weight: 400;">Yes... John, how are you?</text><line x1="75" y1="471" x2="568" y2="471" class="messageLine0" stroke-width="2" stroke="none" style="fill: none;"></line><g><rect x="0" y="491" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="75" y="523.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: Open-Sans, &quot;sans-serif&quot;;"><tspan x="75" dy="0">Alice</tspan></text></g><g><rect x="256" y="491" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="331" y="523.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: Open-Sans, &quot;sans-serif&quot;;"><tspan x="331" dy="0">Bob</tspan></text></g><g><rect x="493" y="491" fill="#eaeaea" stroke="#666" width="150" height="65" rx="3" ry="3" class="actor"></rect><text x="568" y="523.5" dominant-baseline="central" alignment-baseline="central" class="actor" style="text-anchor: middle; font-size: 14px; font-weight: 400; font-family: Open-Sans, &quot;sans-serif&quot;;"><tspan x="568" dy="0">John</tspan></text></g></svg></pre>
<p>And this will produce a flow chart:</p>
<pre class=" language-mermaid"><svg id="mermaid-svg-a08wampVnTScb0Se" width="100%" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" height="174.4375" style="max-width: 502.75px;" viewBox="0 0 502.75 174.4375"><style>#mermaid-svg-a08wampVnTScb0Se{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#000000;}#mermaid-svg-a08wampVnTScb0Se .error-icon{fill:#552222;}#mermaid-svg-a08wampVnTScb0Se .error-text{fill:#552222;stroke:#552222;}#mermaid-svg-a08wampVnTScb0Se .edge-thickness-normal{stroke-width:2px;}#mermaid-svg-a08wampVnTScb0Se .edge-thickness-thick{stroke-width:3.5px;}#mermaid-svg-a08wampVnTScb0Se .edge-pattern-solid{stroke-dasharray:0;}#mermaid-svg-a08wampVnTScb0Se .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-svg-a08wampVnTScb0Se .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-svg-a08wampVnTScb0Se .marker{fill:#666;stroke:#666;}#mermaid-svg-a08wampVnTScb0Se .marker.cross{stroke:#666;}#mermaid-svg-a08wampVnTScb0Se svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-svg-a08wampVnTScb0Se .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#000000;}#mermaid-svg-a08wampVnTScb0Se .cluster-label text{fill:#333;}#mermaid-svg-a08wampVnTScb0Se .cluster-label span{color:#333;}#mermaid-svg-a08wampVnTScb0Se .label text,#mermaid-svg-a08wampVnTScb0Se span{fill:#000000;color:#000000;}#mermaid-svg-a08wampVnTScb0Se .node rect,#mermaid-svg-a08wampVnTScb0Se .node circle,#mermaid-svg-a08wampVnTScb0Se .node ellipse,#mermaid-svg-a08wampVnTScb0Se .node polygon,#mermaid-svg-a08wampVnTScb0Se .node path{fill:#eee;stroke:#999;stroke-width:1px;}#mermaid-svg-a08wampVnTScb0Se .node .label{text-align:center;}#mermaid-svg-a08wampVnTScb0Se .node.clickable{cursor:pointer;}#mermaid-svg-a08wampVnTScb0Se .arrowheadPath{fill:#333333;}#mermaid-svg-a08wampVnTScb0Se .edgePath .path{stroke:#666;stroke-width:1.5px;}#mermaid-svg-a08wampVnTScb0Se .flowchart-link{stroke:#666;fill:none;}#mermaid-svg-a08wampVnTScb0Se .edgeLabel{background-color:white;text-align:center;}#mermaid-svg-a08wampVnTScb0Se .edgeLabel rect{opacity:0.5;background-color:white;fill:white;}#mermaid-svg-a08wampVnTScb0Se .cluster rect{fill:hsl(210,66.6666666667%,95%);stroke:#26a;stroke-width:1px;}#mermaid-svg-a08wampVnTScb0Se .cluster text{fill:#333;}#mermaid-svg-a08wampVnTScb0Se .cluster span{color:#333;}#mermaid-svg-a08wampVnTScb0Se div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(-160,0%,93.3333333333%);border:1px solid #26a;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-svg-a08wampVnTScb0Se:root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}#mermaid-svg-a08wampVnTScb0Se flowchart{fill:apa;}</style><g><g class="output"><g class="clusters"></g><g class="edgePaths"><g class="edgePath LS-A LE-B" id="L-A-B" style="opacity: 1;"><path class="path" d="M109.66244612068965,67.609375L170.0546875,38.859375L246.125,38.859375" marker-end="url(https://stackedit.io/app#arrowhead5)" style="fill:none"></path><defs><marker id="arrowhead5" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath LS-A LE-C" id="L-A-C" style="opacity: 1;"><path class="path" d="M109.66244612068965,114.328125L170.0546875,143.078125L226.921875,143.078125" marker-end="url(https://stackedit.io/app#arrowhead6)" style="fill:none"></path><defs><marker id="arrowhead6" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath LS-B LE-D" id="L-B-D" style="opacity: 1;"><path class="path" d="M307.84375,38.859375L352.046875,38.859375L400.1027516807447,68.9128733192553" marker-end="url(https://stackedit.io/app#arrowhead7)" style="fill:none"></path><defs><marker id="arrowhead7" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath LS-C LE-D" id="L-C-D" style="opacity: 1;"><path class="path" d="M327.046875,143.078125L352.046875,143.078125L400.1027516807447,114.0246266807447" marker-end="url(https://stackedit.io/app#arrowhead8)" style="fill:none"></path><defs><marker id="arrowhead8" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g></g><g class="edgeLabels"><g class="edgeLabel" transform="translate(170.0546875,38.859375)" style="opacity: 1;"><g transform="translate(-31.8671875,-13.359375)" class="label"><rect rx="0" ry="0" width="63.734375" height="26.71875"></rect><foreignObject width="63.734375" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-A-B" class="edgeLabel L-LS-A' L-LE-B">Link text</span></div></foreignObject></g></g><g class="edgeLabel" transform="" style="opacity: 1;"><g transform="translate(0,0)" class="label"><rect rx="0" ry="0" width="0" height="0"></rect><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-A-C" class="edgeLabel L-LS-A' L-LE-C"></span></div></foreignObject></g></g><g class="edgeLabel" transform="" style="opacity: 1;"><g transform="translate(0,0)" class="label"><rect rx="0" ry="0" width="0" height="0"></rect><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-B-D" class="edgeLabel L-LS-B' L-LE-D"></span></div></foreignObject></g></g><g class="edgeLabel" transform="" style="opacity: 1;"><g transform="translate(0,0)" class="label"><rect rx="0" ry="0" width="0" height="0"></rect><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-C-D" class="edgeLabel L-LS-C' L-LE-D"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node default" id="flowchart-A-24" transform="translate(60.59375,90.96875)" style="opacity: 1;"><rect rx="0" ry="0" x="-52.59375" y="-23.359375" width="105.1875" height="46.71875" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-42.59375,-13.359375)"><foreignObject width="85.1875" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Square Rect</div></foreignObject></g></g></g><g class="node default" id="flowchart-B-25" transform="translate(276.984375,38.859375)" style="opacity: 1;"><circle x="-30.859375" y="-23.359375" r="30.859375" class="label-container"></circle><g class="label" transform="translate(0,0)"><g transform="translate(-20.859375,-13.359375)"><foreignObject width="41.71875" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Circle</div></foreignObject></g></g></g><g class="node default" id="flowchart-C-27" transform="translate(276.984375,143.078125)" style="opacity: 1;"><rect rx="5" ry="5" x="-50.0625" y="-23.359375" width="100.125" height="46.71875" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-40.0625,-13.359375)"><foreignObject width="80.125" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Round Rect</div></foreignObject></g></g></g><g class="node default" id="flowchart-D-29" transform="translate(435.8984375,90.96875)" style="opacity: 1;"><polygon points="58.8515625,0 117.703125,-58.8515625 58.8515625,-117.703125 0,-58.8515625" transform="translate(-58.8515625,58.8515625)" class="label-container"></polygon><g class="label" transform="translate(0,0)"><g transform="translate(-32.03125,-13.359375)"><foreignObject width="64.0625" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Rhombus</div></foreignObject></g></g></g></g></g></g></svg></pre>

