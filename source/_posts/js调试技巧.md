---
title: js调试技巧
date: 2018-02-11 16:29:56
tags: [javascript]
---
### 通过vscode 调试

vscode 安装插件 Debugger for chrome

#### attach 模式

"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222



launch.json加入

``` json
{

            "type": "chrome",

            "request": "attach",

            "name": "Attach to Chrome",

            "port": 9222,

            "webRoot": "${workspaceFolder}"

},
```


