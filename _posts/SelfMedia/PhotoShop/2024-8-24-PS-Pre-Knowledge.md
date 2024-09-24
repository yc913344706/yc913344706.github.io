---
layout: post
title: 【PS】01 基础篇01：预置知识
categories: [SelfMedia, PS]
tags: [PS主界面, 分辨率]
---

## 1 学习PS的核心逻辑

- 了解基本概念
了解专业知识的基础概念，了解基本功能的设置，搞清来龙去脉，具备扎实的理论基础

- 掌握操作规律
总结相似的知识，组合同类的情况，挖掘软件的使用规律通过对概念的明确理解，快速学会一系列的操作。

- 开发扩展思维
通过对概念的深刻认识，抓住使用规律，便可以达到触类旁举一反三的效果!

## 2 主界面

- 菜单栏
  - 图像
    - 图像大小：修改图像大小，可以设置宽高，分辨率。 
  - 窗口
    - 工作区
      - 复位基本功能：将工具栏、选项栏、菜单栏等面板全部恢复到初始状态
- 选项栏：是调节工具参数的面板
  - 切换工作区： 
- 工具栏
- 面板
- 工作区
  - 工具面

## 3 新建文档

新建文档时候，可以选择不同的模板，不同的模板对应不同的画布宽、高、分辨率等。 

### 3.1 不同图像

- 像素图：点组成，放大后会模糊，有锯齿
- 矢量图：数学函数组成，放大不会模糊，没有锯齿

### 3.2 不同单位

- 像素：用于屏幕，网页，图片等
- 毫米：用于印刷，文档等

### 3.3 分辨率

- 单位：
  - ppi: 每英寸的像素数（常用）
  - 每厘米的像素数（不常用）
- 使用：
  - 分辨率确实越大越好，但是越大，占用电脑内存就越大，所以，合适就好
- 常用分辨率设置 :
  - 洗印照片，300或以上
  - 杂志，名片等印刷物: 300
  - 海报高清写真: 96-200
  - 网络图片，网页界面 72
  - 大型喷绘 25-50

如果图像很模糊，一半原因有以下几个：
- 分辨率太低
- 图像本身拍摄模糊
- 分辨率被强行改高

### 3.4 颜色模式

- 灰度：纯黑和纯白，其他颜色由灰度值决定
- RGB：红绿蓝，三原色调和而成。广泛用于`显示屏`。
- CMYK：青色、品红、黄色、黑色四种颜色调和而成. 广泛用于`印刷`。

颜色位数

- 8位：256种颜色。
- 16位：65,536种颜色
- 32位：16,777,216种颜色

一般的显示器8位就已经足够了，16位是为了解决显示器的色差问题，32位是为了解决显示器的模糊问题。

## 4 图像大小，文档大小

约束比例：固定宽高比。注意：不会锁定分辨率。
缩放样式：在修改文档大小时，它的图层样式也会改变。 
重定图像像素：文档大小不变的情况下，修改分辨率.

像素锁定了，文档大小也就基本锁定了

$$画面总像素值 = 宽像素值 * 高像素值$$
$$画面总像素值 = 宽(英寸) * 高(英寸) * 分辨率(像素/英寸)^2$$

图片强行放大，并不会提高图片质量

对于网络或者显示屏的图片，分辨率设置为72或96即可。

分辨率单位
- Ppi：Pixels Per Inch，每英寸的像素数，描述图片本身大小。
- Dpi: Dot Per Inch，点分辨率，衡量输出精度。打印机产生的每英寸的油墨点数，或者显示输出设备每英寸的显像点数。
  - 高dpi依赖于：
    - 图片自身的分辨率
    - 纸张的质量要足够好，能承受这么密集的墨点

另：Dpi的计算：
- iphone4
  - 屏幕大小：大概为：1.9*2.9英寸
  - 像素大小：640*960
  - 计算后：Dpi = 640/1.9 = 340

挂网：
- 挂网是印刷的一种专业术语。
- 挂网有一个精度指标：Lpi. Liner Per Inch，每英寸的线数。
- Lpi是指印刷图象所用的网屏的每英寸的网线数。
  - 以后再说，现在大概有个概念。普通报纸、铜版纸（杂志、海报等）

## 5 文件格式

- PSD: Photoshop Document, Photoshop专用格式，可以保存图片、文字、形状、图层、滤镜、特效、调色板、历史记录等。
  - 隐藏图层后，文件大小会更小。
- PSB: Photoshop Big, Photoshop专用格式，可以支持300,000分辨率的超大文件.
- JPG: JPEG, 是最流行的图片文件格式。
- GIF: Graphics Interchange Format，支持动画、透明色. 
- 存储为WEB所用格式: 网页素材，后续再说。 

## 6 开工前准备

首选项 > 性能 > 暂存盘: 
- 用于存放系统运行过程中的临时数据文件，因为内存条是装不下的。
- 不要放到C盘，不要放到软件安装盘即可。 

首选项 > 性能 > 历史记录状态: 
- 历史记录条数

首选项 > 文件处理 > 自动存储恢复信息的间隔
- 自动保存，防止丢失数据

编辑 > 指定快捷键
- 自定义快捷键，可以定义不同的预设快捷键，来回切换。
- 快捷件可以导出、导入

## 100 常用快捷键

| 快捷键 | 功能 |
| --- | --- |
| X | 切换前景色和背景色 |
