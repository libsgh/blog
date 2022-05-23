---
title: 常用jquery、javascript方法合集
categories: javascript
date: 2018-11-9 17:29:10
tags: 
     - javascript
comments: true
---

![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fx1xgzci3fj20dc078dgu.jpg)
<!-- more -->

### 1. javascript

- 格式化时间戳，返回几（刚刚、分钟、小时、天）前字样

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

- 格式化文件大小

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

- 字符串转日期

```javascript
/**
 * 字符串转日期
 * 格式：yyyy-MM-dd HH:mm:ss
 *     yyyy-MM-dd
 * @param strDate
 * @returns {Date}
 */
function strToDate(strDate){
	if(!strDate) return;
	var o = new Date(strDate.replace(/-/g,"/"));
	return o;
	
}
```

- checkbox单选

```javascript
/**
 * checkbox单选
 * @param checkboxName
 * @param form
 */
function singleCheckbox(checkboxName, form) {  
    if (checkboxName == null) return;  
    var f = form || document.forms[0];  
    checkboxs = document.getElementsByName(checkboxName);  
      
    for(i = 0; i < checkboxs.length; i++){  
        checkboxs[i].onclick = function(){  
            for (j = 0; j < checkboxs.length; j++ ){  
                if (this.value != checkboxs[j].value && checkboxs[j].checked == true){  
                    checkboxs[j].checked = false;  
                }  
            }  
        }  
    }  
}
```

- 校验变量值是否为null或空字符

```javascript
/**
 * 校验变量值是否为null或空字符（undefined、null、"" 返回false,其他返回true）
 */
function isNotBlank(value){
	if(value == undefined){
		return false;
	}
	if(value == "undefined"){
		return false;
	}
	if(value == null){
		return false;
	}
	if(value == "null"){
		return false;
	}
	if(value === ""){
		return false;
	}
	return true;
}

```
### 2. JQuery

- 将Form表单序列化成JSON对象

```javascript
$.fn.serializeObject = function() {
	var o = {};
	var a = this.serializeArray();
	$.each(a, function() {
		if (o[this.name]) {
			if (!o[this.name].push) {
				o[this.name] = [ o[this.name] ];
			}
			o[this.name].push(this.value || '');
		} else {
			o[this.name] = this.value || '';
		}
	});
	return o;
};
```

- JSON给表单域赋值，用于编辑和查看

```javascript
$.extend({
	/**
	 * 装载表单数据 jsonStr json数据格式
	 */
	loadFormData : function(jsonStr) {
		var obj = eval("(" + jsonStr + ")");
		var key, value, tagName, type, arr;
		for (x in obj) {
			key = x;
			value = obj[x];

			$("[name='" + key + "'],[name='" + key + "[]']").each(function() {
				tagName = $(this)[0].tagName;
				type = $(this).attr('type');
				if (tagName == 'INPUT') {
					if (type == 'radio') {
						$(this).attr('checked', $(this).val() == value);
					} else if (type == 'checkbox') {
						arr = value.split(',');
						for (var i = 0; i < arr.length; i++) {
							if ($(this).val() == arr[i]) {
								$(this).attr('checked', true);
								break;
							}
						}
					} else {
						$(this).val(value);
					}
				} else if (tagName == 'SELECT' || tagName == 'TEXTAREA') {
					$(this).val(value);
				}

			});
		}
	}
});
```

- 格式化日期

```javascript
/**
* 格式化日期 
* 调用方式： var str = new Date().Format("yyyy-MM-dd HH:mm:ss");
*/
Date.prototype.Format = function (fmt) { //author: meizz  
   var o = { 
       "M+": this.getMonth() + 1, //月份  
       "d+": this.getDate(), //日  
       "H+": this.getHours(), //小时  
       "m+": this.getMinutes(), //分  
       "s+": this.getSeconds(), //秒  
       "q+": Math.floor((this.getMonth() + 3) / 3), //季度  
       "S": this.getMilliseconds() //毫秒  
   }; 
   if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length)); 
   for (var k in o) 
       if (new RegExp("(" + k + ")").test(fmt)) 
           fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length))); 
   return fmt; 
} ;
```

- 数组

```javascript
/**
 * 删除数组元素
 */
Array.prototype.remove=function(dx){ 
    if(isNaN(dx)||dx>this.length){return false;} 
    for(var i=0,n=0;i<this.length;i++) 
    { 
        if(this[i]!=this[dx]) 
        { 
            this[n++]=this[i] 
        } 
    } 
    this.length-=1 
}
//检查是不是数组
function isArray(arg){
  return Object.prototype.toString.call(arg) === '[object Array]';
}
```
