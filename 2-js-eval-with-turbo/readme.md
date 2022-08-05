Will auto-loaded js still work with Turbo?

YES, 但是要注意：因为页面没有full refresh，已经申请的变量注意不要重复申请。error复现：顺序访问 A->B->A->B， B中申请的变量，比如s1、s2等会有redeclaration error。参考c.html的处理方式

***body中的*代码重复执行的问题，参考d.html：**
  1. addEventListener所添加的listener会被反复执行，即使新的页面没有相关代码，因为页面没有刷新，js vm的context也还是同一个
  2. 并且，每次回到d.html的时候，如果是Turbo restore，addEventListener会被Turbo再次执行一次（从cache中取出并render的时候），因此会重复添加相同的listener一次；而如果是Turbo visit（advance）其可能会被[执行两次](https://turbo.hotwired.dev/reference/events)：
  > turbo:render fires after Turbo renders the page. This event fires twice during an application visit to a cached location: once after rendering the cached version, and again after rendering the fresh version.

根据需求选择合适的方法避免：
  1. 使用addEventListener的once选项或每次使用后删除listener。[参考](https://www.educative.io/answers/how-to-ensure-an-event-listener-is-only-fired-once-in-javascript)
  2. 对于UI交互逻辑，使用和Turbo兼容的UI框架，比如[stimulus](https://stimulus.hotwired.dev/)
  3. 使用onclick等方法，只在单一element上绑定一个事件
  4. 使用event delegation等模式，自行实现，统一管理注册和监听的事件。[参考](https://javascript.info/event-delegation)
  5. *把代码放在head里面，而不是body，Turbo只会**append** tag进入head，而不会主动刷新head。这也带来一个问题，就是当head中的js被修改了的时候，新的代码会被append在head中，但是旧的代码不会被remove，因此需要使用data-turbo-track="reload"来主动刷新页面。[参考](https://turbo.hotwired.dev/handbook/building#loading-your-application%E2%80%99s-javascript-bundle)
  6. 在head中添加`name="turbo-cache-control"`或者在script元素上添加`data-turbo-cache="false"`来禁止cache
  7. 添加`data-turbo-eval="false"`以禁止Turbo执行script。 **但是**由Turbo添加的innerHtml中的script element不会被浏览器自动evaluate（上一节已经讨论过），可以在代码执行过一次后再添加annotation：
  `document.currentScript.setAttribute("data-turbo-eval", "false")`
  8. 使操作幂等，注意可能会造成性能问题（主要是在频繁使页面re-render的时候）
  9. 
最好的解决方案还是通过合理组织代码，尽量避免发生上述情况：
  1. 尽量把通用的js代码打包，放入head
  2. 尽量使用html&css实现，比如[dropdown menu等](https://geeknote.net/Rei/posts/1323)
  3. 使用stimulus controller组织UI交互
  4. ...


*待验证：cache在页面render之前就生成了，但是之后页面的更改会依然会更新到cache中，是否是因为DOM饮用的关系？（[refer1](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode)，[refer2](http://lengyun.github.io/js/3-2-3dynamicAndStatic.html)）*



#### [script](https://turbo.hotwired.dev/handbook/building#working-with-script-elements): 

> Your browser automatically loads and evaluates any \<script> elements present on the initial page load.

> When you navigate to a new page, Turbo Drive looks for any \<script> elements in the new page’s <head> which aren’t present on the current page. Then it appends them to the current \<head> where they’re loaded and evaluated by the browser. You can use this to load additional JavaScript files on-demand.

> Turbo Drive evaluates \<script> elements in a page’s \<body> each time it renders the page. You can use inline body scripts to set up per-page JavaScript state or bootstrap client-side models. To install behavior, or to perform more complex operations when the page changes, avoid script elements and use the turbo:load event instead.

>Annotate \<script> elements with data-turbo-eval="false" if you do not want Turbo to evaluate them after rendering. Note that this annotation will not prevent your browser from evaluating scripts on the initial page load.

#### [cache](https://turbo.hotwired.dev/handbook/building#understanding-caching):

> When navigating by history (via Restoration Visits), Turbo Drive will restore the page from cache without loading a fresh copy from the network, if possible.

> Otherwise, during standard navigation (via Application Visits), Turbo Drive will immediately restore the page from cache and display it as a preview while simultaneously loading a fresh copy from the network. This gives the illusion of instantaneous page loads for frequently accessed locations.

> Turbo Drive saves a copy of the current page to its cache just before rendering a new page. Note that Turbo Drive copies the page using cloneNode(true), which means any attached event listeners and associated data are discarded.

#### [cloneNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode):

> Cloning a node copies all of its attributes and their values, including intrinsic (inline) listeners. It does not copy event listeners added using addEventListener() or those assigned to element properties (e.g., node.onclick = someFunction). Additionally, for a \<canvas> element, the painted image is not copied. 

#### [REFER](https://turbo.hotwired.dev/handbook/building#observing-navigation-events)

#### [REFER](https://github.com/turbolinks/turbolinks/issues/167)
