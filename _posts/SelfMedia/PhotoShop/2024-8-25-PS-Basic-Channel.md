---
layout: post
title: 【PS】基础篇04：通道、蒙版
categories: [SelfMedia, PS]
tags: [通道, 蒙版]
published: false
---

## 1 通道

我们可以把通道看成是另外一种图层。

在通道里可以通过颜色信息或者透明度区域信息，把图像分成若干个层次，便于调整图像色彩或者制作各种选区，辅助图层进行图像处理。

带有色彩和透明度信息的一系列通道，同样构成了图像，也可以说通道基于图像的色彩或透明度来分析、编辑，从另外一个角度诠释了图像的构成。

和图层一样，通道也有不同的类型。
- 原色通道。常为RGB三通道，用于编辑图像的色彩。
- 专色通道。
- Alpha通道。
  - 有什么用？回归到前期工作的根本 -- 选区 上。

### 1.1 通道面板

通道标签  面板菜单
通道列表 
将通道作为选区载入
将选区存储为通道
创建新通道
删除通道

### 1.2 基础知识

我们可以通过通道面板来分别观察图像的红绿蓝三个色光的范围和强度

RGB: 复合通道。是RGB三个通道叠加后的预览。
三个原色通道：红绿蓝。
- 单独查看这三个通道，显示都是灰度图像。
- 主要是因为：
  - 灰度图像更便于观察发光强弱，或者是颜色的多少
  - 这个通道什么颜色，我们是已经知道了的，重点是要知道色光的强度，就用黑白灰来表示强度。
    - 黑：色光强度最低，不发光。值为：0
    - 白：色光强度最高，发光最亮。值为：255
  - 所以默认情况下，PS以灰阶图像来表现颜色通道，为了我们更好地查看色光的分布，查看颜色的区域。
  - 这样就可以让我们基于原色通道，来做一些**特殊的选区**。
- 也可以调整
  - CTRK + K，首选项 > 界面 > 用彩色显示通道


色彩模式和通道的联系
- 不同的色彩模式，原色通道的构成，也是不一样的。 
  - RGB 模式下，原色通道是RGB三通道。 
  - CMYK 模式下，原色通道是CMYK四通道。

### 1.3 Alpha通道

不同格式
- jpg格式，不支持Alpha通道的写入。
- PSD,TGA,TIFF等格式的图片可以支持Alpha通道写入，可以保存Alpha通道的信息。
- 虽然jpg格式不支持Alpha通道，但是我们在编辑的时候，可以添加alpha通道。

新建Alpha通道
- 直接通道面板新建一个通道，就是alpha通道。
  - 注意：通道面板，最多支持56个通道层（包括复合通道、原色通道）
- 在画面上做一个选区，将选区存储为通道。

Alpha通道是什么
- 简单来说，Alpha通道就是记录透明度的一种特殊图层。
- 透明和不透明，和选区有什么关系？选区为什么要靠它来记录？
  - 在通道里，是没有蚂蚁线的，但是可以通过**将通道作为选区载入**
  - 在一个通道里，可以将其他通道，作为选区，载入到当前通道。
  - 在通道、路径、图层面板上，都可以通过 CTRL + 点击其他通道、图层、路径，来将其作为选区载入
- Alpha通道里：
  - 黑：表示不包含像素信息，代表透明
  - 白：表示此处为100%的像素覆盖信息，代表不透明的实色区域
  - 一般情况下，我们使用8位通道,即2的8次方=256种色阶
  - 单个通道 0~255灰阶
  - 0=黑=透明
  - 1~254=灰=半透明
  - 255=白=不透明

Alpha通道操作：
- 像搞图层一样搞通道
- 编辑通道和编辑图层一样，可以使用常用的工具和命令
  - 画笔工具
  - 套索工具
  - 等等
- **灰阶像素的调整和形状的编辑，就是为了得到相应的选区**
- 所以可以这么理解：
  - Alpha通道，就是**选区加工厂**的。
- 假如Alpha通道是块黑板，黑板上贴什么就是什么选区
- 通常：
  - 基于原色通道的灰度编辑，可以获得复杂精细的选区


## 2 蒙版

图层蒙版可以隐藏和显示图层上的部分区域。

如果说Alpha通道是“选区加工厂”，那么蒙版就是“选区车间”。


