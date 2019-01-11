## Nginx代理配置和缓存

#### 代理缓存设置在代理层，每个客户端访问时都会经过，可以直接使用，可对通用的缓存内容更高效的提供内容

Cache-control
  - 浏览器：max-age
  - 代理: s-max-age

```nginx
proxy_cache_path cache levels=1:2 keys_zone=my_cache_name:10m
# levels 是否创建二级文件夹  keys_zone 缓存文件空间大小

server {
    listen 80
    server_name test.com;

    location / {
        proxy_pass http://127.0.0.1:8888
        proxy_set_header Host $host;

        proxy_cache my_cache
    }
    
}
```


```js
const http = require('http')
const fs = require('fs')

const wait = (seconds) => {
  return new Promise((resolve) => {
    setTimeout(resolve, seconds * 1000)
  })
}
http.createServer(function (request, response) {
  console.log('request come', request.url)

  if (request.url === '/') {
    const html = fs.readFileSync('test.html', 'utf8')
    response.writeHead(200, {
      'Content-Type': 'text/html'
    })
    response.end(html)
  }

  if (request.url === '/data') {
    response.writeHead(200, {
      'Cache-Control': 'max-age=2, s-maxage=20, private',
      'Vary': 'X-Test-Cache'
    })
    wait(2).then(() => response.end('success'))
  }
}).listen(8888)

console.log('server listening on 8888')
```

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
  <div>This is content, and data is: <span id="data"></span></div>
  <button id="button">click me</button>
</body>
<script>
  var index = 0
  function doRequest () {
    var data = document.getElementById('data')
    data.innerText = ''
    fetch('/data', {
      headers: {
        'X-Test-Cache': index++
      }
    }).then(function (resp) {
      return resp.text()
    }).then(function (text) {
      data.innerText = text
    })
  }
  document.getElementById('button').addEventListener('click', doRequest)
</script>
</html>
```