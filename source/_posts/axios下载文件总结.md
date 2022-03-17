---
title: axios下载文件总结
date: 2018-08-03 17:01:44
categories:
- Javascript
- Vue
tags:
- axios
---
## 总结关于axios下载文件所遇到的问题
1. 下载文件通常用a标签跳转url，但是需要手动处理参数，比如加token等，参数过多时不适用
2. axios下载通常方式为添加response拦截的方式
```javascript
// contenttype为后端返回的type，可以自定义
if (res.headers && (res.headers['content-type'] === 'application/x-msdownload' || res.headers['content-type'] === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet')) {
  downloadUrl(res.request.responseURL)
  return
}

// download url
const downloadUrl = url => {
  let iframe = document.createElement('iframe')
  iframe.style.display = 'none'
  iframe.src = url
  iframe.onload = function () {
    document.body.removeChild(iframe)
  }
  document.body.appendChild(iframe)
}
```
- 这种方式通过构建iframe来跳转下载
<!--more-->
2. 简便方式,不需要拦截,直接通过返回数据转
```javascript
this.$http({
       url: this.$http.adornUrl('/user/ctuser/export'),
       responseType: 'arraybuffer'
     }).then((res) => {
       var myURL = window.URL || window.webkitURL
       let blob = new Blob([res.data], {type: 'application/vnd.ms-excel'})
       let link = document.createElement('a')
       link.href = myURL.createObjectURL(blob)
       link.download = '111.xls'
       link.click()
       /* let objectUrl = myURL.createObjectURL(blob)
       window.location.href = objectUrl */
     })
```
- 此种方式遇到的问题
* 原本直接用的window.URL.createObjectURL方法，会导致chrome浏览器报createObjectURL
方法undefined,找到原因是浏览器内核可能是webkit，但是webkit是手机浏览器内核，不知道
什么情况，但是可以用代码判断选用哪一种：var myURL = window.URL || window.webkitURL
* 上面代码模拟了一个a标签，通过a标签点击条状的方式下载，这样可以自定义文件名
