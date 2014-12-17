---
layout: post
title: "Octopress插入代码的若干种方法"
date: 2012-12-01 16:51
comments: true
categories: 
---
###直接在上下文中插入代码
    ```javascript foo http://foo.url/foo.js 下载
        var foo;
        foo=function(){
            console.log('i\'m foo.');
        };
    ```

Example

```javascript foo http://foo.url/foo.js raw
    var foo;
    foo=function(){
        console.log('i\'m foo.');
    };
```

### Gist Tag
在[Gist](http://gist.github.com)上粘贴代码，得到gist id，让后用以下格式粘贴。
本人推荐这种方式。

    {{ "{% gist 1342001" }} %}
Example

{% gist 1342001 %}

###jsFiddle Tag
前端攻城师喜欢常用的jsFiddle也可以引用
    {{ "{% jsfiddle bSjtc" }} %}
Example
{% jsfiddle bSjtc %}

[Octopress 文档](http://octopress.org/docs/)