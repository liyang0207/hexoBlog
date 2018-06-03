---
title: ckeditor编辑器视频插入及右键事件处理
date: 2017-08-27 23:43:51
tags:
 - JavaScript
 - 工作积累
---
#### 开场扯淡
上次聊了聊ckeditor编辑器实例化及自定义图片上传功能的事儿，今天来说一些杂七杂八的优化及自定义视频插入的小功能。

#### 右键菜单事件
其实ckeditor第四版开始有了右键菜单事件，即在编辑器内右键图片，文本等，会弹出简单的编辑菜单，如复制，粘贴，剪切，编辑图片等等。初衷其实是好的，可是我就是喜欢简洁一些，傻瓜式操作。不过图片的右键编辑功能其实可以保留，方便直接右键编辑现有图片。记得上一篇中的注册右键事件吗？
```javascript
// 注册右键菜单
if ( editor.contextMenu ) {
    editor.addMenuGroup( 'oss');
    editor.addMenuItem( 'image', {
        label: '修改图片',   //右键菜单名称
        command: 'ossupload',   //执行的命令
        group: 'oss'
    } );
}
```
![编辑器4.png](http://upload-images.jianshu.io/upload_images/6589697-0e1ddda60d2884e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

右键之后直接打开弹窗进行编辑即可。为了隐藏或者说取消ckeditor编辑器的默认右键事件，我们在config.js内这么写了一句：
```javascript
CKEDITOR.config.menu_groups = ''; //编辑器内的右键菜单设为空。搜索关键字menuitem，查看源码
```
就是直接将右键菜单事件清空，使用我们自定义的右键菜单事件。
参看源码：[menuitem](https://docs.ckeditor.com/source/plugin63.html#CKEDITOR-menuItem)

#### 自定义iframe插入视频
插入第三方的视频链接也是编辑器常用的一个功能。可惜一个挺简单的功能又让官方搞的很复杂，让人看着就不想使用。同样是发挥自定义精神，我自己做了一个iframe的插入功能。
* 组件的文件结构同上一篇的oss上传，简单起见就不传图了，代码模拟下

```javascript
myiframe
...dialogs
......myiframe.js
...images
......xxx.jpg
......xxx.png
...plugin.js
```

先看最终效果：
![编辑器5.png](http://upload-images.jianshu.io/upload_images/6589697-e5689bd8b57e5c91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 直接插入视频网站的链接，可以自定义视频宽高百分比，同样基于rem的等比例。先定义plugin.js

```javascript
( function() {
    CKEDITOR.plugins.add( 'myiframe', {
        init: function( editor ) {
            editor.addCommand('iframeup', new CKEDITOR.dialogCommand('iframeup')); 
            editor.ui.addButton('myiframe',
            {
                label: '插入视频',
                icon: 'plugins/myiframe/images/myiframe.png',
                command: 'iframeup'
            });
            CKEDITOR.dialog.add( 'iframeup', this.path + 'dialogs/myiframe.js' );
        },
        afterInit: function( editor ) {}
    } );
} )();
```

* 接着是弹窗dialogs/myiframe.js

```javascript
CKEDITOR.dialog.add( 'iframeup', function( editor ) {
    // 视频提示文字
    var remindHtml = new CKEDITOR.template(
            '<p>注：视频的高度和宽度为选填，默认宽度100%，高度是屏幕宽度的80%，</p>'+
            '<p>一般不会出现视频显示不完整的情况，请依据实际情况调整。</p>'
            ).output();
    return {
        title: '插入视频',
        width: 360,
        minHeight: 200,
        contents: [
            {
                id: 'iframe',
                label: '插入视频',
                elements: [
                    {   
                        id: 'source',
                        label: '请输入视频的URL',
                        type: 'text',
                        validate: function() {  //简单的链接验证
                            if ( this.getValue() != '' && this.getValue().indexOf('http') == -1 ) {
                                alert( 'url链接错误，请重新输入' );
                                return false;
                            }
                        }
                    },
                    {
                        type: 'hbox',   //系统自定义水平box
                        widths: [ '50%', '50%'],  //分左右两部分，各50%宽
                        children: [
                            {
                                type: 'text',   //系统自定义input，type=text
                                width: '80%',   //左宽度的80%，即宽度的50%*80%
                                id: 'width',   //取值用到的id
                                label: '宽度',
                                'default': '100%',   //默认input的value值
                                validate: function() {  //验证只能填入百分数
                                    var reg = /^[0-9]+%$/;
                                    if (!reg.test(this.getValue())) {
                                        alert( '请输入正确的百分数宽度' );
                                        return false;
                                    }
                                }
                            },
                            {
                                type: 'text',   //同上
                                id: 'height',
                                width: '80%',
                                label: '高度',
                                'default': '80%',
                                validate: function() {
                                    var reg = /^[0-9]+%$/;
                                    if (!reg.test(this.getValue())) {
                                        alert( '请输入正确的百分数高度' );
                                        return false;
                                    }
                                }
                            }
                        ]
                    },
                    {
                        type: 'html',
                        html: remindHtml    //自定义说明文档
                    }
                ]
            }
            ],
        onOk: function () {   //确定时传值
            var isrc = this.getValueOf('iframe', 'source'),
                width = this.getValueOf('iframe', 'width'),
                height = this.getValueOf('iframe', 'height');
            var rheight = height.replace(/%/,'') * 0.01 * 7.5 + 'rem';
            console.log(rheight);
            if(isrc != '' && isrc.indexOf('http') > -1){
                var ele = CKEDITOR.dom.element.createFromHtml( '<p style="padding:5px 0;"><iframe class="my-iframe" src="'+isrc+'" style="border:none;width:'+width+';height:'+rheight+';"></iframe></p>' );
                editor.insertElement(ele);
                this.commitContent(editor);
            }
        }
   };
} );
```

* iframe默认是有右键菜单的，这里就暂时不注册右键事件了，以后可以优化下，如视频预览图，右键之类的功能。

#### 另一个坑
另外的一个坑是同事玩我编辑器的时候发现的：双击图片会弹出ckeditor默认的图片弹窗。而这一直是我竭力规避的情况。查了下文档，关键字doubleclick，发现这块还是写的很模糊的，搞了一圈也没发现配置选项在哪，最后没办法，只能搜源码暴力解决了。
方法：在ckeditor.js内搜索'doubleclick'，可以看到它注册了很多双击事件，找到控制image的地方，直接将命令为image的代码删除，同时顺手将默认的iframe双击事件也删除，保存即可。这样双击事件就不会再触发了。方法虽然很粗暴，但是很管用。

OK，基本上到现在，我们的编辑器'看起来'还是很舒服的，基本功能都齐全，又是所见即所得的，我暂时很满意，不过不排除以后有什么新的功能需求，到时候再进行开发即可。用了两篇blog来简单说了下ckeditor的使用方法和自己的小优化，等闲下来会将自己的两个组件传到github上，方便以后查询。

下一篇估计会探讨一下plupload的用法，这个上传插件用起来还是很爽的，功能也很强大。就这样...





