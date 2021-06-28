# Rendering


在本章中,我们将学习使用OpenGL渲染场景时发生的过程和三维图形基本渲染过程.如果你习惯于旧版本的OpenGL,也就是固定的函数管道,你可能会在本章结束时想知道为什么它需要如此复杂.你可能会认为,只是在屏幕上画一个简单的形状完全不需要那么多的概念和代码.
让我给你们这些有这种想法的人一个建议。它实际上更简单、更灵活。你只需要给它一个机会：现代的OpenGL让你一次只思考一个问题,它让你以一种更有逻辑的方式组织你的代码和进程.


将一个三维表示绘制到二维屏幕的一系列步骤称为图形管道.OpenGL的第一个版本使用了一个称为固定函数管道的模型.该模型在渲染过程中采用了一组步骤,定义了一组固定的操作,程序员被限制在每个步骤可用的一组函数中.因此,可以应用的效果和操作受到API本身的限制\(例如，“set fog”或“add light”，但这些函数的实现是固定的，不能更改\).



图形管道由以下步骤组成：


![Graphics Pipeline](./rendering_pipeline.png)

但在OPENGL 2.0版本,其引入了可编程流水线渲染的概念.在这个模型中,组成图形管道的不同步骤可以通过一组称为“着色器”的特定程序来控制或编程.(可编程自定义渲染过程,而不是只能使用固定设置)下图描述了OpenGL可编程管道的简化版本:

![Programmable pipeline](./rendering_pipeline_2.png)


渲染开始将顶点缓冲区形式的顶点列表作为其输入.但,什么是顶点?顶点是描述二维或三维空间中的点的数据结构.你该如何描述三维空间中的一个点?————通过指定其x、y和z坐标.那什么是顶点缓冲区?顶点缓冲区是另一种数据结构,它通过使用顶点数组,打包需要渲染的所有顶点,并使这些信息可供图形管道中的着色器使用.



顶点着色器会处理这些顶点,其主要目的是计算每个顶点在屏幕空间中的投影位置.该着色器还可以生成与颜色或纹理相关的其他输出,但其主要目标是将顶点投影到屏幕空间,即生成点.



几何体的处理阶段:连接顶点着色器变换的顶点以形成三角形.它是通过判断顶点的存储顺序并使用不同的模型对它们进行分组来实现的.那为什么是三角形呢?三角形类似于图形卡的基本工作单元.它是一个简单的几何形状,可以组合和转换来构建复杂的三维场景.此阶段还可以使用特定着色器对顶点进行分组.



光栅化阶段:获取前一阶段生成的三角形,对其进行剪辑并将其转换为像素大小的片段.



片段着色器在片段处理阶段会使用这些片段来生成像素,为它们指定写入帧缓冲区的最终颜色.帧缓冲区是图形管道的最终结果.它保存应绘制到屏幕上的每一个像素的值.



请记住,3D加速卡专门设计来用于并行上述所有操作.为了生成最终场景,可以并行处理输入数据.(非常多的线程共同计算,参考英伟达的CUDA芯片,几千个核心共同处理,这是专为渲染图形所设计的硬件方案)


让我们开始编写我们的第一个着色器程序.着色器是使用基于ansic的GLSL语言\(OpenGL着色语言\)编写的.首先,我们将在resources目录下创建一个名为“`vertex.vs`”的文件\(扩展名为vertex Shader\),其内容如下:

```
#version 330

layout (location=0) in vec3 position;

void main()
{
    gl_Position = vec4(position, 1.0);
}
```



