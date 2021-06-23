# More on Rendering
# 有关渲染的详细信息
建一个名为“Mesh”的新的类，它将用一个位置数组作为输入，创建该模型加载到显卡中所需的VBO和VAO对象。

```java
package org.lwjglb.engine.graph;

import java.nio.FloatBuffer; 调用浮动缓冲器
import org.lwjgl.system.MemoryUtil;
import static org.lwjgl.opengl.GL30.*;

public class Mesh {

    private final int vaoId;

    private final int vboId;

    private final int vertexCount;

    public Mesh(float[] positions) {
        FloatBuffer verticesBuffer = null;
        try {
            verticesBuffer = MemoryUtil.memAllocFloat(positions.length);
            vertexCount = positions.length / 3;
            verticesBuffer.put(positions).flip();

            vaoId = glGenVertexArrays();
            glBindVertexArray(vaoId);

            vboId = glGenBuffers();
            glBindBuffer(GL_ARRAY_BUFFER, vboId);
            glBufferData(GL_ARRAY_BUFFER, verticesBuffer, GL_STATIC_DRAW);            
            glEnableVertexAttribArray(0);
            glVertexAttribPointer(0, 3, GL_FLOAT, false, 0, 0);
            glBindBuffer(GL_ARRAY_BUFFER, 0);

            glBindVertexArray(0);         
        } finally {
            if (verticesBuffer  != null) {
                MemoryUtil.memFree(verticesBuffer);
            }
        }
    }

    public int getVaoId() {
        return vaoId;
    }

    public int getVertexCount() {
        return vertexCount;
    }

    public void cleanUp() {
        glDisableVertexAttribArray(0);

        // Delete the VBO 删除这个VBO
        glBindBuffer(GL_ARRAY_BUFFER, 0);
        glDeleteBuffers(vboId);

        // Delete the VAO 删除这个VAO
        glBindVertexArray(0);
        glDeleteVertexArrays(vaoId);
    }
}
```


我们将在`DummyGame`类中创建`Mesh`实例，从`Renderer``init`方法中删除VAO和VBO代码。`Renderer`类中的render方法也将接受要渲染的网格实例。`cleanup`方法也将得到简化，因为`Mesh`类已经提供了释放VAO和VBO资源的方法。

```java
public void render(Mesh mesh) {
    clear();

    if (window.isResized()) {
        glViewport(0, 0, window.getWidth(), window.getHeight());
        window.setResized(false);
    }

    shaderProgram.bind();

    // Draw the mesh
    glBindVertexArray(mesh.getVaoId());
    glDrawArrays(GL_TRIANGLES, 0, mesh.getVertexCount());

    // Restore state
    glBindVertexArray(0);

    shaderProgram.unbind();
}

public void cleanup() {
    if (shaderProgram != null) {
        shaderProgram.cleanup();
    }
}
```

One important thing to note is this line:
需要注意的一点是：

```java
glDrawArrays(GL_TRIANGLES, 0, mesh.getVertexCount());
```

.
我们的`Mesh`通过将位置数组长度除以 3 来计算顶点的数量。（因为我们传递的是X、Y和Z坐标）。现在我们可以渲染更复杂的形状，让我们尝试渲染更复杂的形状————渲染一个四边形！四边形可以通过使用两个三角形来构造，如下图所示

![Quad coordinates](quad_coordinates.png)


如你所见，两个三角形中的每一个，都是由三个顶点组成。第一个由顶点V1、V2和V4构成（橙色的），第二个由顶点V4、V2和V3构成（绿色的）。顶点按逆时针顺序指定，因此要传递的浮点数组将是\[V1，V2，V4，V4，V2，V3\]。因此，`DummyGame`类中的init方法将是：

`DummyGame` class will be: `DummyGame` 将要执行的操作

```java
@Override
public void init() throws Exception {
    renderer.init();
    float[] positions = new float[]{
        -0.5f,  0.5f, 0.0f,
        -0.5f, -0.5f, 0.0f,
         0.5f,  0.5f, 0.0f,
         0.5f,  0.5f, 0.0f,
        -0.5f, -0.5f, 0.0f,
         0.5f, -0.5f, 0.0f,
    };
    mesh = new Mesh(positions);
}
```


现在你应该看到这样渲染的四边形
![Quad rendered](quad_rendered.png)


结束了吗？不幸的是，没有。上面的代码仍然存在一些问题。我们在重复坐标来表示四边形。经过两次V2和V4坐标。有了这个小的外形，看起来没什么，但是想象一下一个更复杂的3D模型。我们会多次重复坐标。请记住，现在我们只是使用三个浮点数来表示顶点的位置。但稍后我们将需要更多的数据来表示纹理等。还要考虑到，在更复杂的形状中，三角形之间共享的顶点数可能会更多，如下图所示（其中一个顶点甚至可以在六个三角形之间共享）。

![Dolphin](dolphin.png)


最后，由于重复的信息，我们需要更多的内存，这就是索引缓冲区的作用所在。为了绘制四边形，我们需要这样指定每个顶点一次：V1，V2，V3，V4\)。每个顶点在数组中都有一个位置。V1有位置0，V2有位置1，等等：

| V1 | V2 | V3 | V4 |
| --- | --- | --- | --- |
| 0 | 1 | 2 | 3 |


