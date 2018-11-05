---
title: spring boot 实现文件上传下载
categories: spring boot
date: 2018-10-18 19:26:29
tags: 
     - spring boot
     - java
comments: true
---
最近在做文件上传下载功能，so记录一下

### 上传文件
>这里实现的是导入功能，效果是点击一个按钮弹出选择文件的框，只能选后缀为<code>xlsx</code>的excel文件，然后通过ajax上传文件，成功后重置隐藏域文件控件，并提示导入成功

<!-- more -->

```html
<input type="file" id="copyrightFile" accept=".xlsx" style="display:none" onchange="fileChange()">
<button class="btn btn-success" type="button" name="import" title="导入" onclick="$('#copyrightFile').click();">
	<i class="glyphicon glyphicon-import"></i> 导入
</button>
```

```
function fileChange (){
	var fileObj = $("#copyrightFile")[0].files[0];// 获取文件对象
    var form = new FormData();
    form.append("file", fileObj);
	$.ajax({  
	     url : "${context }/copyright/c/import",  
	     type : "POST",  
	     contentType: 'multipart/form-data',
         data: form,
         async: true,
         cache: false,
         processData: false, //告诉jQuery不要去设置Content-Type请求头
         contentType: false, //告诉jQuery不要去处理发送的数据
	     success : function(data) {
	    	$('#copyrightFile').replaceWith("<input type='file' id='copyrightFile' accept='.xlsx' style='display:none' onchange='fileChange()'>");	    	 
	    	if(data.resultCode == 2){
		    	alert(data.resultDesc); 
	    	}else if(data.resultCode == 0){
		    	alert(data.resultDesc, refreshTable); 
	    	}else{
	    		alert(data.resultDesc); 
	    	}
	     } 
	});
}
```

```
@RequestMapping(value = { "/c/importShowidFile" })
@ResponseBody
public Object importShowidFile (@RequestParam("file") MultipartFile file){
	return ccService.importShowidFile(file);
}
```

### 下载文件

>1. <code>{filename:.+}</code>避免<code>@PathVariable</code>中丢失后缀名
2. filename使用<code>UTF-8</code>编码可以避免中文文件名乱码问题
3. 如果不在response中指定<code>content-length</code>字段，下载的excel在office中打开时会有错误提示

![](https://ws1.sinaimg.cn/large/6b162853ly1fwclscfkm3j20ki03m3yi.jpg)

``` java
	@RequestMapping(value = { "/c/down/{filename:.+}" }, method = {
			RequestMethod.GET, RequestMethod.POST })
	@ResponseBody
	public void downYouKuFile (HttpServletResponse res, @PathVariable String filename) throws IOException{
		File file = new File(uploadPath+File.separator+"youku"+File.separator+filename);
		res.setContentType("application/octet-stream");
		res.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(filename, "UTF-8"));
		res.setContentLengthLong(file.length());
		byte[] buff = new byte[1024];
	    BufferedInputStream bis = null;
	    OutputStream os = null;
	    try {
	      os = res.getOutputStream();
	      bis = new BufferedInputStream(new FileInputStream(file));
	      int i = bis.read(buff);
	      while (i != -1) {
	        os.write(buff, 0, buff.length);
	        os.flush();
	        i = bis.read(buff);
	      }
	    } catch (IOException e) {
	      logger.error(e.getMessage(), e);
	    } finally {
	      if (bis != null) {
	        try {
	          bis.close();
	        } catch (IOException e) {
	          logger.error(e.getMessage(), e);
	        }
	      }
	    }
	}
```

### 预览文件内容（在浏览器中打开）

>1. 配合<code>fileinput</code>实现pdf、txt、doc等预览
2. 获取<code>content-type</code>在1.7后添加了方法<code>Files.probeContentType(path)</code>

![](https://ws1.sinaimg.cn/large/6b162853ly1fwcmkdbmodj20mp08xt8t.jpg)

```
function editFileInput(fileName) {
	var config={caption: fileName,url:'${basePath}/lawsuit/remove'};
	var postf = fileName.substring(fileName.lastIndexOf("."),fileName.length);
	var url='${basePath}/lawsuit/download/'+fileName;
	var previewUrl='${basePath}/lawsuit/preview/'+fileName;
	if(postf==".pdf"){
		config['type'] = 'pdf';
	}else if(postf==".png"||postf==".jpg"||postf==".bmp"||postf==".jpeg"||postf==".gif"){
		config['type'] = "image";
	}else if(postf==".xls"||postf==".xlsx"||postf==".doc"||postf==".docx"||postf==".ppt"||postf==".pptx"){
		config['type'] = "office";
	}else{
		config['type'] = "text";
		var htmlobj = $.ajax({ url: previewUrl, async: false });
		previewUrl =htmlobj.responseText; 
	}
	$('#file-enclosure').fileinput('destroy').fileinput({
		language: 'zh', //设置语言
        showUpload: false, //是否显示上传按钮
        showDownload: true,
        browseClass: "btn btn-primary", //按钮样式             
        previewFileIcon: "<i class='glyphicon glyphicon-king'></i>",
        maxFileCount: 1,
        initialPreviewAsData: true,
		pdfRendererUrl: 'http://plugins.krajee.com/pdfjs/web/viewer.html',
		initialPreviewDownloadUrl: url,
		initialPreview : [ previewUrl ],
		initialPreviewConfig : [config],
        purifyHtml: true
	});
}
```

``` java
@RequestMapping(value="/preview/{fileName:.+}", method=RequestMethod.GET)
    public void preview(@PathVariable("fileName") String fileName, HttpServletResponse response) throws UnsupportedEncodingException{
    	byte[] buff = new byte[1024];
    	BufferedInputStream bis = null;
    	OutputStream os = null;
    	try {
    		 Path path = Paths.get(uploadFolder+File.separator
     				+"lawsuit"+File.separator+ fileName); 
    		response.setHeader("content-type", Files.probeContentType(path));
    		os = response.getOutputStream();
    		bis = new BufferedInputStream(new FileInputStream(new File(uploadFolder+File.separator
    				+"lawsuit"+File.separator+ fileName)));
    		int i = bis.read(buff);
    		while (i != -1) {
    			os.write(buff, 0, buff.length);
    			os.flush();
    			i = bis.read(buff);
    		}
    	} catch (Exception e) {
    		log.error(e.getMessage(), e);
    	} finally {
    		if (bis != null) {
    			try {
    				bis.close();
    			} catch (IOException e) {
    				log.error(e.getMessage(), e);
    			}
    		}
    	}
    }
```
