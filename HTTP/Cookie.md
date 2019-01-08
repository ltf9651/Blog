## Cookie

Cookie
```js
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html',
      'Set-Cookie': ['id=123; max-age=2', //过期时间
      'abc=456;domain=test.com', // 限制跨域
      'q=1; HttpOnly'
    })
    response.end(html)
  }

}).listen(8888)

console.log('server listening on 8888')
```
