---
layout:     post
title:      Post的ContentType
subtitle:   客户端演示
date:       2023-09-30
author:     BY
catalog: true
tags:
    - Http
---

content-type是http协议标准定义的字段，常见的content-type包括各种资源，html、jpg等等，这里讨论post请求体中的content-type，主要是ajax请求中的content-type，这篇博客暂时不讨论具体服务器如何解析各种content-type。

**1.application/x-www-form-urlencoded**

form表单默认的content-type。传参为key1=value1&key2=value2格式

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <form action="http://localhost:3000" method="post">
        <div>
          <label for="say">What greeting do you want to say?</label>
          <input name="say" id="say" value="Hi" />
        </div>
        <div>
          <label for="to">Who do you want to say it to?</label>
          <input name="to" id="to" value="Mom" />
        </div>
        <div>
          <button>Send my greetings</button>
        </div>
      </form>
</body>

</html>
```

这时发送的content-type默认就是multipart/form-data



**2.multipart/form-data**

可以通过修改form标签的enctype实现，基本等于application/x-www-form-urlencoded，常常用于文件等上传

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <form action="http://localhost:3000" method="post" enctype="multipart/form-data">
        <div>
            <label for="file">选择要上传的文件</label>
            <input type="file" id="file" name="file" multiple />
          </div>
          <div>
            <button>提交</button>
          </div>
      </form>
</body>

</html>
```

![](https://p.sda1.dev/13/e21a9d11f2505ad0e66d4a51c9e47e46/01.png)



**3.application/json**

axios默认的content-type，原生ajax可以通过setRequestHeader设置content-type

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <button class="target">测试</button>
    <script>
        function get(url) {
            return new Promise(function (fulfil, reject) {
                var xhr = new XMLHttpRequest();

                xhr.open('POST', url);

                xhr.onload = function () {
                    fulfil(xhr.responseText);
                };

                // xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
                // xhr.send("foo=bar&lorem=ipsum");

                // xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
                xhr.setRequestHeader("Content-Type", "application/json");
                xhr.send({
                    text: "1"
                });

                xhr.onerror = reject;
            });
        }

        var target = document.querySelector('.target');

        target.addEventListener('click', function () {
            get('http://localhost:3000/')
                .then(function (data) {
                    target.innerHTML = data;
                });
        });

    </script>
</body>

</html>
```

需要注意的是，koa后台需要设置access-control-allow-headers

```javascript
const Koa = require('koa')
const app = new Koa()

app.use(async ctx => {
  // console.log('进来了');
  ctx.status = 200
  ctx.set('Access-Control-Allow-Origin', '*');
  ctx.set('Access-Control-Allow-Credentials', 'true');
  ctx.set("WWW-Authenticate", `Basic realm="oauth2/client"`)
  ctx.set("access-control-allow-headers", "content-type")
  // ctx.cookies.set(
  //     'cid', 
  //     'hello world',
  //     {
  //       domain: 'onlyCookie',  
  //       path: '/onlyCookie',      
  //       maxAge: 10 * 60 * 1000, 
  //       expires: new Date('2024-02-15'),  
  //       httpOnly: false,
  //       overwrite: false  
  //     }
  // )
  ctx.cookies.set('TestCookiePost', 'Test13',
    {
      // domain: '*',
      // path: '/',
      maxAge: 60 * 1000,
      expires: new Date('2024-02-15'),
      httpOnly: false,
      overwrite: false
    }
  );

  // if (ctx.url === '/' && ctx.method === 'GET') {
  //   let html = `
  //     <form action="/" method="post">
  //     <div>
  //       <label for="say">What greeting do you want to say?</label>
  //       <input name="say" id="say" value="Hi" />
  //     </div>
  //     <div>
  //       <label for="to">Who do you want to say it to?</label>
  //       <input name="to" id="to" value="Mom" />
  //     </div>
  //     <div>
  //       <button>Send my greetings</button>
  //     </div>
  //   </form>
  //     `
    ctx.body = 'Hello'
  // }
})

app.listen(3000)

```

**application/json会发送预检请求**

postman将application/json，application/xml等放在了row中，并且提供了binary进行文件上传
