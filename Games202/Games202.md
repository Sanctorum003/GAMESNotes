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

# Real-time Shadows 1