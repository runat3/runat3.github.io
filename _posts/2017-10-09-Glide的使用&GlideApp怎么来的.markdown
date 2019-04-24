---
layout:     post
title:      "Glide的使用&GlideApp怎么来的"
subtitle:   " \"简单易上手的强大图片框架\""
date:       2017-09-10 18:21:00
author:     "Jason Chen"
header-img: "img/post-bg-2015.jpg"
tags:
    - 开发
    - 图像
---

### 1.Glide的优势和劣势or为什么选Glide

###### Glide优势

- Glide.with()能传入activity，fragment等，可以和页面的生命周期绑定，不至于，页面停止了，图片还在加载

- Glide支持多种数据源 本地 网络 uri assets

- Glide加载的图片默认格式是RGB565，相比ARGB8888内存占用减少一半，但图片质量低了

- Glide不用担心大图oom，它不会加载原图，会根据imageview的大小而加载多大

- Glide缓存imageview大小的图片， picasso缓存原始图片，当imageview大小调整了，Glide需要重新下载，再缓存，picasso不用重新下载，只需重新绘制，一定程度上Glide加载比picasso快（时间换空间）

- Glide可以加载动态图gif，可以播放本地视频！！！！！！
  ######Glide劣势

- Glide的包和方法数多
  ######其他框架劣势

- Fresco框架不好用布局文件要用drawees，配置麻烦，学习成本相对高一些，体积很大

- Picasso.with()只能传入context

- UIL没有更新了

  ### 2.Glide 4.0+的使用

  ###### 1)在module的build.gradle中添加依赖

```groovy
repositories {
  mavenCentral()
  maven { url 'https://maven.google.com' }
}

dependencies {
  compile 'com.github.bumptech.glide:glide:4.2.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.2.0'
}
```

![](http://upload-images.jianshu.io/upload_images/7793862-ddc3a7b9f26fcdb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 2)使用GlideApp

在Glide4.0之前使用Glide只需要

```java
 Glide.with(this)
         .load(url)
         .placeholder(R.mipmap.ic_launcher)
         .into(iv);
```

但是在Glide4.0之后load(url)之后就不能调用.placeholder()等方法
查看4.0+的文档发现需要通过GlideApp来调用一系列方法

> GlideApp生成的方法:
> ①新建一个类继承AppGlideModule
> ②类添加GlideModule
> ③Make Module

```java
import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public final class MyAppGlideModule extends AppGlideModule
{
      //可以配置Glide
}
```

现在就可以使用GlideApp了

```java
GlideApp.with(this)
                .load(url)//资源路径
                .placeholder(R.mipmap.ic_launcher)//占位图
                .error(R.mipmap.ic_launcher)//加载失败
                .into(iv);
```

*注意：需要添加网络权限！*

### 3.Glide 4.0+图片转换

在开发中，图片经常要做一些变化，比如剪裁指定形状，圆角（可以使用CircleImageView），这边推荐一个Glide的图片转换库
https://github.com/wasabeef/glide-transformations
实现圆角效果如下：
![](http://upload-images.jianshu.io/upload_images/7793862-dbe888ed1dda3d9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还有一些变化，可以试试

#### 小坑

添加了占位图.placeholder(R.mipmap.ic_launcher)
如果ImageView设置了warp_content占位图会影响加载图的大小
所以，ImageView最好设置固定大小