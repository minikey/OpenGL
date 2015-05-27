OpenGL ES 2.0的渲染管线
===========
OpenGL ES 2.0与1.x渲染管线还是存在许多的相似性，只是OpenGL ES 2.0管线中有一部分已经改变：

>* __顶点着色器__取代了1.x中渲染管线中“变换和光照”阶段，这是得开发3D场景时对顶点的变换、法向量的计算、纹理坐标的变换、光照与材质的应用等均由开发者使用着色器代码完成。
* __片元着色器__取代了1.x中“纹理环境和颜色求和”、“雾”、“Alpha测试”等阶段，这使得纹理处理、颜色求和和以及雾效果均由开发者自行开发。

<img src="https://github.com/minikey/OpenGL/raw/master/images/es2.0.png">

1. ### 顶点着色器

    顶点着色器是一个可编程的处理单元，功能为执行顶点的变换、光照、材质的应用于计算等顶点的相关操作，其每顶点执行一次。工作过程为：首先将原始的顶点集合信息以及其他属性传送至顶点着色器中，经过自己开发的顶点着色器处理后产生的纹理坐标、颜色、点位置后继续流程需要的各项顶点属性信息，然后将其传递给图元装配阶段。
    
    * 顶点着色器的输入主要为待处理顶点的__attribute(属性)变量__、__uniform(全局)变量__、__采样器__以及__临时变量__，输出主要为经过顶点着色器后生成的__varying(易变)变量__以及一些__内建输出变量__。
    *  attribute变量指的是3D物体中每个额顶点各自不同的信息所属的变量，一般顶点的__位置__、__颜色__、__法向量__等每个顶点各自不同的信息都是以attribute变量的方式传入顶点着色器中。例如：“初识OpenGL ES 2.0应用程序”中顶点着色器中的aPosition(顶点位置)和aColor(顶点颜色)变量等。
    *  uniform变量指的是对于同一组顶点组成的单个3D物体中所有的顶点都有相同的量，一般为场景中当前的光源位置、当前的摄像机位置、投影系列矩阵等。例如：“初识OpenGL ES 2.0应用程序”中顶点着色器中的uMVPMatrix(总变换矩阵)变量等。
    *  varying变量是从顶点着色器计算产生并传递到片元着色器的数据变量。顶点着色器可以使用易变变量来传递需要插值到片元的*颜色*、*法向量*、*纹理坐标*等任意值。例如，程序中提到的aaColor变量。
    *  内键输出变量gl\_Position、gl\_FrontFacing和gl_PointSize等。gl\_Position是经过变换矩阵变换、投影后的顶点的最终位置，gl\_FrontFacing指的是片元所在面的朝向，gl\_PointSize指的是点的大小。

    易变量在顶点着色器赋值后并不是直接将赋的值送入到后继的片元着色器中，而是在光栅化阶段由管线根据片元所属图元各个顶点对应的顶点着色器对此易变量的赋值情况以及片元与各顶点的位置关系插值产生。
    
<img src="https://github.com/minikey/OpenGL/raw/master/images/vertex%20shader.png">

2. ### 片元着色器

    片元着色器适用于处理片元值及其相关数据可编程单元，其可以执行__纹理的采样__、__颜色的汇总__、__计算雾颜色__等操作，每片元执行一次。片元着色器主要功能为通过重复执行，将3D物体中的图元光栅化后产生的每个片元的颜色等属性计算出来送入后继阶段，如裁剪测试、深度测试以及模板测试等。

    * Varying0~n指的是从顶点着色器传递到片元着色器的易变数据变量，如前面所介绍，由系统在顶点着色器后的光栅化阶段自动插值产生。其个数是不一定的，取决于具体的需求。
    * fl\_FragColor指的是计算后此片元的颜色。一般在片元着色器中的最后都需要对gl\_FragColor进行赋值。例如，前面提到的三角形案例的片元着色器就有“gl_FragColor=aaColor”语句。
    
<img src="https://github.com/minikey/OpenGL/raw/master/images/fragment%20shader.png">

frag.sh
```
precision mediump float;
varying  vec4 vColor; //接收从顶点着色器过来的参数

void main()                         
{                       
   gl_FragColor = vColor;//给此片元颜色值
}
```

vertex.sh
```
uniform mat4 uMVPMatrix; //总变换矩阵
attribute vec3 aPosition;  //顶点位置
attribute vec4 aColor;    //顶点颜色
varying  vec4 vColor;  //用于传递给片元着色器的变量

void main()     
{                                   
   gl_Position = uMVPMatrix * vec4(aPosition,1); //根据总变换矩阵计算此次绘制此顶点位置
   vColor = aColor;//将接收的颜色传递给片元着色器 
}                      
```
