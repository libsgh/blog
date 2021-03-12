---
title: 阿里teambition网盘接口
categories: java
date: 2020-12-07 17:36:00
tags: 
     - api
     - java
     - PanIndex
comments: true
---
阿里teambition网盘登录，获取文件列表、下载链接等接口

<!-- more -->

### 阿里teambition网盘官网
[https://www.teambition.com](https://www.teambition.com)

> 为了方便分析接口，用了java示例，你可以换成你熟悉的语言

### 运行本代码，pom.xml中需引用

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.4.4</version>
</dependency>
```

### 说明

1. 登录
 * 获取token和clientId
 * 用户名密码登录，记录返回的cookie（`TEAMBITION_SESSIONID`、`TEAMBITION_SESSIONID.sig`）
2. 获取orgId（组织id，可能和团队协作功能有关）, memberId（当前你用户的id）
3. 获取rootId（根目录id）、spaceId(空间id, 可能和底层oss存储有关)
4. 获取driverId
5. 分页获取文件列表，包含下载链接，下载地址有效期可能为5小时（待测试）

> 在测试登录中，暂未出现验证码


### 接口示例

```java
public static void main(String[] args) {
		String body = HttpUtil.get("https://account.teambition.com/login/password");
		//0.登录-获取token
		String token = StrUtil.subBetween(body, "TOKEN\":\"", "\"");
		String clientId = StrUtil.subBetween(body, "CLIENT_ID\":\"", "\"");
		//1.登录-用户名密码登录,获取cookie
		String loginParam = JSONUtil.createObj().set("client_id", clientId)
							.set("email", "xxx@163.com")
							.set("password", "xxx")
							.set("response_type", "session")
							.set("token", token).toString();
		HttpResponse rep = HttpUtil.createPost("https://account.teambition.com/api/login/email").setMaxRedirectCount(1)
				.body(loginParam).execute();
		String seddionId = rep.getCookieValue("TEAMBITION_SESSIONID");
		String seddionIdSig = rep.getCookieValue("TEAMBITION_SESSIONID.sig");
		String cookie = "TEAMBITION_SESSIONID="+seddionId+";TEAMBITION_SESSIONID.sig="+seddionIdSig;
		
		//2. 获orgId, memberId
		body = HttpUtil.createGet("https://www.teambition.com/api/organizations/personal").cookie(cookie).execute().body();
		JSONObject jo = JSONUtil.parseObj(body);
		String orgId = jo.getStr("_id");
		String memberId = jo.getStr("_creatorId");
		//3.获取rootId、spaceId
		body = HttpUtil.createGet(String.format("https://pan.teambition.com/pan/api/spaces?orgId=%s&memberId=%s", orgId, memberId)).cookie(cookie).execute().body();
		String rootId = JSONUtil.parseArray(body).getByPath("$[0].rootId", String.class);
		String spaceId = JSONUtil.parseArray(body).getByPath("$[0].spaceId", String.class);
		//4.获取driverId
		body = HttpUtil.createGet(String.format("https://pan.teambition.com/pan/api/orgs/%s?orgId=%s", orgId, orgId)).cookie(cookie).execute().body();
		String driveId = JSONUtil.parseObj(body).getByPath("$.data.driveId", String.class);
		//5.获取文件列表
		body = HttpUtil.createGet(String.format("https://pan.teambition.com/pan/api/nodes?orgId=%s&offset=0&limit=100&orderBy=updateTime&orderDirection=desc&driveId=%s&spaceId=%s&parentId=%s", orgId, driveId,spaceId, rootId)).cookie(cookie).execute().body();
		System.out.println(body);
	}
}

```
跨域测试
![](https://www.teambition.com/api/works/6049bcf5d3b83500449dd6fe/download/Goose%20house%20-%20%E5%85%89%E3%82%8B%E3%81%AA%E3%82%89.jpg?signature=eyJhbGciOiJIUzI1NiJ9.eyJfd29ya0lkIjoiNjA0OWJjZjVkM2I4MzUwMDQ0OWRkNmZlIiwiZmlsZUtleSI6IjMxMjMzZDI1ODQ3ZTFjMDk3ZTBlOTA4MWZkZDZlYzY2M2JlOSIsIl91c2VySWQiOiI1ZmE0YmI4NDgxZmJhNzE0NjdhYTQ2NzEiLCJleHAiOjE2MTU3MTI2MzYsInN0b3JhZ2UiOiJzdHJpa2VyLWh6In0.ILadZmDl9AMc8rxmHtm74oq_9IEMSye6cCsBn6wfetA&_versionId=6049bcf5d3b83500449dd702)