然后，我们通过参考这些顶点的位置,来指定绘制这些顶点的顺序：

| 0 | 1 | 3 | 3 | 1 | 2 |
| --- | --- | --- | --- | --- | --- |
| V1 | V2 | V4 | V4 | V2 | V3 |


因此，我们需要修改`Mesh`类,以接受另一个参数 -- 一个索引数组，现在要绘制的顶点数将是该索引数组的数。
```java
public Mesh(float[] positions, int[] indices) {
    vertexCount = indices.length;
```


创建存储位置的VBO后，需要创建另一个保存索引的VBO。因此，我们需要重命名保存位置，和VBO标识符的标识符，并为索引VBO\（`idxVboId`\）创建一个新的标识符。这和创建VBO的过程类似，不过类型现在是`GL_ELEMENT_ARRAY_BUFFER`。

```java
idxVboId = glGenBuffers();
indicesBuffer = MemoryUtil.memAllocInt(indices.length);
indicesBuffer.put(indices).flip();
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, idxVboId);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, indicesBuffer, GL_STATIC_DRAW);
memFree(indicesBuffer);
```

.
因为我们要处理整数，所以需要创建一个`IntBuffer`(整数类型)而不是`FloatBuffer`（浮点类型）


这样。VAO现在将包含两个VBO，一个用于位置，另一个用于保存索引并用于渲染。`Mesh`类中的清理释放的方法还必须考虑到有另一个VBO要释放。

```java
public void cleanUp() {
    glDisableVertexAttribArray(0);

    // Delete the VBOs
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glDeleteBuffers(posVboId);
    glDeleteBuffers(idxVboId);

    // Delete the VAO
    glBindVertexArray(0);
    glDeleteVertexArrays(vaoId);
}
```


最后，我们需要修改，使用`glDrawArrays`方法的绘图调用：

```java
glDrawArrays(GL_TRIANGLES, 0, mesh.getVertexCount());
```

To another call that uses the method `glDrawElements`:

```java
glDrawElements(GL_TRIANGLES, mesh.getVertexCount(), GL_UNSIGNED_INT, 0);
```


该方法的参数为：


* 模式：指定用于渲染的基本体，在本示例中为三角形。这里没有变化。

* 计数：指定要呈现的元素的数量。

* 类型：指定索引数据中的值的类型。在这种情况下，我们使用整数。

* 索引：指定一个offset量并应用在索引数据上才能开始转换。


 现在，我们可以通过指定索引，来使用更新的，更有效的方法来绘制复杂的模型。

```java
public void init() throws Exception {
    renderer.init();
    float[] positions = new float[]{
        -0.5f,  0.5f, 0.0f,
        -0.5f, -0.5f, 0.0f,
         0.5f, -0.5f, 0.0f,
         0.5f,  0.5f, 0.0f,
    };
    int[] indices = new int[]{
        0, 1, 3, 3, 1, 2,
    };
    mesh = new Mesh(positions, indices);
}
```

现在，让我们为我们的示例添加一些颜色。我们将把另一个浮点数数组传递给我们的`Mesh`类，该类保存示例四边形中的每个坐标的颜色。

```java
public Mesh(float[] positions, float[] colours, int[] indices) {
```


使用该数组，我们将创建另一个VBO，该VBO将与我们的VAO关联。

```java
// Colour VBO
colourVboId = glGenBuffers();
FloatBuffer colourBuffer = memAllocFloat(colours.length);
colourBuffer.put(colours).flip();
glBindBuffer(GL_ARRAY_BUFFER, colourVboId);
glBufferData(GL_ARRAY_BUFFER, colourBuffer, GL_STATIC_DRAW);
glEnableVertexAttribArray(1);
glVertexAttribPointer(1, 3, GL_FLOAT, false, 0, 0);
```

P
请注意，在`glVertexAttribPointer`的调用中，第一个参数现在是`“1”`。这是我们的着色器期望的该数据的位置\(当然，由于我们有另一个VBO，我们需要在`cleanup`方法中释放它。您可以看到，我们需要在渲染期间启用位置`“1”`处的VAO属性。


下一步是修改着色器。顶点着色器现在需要两个参数，坐标\(位置 0\)和颜色\(位置 1\)。顶点着色器将只输出接收到的颜色，以便片段着色器可以对其进行处理。

```glsl
#version 330

layout (location =0) in vec3 position;
layout (location =1) in vec3 inColour;

out vec3 exColour;

void main()
{
    gl_Position = vec4(position, 1.0);
      exColour = inColour;
}
```


现在，我们的片段着色器接收顶点着色器处理的颜色作为输入，并使用它来生成颜色。

```glsl
#version 330

in  vec3 exColour;
out vec4 fragColor;

void main()
{
    fragColor = vec4(exColour, 1.0);
}
```

 
我们现在可以将这样的颜色数组传递给`Mesh`类，以便为我们的四边形添加一些颜色。

```java
float[] colours = new float[]{
    0.5f, 0.0f, 0.0f,
    0.0f, 0.5f, 0.0f,
    0.0f, 0.0f, 0.5f,
    0.0f, 0.5f, 0.5f,
};
```


然后我们会得到一个像这样的花式彩色四边形！：

![Coloured quad](coloured_quad.png)

