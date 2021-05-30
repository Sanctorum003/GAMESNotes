# 实时渲染概述
* Real-Time High Quality Rendering
  - Real-Time
    - Speed:more tham 30 FPS(frame per second)
      - VR/AR : 90 FPS
    - Interactivity: Each frame generated `on the fly`(运行中)
  - High Quality
    - Realism:advanced approaches to make rendering more realistic
    - Dependability:all-time correctness(exact or approximate),no tolerance to (uncontrollable) failures
  - Rendering
    - 将虚拟场景通过计算方式投射到屏幕上
***
* 课程内容
    - Shadows(and env)
    - Global Illum.(Scene/image space,precomputed)
    - Physically-based shading
    - Real-time ray tracing
* 课程不包括
    - 3d modeling or game engine
    - 离线渲染
    - Neural Rendering
    - Using Opengl
    - Scene /Shader optimization
    - Reverse enginerring of shaders
    - High performace computing
***
* Render-time rendering = fast & approximate offline rendering + systematic engineering
    - Fact:in real-time rendering technologies,the industry is way ahead of the academia
***
* Technological and Algorithmic Milestones
    - Programmable graphics hardware(shaders)(可编程着色器)(20 years ago)
    - Precomputation-based methods(15 years ago)
      - Complex visual effects are (partially) pre-computed
      - Minium rendering cost at run time
      - Precomputation-based methods:Light
        - fix geometry
        - fix viewpoint
        - dynamically change light
    - Interactive Ray Tracing(8-10 years ago:CUDA+OPtiX)
      - Hardware development allows ray tracing on GPUs at low sampling rates(~1 samples per pixel(SSP))
      - Followed by post precessing to denoise

# Recap of CG Basics
## Grapics(Hardware) Pipeline
* Vertex Processing
  - MVP Transform
* Triangle Processing
* Rasterization
  - Sampling Triangle converage
* Fragment Processing
  - Shading
  - Texture mapping & interpolation
* Framebuffer Operations
  - Z - Buffer Visibility 

## Opengl
* 略

## GLSL
* 略

## The Rendering Equation
* Most Impoartant equation in rendering
  - Descring light transport

![](Games202/Origin_Rendering_Equation.png)

***
* In RTR(real-time rendering)
  - Visibility is often explicitly considered
    - Visibility 实时渲染会考虑到物体会不会被光源找到，就引入Visibility 的概念。
    - 在实时渲染中incident light单指从光源发出的光线
      - 那间接光呢？
  - BRDF is often considered together with cosine term
    - brdf可能指原始brdf也可能指cosine-weighted brdf

![](Games202/RTR_Rendering_Equation.png)

* 这样定义便于理解环境光照
  - 对于任意一个shading point只要判断这点与光源之间是否可见
    - 将incident light和有没有光线照到这个点拆开

***
全局光照 = 直接光照 + 间接光照

## 课堂答疑整理

* 1.问：怎么定义哪些点连接成三角形? 
* 答：比如obj格式是先编号一系列点，再定义面，每个面都带着三个点的编号的下标。
***
* 2.问：有无适合全局光照的管线？
* 答：无，但光线追踪渲染管线很完善。
***
* 3.问：纹理坐标怎么参数化的？
* 答：可以理解成物体外面有个盒子，把盒子挤压到物体上，盒子上的uv是好确定的。
***
* 4.问：几个光源就需要几个shadowmap吗？光源也需要深度测试吗？* 答：是
***
* 5.问：opengl支持optix吗？
* 答：可以，但不要指望shader能调用一个光线一个场景怎么运作。
***
* 6.问：纹理是不是是个buffer？
* 答：opengl里两者定义不一样，但也可以这样理解，都是显存里的一块缓存区域。
***
* 7.问：一个pass就是一个frambuffer渲染一次吗？
* 答：一个场景渲染一次
***
* 8.问：不同shader里定义里的变量会通用吗？
* 答：不是

# Real-time Shadows 1
## shadow Mapping

* A 2-Pass Algorithm
    - The light pass generates the SM
    - the camera pass uses the SM(recall last lecture),(真正的渲染)

* An image-space algorithm
  - Pro:no knowledge of scene's geometry is required
  - Con:casuing self occlusion(自遮挡) and aliasing issues(走样)

***
### Shaow Mapping 方法
* Pass1:Render from light
  - output a "depth texture" from the light source
* Pass2:Render from eye
  - Render a standard image from eye
    - 如果眼睛看到的深度等于shaow map的深度说明被光线照到
    - 如果眼睛看到的深度大于shaow map的深度说明在阴影里

