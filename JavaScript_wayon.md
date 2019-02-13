---
title: JavaScript_wayon
date: 2019-02-01 23:37:00
tags:
- javaScript
---

﻿

***
 - 2017.10.26     **属性名和属性值获取**

&emsp;&emsp;获取js对象的所有属性的<font color=#FF5D5A>属性名数组</font>,如获取obj的属性名数组：

`var propArr = Object.getOwnPropertyNames(obj).sort();`

&emsp;&emsp;另一种获取属性名：
```js
for(attribute in obj){ 
    alert(attribute); 
} 
```
&emsp;&emsp;获取属性名对应属性值：
```js
for(attribute in obj){ 
    alert(obj[attribute]); 
} 
```
<!--more-->

***
 - 2017.11.21  **Snippets**
  - 小数保留*位小数，整数保持整数
```
    //如果原来是整数等就取原来的整数，原始值为小数保留两位小数
    if (value.toString().split(".").length > 1) {
        value= value.toFixed(2)
    }
```
 - 时间戳格式化
```
//将时间戳格式化为yyyy-MM-dd hh:mm:ss形式
function formatDate(timestamp) {
    var date = new Date(timestamp);
    Y = date.getFullYear() + '-';
    M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-';
    D = date.getDate() + ' ';
    h = date.getHours() + ':';
    // m = (date.getMinutes()<10?'0'+date.getMinutes():date.getMinutes()) + ':';
    m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes());
    s = (date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds());
    // var result = Y + M + D + h + m + s;
    var result = Y + M + D + h + m;
    return result;
}
```
 - generate sku table
