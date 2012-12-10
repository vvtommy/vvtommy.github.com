---
layout: post
title: "修正Sublime的CoffeeScript插件 0.5.3下的高亮Bug"
date: 2012-12-05 21:05
comments: true
categories: dev sublime coffeescript github git
---
今天Sublime Text2的CoffeeScript插件更新后，语法高亮不能正常工作。长时间没有动过vim的我已然残废，在用vim工作了2分钟后表示我已经废了。
于是只好Google到CoffeeScript插件Github地址，查看issue。果然另一个老兄也报告了这样的bug，项目所有者Xavura回应说，之前合并一个Pull Request时没小心测试，说让等几天他会更新，在此之前找到那个有问题的commit，对着diff自己改回来。

```diff
@@ -261,7 +261,7 @@
         </dict>
       </dict>
       <key>match</key>
-      <string>([a-zA-Z\$_](\w|\$|\.)*\s*(?!\::)((:)|(=))(?!(\s*\(.*\))?\s*((=|-)&gt;)))</string>
+      <string>([a-zA-Z\$_](\w|\$|\.)*\s*(?!\::)((:)|(=[^=]))(?!(\s*\(.*\))?\s*((=|-)&gt;)))</string>
       <key>name</key>
       <string>variable.assignment.coffee</string>
     </dict>
```
看comment好像是为了解决这么一个问题:

```CoffeeScript
if ( derp == herp )
```

这时，derp=被语法分析器解释为赋值语句。结果这哥们完全没有看后面的零宽度负预测先行断言，自顾自的加了[^=]，这意味着只要等号后不是`=`, 那么照样被语法分析器解释为赋值语句… 1秒钟修复。 

```
@@ -262,7 +262,7 @@
                </dict>
            </dict>
            <key>match</key>
-           <string>([a-zA-Z\$_](\w|\$|\.)*\s*(?!\::)((:)|(=))(?!(\s*\(.*\))?\s*(=|(=|-)&gt;)))</string>
+           <string>([a-zA-Z\$_](\w|\$|\.)*\s*(?!\::)((:)|(=[^=]))(?!(\s*\(.*\))?\s*((=|-)&gt;)))</string>
            <key>name</key>
            <string>variable.assignment.coffee</string>
        </dict>
```
fork 然后 发了一个pull request，Done。

