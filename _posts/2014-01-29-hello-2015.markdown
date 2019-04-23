---
layout:     post
title:      "Text4Develop"
subtitle:   " \"A TextView For Android Develop\""
date:       2018-04-19 17:56:00
author:     "jason"
header-img: "img/post-bg-2015.jpg"
tags:
    - 开发
---

#### 场景

在公司做开发的时候，手头上有几台测试机 ，这时候测试随便拿了一台去测，点来点去crash了。然后跑来怼~。其实测试拿过去的手机跑的代码不是最新的，但是最最新代码无法重现bug，你也没改过对应的代码。这时候你不知道你不知道出bug的版本是哪个，无法git回到对应的版本查看bug。也或者是由于Android版本差异性导致的，测试不能马上告知你手机Android版本。

#### 解决方法

在app种显示一个全局的视图，把apk最后更新的时间，版本等等信息展示出来，这样可以初步定位到问题。

#### 效果图

![demo](https://upload-images.jianshu.io/upload_images/7793862-d8b84bf32c786801.gif?imageMogr2/auto-orient/strip)

#### 实现方式

这个其实不难，主要的关键词是“全局”，这里用到了WindowManager。
自定义View继承Textview，然后将view通过添加到WindowManager中
关键代码如下

```
    /**
     * 显示app信息文字
     * show info text
     * @param context 上下文
     */
    private void showInfoText(Context context)
    {
        windowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        // 布局设置
        layoutParams = new WindowManager.LayoutParams();
        // 设置window type
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {//兼容8.0
            layoutParams.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
        } else {
            layoutParams.type = WindowManager.LayoutParams.TYPE_SYSTEM_ALERT;
        }
        // 设置背景透明
        layoutParams.format = PixelFormat.RGBA_8888;
        // 设置显示的位置
        layoutParams.gravity = Gravity.RIGHT | Gravity.TOP;
        // 设置Window flag 不可触摸无焦点
        layoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;
        // 设置视图大小
        layoutParams.width = WindowManager.LayoutParams.WRAP_CONTENT;
        layoutParams.height = WindowManager.LayoutParams.WRAP_CONTENT;
        // 添加视图
        windowManager.addView(this, layoutParams);
    }
```

显示的文字

```
/**
     * 获取app信息
     * get info
     * @param context
     * @return
     */
    public String getDevInfo(Context context)
    {
        StringBuilder info = new StringBuilder();
        PackageManager packageManager = context.getPackageManager();
        try
        {
            PackageInfo packageInfo = packageManager.getPackageInfo(context.getPackageName(), 0);
            // app最后更新时间
            long lastUpdateTime = packageInfo.lastUpdateTime;
            CharSequence relativeTimeSpanString = DateUtils.getRelativeTimeSpanString(lastUpdateTime);
            info.append("LastUpdate：").append(relativeTimeSpanString).append("\n");
            // app版本
            String versionName = packageInfo.versionName;
            info.append("Version：").append(versionName).append("\n");
            // api的host
            info.append("Host：").append(Constants.HOST).append("\n");
            // 测试机android版本
            info.append("Android：").append(Build.VERSION.RELEASE).append("\n");
        } catch (PackageManager.NameNotFoundException e)
        {
            e.printStackTrace();
        }
        return info.toString();
    }
```

- LastUpdate：最后更新时间，可以知道代码是否是最新的

- Version：版本号，可以知道bug在哪个版本产生的

- Android：测试机Android版本，可以知道bug是否由版本差异性而产生（例如:6.0动态权限）

- Host：接口的baseUrl，用于分辨是测试环境还是线上环境
  其他的信息可以根据需求添加删除

#### 用法

1. 拷贝TextView4Dev到你的项目中
2. 在MainActivity中实例化`TextView4Dev textView4Dev = new TextView4Dev(this);`
3. 添加权限`<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>`

注意事项：需要获取权限ACTION_MANAGE_OVERLAY_PERMISSION

完整代码请查看demo->[GitHub](https://github.com/CzSam/TextView4Dev/tree/master)，如果对您有用的话，请star一下，感激不尽~



