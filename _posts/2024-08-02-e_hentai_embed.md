---
layout: post
title: e-hentai 嵌入式元素
category: tech
date: 2024-08-02 16:29:00 +0800
last-modified-date: 2024-08-02 16:53:00 +0800
---

起因完完全全是我想要再我的那篇《失传媒体》中放一些比较有意思的元素进去。

最初我在想，要不要借此次机会熟练一下ASPNET MVC，但是仔细想想还是算了：这个项目大小之小发挥不出来ASPNET MVC的最大特点：易管理性。

所以最后还得是……

![php](https://media.licdn.com/dms/image/D4E12AQEYqTrWsLnG4A/article-cover_image-shrink_720_1280/0/1702616887440?e=2147483647&v=beta&t=fiv7mCqZUx5JaiuZrTb9ID1sbO7GrWWSU5EKXopH2mE)

……拯救天下了。

## Fetching info

我的计算机基础奇差无比，所以对http，头文件等等一窍不通——不过也合理，Web开发可能算下来只占我编程实践万分之一的时间吧。

不管怎么样，通过一些渠道（StackOverflow）总算是开始了。前人的先例让我用php的curl，但是在我看到代码的一刻，我惊呆了：

```php 
$url = "your url";    
$content = json_encode("your data to be sent");

$curl = curl_init($url);
curl_setopt($curl, CURLOPT_HEADER, false);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HTTPHEADER,
        array("Content-type: application/json"));
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $content);

$json_response = curl_exec($curl);

$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);

if ( $status != 201 ) {
    die("Error: call to URL $url failed with status $status, response $json_response, curl_error " . curl_error($curl) . ", curl_errno " . curl_errno($curl));
}


curl_close($curl);

$response = json_decode($json_response, true);
```

---
<center>
<a href="https://stackoverflow.com/questions/6213509/send-json-post-request-with-php">转载</a><br>
这辈子没见过这么傻逼的语法，我是认真的。
</center>


---

然而我作为程序员的德行告诉我不能“以貌取代码”，所以我还是临时采用了这个方法，然而不知为何，`curl_exec()`不会返回任何值…… 正巧我又对它的第一印象不好……

果断放弃。。

正当我犹豫之时，我又看见了同一篇post的下一个回复（还特意点出Without using any external dependency or library，真是哭死）：

```php
$options = array(
  'http' => array(
    'method'  => 'POST',
    'content' => json_encode( $data ),
    'header'=>  "Content-Type: application/json\r\n" .
                "Accept: application/json\r\n"
    )
);

$context  = stream_context_create( $options );
$result = file_get_contents( $url, false, $context );
$response = json_decode( $result );
```

舒服太多了。

我参考了e-hentai wiki中对于e-hentai API的使用方法，写（从示例抄）了一个Inline JSON到`$data`里，结果……

```json
{"error":"No method provided"}
```

怎么会？Options里不是有method？

然而最后，歪打正着，去掉`json_encode()`函数而直接发送`$data`，成功。

我不知道是哪个出错了：是e-hentai的API比较特别还是php这个老东西年久失修了，但是我很确定的是，这个`json_encode()`应该是用在这里的——吧？

Don't quote on me.

——以及另外，还犯了一个错误：StackOverflow有一个回复，其中有代码是用`fopen()`读json文件，当时因为我在那个No method provided的错误上过于崩溃以至于我想试试用文件读能不能成功，结果给我坑惨了：

那个回复写的是`fopen("filename", "w");`，然而他要读取文件为什么用w？（w会直接摧毁文档中一切内容。）评论中也没有correction wtf？？？不管怎样，给我JSON删了，还好vscode比较智能。

## UI
现在该写UI了。



