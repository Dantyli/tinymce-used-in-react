[TOC]

##### 背景

最近在做一个新闻媒体类APP的管理后台，后台用户可以编辑创建新闻内容，需要引入一个富文本的编辑器

##### 需求分析

编辑器需要满足以下几点基本要求：

1. 可以设置文字的样式，如：颜色，大小，字体等
2. 能插入图片，图片上传到自己的服务器
3. 过滤掉从其他地方复制过来的样式
4. 可以缩进等

[选型参考文章](https://blog.csdn.net/davidhzq/article/details/100842866)

##### TinyMCE使用

###### 1.文档配置

中文版地址：http://tinymce.ax-z.cn/

此中文版译制的还是很详细清楚的，可以放心使用

###### 2.配置注意项

```javascript
import { Editor } from '@tinymce/tinymce-react';
<Editor
      initialValue={this.props.data}
      init={{
            height: 500,
            relative_urls : false, //返回图片链接为完整路径时配置
            remove_script_host : false,
            language:'zh_CN',//中文
            branding: false,
            plugins: [ //启用的插件
            'advlist autolink lists link image charmap preview anchor',
            'searchreplace visualblocks code fullscreen',
            'insertdatetime  table paste code help wordcount'
            ],
              toolbar:[
                'undo redo | cut copy paste pastetext | bold italic forecolor backcolor underline          strikethrough link | \
alignleft aligncenter alignright alignjustify outdent indent | \
bullist numlist  ',
                'formatselect fontselect fontsizeselect | blockquote subscript superscript removeformat | image preview fullscreen',
              ],
            images_upload_url:`${baseURL}/upload`,
            images_upload_credentials:true,
            fontsize_formats: '11px 12px 14px 16px 18px 24px 36px 48px',
            font_formats: '宋体=simsun,serif;微软雅黑=Microsoft YaHei,Helvetica Neue,sans-serif;'
        }}
            onEditorChange={this.handleEditorChange}
/>
```

###### 3.搭配React

[官方文档地址](https://www.tiny.cloud/docs/integrations/react/)

3.1 安装tinymce-react

```
$ npm install --save @tinymce/tinymce-react
```

3.2 独立部署（self-hosted）

独立部署不需要申请apikey，且比cdn加载更快(我个人项目是这样的。。)

下载 `TinyMCE` 代码包，解压到根目录

```
/tinymce
  -- langs //语言包
  -- plugins //插件
  -- skins. //皮肤
  -- tinymce.min.js //页面中需要引入的文件
```

根节点所在 `index.html `  中引入

<script src="/tinymce/tinymce.min.js"></script>

##### 附：nodejs 接收图片示例

```javascript
const {Controller}=require('egg')
const fs=require('fs')
const path=require('path')
const pump=require('pump')
const mkdirp=require('mkdirp')
class Tools extends Controller{
    async upload(){
        const {ctx}=this
        const parts=ctx.multipart()
        let stream,filename;
        while((stream=await parts())!=null){
            if(!stream.filename){
                break
            }
            const dir=`./upload` //示例地址
            await mkdirp(dir)
             filename=`${dir}/tinymce-upload${path.extname(stream.filename)}`
            const ws=fs.createWriteStream(filename)
            await pump(stream, ws);
        }
       ctx.body={
            code:'200',
            msg:'success',
            location:'http://dantyli.com/upload/tinymce-upload'+path.extname(filename)
        }
    }
}
module.exports=Tools;
```



###### 