使用：
- 图层蒙版可以隐藏和显示图层上的部分区域
  - 选区完成后：
    - 点击图层蒙版，则：其他部分会被**蒙**起来
    - ALT + 点击图层蒙版，则：选区部分会被**蒙**起来
    - CTRL + 点击图层蒙版，则：创建一个空蒙版，什么也不**蒙**
      - 注意：建立空蒙版后，还可以继续编辑蒙版
  - 蒙版建立后：
    - ALT + 点击蒙版，则会进入蒙版编辑模式
      - 编辑模式，同通道/图层编辑模式
    - 图层蒙版编辑状态下，对应的就是一个临时的Alpha通道
- 蒙版可以停用、启用
  - Shift + 点击图层蒙版，则：启用/停用图层蒙版
- 调整蒙版
  - 和**选区 > 调整边缘**类似，都是为了制作出最理想的选区

蒙版、通道编辑状态理解：
- 黑板上的灰色，则代表透明程度
- 黑板上显示什么，什么就是选区
  - 通道的：
    - 白色，会成为选区
    - 灰色，**根据灰的程度，会被选中的程度不同**
      - 越接近黑色，则在选区内容被复制到新图层上时，显示的越模糊
      - 越接近白色，则在选区内容被复制到新图层上时，显示的越清晰
  - 蒙版的：
    - 白色，代表完全显示
    - 灰色，**根据灰的程度，显示的程度不同**
      - 越接近黑色，则在选区内容被复制到新图层上时，显示的越模糊
      - 越接近白色，则在选区内容被复制到新图层上时，显示的越清晰

### 2.1 快速蒙版

快速蒙版就是一个临时的Alpha通道，也是为了**选区**，只不过它更加快捷、方便（可以看到图像本身）。

快速蒙版模式的优势:
- 1.快捷 
- 2.和图层叠加显示，方便查看

使用：
- 快速蒙版的被蒙区域的显示颜色，可以调整
- 默认是红色

注意：
- 快速蒙版是一种模式，创立的选区是普通的临时选区
- 如果是重要的选区，则要使用：通道、图层蒙版中的选区，是可以长期保存在psd文件中的

## 3 图层高级知识

### 3.1 图层过滤

在图层面板的最上方，图层过滤可以根据图层不同的性质进行查看管理

- 过滤选项
  - 类型、名称、效果、模式、属性、（标记）颜色
- 图层过滤开关

### 3.2 图层锁定

在图层面板的第二行，图层锁定即是对图层或图层某部分进行操作保护

#### 图形锁定方式

锁定透明像素:禁止对透明区域进行操作

- 注意：锁定透明像素，半透明像素依然可以进行操作，只不过，操作结果，也是半透明的

![20-图层高级-图层锁定-锁定透明区域](/assets/images/Designer/PS/20-图层高级-图层锁定-锁定透明区域.png)

锁定图像像素：禁止编辑图像，但可以移动变换

锁定位置: 禁止移动变换

锁定全部: 禁止一切操作

- 注意：隐藏图层，也可以保护图层编辑，但是，还是可以移动变换 

## 3.3 图层链接

图层链接可以多图层统一：
- 移动
- 变换

使用：
- 图层链接可以追加图层
  - 点击已经在链接中的任意一个图层，再选中新图层，点击链接图层，就可以了
- 临时停用链接
  - Shift + 点击图层上的链接符号，则：临时停用该图层的链接

![图层链接、图层编组、智能对象](/assets/images/Designer/PS/20-图层高级-图层整合-操作属性.png)

## 3.4 图层更多

MARK: wait to complete.


## 总结

### 通道颜色总结

原色通道：
- 黑：色光强度最低，不发光。值为：0
- 白：色光强度最高，发光最亮。值为：255

Alpha通道里：
- 黑：表示不包含像素信息，代表透明
- 白：表示此处为100%的像素覆盖信息，代表不透明的实色区域

### 图层整合可操作属性

![20-图层高级-图层整合-操作属性](/assets/images/Designer/PS/20-图层高级-图层整合-操作属性.png)


## 100 常用快捷键

| 快捷键 | 功能 |
| --- | --- |
| **通道** ||
| CTRL + 点击通道 | 将通道作为选区载入 |

