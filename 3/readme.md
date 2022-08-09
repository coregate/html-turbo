1. Frame是通过ID来进行匹配和替换的，所以**请注意ID的值**
2. 两个或多个拥有相同ID但是不同src的frame可以被正确加载，但是不建议这样使用
3. data-turbo-frame的值如果不匹配当前页面上的任何一个frame的ID，turbo就会更新内容到标签（a/from）所在的frame，但是要注意response中frame的ID要匹配这个frame
4. CSS完全遵照浏览器行为，其**作用范围也是全局的**
5. JS的加载情况和上一节讨论过的行为一致
   1. Turbo visit的时候会被执行两次（因为会被render两次）
   2. data-turbo-eval="false"会使js完全不执行（因为这个js是turbo通过replaceWith触发的，而不是浏览器自动执行的）