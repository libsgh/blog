---
title: 常用jquery、javascript方法合集
categories: javascript
date: 2018-11-9 17:29:10
tags: 
     - javascript
comments: true
---

![](https://ws1.sinaimg.cn/large/6b162853ly1fx1xgzci3fj20dc078dgu.jpg)
<!-- more -->

### 1. javascript
1.1. 格式化时间戳，返回几（刚刚、分钟、小时、天）前字样

``` javascript
function timeago(dateTimeStamp){
    var minute = 1000 * 60;
    var hour = minute * 60;
    var day = hour * 24;
    var week = day * 7;
    var halfamonth = day * 15;
    var month = day * 30;
    var now = new Date().getTime();
    var diffValue = now - dateTimeStamp;

    if(diffValue < 0){
        return "";
    }
    var minC = diffValue/minute;
    var hourC = diffValue/hour;
    var dayC = diffValue/day;
    var weekC = diffValue/week;
    var monthC = diffValue/month;
    if(monthC >= 1 && monthC <= 3){
        result = " " + parseInt(monthC) + "月前";
    }else if(weekC >= 1 && weekC <= 3){
        result = " " + parseInt(weekC) + "周前";
    }else if(dayC >= 1 && dayC <= 6){
        result = " " + parseInt(dayC) + "天前";
    }else if(hourC >= 1 && hourC <= 23){
        result = " " + parseInt(hourC) + "小时前";
    }else if(minC >= 1 && minC <= 59){
        result =" " + parseInt(minC) + "分钟前";
    }else if(diffValue >= 0 && diffValue <= minute){
        result = "刚刚";
    }else {
        var datetime = new Date();
        datetime.setTime(dateTimeStamp);
        var Nyear = datetime.getFullYear();
        var Nmonth = datetime.getMonth() + 1 < 10 ? "0" + (datetime.getMonth() + 1) : datetime.getMonth() + 1;
        var Ndate = datetime.getDate() < 10 ? "0" + datetime.getDate() : datetime.getDate();
        var Nhour = datetime.getHours() < 10 ? "0" + datetime.getHours() : datetime.getHours();
        var Nminute = datetime.getMinutes() < 10 ? "0" + datetime.getMinutes() : datetime.getMinutes();
        var Nsecond = datetime.getSeconds() < 10 ? "0" + datetime.getSeconds() : datetime.getSeconds();
        result = Nyear + "-" + Nmonth + "-" + Ndate;
    }
    return result;
}
```

1.2. 格式化文件大小

```javascript
function getFileSize(fileByte) {
    var fileSizeByte = fileByte;
    var fileSizeMsg = "";
    if (fileSizeByte < 1048576) fileSizeMsg = (fileSizeByte / 1024).toFixed(2) + " KB";
    else if (fileSizeByte == 1048576) fileSizeMsg = "1MB";
    else if (fileSizeByte > 1048576 && fileSizeByte < 1073741824) fileSizeMsg = (fileSizeByte / (1024 * 1024)).toFixed(2) + " MB";
    else if (fileSizeByte > 1048576 && fileSizeByte == 1073741824) fileSizeMsg = "1GB";
    else if (fileSizeByte > 1073741824 && fileSizeByte < 1099511627776) fileSizeMsg = (fileSizeByte / (1024 * 1024 * 1024)).toFixed(2) + " GB";
    else fileSizeMsg = "文件超过1TB";
    return fileSizeMsg;
}
```