# 材质
## 3.1 材质的基本原理 原理化BSDF
> **PBR(Physically Based Rendering)**
> **BRDF(Bidirectional Reflectance Distribution Function)**，实际上可以理解为根据一系列的光线、材质的物理性质编写的函数。
> **BSDF = BRDF + BTDF**

- 删除材质（鼠标指针处）  
![BV14u41147YH-P15-[03:22]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-1.png)
- 保留一个未赋予给任何物体的材质（未赋予给任何物体的材质会在软件被关闭时被删除）
![BV14u41147YH-P15-[03:31]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-2.png)
- 删除场景世界的灯光（**只是方便预览**）  
![BV14u41147YH-P15-[05:20]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-3.png)
***世界灯光就是设置一个天然存在的光照场景，将**世界不透明度**调整到1就可以看到是什么样的场景***
- 原理化BSDF中的选项可以分为3类，每个选项的意义建议看原视频，或者动手进行尝试。  
![BV14u41147YH-P15-[07:53]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-4.png)  
![BV14u41147YH-P15-[18:45]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-6.png)

- 同一个物体多个材质，如图鼠标指针位置的“+”。*要注意的是：一个物体同时拥有两个材质，会显示第一个材质，需要进入编辑模式选中特定的面，将其他材质赋予这些面（选择“+”下方的“指定”按钮）。*
![BV14u41147YH-P15-[18:12]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-5.png)

## 3.2 添加有纹理的材质-纹理坐标 映射 颜色渐变 
- 着色界面--操控材质的主要界面  
![BV14u41147YH-P16-[00:30]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-7.png)
- 或者新建一个着色编辑器窗口  
![BV14u41147YH-P16-[00:38]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-8.png)
- 给材质添加纹理的过程：  
**添加纹理节点**
![BV14u41147YH-P16-[01:24]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-9.png)
**映射节点**
![BV14u41147YH-P16-[02:03]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-12.png)
**纹理坐标节点：决定了2d的图片纹理如何映射到3d的物体上**
![BV14u41147YH-P16-[02:25]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-13.png)
- 材质纹理的五个节点主要作用
![BV14u41147YH-P16-[06:21]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-14.png)
### 纹理节点
- 不同纹理节点：生成、UV、物体。  
UV就是将3d的物体展开为一个2d平面，然后跟2d的纹理图像映射；  
生成与物体则分别是基于全局坐标系和局部坐标系：  
**物体模式**
![BV14u41147YH-P16-[08:09]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-16.png)
*移动物体的原点会影响纹理的映射*
**生成模式**
  ![BV14u41147YH-P16-[08:19]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-17.png)
*移动物体的原点不会影响纹理的映射*

- 渐变纹理：  
![BV14u41147YH-P16-[15:56]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-18.png)

- 程序化纹理（blender内置的一些纹理），如**噪波纹理**  
![BV14u41147YH-P16-[16:11]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-19.png)

- 在着色编辑器中，ctrl + x可以删除节点

## 3.3 用蒙版做纹理的叠加

- 自动加节点的插件 ***node wrangler***  
![BV14u41147YH-P17-[02:37]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-20.png)

- 混合节点  
![BV14u41147YH-P17-[06:24]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-21.png)
可以用来将两张贴图进行混合；  
![BV14u41147YH-P17-[08:04]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-22.png)  
**也可以将一张黑白的图片传入到系数节点。（有黑有白的图片输入到“系数”中时，实际上就变成了一个蒙版）**  

## 凹凸感和置换形变
### 凹凸感
![BV14u41147YH-P18-[01:20]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-23.png)
- node wrangler 第二个快捷键。
![BV14u41147YH-P18-[03:10]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-24.png)

- 将纹理的输出连接到凹凸节点的高度上。
![BV14u41147YH-P18-[03:31]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-25.png)

***使用材质中的凹凸节点产生的凹凸是假凹凸，并非面与面之间产生了位移，而是某些面的法线方向被改变，从而导致对关照的反射发生改变，看起来凹凸。***

真正有形变的凹凸是通过“置换”做出来的。

### 置换
1. 置换修改器（之前建立石块的模式时涉及过）  
置换修改器实际上就是造成一些形变，因此第一步需要**增加细分**（编辑模式下右键物体）
![BV14u41147YH-P18-[13:18]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-26.png)
第二步添加置换修改器
2. 置换节点   
**先将渲染器改为cycles，再在*材质设置--表（曲）面*中将“仅凹凸”更换成“置换与凹凸”（这一步很重要）**
创建置换贴图（实际上就是黑白灰三色的图片，用以表示何处凹，何处凸）的纹理节点，并将其输入到“置换节点”。
![BV14u41147YH-P18-[16:51]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-27.png)

**node wrangler第三个快捷键**： 
新建一个材质，选中原理化BSDF节点，ctrl + shift + T后，再跳出的窗口中选择用于颜色、法向、糙度和置换的四张图片后，可以自动生成节点。（要注意的这种使用的是置换节点的方法，只对cycles渲染器有效）  
![BV14u41147YH-P18-[19:09]](./images/bd9772fb-fbf8-459e-8a7b-c2bfdae94cab-28.png)







