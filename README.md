OpenGL
=======
这里会记录一些OpenGL相关资料

### 当发现用模拟器始终无法运行OpenGL的程序的时候（“USE HOST GPU”已经被勾选了）
>尝试在glSurfaceView.setRender()之前执行this.setEGLConfigChooser(8, 8, 8, 8, 16, 0);
