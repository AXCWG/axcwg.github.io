---
layout: post
title: e-hentai 嵌入式元素 改版
category: tech
date: 2024-09-05 18:44 +0800
created-date: 2024-09-05 15:04 +0800
---
终于出头了。

我记得上一篇我还没写完：停在UI部分了。这里我补充一下，先前的老 用php写的那个 有几个问题： 

1. UI问题。老的php版本的widget是直接把exhentai的html和css给搬过来——当时我没有想到缺少响应式界面是一个问题——然后一旦把它的iframe界面缩小一点就全部崩掉了。
2. 难找Host。在我的服务器上host是不现实的：iframe在大部分现代的浏览器中都不允许安插http内容，而我的域名又没有ssl证书……github pages又只提供静态网站，VPS的话国内的大部分都要ICP备案，国外又贼贵……

综上所述，我必须转嫁一下成本。

所以……

![reactjs](https://miro.medium.com/v2/resize:fit:1400/1*x0d41ns8PTQZz4a3VbMrBg.png)

Javascript it is! 

更准确的说，是React.JS。

转嫁成本，指的就是从服务器访问并把信息发送到客户端上，转嫁到客户端发收信息。

根据我丰富的程序员经验，我决定从UI开始写起。

# UI

很简单得，用Bootstrap写响应式界面。

我也不知道在这里说什么好：源代码就在[这里](https://github.com/AXCWG/e-hentai_embed_js)，没什么好说的。

# 后端

这样的一个函数： 

```js
async function actions() {

  await fetch("https://api.e-hentai.org/api.php",
    {
      method: "POST",
      body: '{"method": "gdata","gidlist": [['+searchParams.get("gid")+',"'+searchParams.get("token") + '"]],"namespace": 1}',
      headers: new Headers({
        'Content-Type': 'application/json'
      }),

    }).then(async function (res) { requestResult = await res.json() })
    .catch((error) => console.error("Error: ", error))

  return requestResult.gmetadata[0]

}
```

一开始我的fetch body用了JSON.stringify()，不过也和我上一篇文章说的一样：EHentai处理json的post请求的方式有一点与众不同，需要直接发送字符串而不是stringify一个object，很奇怪。

这个函数return一个`promise`，`object`是`requestResult.gmetadata[0]`。

在函数最上端声明一个函数用来call并unwrap这个async函数。

```js
var gmetadataObject_react = await actions().then((res) => gmetadataObject_react = res);
```

js这个unwrap实属是有点烦人。

不管怎么样，下面就是返回`<App>`的时候了。

```js
{% raw %}
<div className='rounded' style={{ backgroundColor: "#F0F0F0", }}>
      <div className="row">
        <div className="col-md-5 d-md-none d-block" style={{ zoom: 1, textAlign: "center" }}>
          
          <img id="thumb" src={gmetadataObject_react.thumb.replace("l.jpg", "300.jpg")} style={{height:"100%"}} className="p-3" alt='thumbnail' />
        </div>
        <div className="col-md-5 d-md-block d-none" style={{ zoom: 1, textAlign: "center" }}>
          <img id="thumb" src={gmetadataObject_react.thumb} style={{height:"100%"}} className="p-3" alt='thumbnail' />
        </div>


        <div className="col-md-7 p-3" >
          <div className='container'>
            <div className="container" style={{ textAlign: "center", width: "100%" }}>
              <h3 onClick={()=>{window.top.location='https://e-hentai.org/g/'+gmetadataObject_react.gid + '/' + gmetadataObject_react.token}} className="display-6 " style={{ fontSize: "1.5rem", textAlign: "start", }} id="title">{gmetadataObject_react.title}</h3>
              <div style={{textAlign: "start"}} onClick={()=>{window.top.location='https://e-hentai.org/uploader/'+gmetadataObject_react.uploader}}>{gmetadataObject_react.uploader}</div>
              <div onClick={()=>{window.top.location='https://e-hentai.org/'+gmetadataObject_react.category.replaceAll(" ", "").toLowerCase()}} style={{textAlign:"start", fontSize: "1.3rem"}}>{gmetadataObject_react.category}</div>
              <div onClick={()=>{window.top.location='https://e-hentai.org/g/'+gmetadataObject_react.gid + '/' + gmetadataObject_react.token}} style={{textAlign:"start", fontSize: "1.3rem"}}>{gmetadataObject_react.gid}</div>

            </div>
            <div style={{ marginBottom: "2vm" }}>
              {gmetadataObject_react.tags.map((tag) => (

                <div onClick={()=>{window.top.location='https://e-hentai.org/tag/'+tag}} className='rounded px-2 py-1 m-1' style={{ backgroundColor: "gray", textDecoration: "none", color: "#F0F0F0", float: "left" }}><a style={{textDecoration: "none",color: "#F0F0F0",}} >{tag}</a></div>

              ))}
              <br></br>
            </div>
          </div>

        </div>
        
      </div>

    </div>

    {% endraw %}
```

tags由react map生成，剩下的，信息源都填的变量。

说实在的，是一个很简单的项目，但不知为何用了如此之久：一个下午呢。作业都没写。

烦人。
