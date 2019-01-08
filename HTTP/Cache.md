## 缓存

![缓存查询过程](https://github.com/ltf9651/Blog/blob/master/HTTP/Cache_Process.png)


更新已缓存的JS方案：JS打包完成后通过文件名增加哈希码进行比较判断，如果哈希码发生变化说明JS内容发生变化，就去变更URL加载新的JS，相当于加载一个新的静态资源文件

#### 缓存资源验证是否需要更新

- Last-Modified  上次修改时间
  - If-Modified-Since
  - If-Unmodified-Since
- Etag  对比资源的签名判断是否更新
  - If-Match
  - If-None-Match

```js
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html'
    })
    response.end(html)
  }

  if (request.url === '/script.js') {
    
    const etag = request.headers['if-none-match']
    if (etag === '777') {
      // Code 304: 忽略服务端返回的body，直接读取缓存
      response.writeHead(304, {
        'Content-Type': 'text/javascript',
        'Cache-Control': 'max-age=2000000, no-cache',
        'Last-Modified': '123',
        'Etag': '777'
      })
      response.end()
    } else {
      response.writeHead(200, {
        'Content-Type': 'text/javascript',
        'Cache-Control': 'max-age=2000000, no-cache',
        'Last-Modified': '123',
        'Etag': '777'
      })
      response.end('console.log("script loaded twice")')
    }
  }
}).listen(8888)

console.log('server listening on 8888')
```


- no-cache：可以在本地缓存，可以在代理服务器缓存，但是这个缓存要服务器验证才可以使用 
- no-store：彻底得禁用缓冲，本地和代理服务器都不缓冲，每次都从服务器获取