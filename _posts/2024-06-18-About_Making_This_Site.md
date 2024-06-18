---
layout: post
title: 本網站搭建紀錄
category: tech
date: 2024-06-18 13:42:00 +0800
---


居然是Jekyll


不管怎樣，真是一段曲折的路線呢！從6.17到6.18凌晨三點多終於把這個東西搞完了。其中有一些非常隱晦的問題我直到睡覺都沒想清楚，而是等到醒來，準備寫這篇文章前幾小時才想出來解決方法。不過先苦後甜嘛，現在設置好了簡直爽到不行。這篇文章寫來主要是為了避坑，希望看到的人不要走了我的老路。

先簡述一下整個流程的順序：

1. 下載Ruby環境
    - 如果是Windows的話要額外下Gem
2. 用gem下載jekyll和bundler<br><br>
以上兩部都是屬於網站裡講得比較清楚的部分，接下來才是煩人的點。
<br><br>
3. `$ jekyll new .`
    - 不知是因為大陸互聯網還是怎樣，這一步前所未有的慢，並且運行的時候大部分時間是沒有Verbose Output的。一開始跑的時候我還以為宕了，結果最後等了好久（甚至去洗澡）才跑完
4. 配置細節

配置細節這地方要很注意——沒有接觸過“高級”網絡前端的人（也就是大部分人，和我）可能一開始會不太懂概念是什麼。我開始的時候也在想說 馬德給我怎麼複雜是幹嘛，給我一個HTML不就好了 明白之後才搞清楚他的功能強大。

```
.
├── Gemfile
├── Gemfile.lock
├── _config.yml
├── _data
│   └── menu.yml
├── _includes
│   ├── back_link.html
│   ├── goat_counter.html
│   ├── head.html
│   ├── menu_item.html
│   └── post_list.html
├── _layouts
│   ├── archive.html
│   ├── default.html
│   ├── home.html
│   ├── page.html
│   └── post.html
├── _posts
│   ├── 2024-06-18-About_Making_This_Site.md
│   └── 2024-06-18-new.md
├── _sass
│   └── no-style-please.scss
├── _screenshots
│   ├── featured-image.png
│   ├── lighthouse-report.png
│   └── page-speed-insights-report.png
├── _site
│   ├── about
│   │   └── index.html
│   ├── archive
│   │   └── index.html
│   ├── assets
│   │   ├── css
│   │   │   └── main.css
│   │   └── js
│   │       └── mouse_coords.js
│   ├── feed.xml
│   ├── index.html
│   ├── logo.png
│   ├── no-style-please.gemspec
│   ├── normandy-archive
│   │   └── index.html
│   ├── tech
│   │   └── 2024
│   │       └── 06
│   │           └── 18
│   │               └── About_Making_This_Site
│   │                   └── index.html
│   ├── tech-archive
│   │   └── index.html
│   └── test purposes
│       └── 2024
│           └── 06
│               └── 18
│                   └── new
│                       └── index.html
├── about.md
├── archive.md
├── assets
│   ├── css
│   │   └── main.scss
│   └── js
│       └── mouse_coords.js
├── index.md
├── logo.png
├── no-style-please.gemspec
├── normandy-archive.md
└── tech-archive.md
```

這個是抓的第三方的Jekyll所以文件系統可能會有差但大致都一樣。

首先，文件結構。對於一個Blog站，最重要的就是Post，那麼這些post都會在這個<mark>_posts</mark>文件夾裡面。post一定要用嚴格的語法命名。

<center><mark>Year-Month-Date-Title.md</mark></center>

每個post的片頭都要放一些固定格式的內容：

    ---
    layout: post
    title: 本網站搭建紀錄
    category: tech
    date: 2024-06-18 13:42:00 +0800
    ---

這些信息中包含了關於本文所有的內容，例如類別（category），標題（title），日期（date）以及最重要的：樣式（layout）。

這裡就不得不提到Jekyll的一個核心功能了：頁面排版模板。如果是第三方主題有修改過，所有的模板會存放在_layouts中。默認主題的話在默認主題的安裝文件中。這個模板控制了每個頁面最基本的內容，例如標題，返回，時間等。 在本頁面中(layout: post)它的代碼是這樣的：

{%raw%}
```markdown
---
layout: default
---

{%-include back_link.html-%}

<article>
  <p class="post-meta">
    <time datetime="{{ page.date }}">{{ page.date | date: site.theme_config.date_format_post }}</time>
  </p>
  
  <h1>{{ page.title }}</h1>

  {{ content }}
</article>
```
{%endraw%}

這個default又寫：
{%raw%}
```html
<!DOCTYPE html>
<html lang="{{ page.lang | default: "en" }}">
  {%- include head.html -%}
  <body a="{{ site.theme_config.appearance | default: "auto" }}">
    <main class="page-content" aria-label="Content">
      <div class="w">
        {{ content }}
      </div>
    </main>

    {%-if site.goat_counter and jekyll.environment == "production"-%}
      {%-include goat_counter.html-%}
    {%-endif-%}

    {% if page.custom_js %}
      {% for js_file in page.custom_js %}
        <script type="text/javascript" src="{{ site.baseurl }}/assets/js/{{ js_file }}.js"></script>
      {% endfor %}
    {% endif %}
  </body>
</html>
```
{%endraw%}
Jekyll提供Liquid語句的使用來實現多功能，具體的Liquid我不詳細解釋（因為不懂），可以去[這裡](https://jekyllrb.com/docs/liquid/)以及[這裡](https://shopify.github.io/liquid/)看詳細解釋。

關於時間
---
不知為何Jekyll默認編譯的時區居然是-0400而不是讀取PC上的時間。這一點可以在Liquid和附帶的RSS插件看到，真的是很匪夷所思的設計。這一點我看了很久才看出來，大家要注意。

關於主題
---
如果看到心儀的主題，安裝下來發現變白屏，請更改layout一條目（片頭中的）。

以上。
