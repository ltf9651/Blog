## 缓存

![缓存查询过程](https://github.com/ltf9651/Blog/blob/master/HTTP/cache_validate.png)


更新已缓存的JS方案：JS打包完成后通过文件名增加哈希码进行比较判断，如果哈希码发生变化说明JS内容发生变化，就去变更URL加载新的JS，相当于加载一个新的静态资源文件

#### 缓存验证

