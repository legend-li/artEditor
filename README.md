# artEditor
artEditor是一款基于jQuery的移动端富文本编辑器，支持插入图片，后续完善其他功能。新增图片上传之前压缩功能
[demo](http://baixuexiyang.github.io/artEditor/)，为了更好的效果请将浏览器设置为手机模式。   
使用交流：[交流](https://gitter.im/artEditor/Lobby)    
   
新增开发[web editor](./web/README.md)      
    
# 引用
在页面中引入下面资源 
```   
<script src="http://code.jquery.com/jquery-1.11.0.min.js"></script>  
<script src="artEditor.min.js"></script>   
```
# CDN    
artEditor cdn 加载方式：    
```   
<script src="https://unpkg.com/arteditor"></script>  
```   
# Options
### imgTar
  图片上传按钮
### limitSize
  图片最大限制，默认3兆
### showServer
  显示从服务端返回的图片，默认是显示本地资源的图片
### uploadUrl
  图片上传路劲
### data
  上传图片其他参数，例如：{type: 1}  
### uploadField
  上传图片字段, 默认值：uploadfile
### placeholader
  富文本编辑器holder
### validHtml
  粘贴时，去除不合法的html标签
### uploadSuccess
  图片上传成功回调
### uploadError
  图片上传失败回调
### formInputId
  表单隐藏域id，如果设置，则编辑器内容会储存到该元素value值
### compressSize
  图片超过大小会被压缩，单位（兆）
### breaks  
  换行符，非Firefox默认为div，如果该值为true，则换行符替换为br   
### beforeUpload   
  图片上传之前的回调（function）,参数图片base64数据

# Methods

### getValue
    获取值，$('#content').getValue()
### setValue
    设置值，$('#content').setValue('<div></div>')


# Example
html:
```
<div class="article-content" id="content">
</div>
```
js:

```
$('#content').artEditor({
	imgTar: '#imageUpload',
	limitSize: 5,   // 兆
	showServer: false,
	uploadUrl: '',
	data: {},
	uploadField: 'image',
	placeholader: '<p>请输入文章正文内容</p>',
	validHtml: ["br"],
  beforeUpload: function(imgBase64) {
    // 处理完之后，必须return图片数据   
    // return imgBase64         
  },
	uploadSuccess: function(res) {
            // 这里是处理返回数据业务逻辑的地方
            // `res`为服务器返回`status==200`的`response`
            // 如果这里`return <path>`将会以`<img src='path'>`的形式插入到页面
            // 如果发现`res`不符合业务逻辑
            // 比如后台告诉你这张图片不对劲
            // 麻烦返回 `false`
            // 当然如果`showServer==false`
            // 无所谓咯
		return res.path;
	},
	uploadError: function(res) {
		//这里做上传失败的操作
        //也就是http返回码非200的时候
		console.log(res);
	}
});
```

# Issues
[new Issue](https://github.com/baixuexiyang/artEditor/issues/new)


# Release
 + 增加获取值和设置值的方法    
 + 去掉粘贴时过滤不允许的HTML标签    
 + 插入图片自动换行       
2016-03-28       
 + 增加拍照上传图片    
2016-04-17    
 + 增加表单提交功能    
2017-03-22    
 + 增加图片压缩    
2017-05-24      
 + 解决上传图片光标定位问题    
2017-05-26    
 + 解决Firefox图片压缩的问题    
 + 解决上传字段的问题  
2017-06-02   
 + 解决chrome换行额外增加div   
2017-12-25   
 + 增加beforeUpload回调  
2017-12-25   
 + 增加服务端demo   
      
   
   
### 项目文件说明
|- dist 项目打包结果文件夹
|- example 项目案例文件夹
|- service 项目后台数据文件夹
|- src 项目源文件

### 项目运行方式
1. `npm install`下载用来工具
2. `npm start`页面    
    
    
### 增加服务端demo   
1、npm install    
2、node server/index.js   
访问:   
127.0.0.1:9091    
   
   
   
# 最后

## html5图片压缩

前端在获取到用户上传的图片之后，有时图片过大需要压缩，html5图片压缩主要是通过canvas来处理。但是在ios中存在几个问题：

- 首先是图片的大小，如果图片的大小超过两百万像素，图片是无法绘制到canvas上的，调用drawImage的时候不会报错，但是你用toDataURL获取图片数据的时候获取到的是空的图片数据。

- 再者就是canvas的大小有限制，如果canvas的大小大于大概五百万像素（即宽高乘积）的时候，不仅图片画不出来，其他什么东西也都是画不出来的。

## 解决办法

- 问题1解决办法是将图片分割成多块绘制到canvas上
- 问题2对图片的宽高进行适当压缩，保证处于canvas的最大大小限制以内（大概500万像素以内），更早期的手机甚至限制canvas最大大小为300万像素左右，这个早起手机就不用理会了，毕竟用的人不多。

## 代码

``` js
function compressHandler(img) {   
    var canvas = document.createElement("canvas");  
    var ctx = canvas.getContext('2d');  
    var tCanvas = document.createElement("canvas");  
    var tctx = tCanvas.getContext("2d");  
    var initSize = img.src.length;   
    var width = img.width;  
    var height = img.height;  
    var ratio;   
    if ((ratio = width * height / 4000000) > 1) {  
        ratio = Math.sqrt(ratio);   
        width /= ratio;  
        height /= ratio;  
    } else {  
        ratio = 1;  
    }  
    canvas.width = width;  
    canvas.height = height;  
    ctx.fillStyle = "#fff";  
    ctx.fillRect(0, 0, canvas.width, canvas.height);  
    var count;  
    if ((count = width * height / 1000000) > 1) {  
        count = ~~(Math.sqrt(count) + 1);  
        var nw = ~~(width / count);  
        var nh = ~~(height / count);  
        tCanvas.width = nw;  
        tCanvas.height = nh;  
        for (var i = 0; i < count; i++) {  
            for (var j = 0; j < count; j++) {  
                tctx.drawImage(img, i * nw * ratio, j * nh * ratio, nw * ratio, nh * ratio, 0, 0, nw, nh);  
                ctx.drawImage(tCanvas, i * nw, j * nh, nw, nh);  
            }  
        }  
    } else {  
        ctx.drawImage(img, 0, 0, width, height);  
    }  
    var ndata = canvas.toDataURL('image/jpeg', 0.1);  
    tCanvas.width = tCanvas.height = canvas.width = canvas.height = 0;  
    return ndata;   
}
```
