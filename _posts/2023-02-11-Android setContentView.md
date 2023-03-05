---
layout: post
title: Android setContentView
subtitle: 
categories: android
tags: 源码
---

再看一次

再看一次

再看一次

# 流程过一遍

`setContentView(layoutId)`源码

```java
    @Override
    public void setContentView(@LayoutRes int layoutResID) {
        initViewTreeOwners();
        getDelegate().setContentView(layoutResID);
    }
```

看到`getDelegate().setContentView(layoutResId)`，其中`getDelegate`获取到的是`AppCompatDelegate`的代理，具体实现为`AppCompatDelegateImpl`，所以直接看impl的setContentView方法

```java
    @Override
    public void setContentView(int resId) {
        ensureSubDecor();
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mAppCompatWindowCallback.bypassOnContentChanged(mWindow.getCallback());
    }
```

首先调用`ensureSubDecor`确保存在`SubDecor`

```java
    private void ensureSubDecor() {
        if (!mSubDecorInstalled) {
            mSubDecor = createSubDecor();

            //...
            applyFixedSizeWindow();

            onSubDecorInstalled(mSubDecor);

            mSubDecorInstalled = true;
            //...
        }
    }
```

看`createSubDecor`方法

```java
    private ViewGroup createSubDecor() {
        //...获取一些属性

        // Now let's make sure that the Window has installed its decor by retrieving it
        ensureWindow();
        mWindow.getDecorView();

        final LayoutInflater inflater = LayoutInflater.from(mContext);
        ViewGroup subDecor = null;

        //...

        final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);

        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
            // There might be Views already added to the Window's content view so we need to
            // migrate them to our content view
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }

            // Change our content FrameLayout to use the android.R.id.content id.
            // Useful for fragments.
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);

            // The decorContent may have a foreground drawable set (windowContentOverlay).
            // Remove this as we handle it ourselves
            if (windowContentView instanceof FrameLayout) {
                ((FrameLayout) windowContentView).setForeground(null);
            }
        }

        // Now set the Window's content view with the decor
        mWindow.setContentView(subDecor);
        //...

        return subDecor;
    }
```

在`ensureWindow`方法种首先确保window存在，在这个方法中，调用`attachToWindow`，类对象`mWindow`被赋值为强转成`activity`的`getWindow`，看源码得知该window为`PhoneWindow`，然后调用window的`getDecorView`方法（即**PhoneWindow**的`getDecorView`）

```java
    @Override
    public final @NonNull View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            installDecor();
        }
        return mDecor;
    }
```

在该方法中安装了Decor。看实现

```java
    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            //...
        } else {
            mDecor.setWindow(this);
        }
        //...下面再看
    }
```

首先调用`generateDecor`创建一个DecorView对象，并赋值给mDecor，然后继续

```java
private void installDecor() {
    //...
    if (mContentParent == null) {
        mContentParent = generateLayout(mDecor);
        //设置一些属性
    }
}
```

`mContentParent`是一个ViewGroup，被赋值为`generateLayout`方法返回值。看实现

```java
    protected ViewGroup generateLayout(DecorView decor) {
        //...设置一些标志位和属性

        //此处的ID_ANDROID_CONTENT为  com.android.internal.R.id.content
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        //...找到对应id的控件

        //...一些配置

        return contentParent;
    }
```

使用`findViewById`找到`ID_ANDROID_CONTENT`对应的View，强转为ViewGroup，其中`findViewById`使用的是`getDecorView().findViewById()`，故此处实际为调用mDecor的findViewById

至此`installDecor`走完，`mWindow.getDecorView()`走完。回到`createSubDecor`方法，继续

```java
    private ViewGroup createSubDecor() {
        //...
        ensureWindow();
        mWindow.getDecorView();

        final LayoutInflater inflater = LayoutInflater.from(mContext);
        ViewGroup subDecor = null;

        if (!mWindowNoTitle) {
            if (mIsFloating) {
                // If we're floating, inflate the dialog title decor
                subDecor = (ViewGroup) inflater.inflate(
                        R.layout.abc_dialog_title_material, null);
            } else if (mHasActionBar) {
                //...
                subDecor = (ViewGroup) LayoutInflater.from(themedContext)
                        .inflate(R.layout.abc_screen_toolbar, null);
                //...
            }
        } else {
            if (mOverlayActionMode) {
                subDecor = (ViewGroup) inflater.inflate(
                        R.layout.abc_screen_simple_overlay_action_mode, null);
            } else {
                subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
            }
        }
        //...

        if (mDecorContentParent == null) {
            mTitleView = (TextView) subDecor.findViewById(R.id.title);
        }

        // Make the decor optionally fit system windows, like the window's decor
        ViewUtils.makeOptionalFitsSystemWindows(subDecor);

        final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);

        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
            // There might be Views already added to the Window's content view so we need to
            // migrate them to our content view
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }

            // Change our content FrameLayout to use the android.R.id.content id.
            // Useful for fragments.
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);

            // The decorContent may have a foreground drawable set (windowContentOverlay).
            // Remove this as we handle it ourselves
            if (windowContentView instanceof FrameLayout) {
                ((FrameLayout) windowContentView).setForeground(null);
            }
        }

        // Now set the Window's content view with the decor
        mWindow.setContentView(subDecor);
        //...

        return subDecor;
    }
```

首先根据`mWindowNoTitle`变量来创建对应视图，赋给`subDecor`，此处的`subDecor`是局部变量而**不是window.mDecor**。无论是呐哪个布局，都有include到`abc_screen_content_include.xml`文件，所以在下面通过`subDecor.findViewById(R.id.action_bar_activity_content)`，获取到`contentView`，类型为`ContentFrameLayout`。继续，调用`mWindow.findViewById(android.R.id.content)`获取到`window.mDecor`对应的`content`布局，将后者的child添加到前者，并将后者的`id`置空，将`android.R.id.content`赋给此处局部变量`subDecor`对应的`contentView`，然后mWindow调用setContentView设置新布局。

```java
    @Override
    public void setContentView(View view, ViewGroup.LayoutParams params) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            view.setLayoutParams(params);
            final Scene newScene = new Scene(mContentParent, view);
            transitionTo(newScene);
        } else {
            mContentParent.addView(view, params);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```



至此`ensureSubDecor`走完。

回到我们自己`xxActivity`调用的`setContentView(id)`方法

```java
    @Override
    public void setContentView(int resId) {
        ensureSubDecor();
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mAppCompatWindowCallback.bypassOnContentChanged(mWindow.getCallback());
    }
```

可以看到剩下的就是将我们自己的布局添加到content中，此处的content即为上述subDecor对应的contentView