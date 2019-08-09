In this tutorial, we create a new class called TextureGPU,
which is very similar to the last tutorial's BufferGPU, which
will allow us to make images that are in VRAM. We will use
this to draw a 3D Cube with some colors. 

To use a depth buffer in our rendering to create a 3D image,
first we need to create the depth buffer, and then we need 
to tell the GPU exactly how to use the depth buffer. 
If you want to look in the TextureGPU class and learn how
it works, feel free, the comments explain everything.

In Demo.h we add a TextureGPU called depthBufferGPU, and
we have a function called Demo::prepare_depth_buffer().
Every time we call prepare(), we have to rebuilt this
depth buffer to have the same dimensions as the screen,
so every time the window is resized, this buffer is 
destroyed and recreated. We delete depthBufferGPU in the
delete_resolution_dependencies() function, and we call
prepare_depth_buffer() in prepare(), without checking
for firstInit.

We need to add the depth buffer to the renderpass, 
all of the framebuffers. This will require changes in
prepare_render_pass and prepare_framebuffers. 

In the last tutorial, we
only used initCmd during firstInit of prepare, but
now we need to use initCmd every time we call prepare(),
so that the depth buffer can be rebuilt. Please read
the comments of prepare(), to see how we can reuse
initCmd, while only using it for the depth buffer
after firstInit is complete. We need to change the
execute_init_cmd() function, so that the command buffer is
not destroyed after execution, but reset, so that we can use
the command buffer the next time we call prepare, then
we need to destroy initCmd and initFence in the Demo destructor

We also have to alter VkPipelineDepthStencilStateCreateInfo
when we create the pipeline, so that the pipeline
knows what to do with the depth buffer. Otherwise, the GPU
simply won't use the depth buffer that we worked so hard 
to make.

In our build_swapchain_cmds() function, we need to add another
element to the VkClearValues array, to allow the depth buffer
to be cleared. We also need to change VkViewport to have a 
minimum and maximum depth value.

After that, we change the "model" matrix in our uniform
buffer to be an MVP matrix, which is a combination of 
model, view, and projection. View and Projection matrices
help us manage our 3D Camera. These can be found in the
prepare_uniform_buffers() function, which is called in prepare(),
and it can be found again in update_uniform_buffer()