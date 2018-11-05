---
title: 为项目添加钉钉、邮件、微信告警通知
categories: java
date: 2018-10-12 19:07:18
tags: 
     - java
comments: true
---
通常我们在开发程序的时候，需要监测程序的状态，任务执行的结果反馈，异常报警。通常有会三种方式钉钉、邮件、微信

<!-- more -->

### 微信

1. 非spring项目，httpUtil为hutool中封装的工具类，http请求post方式，提交json数据，其中desc即为你要通知的内容，access_token为群自定义机器人的token（钉钉处查看）

``` java
String body = "{\"msgtype\": \"text\",\"text\": {\"content\": \""+desc+"\"}}";
HttpUtil.post("https://oapi.dingtalk.com/robot/send?access_token="+PropKit.get("access_token"), body);
```

2. spring 项目，不需要任何工具类，使用<code>RestTemplate</code>就可以

``` java
	String url="https://oapi.dingtalk.com/robot/send?access_token=" + accessToken;
	HttpHeaders headers = new HttpHeaders();
	headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
	String body = "{\"msgtype\": \"text\",\"text\": {\"content\": \""+desc+"\"}}";
	HttpEntity<String> entity = new HttpEntity<String>(body, headers);
	restTemplate.exchange(url, HttpMethod.POST, entity, String.class);
```

### 邮件

>示例是在spring boot项目中，添加附件，使用freemarker模板（<code>message.ftl</code>）自定义发送样式，在正文中还可以关联图片
注意jar包中图片的获取方式<code>InputStream stream = getClass().getClassLoader().getResourceAsStream("static/maillogo.png")</code>

``` bash
@Autowired
private JavaMailSender jms;

private Session s;

 //邮件的发送者
@Value("${spring.mail.username}")
private String from;

@Value("${spring.mail.receiver}")
private String receiver;

private static final Logger logger = LoggerFactory.getLogger(MailTask.class);

@Autowired
private FreeMarkerConfigurer configurer;
public void send() {
	System.getProperties().setProperty("mail.mime.splitlongparameters", "false");
	try {
		MimeMessage message = new MimeMessage(s);
		message.setFrom(new InternetAddress(from));
		InternetAddress[] listAddress = new InternetAddress[1];
		listAddress[0] = new InternetAddress(IDNMailHelper.toIdnAddress(receiver));
		message.addRecipients(Message.RecipientType.TO, listAddress);
		message.setSubject("xxx");
		Map<String, Object> model = Maps.newHashMap();
		List<CopyrightContent> list = ccdao.selectExpireList();
		if(list.size() > 0) {
			model.put("list",list);
			Template template = configurer.getConfiguration().getTemplate("message.ftl");
			String text = FreeMarkerTemplateUtils.processTemplateIntoString(template, model);
			InputStream stream = getClass().getClassLoader().getResourceAsStream("static/maillogo.png");
			File targetFile = new File("maillogo.png");
			FileUtils.copyInputStreamToFile(stream, targetFile);
			//关联图片
			MimeBodyPart img = new MimeBodyPart();
			DataHandler dh  = new DataHandler(new FileDataSource(targetFile));
			img.setDataHandler(dh);
			img.setContentID("a");
			//正文部分
			MimeBodyPart mbp = new MimeBodyPart();
			mbp.setContent(text, "text/html;charset=utf-8");
			//附件
			BodyPart filePart = new MimeBodyPart();
			Calendar calendar = Calendar.getInstance();
			SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
			calendar.add(Calendar.DATE, 60);
			String after60 = sdf.format(calendar.getTime());
			File attacheFile = new File("xxx-"+after60+".xlsx");
			generalExcel(attacheFile);
			DataSource source = new FileDataSource(attacheFile); 
			filePart.setDataHandler(new DataHandler(source)); 
		     
		    //messageBodyPart.setFileName(filename); 
		    //处理附件名称中文（附带文件路径）乱码问题 
			filePart.setFileName(MimeUtility.encodeText(attacheFile.getName()));
			MimeMultipart mm = new MimeMultipart();
			mm.addBodyPart(mbp);
			mm.addBodyPart(img);
			mm.addBodyPart(filePart);
			mm.setSubType("related");// 设置正文与图片之间的关系
			message.setContent(mm);
			jms.send(message);
		}
	} catch (Exception e) {
		logger.error(e.getMessage(), e);
	}
}
```

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>xxx到期通知</title>
</head>

