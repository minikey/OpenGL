OpenGL ES 1.0的渲染管线
==========
OpenGL ES中的渲染管线实质上指的是一系列绘制过程。这个过程输入的是待渲染3D物体的相关描述信息数据，经过渲染管线，输出的是一帧想要的图像。

1. ### 基本处理
设定3D空间物体的顶点坐标、顶点对应的颜色、顶点的纹理坐标等属性，并且指定绘制方式，如点绘制、线段绘制或者三角形绘制等。

2. ### 顶点缓冲对象
这个功能可选，如果顶点的基本数据不变的情况下。可以在初始化阶段将顶点数据经过基本处理后送入顶点缓冲对象中，在绘制每一帧想要的图像时就省去了顶点数据IO的问题。

3. ### 变换和光照
这个阶段的工作主要是进行顶点变换以及根据程序中设置的光照属性对顶点进行光照计算。顶点变换的任务是对3D物体的各顶点进行平移、旋转或者缩放等操作。光照计算的任务是根据程序送入的光源位置、性质、各通道强度、物体的材质等，同时再根据一定的光照数学模型计算各个顶点的光照情况。

4. ### 图元装配
这个阶段主要有两个任务：
    * 图元组装
    >顶点数据根据设置的绘制方式被结合成完整的图元。例如绘制方式仅需要一个单独的顶点，此方式下每个顶点为一个图元；线段绘制方式则需要两个顶点，此方式下每两个顶点构成一个图元；三角形绘制方式下需要三个顶点构成一个**图元**。
    
    * 图元处理
    >图元处理最重要的工作就是裁剪，其任务是消除位于半空间(half-space)之外的部分集合图元，这个半空间是由一个裁剪平面所定义的。例如，点裁剪就是简单的接收或者拒绝顶点，线段或者多边形裁剪可能需要增加额外的顶点，具体取决于直线或者多边形与裁剪平面之间的位置关系

5. ### 光栅化
将虚拟3D世界中的物体投影到视平面上。由于虚拟3D世界当中物体的集合信息一般采取连续的数学量来表示，因此投影的平面结果也是用连续数学量表示的，但目前的显示设备屏幕都是离散化的(由一个一个的像素组成)，因此还需要将投影的结果离散化。_将其分解为一个一个离散化的小单元，这些小单元就被称为**片元**_。
其实每个片元都对应于缓冲区中的一个像素，之所以不能直接称为__像素__是因为3D空间中的物体是可以相互遮挡的，而一个3D场景最终显示到屏幕上虽然是一个整体，但每个3D物体的每个图元是独立处理的。这就可能出现这样的情况，系统先处理的位于离观察点较远的图元，其光栅化成为了一组片元，暂时送入缓冲区中的对应位置。
但后面继续处理离观察点较近的图元时也光栅化了一组片元，两组片元中又对应到缓冲区中同一个位置的，这时距离近的片元将覆盖距离远的片元(*如何覆盖的检测是在深度检测阶段完成*)，因此某片元就不一定能够成为屏幕上的像素，称之为像素就不准确，可以将其理解为候选像素。

6. ### 纹理环境和颜色求和
该阶段主要有两个任务：
    * 纹理采样任务
    >根据当前需处理片元的纹理坐标及采用的纹理id对应的纹理图进行纹理采样，获取采样值。

    * 颜色求和
    >执行颜色的变化，根据纹理采样及光照计算等的结果综合生成需要处理片元的颜色。

7. ### 雾
根据程序中设置的雾的相关参数，如：颜色、浓度、范围等，以及某种雾的数学模型来计算当前处理的片元受雾处理后的颜色。

8. ### Alpha测试
如果程序启用了Alpha测试，OpenGL ES会检查每个片元的Alpha值，只有Alpha只符合测试条件的片元才会送入下个阶段，不满足的片元则被丢弃。

9. ### 裁剪测试
如果程序启用了裁剪测试，OpenGL ES会检查每个片元在缓冲中对应的位置，若对应位置再裁剪窗口中则将此片元送入下个阶段，否则丢弃此片元。

10. ### 深度测试和模板测试
	* 深度测试
	>将输入片元的深度值和缓冲区中存储的对应位置偏的深度进行比较，若输入片元的深度小则将输入片元送入下一个阶段准备覆盖缓冲区中的原片元或与缓冲中的元片元混合，否则丢弃输入片元。

	* 模板测试
	>模板测试的主要功能为将绘制区域限定在一定的范围内，一般用在湖面倒影、镜像等场合。

11. ### 颜色缓冲混合
如果程序开启了Alpha混合，则根据混合因子将上一阶段送来的片元与缓冲中对应位置的片元进行Alpha混合；否则送入的片元将覆盖缓冲区对应位置的片元。

12. ### 抖动
抖动是一种简单的操作，其允许使用少量的颜色模拟出更宽的颜色显示范围从而使颜色视觉效果更丰富。例如，可以使用白色以及黑色模拟出一种过渡的灰色。
固有缺点：会损失一部分分辨率，因此对于现在主流的原生颜色就很丰富的显示器设别一般是不需要开启抖动的。

13. ### 帧缓冲
OpenGL ES中的物体绘制并不是直接在屏幕上进行的，而是预先在缓冲区域中进行绘制，每绘制完一帧再将绘制的结果交换到屏幕上。因此，在每次绘制新的一帧时需要清除缓冲区中的相关数据，否则有可能产生不正确的结果。
应对不同的需求，帧缓冲是由一套组件组成的，主要包括颜色缓冲、深度缓冲以及模板缓冲，各组件的具体用途如下：
    * 颜色缓冲
    >用于存储每个片元的颜色值，每个颜色值包括RGBA4个色彩通道，应用程序运行时在屏幕上看到的就是颜色缓冲中的内容。

    * 深度缓冲
    >用来存储每个片元的深度值，所谓深度值是指以特定的内部格式表示的从片元处到观察点的距离。

    * 模板缓冲
    >用来存储每个片元的模板值，供模板测试使用。*模板测试是几种测试中最为灵活和复杂的一种*。
