# 材质

## 创造一个 3D 立方体

在这一章中我们将会学习怎样在渲染过程中加载与使用材质. 为了展示所有的与材质有关联的概念, 我们将会把我们在之前的章节中使用的四方形变换成一个 3D 立方体. 通过我们构建的基本代码, 我们得以正确地定义一个立方体的坐标, 然后他就会正确地被画出来了.

要画一个立方体, 只需定义 8 个定点.

![Cube coords](cube_coords.png)

所以这一互相关联的坐标数组应该像下面这样:

```java
float[] positions = new float[] {
    // VO
    -0.5f,  0.5f,  0.5f,
    // V1
    -0.5f, -0.5f,  0.5f,
    // V2
    0.5f, -0.5f,  0.5f,
    // V3
     0.5f,  0.5f,  0.5f,
    // V4
    -0.5f,  0.5f, -0.5f,
    // V5
     0.5f,  0.5f, -0.5f,
    // V6
    -0.5f, -0.5f, -0.5f,
    // V7
     0.5f, -0.5f, -0.5f,
};
```

当然了, 既然我们已经有 4 个以上的顶点了, 我们就需要更新那些颜色数组. 现在我们只要把先前的四个项目重复一遍就行了.

```java
float[] colours = new float[]{
    0.5f, 0.0f, 0.0f,
    0.0f, 0.5f, 0.0f,
    0.0f, 0.0f, 0.5f,
    0.0f, 0.5f, 0.5f,
    0.5f, 0.0f, 0.0f,
    0.0f, 0.5f, 0.0f,
    0.0f, 0.0f, 0.5f,
    0.0f, 0.5f, 0.5f,
};
```

最后, 因为一个立方体是由六个面组成的, 这意味着我们需要画上 12 个三角形 (每面 2 个). 所以我们需要更新指数数组. 请记住, 这些三角形应该按照逆时针顺序定义. 如果你人工操作的话, 很容易出错. 请始终把你想定义索引的面放在面前. 然后, 确定顶点并按逆时针顺序绘制三角形即可.

```java
int[] indices = new int[] {
    // Front face
    0, 1, 3, 3, 1, 2,
    // Top Face
    4, 0, 3, 5, 4, 3,
    // Right face
    3, 2, 7, 5, 3, 7,
    // Left face
    6, 1, 0, 6, 0, 4,
    // Bottom face
    2, 1, 6, 2, 6, 7,
    // Back face
    7, 6, 4, 7, 4, 5,
};
```

为了更好地查看这个立方体, 我们把 `DummyGame` 类中的旋转模型的代码改成沿三个轴旋转的.

```java
// Update rotation angle
float rotation = gameItem.getRotation().x + 1.5f;
if ( rotation > 360 ) {
    rotation = 0;
}
gameItem.setRotation(rotation, rotation, rotation);
```

这就是全部了. 我们现在得以显示一个旋转的 3D 立方体了. 你现在可以编译运行你的项目, 然后你就会看到像这样旋转着的物体.

![Cube with no depth tests](cube_no_depth_test.png)

但是这个立方体看起来有点诡异, 一些面并未正确地绘制. 发生了什么? 实际上, 之所以会有这种情况, 是因为组成这个立方体的三角形是按某种随机顺序绘制的. 较远的像素点应该在较近的像素点之前绘制. 现在这不会发生, 除非我们进一步测试下去.

这可以在 `init` 方法结尾的 `Window` 类中实现.

```java
glEnable(GL_DEPTH_TEST);
```

我们的立方体现在被成功地渲染了!

![Cube with depth test](cube_depth_test.png)

如果你已经看过了这章的源代码, 你可能会发现我们已经在 `Mesh` 类中做了一个小小的重组. VBO 的标识符现在被存储在一个列表中, 以便我们轻松地遍历它们.

## 向立方体添加材质