```js

var result = [];//组合成产品规格集
/* specResultList 
0:{id: 36, attrName: "容量", alias: null, classifyId: 11, propValues: Array(6), …}
1:{id: 43, attrName: "颜色", alias: null, classifyId: 11, propValues: Array(7), …}
2:{id: 69, attrName: "尺寸", alias: null, classifyId: 11, propValues: Array(12), …}

propValues
0:{id: 117, propValue: "锥形1500", propName: null, propNameId: 36, gmtDatetime: null, …}
1:{id: 118, propValue: "锥形1800", propName: null, propNameId: 36, gmtDatetime: null, …}
2:{id: 119, propValue: "锥形1000", propName: null, propNameId: 36, gmtDatetime: null, …}
3:{id: 231, propValue: "250ML", propName: null, propNameId: 36, gmtDatetime: "2017-09-19 18:59:08", …}
4:{id: 232, propValue: "350ML", propName: null, propNameId: 36, gmtDatetime: "2017-09-19 18:59:08", …}
5:{id: 233, propValue: "450ML", propName: null, propNameId: 36, gmtDatetime: "2017-09-19 18:59:08", …}
 */
//某分类下规格结果集合
var specResultList = [];

//根据规格集合生成sku笛卡尔积
function generate(index, current) {
    if (index < specResultList.length - 1) {
        var specItem = specResultList[index];
        var keya = specItem.attrName;
        var items = specItem.propValues;
        if (items.length == 0) {
            generate(index + 1, current);
        }
        for (var i = 0; i < items.length; i++) {
            if (!items[i]) continue;
            var newMap = {};
            newMap = $.extend(newMap, current);
            newMap[keya] = items[i];
            generate(index + 1, newMap);
        }

    } else if (index == specResultList.length - 1) {
        var specItem = specResultList[index];
        var keya = specItem.attrName;
        var items = specItem.propValues;
        if (items.length == 0) {
            result.push(current);
        }
        for (var i = 0; i < items.length; i++) {
            if (!items[i]) continue;
            var newMap = {};
            newMap = $.extend(newMap, current);
            newMap[keya] = items[i];
            result.push(newMap);
        }
    }
}
```
```js
//生成sku表格,参数result为转化后的sku数组
function generateSkuTable(result) {
    if (result == null || result == undefined || result.length == 0)
        return;
    $(".sku-table").html("");
    //获取其下属性数组，举例格式为["大小", "容量", "颜色"]
    var propArr = Object.getOwnPropertyNames(result[0]).sort();
    console.log("propArr:", propArr);
    var skuTheadStr = "<thead>\n" +
        "                                                <tr>\n" +
        "                                                    <td class=\"thead-sku-id\" style=\"display: none\"></td>\n";
    for (var prop_p of propArr) {
        skuTheadStr += "<td class=\"thead-prop-name\">" + prop_p + "</td>\n";
    }
    skuTheadStr += "<td class=\"thead-sku-prop-str\" style=\"display: none\"></td>\n" +
        "<td class=\"thead-sku-prop-values\" style=\"display: none\"></td>\n" +
        "                                                    <td>商品图片</td>\n" +
        "                                                    <td>价格</td>\n" +
        "                                                    <td>库存</td>\n" +
        "                                                    <td width=\"80\">删除操作</td>\n" +
        "                                                </tr>\n" +
        "                                                </thead>";
    $(".sku-table").append(skuTheadStr);
    var skuTbodyStr = "        <tbody>\n";
    var tempSku = [];
    var zz = 0;
    console.log("result array for generate table:", result);
    for (tempSku of result) {
        zz++;
        skuTbodyStr += "<tr class=\"js-sku-content\">\n";
        skuTbodyStr += "<td class=\"td-sku-id\" style=\"display: none\"></td>\n";
        console.log(tempSku);
        var str = "";
        //要存储的属性字符串
        var prop_str = "";
        //要存储的属性值名组合（订单显示用）
        var prop_values = "";
        for (var prop_p of propArr) {
            var prop_pid = tempSku[prop_p].propNameId;
            var prop_vid = tempSku[prop_p].id;
            var prop_v = tempSku[prop_p].propValue;
            str += prop_p + ":" + prop_v + "::" + prop_pid + "-" + prop_vid + ",";
            prop_str += prop_pid + "-" + prop_vid + ",";
            prop_values += "、" + prop_v;
            skuTbodyStr += "<td class=\"td-prop-value\">" + prop_v + "</td>\n";
        }
        prop_values = prop_values.substr(1);
        console.log(str);
        console.log(prop_str);
        console.log(prop_values);
        var skuUrl = "";
        skuTbodyStr += "<td class=\"td-sku-prop-str\" style=\"display: none\">" + prop_str + "</td>\n" +
            "<td class=\"td-sku-prop-values\" style=\"display: none\">" + prop_values + "</td>\n" +
            " <td class=\"sku-url\" data-url=\"" + skuUrl + "\">" +
            "<div id='dz" + zz + "' data-url=\"\" class=\"sku-url-div dropzone\" name=\"file\">\n" +
            "                                    <div id='pr" + zz + "' class=\"dropzone-previews drop-preview\"></div>\n" +
            "                                    <a class=\"upload_sku_pic btn btn-inline btn-sm dz-message drop-a\">上传图片</a>\n" +
            "                                </div>" +
            "</td>" +
            "                                                    <td>\n" +
            "                                                        <div class=\"validator-form\">\n" +
            "                                                            <input custom-number=\"\" decimal=\"2\"\n" +
            "                                                                   class=\"form-control input-sm sku-price-input ng-pristine ng-untouched ng-isolate-scope ng-invalid ng-invalid-custom-number\"\n" +
            "                                                                   type=\"text\" name=\"sku_price\" value=\"\"\n" +
            "                                                                   style=\"\"></div>\n" +
            "                                                    </td>\n" +
            "                                                    <td>\n" +
            "                                                        <div class=\"validator-form\">\n" +
            "                                                            <input custom-number=\"\"\n" +
            "                                                                   class=\"form-control input-sm sku-stock-input ng-pristine ng-untouched ng-isolate-scope ng-invalid ng-invalid-custom-number\"\n" +
            "                                                                   type=\"text\" name=\"sku_inventory\" value=\"\"\n" +
            "                                                                   style=\"\"></div>\n" +
            "                                                    </td>\n" +
            "                                                    <td class=\"remove-a-sku\"><a placement=\"bottom\"\n" +
            "                                                                                maximize=\"true\"><i\n" +
            "                                                            class=\"fa fa-remove\"></i></a>\n" +
            "                                                    </td>\n" +
            "                                                </tr>\n";
    }
    skuTbodyStr += "</tbody>";
    $(".sku-table").append(skuTbodyStr);

    $(".sku-table input").validate();
    //为表格中每个图片上传处初始化dropzone设置
    initTableDropzone();
}
```
 - sku表格创建
```
//表格创建
$(document).on("change", '.sku_value', function () {
    var skuTypeArr = [];//存放SKU类型的数组
    var totalRow = 1;//总行数
    //获取所选规格数组
    skuTypeArr = getCkeckedSkuTypeArr(skuTypeArr, totalRow);

    /* skuTypeArr结果形如
     0:{attrName: "试试", propNameId: "26", propValues: Array(1), skuValueLen: 1}
     1:{attrName: "你好", propNameId: "24", propValues: Array(1), skuValueLen: 1}*/
    if (skuTypeArr.length == 0) {
        $(".sku-table").html("");
    } else {
        //已选规格集合
        specResultList = skuTypeArr;
        result = [];
        //做规格笛卡尔积，转化规格为多个sku
        generate(0, {});
        //生成sku表格
        generateSkuTable(result);
    }
});
```
***
 - 2017.11.27 **获取页面宽度，设置字体大小**
