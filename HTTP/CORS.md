## 跨域请求

Server1  发送请求
```js
const http = require('http')
const fs = require('fs')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  const html = fs.readFileSync('test.html', 'utf8')
  response.writeHead(200, {
    'Content-Type': 'text/html'
  })
  response.end(html)
}).listen(8888)

console.log('server listening on 8888')
```
test.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
    
</body>
<!-- cors1 -->
<!-- <script>
  var xhr = new XMLHttpRequest()
  xhr.open('GET', 'http://localhost:8887/')
  xhr.send()
</script> -->
<script>
  fetch('http://localhost:8887', {
    method: 'POST',
    headers: {
      'X-Test-Cors': '123'
    }
  })
</script>
</html>
```

Server2 接收请求
```js
const http = require('http')

http.createServer(function (request, response) {
  console.log('request come', request.url)

  response.writeHead(200, {
    'Access-Control-Allow-Origin': 'http://localhost:8888', 
    // Access-Control-Allow-Origin如果没填写Server1或 * 则会被拒绝请求
    'Access-Control-Allow-Headers': 'X-Test-Cors',
    'Access-Control-Allow-Methods': 'POST, PUT, DELETE',
    'Access-Control-Max-Age': '1000'//跨域请求最长时间1000秒
  })
  response.end('123')
}).listen(8887)

console.log('server listening on 8887')
```