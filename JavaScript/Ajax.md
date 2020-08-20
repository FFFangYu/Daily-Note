这一部分，主要实现手写Ajax

### 步骤

- 创建XHR
- 发出HTTP请求
- 服务器返回xml格式字符串
- JS解析xml，并更新页面

### XHR的属性

- readState
- status
- onreadystatechange
- responseText

### XHR的方法

- open（）
- send（）
- setRequestHeader（）
- getResponseHeader（）
- getAllResponseHeaders（）

### 简单实现

```js
function ajax(){
    // 创建xhr
    let xhr = new XMLHTTPRequest()
    // 发出http请求
    xhr.open('get','https://www.gogle.com')
    // 回调的实现
    xhr.onreadystatechange = () => {
        // 表示请求已经完成
        if(xhr.readystate === 4){
            // 表示请求成功
            if(xhr.status >= 200 && xhr.status < 300){
                // 服务器返回的字符
                let str = xhr.responseText
                // 将字符解析为Json
                let obj = JSON.parse(str)
            }
        }
    }
}
```

### 最终版本

- url：地址
- method：请求方法
- body：请求体
- success：成功的回调
- fail：失败的回调

```js
function ajax = (url,method,body,success,fail) => {
    let xhr = new XMLHTTPRequest()
    xhr.open(method,url)
    xhr.onreadystatechange = () => {
        if(xhr.readystate === 4){
            if(xhr.status >=200 &&xhr.status<300){
                // 成功就将返回的结果传给成功的回调函数处理
                success.call(undefined,xhr.responseText)
            }else if(xhr.status >= 400){
                // 失败就将这个xhr传过去处理
                fail.call(undefined,xhr)
            }
        }
    }
}
```