```js


//获取页面宽度，设置字体大小

(function (doc, win) {
    var docEl = doc.documentElement,
        resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
        recalc = function () {
            var clientWidth = docEl.clientWidth;
            if (!clientWidth) return;
            if (clientWidth < 750) {
                docEl.style.fontSize = 100 * (clientWidth / 750) + 'px';
            }
            else {
                docEl.style.fontSize = '100px';
            }
        };

    if (!doc.addEventListener) return;
    win.addEventListener(resizeEvt, recalc, false);
    doc.addEventListener('DOMContentLoaded', recalc, false);
})(document, window);
```
```
    //setTimeout轮询
    function showTime() {
        var today = new Date();
        alert("The time is: " + today.toString());
        //每2秒执行一次,showTime()
        if(xxx){
            setTimeout("showTime()", 2000);
        }
    }
```
***
 - 2019.01.23 **Snippet**
```js
//限制input位数和数字
oninput="if(value.length>6)value=value.slice(0,6);value=value.replace(/[^\d]/g,'')"

//小数
oninput="if(value.length>6)value=value.slice(0,6);value=value.replace(/[^\d\.]/g,'')"
```
```js
    function timestampToTime(timestamp) {
        var date = new Date(timestamp );
        Y = date.getFullYear() + '-';
        M = (date.getMonth()+1 < 10 ? '0'+(date.getMonth()+1) : date.getMonth()+1) + '-';
        D = (date.getDate()<10 ? '0'+date.getDate() : date.getDate()) + ' ';
        h = (date.getHours()<10 ? '0'+date.getHours() : date.getHours()) + ':';
        m = (date.getMinutes() <10 ? '0'+date.getMinutes() : date.getMinutes())+ ':';
        s = date.getSeconds()<10 ? '0'+date.getSeconds() : date.getSeconds();
        return Y+M+D+h+m+s;
    }
```
```js
 objPropertyRecursively2String(obj) {
                //循环将obj中的数字类型值转为字符串类型
                for (var prop in obj) {
                    if (judgeArrObj(obj[prop])) {
                        vm.objPropertyRecursively2String(obj[prop]);
                    } else if (Object.prototype.toString.call(obj[prop]) === '[object Number]') {
                        obj[prop] = this.convert2String(obj[prop])
                    }
                }
                return obj;

                function judgeArrObj(it) {
                    return !!(Object.prototype.toString.call(it) === '[object Object]' || Object.prototype.toString.call(it) === '[object Array]')

                }
            },
            convert2String(value) {
                if(value){
                    console.log("value:",value)
                    console.log("valueStr:",value.toString())
                    return value.toString();
                }else{
                    return "";
                }
            }
```

```js
   if (openInWebview()) {
                // 在app内打开
                window.postMessage("{\"type\" : \"toLogin\"}","*");
            } else {
                // 其他地方
            }
            
// 判断是否在Webview打开
function openInWebview () {
    var ua = navigator.userAgent.toLowerCase()
    if (ua.match(/MicroMessenger/i) == 'micromessenger') { // 微信浏览器判断
        return false
    } else if (ua.match(/QQ/i) == 'qq') { // QQ浏览器判断
        return false
    } else if (ua.match(/WeiBo/i) == "weibo") {
        return false
    } else {
        if (ua.match(/Android/i) != null) {
            return ua.match(/browser/i) == null
        } else if (ua.match(/iPhone/i) != null) {
            return ua.match(/safari/i) == null
        } else {
            return (ua.match(/macintosh/i) == null && ua.match(/windows/i) == null)
        }
    }
}
```
***

- **Service Worker**

```
 Service workers也可以用来做这些事情：

后台数据同步
响应来自其它源的资源请求
集中接收计算成本高的数据更新，比如地理位置和陀螺仪信息，这样多个页面就可以利用同一组数据
在客户端进行CoffeeScript，LESS，CJS/AMD等模块编译和依赖管理（用于开发目的）
后台服务钩子
自定义模板用于特定URL模式
性能增强，比如预取用户可能需要的资源，比如相册中的后面数张图片
未来service workers能够用来做更多使web平台接近原生应用的事。 值得关注的是，其他标准也能并且将会使用service worker，例如:

后台同步：启动一个service worker即使没有用户访问特定站点，也可以更新缓存
响应推送：启动一个service worker向用户发送一条信息通知新的内容可用
对时间或日期作出响应
进入地理围栏
```

------

- **npm 安装包时 npm ERR! code Z_BUF_ERROR**

 2019.02.05 安装hexo的git部署模块 *hexo-deploy-git* 的时候屡次报错

`npm ERR! code Z_BUF_ERROR`

由google后在爆栈网中找到错误原因为npm缓存的问题，所以照着answer清除npm缓存

`npm cache clean --force` 

`npm cache verify`

重试之后可行。但是感觉隐隐约约不是这个问题，就看了看其他中文回答，说是访问外网网络不稳定原因（我想了下是有可能的，因为中间切换了一下自己SSR的配置），用配置阿里npm代理的cnpm`npm install -g cnpm --registry=https://registry.npm.taobao.org`。几个方法都列举了下，谨作参考。

***