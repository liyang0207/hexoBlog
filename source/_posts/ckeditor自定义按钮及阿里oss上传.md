---
title: ckeditor自定义按钮及阿里oss上传
date: 2017-08-22 22:08:22
tags:
 - 工作积累
 - JavaScript
---
#### 啰嗦需求
公司的百度编辑器已经千疮百孔了，各种bug层出不穷，因为公司图片这块的处理全是oss直传阿里的，上个前端也是费了巨力重新编辑修改了百度编辑器的图片上传功能，来贴合公司的实际需求。但是我觉得编辑器这块最重要是需要贴合用户的实际使用体验，编辑出来的页面是要在移动端展示的，最好能实现真正的“所见即所得”，傻瓜式的编辑体验，简单粗暴，够用就好，花里胡哨的功能我们也暂时用不到。参考了微信公众号编辑界面，知乎文章编辑界面，发现都是“简单”的界面实现了很“实用”的功能。
接下来就是现有编辑器的选择了，毕竟自己现写个编辑器根本不现实（听说知乎的编辑器是30人的团队维护的...）。查阅了几家编辑器：wangEditor、bootstrap-wysiwyg、tinymce...感觉都对不上，不是文档太少，就是感觉“弱不经风”，官网的编辑器随手试试都能搞一两个bug出来，优化体验太差。应了知乎大神那句“编辑器是个坑，你们千万别往里跳”，一个好的编辑器真是需要精力去维护和更新的。
最后选择了比较稳重的ckeditor，文档很全，支持插件，配置什么的也很友好，先给个文档链接感受一下：ckeditor在线文档https://docs.ckeditor.com/#!/guide/dev_installation。

