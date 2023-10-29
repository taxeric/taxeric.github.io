---
layout: post
title: ViewModel
subtitle: 
categories: Android
tags: 源码
---



As we all know，ViewModel可以用来持久保留界面数据

初始化

```kotlin
    vm = ViewModelProvider(this)[MainVM::class.java]
```

调用构造

```kotlin
    public constructor(
        owner: ViewModelStoreOwner
    ) : this(owner.viewModelStore, defaultFactory(owner), defaultCreationExtras(owner))
```

其中`owner.viewModelStore`为

```java
    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        ensureViewModelStore();
        return mViewModelStore;
    }
```

`ensureViewModelStore`方法会创建一个`ViewModelStore`，而后者维护一个HashMap，如下

```java
private final HashMap<String, ViewModel> mMap = new HashMap<>();
```

