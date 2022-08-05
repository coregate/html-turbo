Will browser auto evaluate js added by another script (and after the page laoded)?

YES, when it is firstly being appended to the DOM
NO, when:
  1. 重复添加同一个script element object. [标准文档4.3.1 The script element](http://dev.w3.org/html5/spec/the-script-element.html#attr-script-src):
  > Changing the src, type, charset, async, and defer attributes dynamically has no direct effect; these attribute are only used at specific times described below.

  2. 改变已经在DOM中的script element的innerHtml， 理由同上。
  3. 直接添加innerHtml，  可以使用Element.replaceWith使用新的script element来替换原有的script。这是由于安全原因（[refer1](https://github.com/hotwired/turbo/issues/186#issuecomment-959814926), [refer2](https://stackoverflow.com/questions/13390588/script-tag-create-with-innerhtml-of-a-div-doesnt-work)），参考[标准文档 2.5.2](https://www.w3.org/TR/2008/WD-html5-20080610/dom.html#innerhtml0):
  > script elements inserted using innerHTML do not execute when they are inserted

  参考[标准文档13.2.4.5](https://html.spec.whatwg.org/multipage/parsing.html#other-parsing-state-flags):
  > The scripting flag can be enabled even when the parser was created as part of the HTML fragment parsing algorithm, even though script elements don't execute in that case.

trick: use [Node.isConnected](https://developer.mozilla.org/en-US/docs/Web/API/Node/isConnected) to check if the element declared but not in DOM or shadow DOM

RECOMMEND to use ES6 module to load addionally js code

See other ways to do this: https://stackoverflow.com/questions/950087/how-do-i-include-a-javascript-file-in-another-javascript-file