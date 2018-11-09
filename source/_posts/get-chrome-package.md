---
title: 获取chrome离线版下载地址
categories: 杂
date: 2018-11-9 17:55:33
tags: 
     - 杂
comments: true
---
![](https://ws1.sinaimg.cn/large/6b162853ly1fx1yr4czpfj20go098t8o.jpg)
<!-- more -->

## 说明

### 为什么要这样做

1. 由于众所周知的原因，我们获取谷歌浏览器会受到一定的阻碍，当浏览器无法更新，或更新速度很慢时，我们需要用到离线包下载地址
2. 下载<code>70.0.3538.77_chrome_installer.exe</code>包，用解压软件打开即可解压，无须安装
3. 配合[GreenChrome](https://shuax.com/portfolio/greenchrome/)，实现chrome的便携化、功能强化、检查更新等

### 获取原理

1. 抓取chrome地址

>platform:<code>win</code>、<code>mac</code> 对应version: <code>6.3</code>、<code>46.0.2490.86</code>

```
POST https://tools.google.com/service/update2
DATA:
<?xml version='1.0' encoding='UTF-8'?>
<request protocol='3.0' version='1.3.23.9' shell_version='1.3.21.103' ismachine='0'
         sessionid='{3597644B-2952-4F92-AE55-D315F45F80A5}' installsource='ondemandcheckforupdate'
         requestid='{CD7523AD-A40D-49F4-AEEF-8C114B804658}' dedup='cr'>
<hw sse='1' sse2='1' sse3='1' ssse3='1' sse41='1' sse42='1' avx='1' physmemory='12582912' />
<os platform='win' version='6.3' arch='{{arch}}'/>
<app appid='{{appid}}' ap='{{ap}}' version='' nextversion='' lang='' brand='GGLS' client=''><updatecheck/></app>
</request>
```

2. {{appid}}，{{arch}} 和 {{ap}} 对应不同的版本号

```
{{appid}}
win_stable: 8A69D345-D564-463C-AFF1-A69D9E530F96,
win_beta: 8A69D345-D564-463C-AFF1-A69D9E530F96,
win_dev: 8A69D345-D564-463C-AFF1-A69D9E530F96,
win_canary: 4EA16AC7-FD5A-47C3-875B-DBF4A2008C20
mac_stable: com.google.Chrome
mac_beta: com.google.Chrome
mac_dev: com.google.Chrome
mac_dev: com.google.Chrome.Canary
```

```
{{arch}}
x64, x86
```

```
{{arch}}
x64, x86
```

```
{{ap}}
win_stable
    "x86": "-multi-chrome",
    "x64": "x64-stable-multi-chrome"
win_beta
    "x86": "1.1-beta",
    "x64": "x64-beta-multi-chrome"
win_dev
    "x86": "2.0-dev",
    "x64": "x64-dev-multi-chrome"
win_canary
    "x86": "",
    "x64": "x64-canary"
mac_stable
    "x86": "",
    "x64": ""
mac_beta
    "x86": "betachannel",
    "x64": "betachannel"
mac_dev
    "x86": "devchannel",
    "x64": "devchannel"
mac_canary
    "x86": "",
    "x64": ""
```

3. 返回xml中包含chrome信息

```xml
<?xml version="1.0" encoding="utf-8"?>
<response protocol="3.0" server="prod">
  <daystart elapsed_days="4330" elapsed_seconds="8996"/>
  <app appid="{4EA16AC7-FD5A-47C3-875B-DBF4A2008C20}" cohort="1:jn:pyl@0.05" cohortname="Clang-64" status="ok">
    <updatecheck status="ok">
      <urls>
        <url codebase="http://redirector.gvt1.com/edgedl/release2/chrome/KflURoSSry0_72.0.3606.0/"/>
        <url codebase="https://redirector.gvt1.com/edgedl/release2/chrome/KflURoSSry0_72.0.3606.0/"/>
        <url codebase="http://dl.google.com/release2/chrome/KflURoSSry0_72.0.3606.0/"/>
        <url codebase="https://dl.google.com/release2/chrome/KflURoSSry0_72.0.3606.0/"/>
        <url codebase="http://www.google.com/dl/release2/chrome/KflURoSSry0_72.0.3606.0/"/>
        <url codebase="https://www.google.com/dl/release2/chrome/KflURoSSry0_72.0.3606.0/"/>
      </urls>
      <manifest version="72.0.3606.0">
        <actions>
          <action arguments="--do-not-launch-chrome --chrome-sxs" event="install" run="72.0.3606.0_chrome_installer.exe"/>
          <action Version="72.0.3606.0" channel="canary64" event="postinstall" onsuccess="exitsilentlyonlaunchcmd"/>
        </actions>
        <packages>
          <package fp="1.8d3b2e00c896e250c822c8afcbb01a332239f6fd7c45dcda164dbc51e67db846" hash="/BLhz88jvTqeiaGavzEOchqxi2s=" hash_sha256="8d3b2e00c896e250c822c8afcbb01a332239f6fd7c45dcda164dbc51e67db846" name="72.0.3606.0_chrome_installer.exe" required="true" size="54769768"/>
        </packages>
      </manifest>
    </updatecheck>
  </app>
</response>
```

**从xml中科院解析出版本、大小、文件名、下载地址，其中下载地址为<code>codebase</code>+<code>name</code>**

4. 以java获取示例

```java
package com.libs;

import cn.hutool.http.HttpRequest;

public class Test {

	public static void main(String[] args) {
		String body = "<?xml version='1.0' encoding='UTF-8'?>" + 
				"<request protocol='3.0' version='1.3.23.9' shell_version='1.3.21.103' ismachine='0'" + 
				"         sessionid='{3597644B-2952-4F92-AE55-D315F45F80A5}' installsource='ondemandcheckforupdate'" + 
				"         requestid='{CD7523AD-A40D-49F4-AEEF-8C114B804658}' dedup='cr'>" + 
				"<hw sse='1' sse2='1' sse3='1' ssse3='1' sse41='1' sse42='1' avx='1' physmemory='12582912' />" + 
				"<os platform='win' version='6.3' arch='x64'/>" + 
				"<app appid='{4EA16AC7-FD5A-47C3-875B-DBF4A2008C20}' ap='x64-canary' version='' nextversion='' lang='' brand='GGLS' client=''>" + 
				"    <updatecheck/>" + 
				"</app>" + 
				"</request>";
		String result = HttpRequest
				.post("https://tools.google.com/service/update2")
				.body(body)
				.execute()
				.body();
		System.out.println(result);
	}
}
```

### 造一个轮子

根据以上，我们用定时查询，即可获取最新chrome离线包
http://noki.tk/chrome
Api: http://noki.tk/chrome/info
![](https://ws1.sinaimg.cn/large/6b162853ly1fx1zxrcsvmj20y80l4mzr.jpg)

### 后续
[GreenChrome](https://shuax.com/portfolio/greenchrome/)、离线包，这些需要手动下载解压，对于一些电脑盲来说简直是噩梦
所以就有了自动打包便携Chrome的想法，原理就是，自动获取最新包，解压，文件移动，最终压缩获得zip
不过由于程序部署在heroku上，下载速度比较慢，后期考虑传到网盘上

### 参考

https://tools.shuax.com/chrome/#/
https://csharp.love/chrome-update-tool.html
https://i-meto.com/chrome-binary/
https://github.com/google/omaha/blob/master/doc/ServerProtocolV3.md
https://github.com/alick/fzug-repo-web-test/blob/3f0d2cd0691e78d13479c8e67dd791eb5e768a03/api/utils/chrome.py