第一行是一个指令,说明了我们使用的GLSL语言的版本.下表则说明了GLSL版本、和与该版本匹配的OpenGL以及要使用的指令.\(维基百科: [https://en.wikipedia.org/wiki/OpenGL\_Shading\_Language\#Versions](https://en.wikipedia.org/wiki/OpenGL_Shading_Language#Versions)\).

| GLS版本 | OpenGL 版本 | 着色器预处理器 |
| --- | --- | --- |
| 1.10.59 | 2.0 | \#version 110 |
| 1.20.8 | 2.1 | \#version 120 |
| 1.30.10 | 3.0 | \#version 130 |
| 1.40.08 | 3.1 | \#version 140 |
| 1.50.11 | 3.2 | \#version 150 |
| 3.30.6 | 3.3 | \#version 330 |
| 4.00.9 | 4.0 | \#version 400 |
| 4.10.6 | 4.1 | \#version 410 |
| 4.20.11 | 4.2 | \#version 420 |
| 4.30.8 | 4.3 | \#version 430 |
| 4.40 | 4.4 | \#version 440 |
| 4.50 | 4.5 | \#version 450 |


第二行用来指定此着色器的输入格式.OpenGL缓冲区中的数据都可以是我们想要的任何数据,也就是说,该语言不会强制你传递具有预定义语义的特定数据结构.从着色器的角度来看,它希望接收一个包含数据的缓冲区.它可以是一个位置,一个有额外信息的位置,或者我们想要的任何位置.当顶点着色器刚刚接收一个浮点数组,填充缓冲区时,定义将由着色器处理的缓冲区块来定义.



所以,首先,我们需要把这些转化成对我们有意义的东西.在本例中,从位置0开始,我们期望接收一个由3个参数组成的向量坐标\(x, y, z\).



着色器有一个主块,就像任何其他的C语言程序一样,在本案例中非常简单,它要做的只是返回输出变量`gl_Position`中接收到的位置,而不应用任何转换.现在你可能想知道为什么三个属性的向量会被转换成四个属性的向量,这到底是怎么实现的\(vec4\)？这是因为`gl_Position`使用的是齐次坐标格式,因此预期结果为vec4格式.也就是说,它需要某种形式的四个变量\(x，y，z，w\)s,其中w表示一个额外的维度.那为什么要添加另一个维度呢?在后面的章节中,你将看到我们需要做的大多数操作都是基于向量和矩阵的.如果没有那个额外的维度,这些操作中的一些就不能合并.例如,如果没有这个额外的维度,将不能结合旋转和平移操作.\(如果你想了解更多这个额外的维度，以允许结合仿射和线性变换。你可以通过阅读Fletcher Dunn和Ian Parberry大佬的优秀著作《3D Math Primer for Graphics and Game Development\)以了解更多信息.



现在让我们看看我们的第一个片段着色器.我们将在resources目录下创建一个名为“`fragment.fs`”的方法\(扩展名是 Fragment Shader\)位于resources目录下,包含以下内容:

```
#version 330

out vec4 fragColor;

void main()
{
    fragColor = vec4(0.0, 0.5, 0.5, 1.0);
}
```


其结构与我们的顶点着色器非常相似,在这种情况下,我们将为每个片段设置一个固定的颜色,输出变量在第二行中,定义并设置为vec4 fragColor(花色).
既然已经创建了着色器,那么该如何使用它们呢》?我们需要遵循以下步骤:
 
1.    创建一个OpenGL程序.
2.    加载顶点和片段着色器代码文件. 
3.    为每个着色器创建一个新的着色器程序并指定其类型\(顶点,片段\)
4.    编译着色器. 
5.    将着色器附加到程序中.
6.    链接程序.

最后,着色器将加载到图形卡中.我们可以通过引用一个标识符(程序标识符)来使用它.


```java
package org.lwjglb.engine.graph;

import static org.lwjgl.opengl.GL20.*;

public class ShaderProgram {

    private final int programId;

    private int vertexShaderId;

    private int fragmentShaderId;

    public ShaderProgram() throws Exception {
        programId = glCreateProgram();
        if (programId == 0) {
            throw new Exception("Could not create Shader");
        }
    }

    public void createVertexShader(String shaderCode) throws Exception {
        vertexShaderId = createShader(shaderCode, GL_VERTEX_SHADER);
    }

    public void createFragmentShader(String shaderCode) throws Exception {
        fragmentShaderId = createShader(shaderCode, GL_FRAGMENT_SHADER);
    }

    protected int createShader(String shaderCode, int shaderType) throws Exception {
        int shaderId = glCreateShader(shaderType);
        if (shaderId == 0) {
            throw new Exception("Error creating shader. Type: " + shaderType);
        }

        glShaderSource(shaderId, shaderCode);
        glCompileShader(shaderId);

        if (glGetShaderi(shaderId, GL_COMPILE_STATUS) == 0) {
            throw new Exception("Error compiling Shader code: " + glGetShaderInfoLog(shaderId, 1024));
        }

        glAttachShader(programId, shaderId);

        return shaderId;
    }

    public void link() throws Exception {
        glLinkProgram(programId);
        if (glGetProgrami(programId, GL_LINK_STATUS) == 0) {
            throw new Exception("Error linking Shader code: " + glGetProgramInfoLog(programId, 1024));
        }

        if (vertexShaderId != 0) {
            glDetachShader(programId, vertexShaderId);
        }
        if (fragmentShaderId != 0) {
            glDetachShader(programId, fragmentShaderId);
        }

        glValidateProgram(programId);
        if (glGetProgrami(programId, GL_VALIDATE_STATUS) == 0) {
            System.err.println("Warning validating Shader code: " + glGetProgramInfoLog(programId, 1024));
        }

    }

    public void bind() {
        glUseProgram(programId);
    }

    public void unbind() {
        glUseProgram(0);
    }

    public void cleanup() {
        unbind();
        if (programId != 0) {
            glDeleteProgram(programId);
        }
    }
}
```

这个叫`ShaderProgram`的构造函数在OpenGL中创建了一个新程序,并提供了添加顶点和片段着色器的方法.这些着色器被编译并附加到OpenGL程序.当附加所有着色器时,应该调用link方法,该方法负责链接所有代码,并验证所有操作是否正确.


当链接着色器程序后,可以释放已编译的顶点和片段着色器\(通过调用 `glDetachShader`\).


此方法主要用于调试目的,当游戏进入生产阶段时应将其删除.此方法尝试验证给定**当前OpenGL状态**的着色器是否正确.这意味着,在某些情况下,即使着色器是正确的,验证也可能失败,因为当前状态不够完整,无法运行着色器\(某些数据可能尚未上载\),因此,我们只需将错误消息打印到标准错误输出就好,而不是认为运行失败.


`ShaderProgram`还提供了一些方法来激活此程序以渲染\(bind\),和停止使用它\(unbind\).最后,它提供了一种清理方法,在不再需要资源时释放所有资源.


既然我们有一个cleanup方法,那就让我们更改`IGameLogic`接口类以添加一个cleanup方法:

```java
void cleanup();
```


此方法将在游戏循环完成时调用,因此我们需要修改`GameEngine`类的run方法:

```java
@Override
public void run() {
    try {
        init();
        gameLoop();
    } catch (Exception excp) {
        excp.printStackTrace();
    } finally {
        cleanup();
    }
}

protected void cleanup() {
    gameLogic.cleanup();                
}
```

现在我们可以使用着色器来显示三角形了.我们将在`Renderer`类的`init`方法中执行此操作.首先,我们创建着色器程序:

```java
private ShaderProgram shaderProgram;

public void init() throws Exception {
    shaderProgram = new ShaderProgram();
    shaderProgram.createVertexShader(Utils.loadResource("/vertex.vs"));
    shaderProgram.createFragmentShader(Utils.loadResource("/fragment.fs"));
    shaderProgram.link();
}
```


我们已经创建了一个实用程序类,现在它提供了一个从类路径检索文件内容的方法,此方法用于检索着色器的内容.


现在我们可以将三角形定义为一个浮点数数组了,所以我们创建一个浮点数数组来定义三角形的顶点.如你所见,数组中还没有结构.现在,OpenGL无法知道这些数据的结构.这看起来只是一系列的浮点数:

```java
float[] vertices = new float[]{
     0.0f,  0.5f, 0.0f,
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f
};
```


下图描绘了我们坐标系中的三角形.

![Triangle](./triangle_coordinates.png)


现在我们有了坐标,我们需要将它们存储到GPU中,并告诉OpenGL数据的结构.我们现在将介绍两个重要的概念,顶点数组对象\(VAOs\)和顶点缓冲区对象\(VBOs\).如果你在接下来的代码片段中迷失了方向,请记住,最后我们要做的是将要绘制的对象的模型数据发送到GPU的内存中.当我们存储它时,我们会得到一个标识符,以便稍后在绘图时引用它.


让我们首先从顶点缓冲区对象\(VBOs\)开始.VBO只是存储在图形卡内存中的内存缓冲区,用于存储顶点.我们将在这里传递一组模拟三角形的浮点数.如前所述,OpenGL对我们的数据结构一无所知.事实上,它不仅可以保存坐标,还可以保存其他信息,如纹理、颜色等.
顶点数组对象\(VAOs\)是包含一个或多个VBO(通常称为属性列表)的对象.每个属性列表可以保存一种类型的数据:位置、颜色、纹理等,这样你就可以可以随意在每个插槽中存储任何你想要的数据.


VAO就像一个包装器,它为将要存储在图形卡中的数据分组为一组一组的定义,当我们创建一个VAO时,我们得到一个标识符.我们使用该标识符来呈现它和它包含的元素,使用我们在创建期间指定的定义.


让我们继续编写我们的示例.我们必须做的第一件事是将浮点数组存储到`FloatBuffer`中.这主要是因为我们必须与基于C语言的OpenGL库接口,所以我们必须将我们的float数组转换成库可以管理的东西.

```java
FloatBuffer verticesBuffer = MemoryUtil.memAllocFloat(vertices.length);
verticesBuffer.put(vertices).flip();
```


我们使用`MemoryUtil`类在堆外内存中创建缓冲区,以便OpenGL库可以访问它.在我们存储了数据\(使用put方法\)之后,我们还需要使用flip方法\(也就是说,我们已经完成了对它的写入\)将缓冲区的位置重置为0位置.请记住,Java对象是在称为堆的空间中分配的.堆是JVM进程内存中保留的大量内存空间.存储在堆中的内存不能被本机代码访问\(JNI,允许从Java调用本机代码的机制不允许这样\).在Java和本机代码之间共享内存数据的唯一方法是直接在Java中分配内存.



如果你是LWJGL的早期版本使用者,那么强调几个主题是很重要的.你可能已经注意到,我们没有使用实用程序类`BufferUtils`来创建缓冲区.相反,我们使用`MemoryUtil`类.这是由于`BufferUtils`的效率不高,而且只是为向后兼容而设计的.相反,LWJGL 3则提出了两种缓冲区管理方法:


* 自动管理的缓冲区,即由垃圾收集器自动收集的缓冲区.这些缓冲区主要用于短期操作,或用于传输到GPU且不需要存在于进程内存中的数据.这是通过使用`org.lwjgl.system.MemoryStack`类实现的.


* 我们可以手动管理缓冲区.在这种情况下,我们需要小心地释放他们,一旦我们完成了.这些缓冲区将用于长时间操作或大量数据.这是通过使用`MemoryUtil`类实现的.


你可以在此处查阅详细信息:[https://blog.lwjgl.org/memory-management-in-lwjgl-3/](https://blog.lwjgl.org/memory-management-in-lwjgl-3/ "这里").


在这种情况下，我们的数据将会被发送到GPU，这样我们就可以使用自动管理的缓冲区.但是,由于以后我们将使用它们来保存潜在的大量数据,因此我们需要手动管理它们.这就是我们使用`MemoryUtil`类的原因,也是我们释放finally块中缓冲区的原因.在下一章中,我们将学习如何使用自动管理的缓冲区.


现在我们需要创建VAO并绑定它.

```java
vaoId = glGenVertexArrays();
glBindVertexArray(vaoId);
```

然后我们需要创建VBO,绑定它并将数据放入其中.

```java
vboId = glGenBuffers();
glBindBuffer(GL_ARRAY_BUFFER, vboId);
glBufferData(GL_ARRAY_BUFFER, verticesBuffer, GL_STATIC_DRAW);
glEnableVertexAttribArray(0);
```

现在是最重要的部分.我们需要定义数据的结构并将其存储在VAO的一个属性列表中,这是通过以下代码完成的.

```java
glVertexAttribPointer(0, 3, GL_FLOAT, false, 0, 0);
```

参数包括:

* 索引:指定着色器需要的数据的位置.
* 大小:指定每个顶点的组件数属性\(from 1 to 4\).在这个例子中,我们传递的是三维坐标,所以它应该是3.
* 类型:指定数组中每个组件的类型,在本例中为浮点数.
* 格式化:指定值是否应格式化.
* 跨距:指定连续常规顶点属性之间的字节偏移量\(我们稍后再解释\).
* 偏移:指定缓冲区中第一个组件的偏移量.

完成VBO后,我们可以解除它和VAO的绑定\(将它们绑定到0\)

```java
// 解除VBO绑定
glBindBuffer(GL_ARRAY_BUFFER, 0);

// 关闭VAO
glBindVertexArray(0);
```


完成后,我们**必须**释放FloatBuffer分配的堆外内存.这是通过手动调用memFree来完成的,因为Java垃圾收集不会清除堆外分配.

```
if (verticesBuffer != null) {
    MemoryUtil.memFree(verticesBuffer);
}
```


这就是我们的`init`方法中应该包含的所有代码.我们的数据已经在显卡中,可以使用了.我们只需要修改`render`方法,在游戏循环的每个渲染步骤中使用它.

```java
public void render(Window window) {
    clear();

    if (window.isResized()) {
        glViewport(0, 0, window.getWidth(), window.getHeight());
        window.setResized(false);
    }

    shaderProgram.bind();

    // 绑定到VAO
    glBindVertexArray(vaoId);

    // 绘制顶点
    glDrawArrays(GL_TRIANGLES, 0, 3);

    // 还原状态
    glBindVertexArray(0);

    shaderProgram.unbind();
}
```

如你所见,我们只需清除窗口、绑定着色器程序、绑定VAO、绘制与VAO关联的VBO中存储的顶点并恢复状态,就这样.

我们还向`Renderer`类添加了一个cleanup方法,该方法会释放获取了的资源.

```java
public void cleanup() {
    if (shaderProgram != null) {
        shaderProgram.cleanup();
    }

    glDisableVertexAttribArray(0);

    // 删除VBO
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glDeleteBuffers(vboId);

    // 删除VAO
    glBindVertexArray(0);
    glDeleteVertexArrays(vaoId);
}
```

这就是所有的操作!如果你仔细地遵循这些步骤,你将会看到以下内容:

![Triangle game](./triangle_window.png)


这是我们的第一个三角形!你可能会认为这还没法变成一个很好的游戏....你是正确的.你也可能认为这些工作量太大,如此繁琐的操作,只是为了绘制一个无聊的三角形.但请记住,我们正在引入关键的概念,并准备基础基础架构来完成更复杂的任务,所以请耐心阅读下去吧;)
