1. 如果服务器端的[CROS](ht的frametps://developer.mozilla.org/en-US/docs/Glossary/CORS)设置正确，Turbo可以自动获取来自其他origin的frame
2. Frame的src一定要是full path(以'//', 'http://'或'https://'开头）的形式，否则请求的URL会不正确
3. Responsed frame中的js/css文件的src也要是full path
4. 除上述情况外，其他方法都不能正确加载CROS frame，比如设置frame内的a/from标签的data-turbo-frame
5. 虽然可以使用CROS frame， 但是不建议频繁使用，因为：
   1. 安全：服务器端允许跨域请求可能会泄露数据和用户信息
   2. 性能：CROS请求的preflight fetch增加了RTT
   3. 复杂度：增加了业务逻辑代码的负载度

注：这里的CROS是指需要加载的内容来自一个与[turbo-root](https://turbo.hotwired.dev/handbook/drive#setting-a-root-location)不同的origin。有关于CROS本身的解释，请参考[这里](https://javascript.info/fetch-crossorigin)