### Issues in Shadow Mapping
#### Self occlusion && detached shadow
* Self occlusion(自遮挡问题)
  - 表现:地面上的奇怪纹路,也叫阴影失真(Shadow Acne)
![](Games202/Shadow_Mapping_Occlusion.png)

* 原因:
  - 由于shadowmap记录的深度是不连续的，是一个一个像素的深度值组成的，如下图右，眼睛观察到的像素会误认为深度在前面的像素后面，形成遮挡，产生错误的阴影。当light和平面趋于平行时最严重。

* 解决方法
  - 通过一个shaow bias(阴影偏移)，使得在shading point上shadow map中的深度和从eye看到的深度在一定范围内的时候,不算遮挡住

  ```cpp
  float bias = 0.005;
  float shadow = currentDepth - closestDepth > bias ? 1.0 : 0.0;
  ```
  - 在grazing angle(几乎是平行)的时候self occlusion问题最严重,在光线垂直投射的时候最不严重,可以将bias设计的与光线夹角有关.

  ```cpp
  float bias = max(0.05 * (1.0 - dot(normal, lightDir)), 0.005);
  ````

* 新问题:detached shadow
  - 忽略一定的深度的情况下,肯定会将一些本来就差距很小的阴影给舍弃掉,如下图脚后跟的阴影
  - 也叫悬浮(Peter Panning)，因为物体看起来轻轻悬浮在表面之上
  ![](Games202/Shadow_mapping_Detached_shaow.png)

* 一种解决方法:Second-depth shadow mapping
  - Using the midpoint between first and second depths in SM
    - 可以在第一个pass时，可以在渲染一张light看到最近深度的图同时渲染一张次近的图（个人理解是透过最近的深度的像素再渲染一张最近深度），将两个深度的中间深度来算遮挡阴影。
  - Unfortunately,requires objects to be watertight
    - 一个是场景内的物体必须都是watertight（非面片
  - And the overhead may not worth it
    - 算的复杂，开销太大，实时渲染不相信复杂度。

![](Games202/Second_depth_shadow_mapping.png)

#### Aliasing(走样)

## The Math behind shaow mapping
* there are a lot of useful inequalities in calculus
* but in RTR,we care more about "approximately equal"
  - 微积分在实时渲染中更关心近似相等，把不等式当约等于来用
* An important approximation throughout RTR

  $$\int_{\Omega}f(x)g(x)dx\approx\frac{\int_{\Omega}f(x)dx}{\int_{\Omega}dx}\cdot\int_{\Omega}g(x)dx$$

  - 除以一个积分是为了式子拆分以后，不希望得到的能量波动太大，就需要除以一个积分归一化。

  - what is it (more) accurate?
    - 当g的积分的范围很小的时候或者g这个函数足够光滑(低频,变化不大)的时候就会比较准确。

### shaow mapping的理论基础
* Recall the rendering equation with explicit visibility

![](Games202/Rendering_Equation_Approximated_For_Shadow_Mapping.png)

* 有了这个方法就能把the rendering equation拆开来写，把V拆出来，这样式子就变成了左边是visibility(遮挡)右边是shading，这样就和shadowmap的思路相吻合。

* when is it accurate?(两个满足一个即可)
  - Small support(point / directional light)
    - 当只有点光源的时候和方向光源的时候，积分范围小到基本没积分什么事，这个式子就成立。
  - Smooth integrand(低频,变化不大)(diffuse(漫反射) bsdf(=brdf+btsd)/constant radiance area light)
    - 当在光源的radiance恒定(uniform)情况下，就比如一个面光源，当brdf是diffuse的情况下(glossy就不行)，右边的函数足够光滑，得到的结果也正确的。

## Percentage closer soft shadows
### Percentage Closer Filtering(PCF)
* Provides `anti-aliasing` at shadows' edge
  - Not for soft shadows(PCSS is,introducing later)
  - Filtering the results of shadow comparisons
* 不是对shadow map进行filitering
  - 如果直接在shadow map上filtering就会造成阴影和物体交界直接糊起来
    - Texture filitering just averages color componets,i.e. you will get blurred shadow map first
  - 做深度测试的时候,判断结果任然是非0即1的(任然是阴影硬)
    - Averaging depth vlaues,then comparing,you still get a binary visibility
* 不是对最后得到有锯齿的阴影硬图做filtering
  - 这样做只能得到模糊的带锯齿的图,还剩没有阴影硬也没有去掉锯齿
    - 具体可以回顾GAMES101中的抗锯齿

    ![](Games202/Filter_the_shadow_map.png)

***

* Solution[Reeves,SIGGAPRH 87]
  - Perform multiple(e.g. 7*7) depth comparsions for each fragment
    -对着色点周围一圈的像素与改点在shadow map中记录的深度做比较
  - Then，averages results of comparisons
    - 对这些比较结果做平均得到一个在0~1之间的值

***
* filitering size ->
  - Small -> sharper
  - Large -> softer

## PCSS(Percentage Closer Soft Shadows)
* 在面光源下,什么地方产生阴影硬,什么地方产生软阴影
  - Filter size <-> blocker distance
    - 遮挡物和阴影接受物的距离
  - More accurately,`relative` `average` projected blocker depth
    - 更准确，相对平均投影阻挡深度
  - A mathematical "translation"
    - 可以用相似三角形得出，filtering的范围大小可以通过遮挡物到被遮挡面的距离除以遮挡物到光源的距离乘以光源面积得到。
    - $W_{penumbra} = (d_{Receiver} - d_{Blocker}) \cdot w_{Light} / d_{Blocker}$

    ![](Games202/PCSS.png)

***
* Now the only question:
  - what's the blocker dpeth $d_{Blocker}$
    - 对于场景中的物体,遮挡物一般是不是一个点或者一个平面,收益对于某一着色点Blocker的$d_{Blocker}$需要对周围一定范围内的被遮挡物求平均,计算平均遮挡深度.

* The complete algorithm of PCSS
  - Step 1:Blocker serch
    - getting the average blocker depth `in a certain region`
      - 只对能够遮挡住的像素求平均
  - Step 2:Penumbra estimation(半影估计)
    - use the average blocker depth to determine filter size
  - Step 3:Percentage Closer Filtering

* 注意面光源无法生成Shadow map,这里生成shadow map通过将light放在面光源中心,当做点光源生成shadow map

* which region to perform blocker search?
  - can be set constant (e.g. 5*5),but can be better heuristics(启发式)

* which region(on the shadow map) to perform blocker serach?
  - depends on the light size
  - and receiver's distance from the light
>  可以把shader point连向light，看在shadow map(就是视锥体的近平面,就说用于光栅化的平面，在第一个PASS用于生成shadow map)所占范围来确定，因为离光源越远，遮挡物也会更多。

![](Games202/PCSS_Blocker_Search_Size.png)

## 答疑

1.纯工业界问题最后一节说

2.2倍的n不太能接受。 的确，会慢很多，还是很在乎常数的，不止复杂度

3.离线渲染有这样用近似的吗。 好像没见过。

4.有误差公式吗？ 没有必要

5.smooth对左右两个函数都有要求吗。 只要一个

6点光源怎么用pcss。 点光源本身生成的就是硬阴影，不用pcss

7.shadow maping好处就是说任何一帧你都知道那个物体在哪，反正阴影都是重新算的，shodow map也是重新算的，和运动物体没关系。

## More on PCF and PCSS(A Deeper Look at PCF)
* The math behind PCF
  - Filtering / convolution:
    - $[ w * f](p) = \Sigma_{q \in N(p)} w(p,q)f(q)$
      - 这个卷积公式,取p点领域内的所有点q,根据权值项w(p,q)对所有的f(q)进行加权平均
      - w(p,q):根据p,q之间距离的一种权值
  - PCSS
    - $V(x) = \Sigma_{q \in N(p)} w(p,q) \cdot \chi^{+}[D_{SM}(q) - D_{scene}(x)]$
      - $\chi^{+}[]$:为符号函数
      - 这个公式将shading point点x对应shadow map上点p周围领域上的所有点q进行判断,这个点q是否能遮挡住shading point点x(即在shadow map上q点的深度是否小于p点的深度),并对所有的判断进行加权平均
    ![](Games202/Deeper_Look%20_at%20_PCF.png)
  - PCSS不是filteringshadow map
    - $V(X) \neq \chi^{+}\{[w * D_{sm}](q) - D_{scene}(x)\}$
  - PCSS不是对最后生成的图进行filtering
    - $V(x) \neq \Sigma_{y \in N(x)}w(x,y)V(y)$

***
* in PCSS,which step(s) can be slow?
  - Looking at very texel inside a region(step 1 and 3)
    - 都要对区域内所有像素进行比较
  - Softer -> larger filtering region -> slower

  ## VSSM(Variance Soft Shadow Mapping)