<style type="text/css">
    table {
        font-family: "Trebuchet MS", Arial, Helvetica, sans-serif;
        width: 100%;
        border-collapse: collapse;
    }

    td, th {
        font-size: 1em;
        border: 1px solid #5B4A42;
        padding: 3px 7px 2px 7px;
        text-align: center;
    }

    th {
        font-size: 1.1em;
        text-align: center;
        padding-top: 5px;
        padding-bottom: 4px;
        background-color: #666666;
        color: #ffffff;
    }
</style>
<body>
<div>
    <p>你好，以下xxx即将到期</p>
    <table id="customers" style="width:80%">
        <tr>
            <th>xxx名称</th>
            <th>xxx分类</th>
            <th>xxxid</th>
            <th>xxx名称</th>
            <th>产地</th>
            <th>集数</th>
            <th>出品年代</th>
            <th>xxx时间</th>
        </tr>
        	<#assign x=0 />
        	<#list list as bean>
        		<#assign x=x+1 />
	        	<tr>
		            <td>${bean.copyrightPartyName}</td>
		            <td>${bean.programTypeName}</td>
		            <td>${bean.id?string}</td>
		            <td>${bean.seriesName}</td>
		            <td>${bean.productArea}</td>
		            <td>${bean.volumnCount?string}</td>
		            <td>${bean.releaseYear}</td>
		            <td>${bean.endTime?string('yyyy/MM/dd')}</td>
	        	</tr>
	        	<#if x == 4> <#break> </#if>
			</#list>
    </table>
    <#if x == 4><p>......</p></#if>
</div>
<br><br>
<hr style="width: 210px; height: 1px;" color="#b5c4df" size="1" align="left" /> 
  <div>
   <span>
    <div style="MARGIN: 10px; FONT-FAMILY: verdana; FONT-SIZE: 10pt">
     <div>
      <a href="mailto:xxx@xxx.cn" style="font-size: 10pt; line-height: 1.5; background-color: window;">xxx@xxx.cn</a>
      <span style="font-size: 10pt; color: rgb(0, 0, 0); line-height: 1.5; background-color: rgba(0, 0, 0, 0);">&nbsp;</span>
     </div>
     <div>
      <span style="color: rgb(0, 0, 0); background-color: rgba(0, 0, 0, 0);">xxx股份有限公司&nbsp;&nbsp;<br />Global&nbsp;United&nbsp;Technology&nbsp;Co.,Ltd&nbsp;<br />地址：北京市海淀区xxx&nbsp;100080&nbsp;</span>
     </div>
     <div>
      <img src="cid:a" border="0" />
     </div>
     <span style="color: rgb(0, 0, 0); background-color: rgba(0, 0, 0, 0);"></span>
    </div></span>
  </div> 
</body>
</html>
```

### 微信

``` java
package com.baosight.wechat.service;
 
import net.sf.json.JSONObject;
 
import org.apache.commons.httpclient.HttpStatus;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;
import org.junit.Test;
 
import com.baosight.wechat.project.baosightecmp.ConstantUtilEcmp;
import com.baosight.wechat.util.HttpUtil;
 
public class TestUnit
{
 
    @Test
    public void Test1()
    {
 
        String Access_token = HttpUtil.getAccess_token_server(ConstantUtilEcmp.APPID, ConstantUtilEcmp.APPSECRET);
        // String open_id = "oe7rSjlz1flhx7HP3-DnlgrpobqM";
        JSONObject obj = JSONObject.fromObject(Access_token);
        String token = obj.getString("access_token");
        String strJson = "{\"touser\" :\"oe7rSjlz1flhx7HP3-DnlgrpobqM\",";
        strJson += "\"msgtype\":\"text\",";
        strJson += "\"text\":{";
        strJson += "\"content\":\"Hello World\"";
        strJson += "}}";
        String url = "https://api.weixin.qq.com/cgi-bin/message/custom/send?&body=0&access_token=" + token;
 
        System.out.println(url);
        this.post(url, strJson);
    }
 
    public void post(String url, String json)
    {
        DefaultHttpClient client = new DefaultHttpClient();
        HttpPost post = new HttpPost(url);
        try
        {
            StringEntity s = new StringEntity(json);
            s.setContentEncoding("UTF-8");
            s.setContentType("application/json");
            post.setEntity(s);
 
            HttpResponse res = client.execute(post);
            if (res.getStatusLine().getStatusCode() == HttpStatus.SC_OK)
            {
                HttpEntity entity = res.getEntity();
                System.out.println(EntityUtils.toString(entity, "utf-8"));
            }
        }
        catch (Exception e)
        {
            throw new RuntimeException(e);
        }
    }
 
}
```

![](https://ws1.sinaimg.cn/large/6b162853ly1fw5o0urca4j20vj01ujrc.jpg)

转自：https://www.cnblogs.com/wggWeb/p/3767111.html