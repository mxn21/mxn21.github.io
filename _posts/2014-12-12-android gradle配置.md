---
layout: post
author: mxn
title: android gradle配置
category: 技术博文
tag: [gradle]
---


#####1.去除无用的资源：

```
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
        }
    }
}

```
