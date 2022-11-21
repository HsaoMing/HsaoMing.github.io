---
title: Drawing a triangle
date: 2022-11-12 00:17:01
categories:
- Learning note
tags:
- Vulkan 
---

### General structure

```c++
#include <vulkan/vulkan.h>

#include <iostream>
#include <stdexcept>
#include <cstdlib>

class HelloTriangleApplication {
public:
    void run() {
        initVulkan();
        // the function include a loop that iterates until the window is closed in a moment.
        mainLoop();
        // Deallocating the resources which were used in this function.
        cleanup();
    }

private:
    void initVulkan() {

    }

    void mainLoop() {

    }

    void cleanup() {

    }
};

int main() {
    HelloTriangleApplication app;

    try {
        app.run();
    } catch (const std::exception& e) {
        std::cerr << e.what() << std::endl;
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

### Resource management

Just like each chunk of memory allocated with `malloc` requires a call to `free`, every Vulkan object that we create needs to be explicitly destroyed when we no longer need it.

The parameters for these functions generally vary for different types of objects, but there is one parameter that they all share: `pAllocator`. This is an optional parameter that allows you to specify callbacks for a custom memory allocator. 

### Integrating GLFW

That way GLFW will include its own definitions and automatically load the Vulkan header with it. Replace the `#include <vulkan/vulkan.h>` line with

```c++
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>
```

Add a `initWindow` function and add a call to it from the `run` function before the other calls.

```c++
void run() {
	initWindow();
    initVulkan();
    mainLoop();
    cleanup();
}
private:
    void initWindow() {

    }
```