现在,我们将要向立方体添加材质. 材质是一种绘制针对一个特定的模型的像素点的颜色的图像. 你可以吧材质看作是围绕着你的 3D 模型的皮肤. 你只需要将图像中的点指定给你的模型的顶点. 根据这些信息, OpenGL 就能够计算颜色, 并以材质图像作为基底来绘制像素点的图像.


![Texture mapping](texture_mapping.png)

The texture image does not have to be the same size as the model. It can be larger or smaller. OpenGL will extrapolate the colour if the pixel to be processed cannot be mapped to a specific point in the texture. You can control how this process is done when a specific texture is created.

So basically what we must do, in order to apply a texture to a model, is assigning texture coordinates to each of our vertices. The texture coordinate system is a bit different from the coordinate system of our model. First of all, we have a 2D texture so our coordinates will only have two components, x and y. Besides that, the origin is setup in the top left corner of the image and the maximum value of the x or y value is 1.

![Texture coordinates](texture_coordinates.png)

How do we relate texture coordinates with our position coordinates? Easy, in the same way we passed the colour information. We set up a VBO which will have a texture coordinate for each vertex position.

So let’s start modifying the code base to use textures in our 3D cube. The first step is to load the image that will be used as a texture. For this task, in previous versions of LWJGL, the Slick2D library was commonly used. At the moment of this writing it seems that this library is not compatible with LWJGL 3 so we will need to follow another approach. We will use the LWJGL wrapper for the [stb](https://github.com/nothings/stb) library. In order to do that,  we need first to declare that dependency, including the natives in our `pom.xml` file.

```xml
<dependency>
    <groupId>org.lwjgl</groupId>
    <artifactId>lwjgl-stb</artifactId>
    <version>${lwjgl.version}</version>
</dependency>
[...]
<dependency>
    <groupId>org.lwjgl</groupId>
    <artifactId>lwjgl-stb</artifactId>
    <version>${lwjgl.version}</version>
    <classifier>${native.target}</classifier>
    <scope>runtime</scope>
</dependency>
```

One thing that you may see in some web pages is that the first thing we must do is enable the textures in our OpenGL context by calling `glEnable(GL_TEXTURE_2D)`. This is true if you are using the fixed-function pipeline. Since we are using GLSL shaders it is not required anymore.

Now we will create a new `Texture` class that will perform all the necessary steps to load a texture. First we need to load the image data into a `ByteBuffer`. The code is defined as this:

```java
private static int loadTexture(String fileName) throws Exception {
    int width;
    int height;
    ByteBuffer buf;
    // Load Texture file
    try (MemoryStack stack = MemoryStack.stackPush()) {
        IntBuffer w = stack.mallocInt(1);
        IntBuffer h = stack.mallocInt(1);
        IntBuffer channels = stack.mallocInt(1);

        buf = stbi_load(fileName, w, h, channels, 4);
        if (buf == null) {
            throw new Exception("Image file [" + fileName  + "] not loaded: " + stbi_failure_reason());
        }
    
        /* Get width and height of image */
        width = w.get();
        height = h.get();
     }
	 [... More next ....]
```
The first thing we do is to allocate `IntBuffer`s for the library to return the image size and number of channels. Then, we call the `stbi_load` method to actually load the image into a `ByteBuffer`. This method requires the following parameters:

* `filePath`: The absolute path to the file. The stb library is native and does not understand anything about `CLASSPATH`. Therefore, we will be using regular file system paths.
* `width`:  Image width. This will be populated with the image width.
* `height`: Image height. This will be populated with the image height.
* `channels`: The image channels.
* `desired_channels`: The desired image channels. We pass 4 (RGBA).

One important thing to remember is that OpenGL, for historical reasons, requires that texture images have a size \(number of texels in each dimension\) of a power of two \(2, 4, 8, 16, ....\). Some drivers remove that constraint but it’s better to stick to it to avoid problems.

The next step is to upload the texture into the graphics card memory. First of all we need to create a new texture identifier. Each operation related to that texture will use that identifier so we need to bind it.

```java
// Create a new OpenGL texture 
int textureId = glGenTextures();
// Bind the texture
glBindTexture(GL_TEXTURE_2D, textureId);
```

Then we need to tell OpenGL how to unpack our RGBA bytes. Since each component is one byte in size we need to add the following line:

```java
glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
```

And finally we can upload our texture data:

```java
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height,
    0, GL_RGBA, GL_UNSIGNED_BYTE, buf);
```

The `glTexImage2D` method has the following parameters:

* `target`: Specifies the target texture \(its type\). In this case: `GL_TEXTURE_2D`. 
* `level`: Specifies the level-of-detail number. Level 0 is the base image level. Level n is the nth mipmap reduction image. More on this later.
* `internal format`: Specifies the number of colour components in the texture.
* `width`: Specifies the width of the texture image.
* `height`: Specifies the height of the texture image.
* `border`: This value must be zero.
* `format`: Specifies the format of the pixel data: RGBA in this case.
* `type`: Specifies the data type of the pixel data. We are using unsigned bytes for this.
* `data`: The buffer that stores our data.

In some code snippets that you may find, you will probably see that filtering parameters are set up before calling the `glTexImage2D` method. Filtering refers to how the image will be drawn when scaling and how pixels will be interpolated. If filtering parameters are not set the texture will not be displayed. So before the `glTexImage2D` method you could see something like this:

```java
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
```

These parameters basically say that when a pixel is drawn with no direct one to one association to a texture coordinate it will pick the nearest texture coordinate point.

By this moment we will not set up filtering parameters. Instead we will generate a mipmap. A mipmap is a decreasing resolution set of images generated from a high detailed texture. These lower resolution images will be used automatically when our object is scaled.

In order to generate mipmaps we just need to set the following line \(in this case after the `glTexImage2D` method\):

```java
glGenerateMipmap(GL_TEXTURE_2D);
```

Finally we can free the memory for the raw image data itself:

```
stbi_image_free(buf);
```

And that’s all, we have successfully loaded our texture. Now we need to use it. As we said before we need to pass texture coordinates as another VBO. So we will modify our `Mesh` class to accept an array of floats that contains texture coordinates instead of colours \(we could have both but in order to simplify it we will strip colours off\). Our constructor will be like this:

```java
public Mesh(float[] positions, float[] textCoords, int[] indices,
    Texture texture)
```

The texture coordinates VBO is created in the same way as the colour one. The only difference is that it has two components per vertex attribute instead of three:

```java
vboId = glGenBuffers();
vboIdList.add(vboId);
textCoordsBuffer = MemoryUtil.memAllocFloat(textCoords.length);
textCoordsBuffer.put(textCoords).flip();
glBindBuffer(GL_ARRAY_BUFFER, vboId);
glBufferData(GL_ARRAY_BUFFER, textCoordsBuffer, GL_STATIC_DRAW);
glEnableVertexAttribArray(1);
glVertexAttribPointer(1, 2, GL_FLOAT, false, 0, 0);
```

Now we need to use the texture in our shader. In the vertex shader we have changed the second input parameter because now it’s a `vec2` \(we also changed the parameter name). The vertex shader, as in the colour case, just passes the texture coordinates to be used by the fragment shader.

```glsl
#version 330

layout (location=0) in vec3 position;
layout (location=1) in vec2 texCoord;

out vec2 outTexCoord;

uniform mat4 worldMatrix;
uniform mat4 projectionMatrix;

void main()
{
    gl_Position = projectionMatrix * worldMatrix * vec4(position, 1.0);
    outTexCoord = texCoord;
}
```

In the fragment shader we must use the texture coordinates in order to set the pixel colours:

```glsl
#version 330

in  vec2 outTexCoord;
out vec4 fragColor;

uniform sampler2D texture_sampler;

void main()
{
    fragColor = texture(texture_sampler, outTexCoord);
}
```

Before analyzing the code let’s clarify some concepts. A graphics card has several spaces or slots to store textures. Each of these spaces is called a texture unit. When we are working with textures we must set the texture unit that we want to work with. As you can see we have a new uniform named `texture_sampler`. That uniform has a `sampler2D` type and will hold the value of the texture unit that we want to work with.

In the `main` function we use the texture lookup function named `texture`. This function takes two arguments: a sampler and a texture coordinate and will return the correct colour. The sampler uniform allows us to do multi-texturing. We will not cover that topic right now but we will try to prepare the code to add it easily later on.

Thus, in our `ShaderProgram` class we will create a new method that allows us to set an integer value for a uniform:

```java
public void setUniform(String uniformName, int value) {
    glUniform1i(uniforms.get(uniformName), value);
}
```

In the `init` method of the `Renderer` class we will create a new uniform:

```java
shaderProgram.createUniform("texture_sampler");
```

Also, in the `render` method of our `Renderer` class we will set the uniform value to 0. \(We are not using several textures right now so we are just using unit 0\).

```java
shaderProgram.setUniform("texture_sampler", 0);
```

Finally we just need to change the `render` method of the `Mesh` class to use the texture. At the beginning of that method we put the following lines:

```java
// Activate first texture unit
glActiveTexture(GL_TEXTURE0);
// Bind the texture
glBindTexture(GL_TEXTURE_2D, texture.getId());
```

We basically are binding the texture identified by `texture.getId()` to the texture unit 0.

Right now, we have just modified our code base to support textures. Now we need to setup texture coordinates for our 3D cube. Our texture image file will be something like this:

![Cube texture](cube_texture.png)

In our 3D model we have eight vertices. Let’s see how this can be done. Let’s first define the front face texture coordinates for each vertex.

![Texture coordinates front face](cube_texture_front_face.png)

| Vertex | Texture Coordinate |
| --- | --- |
| V0 | \(0.0, 0.0\) |
| V1 | \(0.0, 0.5\) |
| V2 | \(0.5, 0.5\) |
| V3 | \(0.5, 0.0\) |

Now, let’s define the texture mapping of the top face.

![Texture coordinates top face](cube_texture_top_face.png)

| Vertex | Texture Coordinate |
| --- | --- |
| V4 | \(0.0, 0.5\) |
| V5 | \(0.5, 0.5\) |
| V0 | \(0.0, 1.0\) |
| V3 | \(0.5, 1.0\) |

As you can see we have a problem, we need to setup different texture coordinates for the same vertices \(V0 and V3\). How can we solve this? The only way to solve it is to repeat some vertices and associate different texture coordinates. For the top face we need to repeat the four vertices and assign them the correct texture coordinates.

Since the front, back and lateral faces use the same texture we will not need to repeat all of these vertices. You have the complete definition in the source code, but we needed to move from 8 points to 20. The final result is like this.

![Cube with texture](cube_with_texture.png)

In the next chapters we will learn how to load models generated by 3D modeling tools so we won’t need to define by hand the positions and texture coordinates \(which by the way, would be impractical for more complex models\).

## A brief note about transparencies

As you have seen, when we load an image we retrieve the four RGBA components including the transparency level. But if you load a texture with transparencies you may not see anything. In order to support transparency we need to enable blending. This is done by this fragment of code:

```java
glEnable(GL_BLEND);
```

But just by enabling blending, transparencies still will not show up. We need also to instruct OpenGL about how the blending will be applied. This is done through the `glBlendFunc` method:

```java
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```

You can check an excellent explanation about the details of the different functions that can be applied [here]( https://learnopengl.com/Advanced-OpenGL/Blending).

Even after you have enabled blending and set up a function you may not see correct values for transparencies. The reason behind that is depth testing. When fragments are discarded using depth values, we can end up blending fragments with transparent values with the background, not with fragments that were behind them. This will result in wrong rendering artifacts. In order to solve this we need to draw first opaque objects and then render objects that have transparencies in decreasing depth order (far objects should be drawn first).
