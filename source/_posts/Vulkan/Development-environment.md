---
title: Development environment
date: 2022-11-09 19:32:28
categories:
- Learning note
tags:
- Vulkan 
---

### Environment

Developing for windows, and using Visual Studio to compile the code.

### Vulkan SDK

Vulkan SDK includes the headers, standard validation, debugging tools and a loader for the Vulkan functions. The loader looks up the functions in the driver at runtime, similarly to GLEW for OpenGL.

The SDK can be downloaded from [here](https://vulkan.lunarg.com/) using the buttons at bottom of the page.

After installing, open the **bin** directory and run the **vkcube.exe** demo to verify that the graphics card and driver properly support Vulkan.

### GLFW

[GLFW](https://www.glfw.org/) library is used to create a window. The latest release of GLFW on [here](https://www.glfw.org/download.html). 

### GLM

Vulkan does not include a library for linear algebra operations. [GLM](http://glm.g-truc.net/) is a nice library that is designed for use with graphics APIs and is also commonly used with OpenGL. The latest release of GLFW on [here](https://github.com/g-truc/glm/releases). 

### Setting up Visual Studio

Setting up a basic Visual Studio project for Vulkan and write a little bit of code to make sure that everything works.
