---
layout:     post
title:      "探究LayoutInflater和RecyclerView中item设置宽高无效"
subtitle:   "\"LayoutInflater解密\""
date:       2017-10-30 17:28:00
author:     "Jason Chen"
header-img: "img/post-bg-2015.jpg"
tags:
    - Android
---

# 1. LayoutInflater是做什么的

```
Instantiates a layout XML file into its corresponding {@link android.view.View}objects. 
实例化一个布局XML文件转换为相应的{ @link android.view。视图对象};
```

在Android开发中LayoutInflater经常要用到，Fragment，adapter中的视图都是需要LayoutInflater将XML文件转换为View对象

# 2. 获取LayoutInflater对象

获取LayoutInflater对象有三种方法

**(1)getLayoutInflater()**
调用Activity的getLayoutInflater()

**(2)LayoutInflater.from(context)**
调用LayoutInflater的静态方法从context中获取LayoutInflater对象

**(3)(LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)**
调用系统服务获取LayoutInflater对象

# 3.将XML文件转换为View对象

最常用的做法

```java
 View view = layoutinflater.inflate(R.layout.XXX,null);
```

> ###### 到此为止，上述的方法基本能实现布局从xml转换为想要的View对象
>
> ###### 但是前几天在做RecyclerView的时候发现一个问题，如下

# 4.RecyclerView中item设置宽高无效

item_recyclerview.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="200dp"
                android:layout_marginLeft="8dp"
                android:layout_marginRight="8dp"
                android:layout_marginTop="5dp"
                android:layout_marginBottom="5dp"
                android:padding="20dp"
                android:background="#44ff0000">

    <TextView
        android:id="@+id/tv_item"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:textColor="#ffffff"
        android:textSize="26sp"/>

</RelativeLayout>
```

recyclerview的adapter中布局转换

```java
    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        View view = lif.inflate(R.layout.item_recyclerview,null);
        MyViewHolder myViewHolder = new MyViewHolder(view);
        return myViewHolder;
    }
```

简单的item布局，实现的效果如下

![image.png](http://upload-images.jianshu.io/upload_images/7793862-ff4e6969faaff6c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
发现明明设置了宽度是match_parent，显示出来的效果确实有点像wrap_content
然后调整item的高度，发现也是不起作用的
在检查代码之后找不到出现问题的原因，百度了一波找到原因
**※在布局转换的时候使用**
**View view = layoutinflater.inflate(R.layout.item_XXX,ViewGroup,false);**
修改之后，效果如下

![](http://upload-images.jianshu.io/upload_images/7793862-8654164db263cc3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
符合预期效果。

###### ※问题：

（1）为什么使用三个参数的inflate方法就能实现效果？
（2）inflate三个参数和两个参数有什么区别？

# 5.LayoutInflater.inflate方法分析

![](http://upload-images.jianshu.io/upload_images/7793862-eae1ccc235c12691.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

inflate有重载4个方法

```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root)
```

```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root)
```

```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot)
```

```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) 
```

查看源码发现不管是三个参数还是两个参数的inflate方法
最终是实现了
 public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot)
attachToRoot的值是根据 root != null来赋值

```java
/**
     * Inflate a new view hierarchy from the specified XML node. Throws
     * {@link InflateException} if there is an error.
     * <p>
     * <em><strong>Important</strong></em>   For performance
     * reasons, view inflation relies heavily on pre-processing of XML files
     * that is done at build time. Therefore, it is not currently possible to
     * use LayoutInflater with an XmlPullParser over a plain XML file at runtime.
     * 
     * @param parser XML dom node containing the description of the view
     *        hierarchy.
     * @param root Optional view to be the parent of the generated hierarchy (if
     *        <em>attachToRoot</em> is true), or else simply an object that
     *        provides a set of LayoutParams values for root of the returned
     *        hierarchy (if <em>attachToRoot</em> is false.)
     * @param attachToRoot Whether the inflated hierarchy should be attached to
     *        the root parameter? If false, root is only used to create the
     *        correct subclass of LayoutParams for the root view in the XML.
     * @return The root View of the inflated hierarchy. If root was supplied and
     *         attachToRoot is true, this is root; otherwise it is the root of
     *         the inflated XML file.
     */
    public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;

            try {
                // Look for the root node.
                int type;
                while ((type = parser.next()) != XmlPullParser.START_TAG &&
                        type != XmlPullParser.END_DOCUMENT) {
                    // Empty
                }

                if (type != XmlPullParser.START_TAG) {
                    throw new InflateException(parser.getPositionDescription()
                            + ": No start tag found!");
                }

                final String name = parser.getName();
                
                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }

                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    // Temp is the root view that was found in the xml
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }

                    // Inflate all children under temp against its context.
                    rInflateChildren(parser, temp, attrs, true);

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(parser.getPositionDescription()
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;

                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }

            return result;
        }
    }
```

#### (1)第一个参数parser

parser：xml文件
如果传入的是layout的id，会调用
final XmlResourceParser parser = res.getLayout(resource)；转换成xml文件

#### (2)第二个参数root，第三个参数attachToRoot

- root不为空
  和第三个参数attachToRoot有关
  ① 如果attachToRoot为true
  root成为inflate出来的view对象的父布局
  ② 如果attachToRoot为false
  inflate出来的view的根布局提供LayoutParams参数的控件

```java
final AttributeSet attrs = Xml.asAttributeSet(parser);
....
 final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }
```

当root不为空，attachToRoot为false时候。
获得xml文件中的属性
通过createViewFromTag生成View
再通过generateLayoutParams(attrs)生成layoutparams参数
设置到view中，使得view有layoutParams

- root为空（null）
  **※不指定父布局，那么inflate出来的view对象的根布局的某些参数会失效，比如Layout_width和Layout_height会失效**

  # 6.写后感

  源码封装的太好了，代码跳来跳去，对于不经常看源码的我来说很容易看不下去。不过还是硬着头皮慢慢看下去，不要往深层去看，有些可以选择性忽略，只要看懂注释大概懂得，等知识累积的更多了相信再去重新解读源码会有新收获。