#### 安装配置
官网下载文件，下载界面可选：Basic Package、Standard Package、Full Package、Customize。根据自己需求下载不同的安装包，而且每种都有压缩版和源码版可选。其中Customize版本顾名思义可自定义选择自己需要的模块，官方也推荐使用这种方式自定义下载，免得像我一样傻呵呵的自己下载需要的插件，结果安装依赖包安装到怀疑人生。
接下来很简单，傻瓜式安装：
* html页面引入ckeditor.js文件，也可以使用官方CDN
```javascript
<script type="text/javascript" src="ckeditor/ckeditor.js />
//<script src="https://cdn.ckeditor.com/4.7.2/standard/ckeditor.js"></script>  //官方CDN
```
* 替换textarea
```javascript
<textarea name="editor1" id="editor1" rows="10" cols="80">This is my textarea to be replaced with CKEditor.</textarea>
```
* 实例化ckeditor
```javascript
<script>
    // Replace the <textarea id="editor1"> with a CKEditor
    // instance, using default configuration.
    CKEDITOR.replace( 'editor1' );
</script>
```
这样，一个基本可以使用的编辑器就搞好了，大概如下图：
![截图1.png](http://upload-images.jianshu.io/upload_images/6589697-ecd5ffcf1382bca0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 个性化配置
* ckeditor的个性化配置功能很人性化，打开下载好的ckeditor文件夹，ckeditor/samples/index.html打开，点击右上TOOLBAR CONFIGURATOR，切换到配置功能页，勾选自己需要的功能块儿，也可以调整显示顺序，上方是预览。
![截图2.png](http://upload-images.jianshu.io/upload_images/6589697-5a53946920c5043e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 点击右边的`Get toolbar config`按钮，切换至我们配置好的config源码部分，复制该配置代码至ckeditor/config.js，直接覆盖就好。基本上所做的个性化配置都在这个文件里定义（还有另外两种配置方式，具体可以查看文档）。
```javascript
CKEDITOR.editorConfig = function( config ) {
    config.toolbarGroups = [
        { name: 'document', groups: [ 'mode', 'document', 'doctools' ] },
        { name: 'clipboard', groups: [ 'clipboard', 'undo' ] },
        { name: 'editing', groups: [ 'find', 'selection', 'spellchecker', 'editing' ] },
        { name: 'forms', groups: [ 'forms' ] },
        { name: 'styles', groups: [ 'styles' ] },
        { name: 'basicstyles', groups: [ 'basicstyles', 'cleanup' ] },
        { name: 'colors', groups: [ 'colors' ] },
        { name: 'paragraph', groups: [ 'list', 'indent', 'blocks', 'align', 'bidi', 'paragraph' ] },
        { name: 'links', groups: [ 'links' ] },
        { name: 'insert', groups: [ 'insert' ] },
        { name: 'tools', groups: [ 'tools' ] },
        { name: 'others', groups: [ 'others' ] },
        { name: 'about', groups: [ 'about' ] }
    ];
    config.removeButtons = 'Image,Source,Save,NewPage,Print,Templates,PasteFromWord,PasteText,Paste,Copy,Cut,Find,Replace,SelectAll,Scayt,Form,Subscript,Superscript,CopyFormatting,Checkbox,Radio,TextField,Textarea,Select,Button,ImageButton,HiddenField,Outdent,Indent,CreateDiv,BidiLtr,BidiRtl,Language,Anchor,Table,HorizontalRule,Smiley,SpecialChar,PageBreak,ShowBlocks,Blockquote,Styles,Font,Preview,Flash,Link,Unlink,Maximize,Format,About,Iframe';
};
```
* 刷新页面，就可以看到配置完的个性化编辑器了。

#### 自定义按钮（功能）
毕竟每个人的需求都是不一样的，这时候一个编辑器的“可扩展性”就必不可少了。上面说道，图片功能需要结合阿里的oss上传，policy,signature等等的验证，同时需要能有微信公众号编辑界面一样，实现图片的100%显示和原尺寸显示的自由切换。
* 在ckeditor/plugins文件夹下，新建一个名为oss的文件夹：内含dialogs文件夹，存放我们的弹窗内容；images文件夹，用来存放用到的小图标；plugin.js文件，定义oss插件的入口。方便起见我就用代码模拟一下，不传图了。
```javascript
oss
...dialogs
......oss.js
...images
......xxx.jpg
......xxx.png
...plugin.js
```
* 配置oss/plugin.js文件
```javascript

( function() {
    CKEDITOR.plugins.add( 'oss', {
        onLoad: function() {},
        init: function( editor ) {   //上传图片插件初始化
            editor.addCommand('ossupload', new CKEDITOR.dialogCommand('ossupload'));   //基于dialog弹窗插件
            editor.ui.addButton('oss',
            {
                label: '插入图片',   //鼠标hover到按钮上显示文字
                icon: 'plugins/oss/images/oss.png',   //自定义按钮的图标
                command: 'ossupload'  //点击按钮执行的命令
            });
            // 注册右键菜单
            if ( editor.contextMenu ) {
                editor.addMenuGroup( 'oss');
                editor.addMenuItem( 'image', {
                    label: '修改图片',
                    command: 'ossupload',
                    group: 'oss'
                } );
            }
            CKEDITOR.dialog.add( 'ossupload', this.path + 'dialogs/oss.js' );  //弹窗文件，指向dialogs文件夹下的oss.js
        },
        afterInit: function() {}
    } );
} )();

```
接下来定义弹窗的内容，即dialogs内的oss.js文件，先看下实际效果
![编辑器1.png](http://upload-images.jianshu.io/upload_images/6589697-98457a20dd53ea72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```javascript
CKEDITOR.dialog.add( 'ossupload', function( editor ) {
    // 图片尺寸html模板
    var sizeHtml = new CKEDITOR.template(
            '<div class="myradio">' +
                '<div class="item" style="padding-bottom:5px;">'+
                    '<input type="radio" name="size" value="100%" id="100" checked style="vertical-align:middle;margin:0 5px 0 0;">' +
                    '<label for="100">图片满屏显示</label>'+
                '</div>'+
                '<div class="item">'+
                    '<input type="radio" name="size" value="auto" id="auto" style="vertical-align:middle;margin:0 5px 0 0;">' +
                    '<label for="auto">图片原尺寸显示</label>'+
                '</div>'+
            '</div>').output();
    var currentSrc = null;  //编辑时图片源地址
    // 阿里oss图片上传
    function plupload1() {
        //阿里oss上传组件，使用了官网的例子，基于plupload组件
    };
    return {
        title: '插入图片',
        width: 302,   //弹窗的宽px
        minHeight: 302,   //弹窗的最小高度px,上传图片高度大于300自动撑开
        contents: [
            {
                id: 'ossimage',
                label: '上传图片',
                elements: [    //自定义弹窗的内容，可以使用模板，也可自定义html及样式
                    {
                        id: 'myimage',    //选择图片按钮
                        type: 'html',
                        html: '<a href="javascript:;" id="selectfile">选择图片</a>',  //plupload按钮
                        style: 'display:block;width:82px;line-height:34px;background-color:#3366b7;font-size:14px;color:#fff;text-align:center;border-radius:4px;',  //html的样式，直接作用于上面的a元素
                        onShow: function() {  //当该元素show的时候执行的方法
                            if(currentSrc && currentSrc !== 'undefined'){   //当右键编辑图片时执行，下篇说
                                $('#selectfile').hide();
                                $('.moxie-shim').hide();
                            }
                        },
                        onLoad: function() {
                            plupload1();  //加载时实例化plupload上传组件
                        }
                    },
                    {
                        type: 'html',    //图片上传成功后的容器
                        html: '<div class="container"><div class="imgbox"></div><a style="display:none;position:absolute;top:0;left:0;z-index:1;width:16px;height:16px;background-image:url(/assets/js/ckeditor/skins/moono-lisa/images/close2.png)"></a></div>',
                        style: 'width:300px;height:auto;position:relative;',  //对应的样式
                        onShow: function() {
                            if(currentSrc && currentSrc !== 'undefined'){
                                //右键编辑图片时，将图片src传入容器，同时注册关闭图片事件
                            }
                        },
                        commit: function( editor ){  //点击确定按钮时，将图片src传入全局src中
                            src = $('.imgbox img').attr('src');
                        }
                    },
                    {
                        id: 'size',   //图片全屏显示及按实际尺寸显示radio
                        type: 'html',
                        html: sizeHtml,   //html写到了上面
                        commit: function( editor ){
                            var tt = document.getElementsByName('size'); //取radio选项
                            for (var i = 0; i < tt.length ; i++ ){    
                                if(tt[i].checked){
                                    imgsize = tt[i].value;    
                                    break;
                                }
                            }
                            if(src != undefined){
                                //创建element,即确定后填到编辑器中的代码，此处最好使用p标签包裹img标签，ckeditor都是使用p标签包裹元素，保持一致
                                var ele = CKEDITOR.dom.element.createFromHtml( '<p style="padding:5px 0;"><img style="max-width: 100%;width:'+imgsize+'" src="'+src+'"/></p>' );  
                                editor.insertElement(ele);   //将element插入editor
                            }
                        }
                    }
                ]
            }
            ],
        onShow: function() {
            currentSrc = null;   //弹窗显示的时候判断‘选中元素’的currentSrc是否存在
            var element = editor.getSelection().getSelectedElement();
            if(element){
                console.log(element);
                console.log(element.$.currentSrc);
                currentSrc = element.$.currentSrc;//源图片地址
            }
        },
        onOk: function () {
            this.commitContent(editor);
            //点击确定时dom操作
            currentSrc = null; 
        },
        onCancel: function () {
            //取消时dom操作
            currentSrc = null;  //取消或者确定时清空currentSrc
        }
   };
} );
```
看下上传图片后的实际效果
![编辑器2.png](http://upload-images.jianshu.io/upload_images/6589697-e45566e99a1382e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![编辑器3.png](http://upload-images.jianshu.io/upload_images/6589697-3f9c2d2dd0cb97df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图片满屏显示会将图片宽度设为100%，高度自动；图片原尺寸显示会显示图片原本的尺寸。这里注意所谓图片原尺寸是指图片的实际大小，假若图片宽度200px,那么图片不管在什么设备下，宽度始终为200px，不过若是图片原尺寸超过了设备的实际宽度，那么图片的最大宽度就是设备的宽度，不会出现左右滚动条。

最后记得将我们自定义的组件注册到config.js中
```javascript
config.extraPlugins = 'myiframe,oss,dialog,dialogui,lineheight,richcombo,floatpanel,panel,listblock';  //除了oss,myiframe,lineheight外，其他都为依赖组件
```

最后是编辑器的一些其他优化，如使用rem使编辑器宽度，字体大小等比例显示，行高组件的添加等，都是为了真正的实现所见即所得的个性化移动编辑器，真正做到了pc端编辑出来的效果就是在移动展示的实际比例效果。
```javascript
config.fontSize_defaultLabel = '12';
config.fontSize_sizes='12/0.24rem;14/0.28rem;16/0.32rem;18/0.36rem;20/0.4rem;22/0.44rem;24/0.48rem;26/0.52rem;28/0.56rem;36/0.72rem;48/0.96rem;72/1.44rem';
CKEDITOR.config.menu_groups = ''; //编辑器内的右键菜单设为空。搜索关键字menuitem，查看源码
config.line_height = '1;1.2;1.5;2;2.2;2.5;3;3.5;4;5';  //定义行高
```
