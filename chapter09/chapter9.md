# Loading more complex models

In this chapter we will learn to load more complex models defined in external files. These models will be created by 3D modelling tools \(such as [Blender](https://www.blender.org/)\). Up to now we have been creating models by coding directly the arrays that define their geometry, but in this chapter we will learn how to load models defined in OBJ format.
在本章中,我们将学习加载在外部文件中定义的更复杂的模型,这些模型将由3D建模工具\(如[Blender]创建(https://www.blender.org/)\).到目前为止,我们一直在通过直接编码定义几何体的数组来创建模型,但在本章中,我们将学习如何加载以OBJ格式定义的模型.

OBJ \(or .OBJ\) is a geometry definition open file format developed by Wavefront Technologies which has been widely adopted. An OBJ file defines the vertices, texture coordinates and polygons that compose a 3D model. It’s a relatively easy format to parse since it is text based and each line defines an element \(a vertex, a texture coordinate, etc.\).
OBJ\(或.OBJ\)是由波前技术开发的一种几何定义的开放文件格式,已被广泛采用.OBJ文件定义组成三维模型的顶点、纹理坐标和多边形,这是一种相对容易解析的格式,因为它是基于文本的，每行定义一个元素\(顶点、纹理坐标等\).

In an .obj file each line starts with a token which identifies the type of element:
在.obj文件中,每行以标识元素类型的标记开始:

* Comments are lines which start with \#.
* 注释是以\#开头

* The token “v” defines a geometric vertex with coordinates \(x, y, z, w\). Example: v 0.155 0.211 0.32 1.0.
* 标记"v“表示用坐标\(x，y，z，w\)定义一个几何顶点.示例:v 0.155 0.211 0.32 1.0.

* The token “vn” defines a vertex normal with coordinates \(x, y, z\). Example: vn 0.71 0.21 0.82. More on this later.
* 标记"vn"表示用坐标\(x，y，z\)定义顶点法线.示例:vn 0.71 0.21 0.82.我们稍后会详细介绍.

* The token “vt” defines a texture coordinate. Example: vt 0.500 1. 
* 标记"vt"表示定义纹理坐标.示例: vt 0.500 1.

* The token “f” defines a face. With the information contained in these lines we will construct our indices array. We will handle only the case where faces are exported as triangles. It can have several variants:
* 标记"f"表示定义一个面.利用这些行中包含的信息,我们将构造索引数组.我们将只处理面作为三角形导出的情况,它可以有几个变体:

  * It can define just vertex positions \(f v1 v2 v3\). Example: f 6 3 1. In this case this triangle is defined by the geometric vertices that occupy positions 6, 3 a and 1. \(Vertex indices always starts by 1\).
  * 它只能定义顶点位置\(f v1 v2 v3\).示例:f 6 3 1.在这种情况下,该三角形由占据位置6, 3 和 1的几何顶点定义.\(顶点索引总是从1开始.\)

  * It can define vertex positions, texture coordinates and normals \(f v1/t1/n1 v2/t2/n2 V3/t3/n3\). Example: f 6/4/1 3/5/3 7/6/5. The first block is “6/4/1” and defines the coordinates, texture coordinates and normal vertex. What you see here is the position, so we are saying: pick the geometric vertex number six, the texture coordinate number four and the vertex normal number one.
  * 它可以定义顶点位置,纹理坐标和法线\(f v1/t1/n1 v2/t2/n2 V3/t3/n3\).示例:f 6/4/1 3/5/3 7/6/5.第一个块是“6/4/1”,定义坐标、纹理坐标和法线顶点.你在这里看到的是位置,所以我们说的是:选取几何顶点6,纹理坐标4和顶点法线1.

OBJ format has many more entry types \(like one to group polygons, defining materials, etc.\). For now we will stick to this subset, our OBJ loader will ignore other entry types.
OBJ格式有更多的条目类型\(比如分组多边形、定义材质等\).现在我们将继续使用这个子集,我们的OBJ加载器将忽略其他条目类型.

But what is a normal? Let’s define it first. The normal of a plane is a vector perpendicular to that plane which has a length equal to one.
但什么是正常的?我们先来定义一下.一个平面的法线是一个垂直于该平面的向量,其长度等于1.

![Normals](normals.png)

As you can see in the figure above, a plane can have two normals. Which one should we use? Normals in 3D graphics are used for lighting, so we should choose the normal which is oriented towards the source of light. In other words, we should choose the normal that points out from the external face of our model.
如上图所示,一个平面可以有两条法线.我们应该用哪一个?三维图形中的法线是用来照明的,所以我们应该选择朝向光源的法线.换句话说,我们应该选择从模型的外表面指出的法线.

When we have a 3D model, it is composed by polygons, triangles in our case. Each triangle is composed by three vertices. The Normal vector for a triangle will be the vector perpendicular to the triangle surface which has a length equal to one.
当我们有一个三维模型时，它是由多边形组成的，在我们的例子中是三角形.每个三角形由三个顶点组成.三角形的法向量,是垂直于长度等于1的三角形表面的向量.

A vertex normal is associated to a specific vertex and is the combination of the normals of the surrounding triangles \(of course its length is equal to one\). Here you can see the vertex models of a 3D mesh \(taken from [Wikipedia](https://en.wikipedia.org/wiki/Vertex_normal#/media/File:Vertex_normals.png)\)
顶点法线与特定顶点相关联,是周围三角形法线的组合\(当然，其长度等于1\).在这里你可以看到三维网格的顶点模型\(摘自[维基百科](https://en.wikipedia.org/wiki/Vertex_normal#/media/File：Vertex\u normals.png）\)

![Vertex normals](vertex_normals.png)

Let’s now start creating our OBJ loader. First of all we will modify our `Mesh` class since now it’s mandatory to use a texture. Some of the obj files that we may load may not define texture coordinates and we must be able to render them using a colour instead of a texture. In this case the face definition will be of the form: “f v//n”.
现在让我们开始创建OBJ加载程序.首先,我们将修改`Mesh`类,因为现在必须使用纹理.可能我们加载的一些obj文件会没有定义纹理坐标,我们必须能够使用颜色而不是纹理来渲染它们.在这种情况下,面定义的形式为:“f v//n”.

Our `Mesh` class will have a new attribute named `colour`.
我们的`Mesh`类将有一个名为`colour`的新属性.

```java
private Vector3f colour;
```

And the constructor will no longer require a `Texture` instance. Instead we will provide getters and setters for texture and colour attributes.
构造函数将不再需要`Texture`实例.相反,我们将为纹理和颜色属性提供getter和setter.

```java
public Mesh(float[] positions, float[] textCoords, float[] normals, int[] indices) {
```

Of course, in the `render` and `cleanup` methods we must check if the texture attribute is not null before using it. As you can see in the constructor we now pass a new array of floats named `normals`. How do we use normals for rendering? The answer is easy: it will be just another VBO inside our VAO, so we need to add this code.
当然，在`render`和“`cleanup`方法中,我们必须在使用纹理属性之前检查它是否为null状态.正如你在构造函数中看到的,我们现在传递一个名为`normals`的新浮点数组.如何使用法线进行渲染?答案很简单:这将只是我们的VAO中的另一个VBO,所以我们需要添加此代码.

```java
// Vertex normals VBO
vboId = glGenBuffers();
vboIdList.add(vboId);
vecNormalsBuffer = MemoryUtil.memAllocFloat(normals.length);
vecNormalsBuffer.put(normals).flip();
glBindBuffer(GL_ARRAY_BUFFER, vboId);
glBufferData(GL_ARRAY_BUFFER, vecNormalsBuffer, GL_STATIC_DRAW);
glEnableVertexAttribArray(2);
glVertexAttribPointer(2, 3, GL_FLOAT, false, 0, 0);
```

Now that we have finished the modifications in the `Mesh` class we can change our code to use either texture coordinates or a fixed colour. Thus we need to modify our fragment shader like this:
现在我们已经完成了`Mesh`类中的修改,我们可以将代码更改为使用纹理坐标或固定颜色.因此,我们需要修改片段着色器,如下所示:

```glsl
#version 330

in  vec2 outTexCoord;
out vec4 fragColor;

uniform sampler2D texture_sampler;
uniform vec3 colour;
uniform int useColour;

void main()
{
    if ( useColour == 1 )
    {
        fragColor = vec4(colour, 1);
    }
    else
    {
        fragColor = texture(texture_sampler, outTexCoord);
    }
}
```

As you can see we have created two new uniforms:
如你所见,我们已经新建了两个统一方法:

* `colour`: Will contain the base colour.
* `colour`:将包含基本颜色.
* `useColour`: It’s a flag that we will set to 1 when we don’t want to use textures.
* `useColour`:当我们不想使用纹理时,它是一个将被设置为1的标志.

In the `Renderer` class we need to create those two uniforms.
在`Renderer`类中,我们也需要创建这两个统一方法:

```java
// Create uniform for default colour and the flag that controls it
shaderProgram.createUniform("colour");
shaderProgram.createUniform("useColour");
```

And like any other uniform, in the `render` method of the `Renderer` class we need to set the values for this uniforms for each `gameItem`.
和其他统一的方法一样,在Renderer类的render方法中，我们需要为每个gameItem设置统一的值。

```java
for (GameItem gameItem : gameItems) {
    Mesh mesh = gameItem.getMesh();
    // Set model view matrix for this item
    Matrix4f modelViewMatrix = transformation.getModelViewMatrix(gameItem, viewMatrix);
    shaderProgram.setUniform("modelViewMatrix", modelViewMatrix);
    // Render the mesh for this game item
    shaderProgram.setUniform("colour", mesh.getColour());
    shaderProgram.setUniform("useColour", mesh.isTextured() ? 0 : 1);
    mesh.render();
}
```

Now we can create a new class named `OBJLoader` which parses OBJ files and creates a `Mesh` instance with the data contained in it. You may find some other implementations in the web that may be a bit more efficient than this one, but I think this version is simpler to understand. This will be an utility class which will have a static method like this:
现在,我们可以创建一个名为`OBJLoader`的新类,该类解析OBJ文件并使用其中包含的数据创建一个`Mesh`实例.你可能会在web上发现一些其他的实现,它们可能比这个更有效,但是我认为这个版本更容易理解.这将是一个实用程序类,其静态方法如下:

```java
public static Mesh loadMesh(String fileName) throws Exception {
```

The parameter `filename` specifies the name of the file, which must be in the CLASSPATH, that contains the OBJ model.
参数`filename`指定包含OBJ模型的文件名,该文件必须位于类路径中.

The first thing that we will do in that method is to read the file contents and store all the lines in an array. Then we create several lists that will hold the vertices, the texture coordinates, the normals and the faces.
在该方法中,我们要做的第一件事是读取文件内容,并将所有行存储在一个数组中.然后我们创建几个列表来保存顶点、纹理坐标、法线和面.

```java
List<String> lines = Utils.readAllLines(fileName);

List<Vector3f> vertices = new ArrayList<>();
List<Vector2f> textures = new ArrayList<>();
List<Vector3f> normals = new ArrayList<>();
List<Face> faces = new ArrayList<>();
```

We then parse each line and, depending on the starting token, we will get a vertex position, a texture coordinate, a vertex normal or a face definition. At the end we will need to reorder that information.
然后我们分析每一行,根据起始标记,我们将得到顶点位置、纹理坐标、顶点法线或面定义.最后我们需要重新整理这些信息.

```java
for (String line : lines) {
    String[] tokens = line.split("\\s+");
    switch (tokens[0]) {
        case "v":
            // Geometric vertex
            Vector3f vec3f = new Vector3f(
                Float.parseFloat(tokens[1]),
                Float.parseFloat(tokens[2]),
                Float.parseFloat(tokens[3]));
            vertices.add(vec3f);
            break;
        case "vt":
            // Texture coordinate
            Vector2f vec2f = new Vector2f(
                Float.parseFloat(tokens[1]),
                Float.parseFloat(tokens[2]));
            textures.add(vec2f);
            break;
        case "vn":
            // Vertex normal
            Vector3f vec3fNorm = new Vector3f(
                Float.parseFloat(tokens[1]),
                Float.parseFloat(tokens[2]),
                Float.parseFloat(tokens[3]));
            normals.add(vec3fNorm);
            break;
        case "f":
            Face face = new Face(tokens[1], tokens[2], tokens[3]);
            faces.add(face);
            break;
        default:
            // Ignore other lines
            break;
    }
}
return reorderLists(vertices, textures, normals, faces);
```

Before talking about reordering let’s see how face definitions are parsed. We have create a class named `Face` which parses the definition of a face. A `Face` is composed by a list of indices groups, in this case since we are dealing with triangles we will have three indices group\).
在讨论重新排序之前,让我们先看看如何解析面定义.我们创建了一个名为`Face`的类,它解析一个面的定义.`Face`由索引组列表组成,在本例中,由于我们处理的是三角形,因此将有三个索引组\).

![Face definition](face_definition.png)

We will create another inner class named `IndexGroup` that will hold the information for a group.
我们将创建另一个名为`IndexGroup`的内部类,它将保存组的信息.

```java
protected static class IdxGroup {

    public static final int NO_VALUE = -1;

    public int idxPos;

    public int idxTextCoord;

    public int idxVecNormal;

    public IdxGroup() {
        idxPos = NO_VALUE;
        idxTextCoord = NO_VALUE;
        idxVecNormal = NO_VALUE;
        }
}
```

Our `Face` class will be like this.
我们的`Face`类则是这样的:

```java
protected static class Face {

    /**
     * List of idxGroup groups for a face triangle (3 vertices per face).
    */
    private IdxGroup[] idxGroups = new IdxGroup[3];

    public Face(String v1, String v2, String v3) {
        idxGroups = new IdxGroup[3];
        // Parse the lines
        idxGroups[0] = parseLine(v1);
        idxGroups[1] = parseLine(v2);
        idxGroups[2] = parseLine(v3);
    }

    private IdxGroup parseLine(String line) {
        IdxGroup idxGroup = new IdxGroup();

        String[] lineTokens = line.split("/");
        int length = lineTokens.length;
        idxGroup.idxPos = Integer.parseInt(lineTokens[0]) - 1;
        if (length > 1) {
            // It can be empty if the obj does not define text coords
            String textCoord = lineTokens[1];
            idxGroup.idxTextCoord = textCoord.length() > 0 ? Integer.parseInt(textCoord) - 1 : IdxGroup.NO_VALUE;
            if (length > 2) {
                idxGroup.idxVecNormal = Integer.parseInt(lineTokens[2]) - 1;
            }
        }

        return idxGroup;
    }

    public IdxGroup[] getFaceVertexIndices() {
        return idxGroups;
    }
}
```

When parsing faces we may encounter objects with no textures but with vector normals. In this case a face line could be like `f 11//1 17//1 13//1`, so we need to detect those cases.
当分析面时,我们可能会遇到没有纹理但具有向量法线的对象.在本例中,面线可能类似于`f 11//1 17//1 13//1`这样,因此我们需要检测这些情况.

Now we can talk about how to reorder the information we have. Our `Mesh` class expects four arrays, one for position coordinates, one for texture coordinates, one for vector normals and another one for the indices. The first three arrays will have the same number of elements since the indices array is unique \(note that the same number of elements does not imply the same length. Position elements, vertex coordinates, are 3D and are composed by three floats. Texture elements, texture coordinates, are 2D and thus are composed by two floats\). OpenGL does not allow us to define different indices arrays per type of element \(if so, we would not need to repeat vertices while applying textures\).
现在我们可以讨论如何重新排序我们拥有的信息.我们的`Mesh`类需要四个数组,一个用于位置坐标,一个用于纹理坐标,一个用于向量法线,另一个用于索引.前三个数组将具有相同的元素数,因为索引数组是唯一的\(请注意,相同的元素数并不意味着相同的长度.位置元素顶点坐标是三维的,由三个浮点数组成.但纹理元素、纹理坐标是二维的.因此由两个浮点数组成\).OpenGL不允许我们为每种类型的元素定义不同的索引数组\(如果是这样，我们就不需要在应用纹理时重复顶点了\).

When you open an OBJ line you will first probably see that the list that holds the vertices positions has a higher number of elements than the lists that hold the texture coordinates and the number of vertices. That’s something that we need to solve. Let’s use a simple example which defines a quad with a texture with a pixel height \(just for illustration purposes\). The OBJ file may be like this \(don’t pay too much attention about the normals coordinate since it’s just for illustration purpose\).
当你打开一条作为对象的线时,你首先可能会看到保存顶点位置的列表的元素数比保存纹理坐标和顶点数的列表的元素数要多.这是一个我们需要解决的问题.让我们用一个简单的例子来定义一个具有像素高度的纹理的四边形\(仅用于说明目的\).OBJ文件可能是这样的\(不要过于注意法线坐标,因为它只是为了演示\).

```java
v 0 0 0
v 1 0 0
v 1 1 0
v 0 1 0

vt 0 1
vt 1 1

vn 0 0 1

f 1/2/1 2/1/1 3/2/1
f 1/2/1 3/2/1 4/1/1
```

When we have finished parsing the file we have the following lists \(the number of each element is its position in the file upon order of appearance\)
当我们解析完文件后,我们有以下列表\(每个元素的编号是它在文件中的位置,按出现顺序排列\)

![Ordering I](ordering_i.png)

Now we will use the face definitions to construct the final arrays including the indices. A thing to take into consideration is that the order in which texture coordinates and vector normals are defined does not correspond to the order in which vertices are defined. If the size of the lists were the same and they were ordered, face definition lines would only just need to include a number per vertex.
现在,我们将使用面定义来构造包含索引的最终数组.需要考虑的一点是,定义纹理坐标和向量法线的顺序与定义顶点的顺序并不一致.如果列表的大小相同并且是有序的,则面定义线只需要包含每个顶点的数字.

So we need to order the data and setup accordingly to our needs. The first thing that we must do is create three arrays (for the vertices, the texture coordinates, and the normals) and one list for the indices. As we said before, the three arrays will have the same number of elements \(equal to the number of vertices\). The vertices array will have a copy of the list of vertices.
因此,我们需要对数据进行排序,并根据需要进行相应的设置.我们必须做的第一件事是创建三个数组\(顶点、纹理坐标和法线\)和一个索引列表.如前所述,这三个数组的元素数相同\(等于顶点数\).顶点数组将有一个顶点列表的副本.

![Ordering II](ordering_ii.png)

Now we start processing the faces. The first index group of the first face is 1/2/1. We use the first index in the index group, the one that defines the geometric vertex, to construct the index list. Let’s call it `posIndex`.
现在我们开始处理一个面,第一个面的第一个索引组是 1/2/1 .我们使用索引组中的第一个索引,即定义几何顶点的索引,来构造索引列表.我们叫它`posIndex`.
Our face is specifying that the we should add the index of the element that occupies the first position into our indices list. So we put the value of `posIndex` minus one into the `indicesList` \(we must subtract 1 since arrays start at 0 but the OBJ file format assumes that they start at 1\).
我们的任务,是指定我们应该将占据第一个位置的元素的索引,将它添加到索引列表中.因此,我们将`posIndex`的值减去1,然后放入`indicesList`中\(由于数组从0开始，所以必须减去1,但OBJ文件格式会假定它们从1开始\).

![Ordering III](ordering_iii.png)

Then we use the rest of the indices of the index group to set up the `texturesArray` and `normalsArray`. The second index in the index group is 2, so what we must do is put the second texture coordinate in the same position as the one that occupies the vertex designated posIndex \(V1\).
然后我们使用索引组的其余索引来设置`texturesArray`和`normalsArray`.索引组中的第二个索引是2,因此我们必须将第二个纹理坐标,放置在与占据指定顶点`posIndex`\(V1\)的坐标相同的位置.

![Ordering IV](ordering_iv.png)

Then we pick the third index, which is 1, so what we must do is put the first vector normal coordinate in the same position as the one that occupies the vertex designated `posIndex` \(V1\).
然后我们选择第三个索引,也就是1.重复一次,我们要做的是把第一个向量法坐标放在与占据指定顶点`posIndex`\(V1\)的位置相同的位置.

![Ordering V](ordering_v.png)

After we have processed the first face the arrays and lists will be like this.
处理完第一个面后,数组和列表将如下所示:

![Ordering VI](ordering_vi.png)

After we have processed the second face the arrays and lists will be like this.
处理完第二个面后,数组和列表将如下所示:

![Ordering VII](ordering_vii.png)

The second face defines vertices which already have been assigned, but they contain the same values, so there’s no problem in reprocessing this. I hope the process has been clarified enough, it can be somewhat tricky until you get it. The methods that reorder the data are shown below. Keep in mind that what we have are float arrays so we must transform those arrays of vertices, textures and normals into arrays of floats. So the length of these arrays will be the length of the vertices list multiplied by the number three in the case of vertices and normals or multiplied by two in the case of texture coordinates.
第二个面定义了已经指定的顶点,但是它们包含相同的值,所以重新处理这个没有问题.我希望这个过程已经足够清楚了,在你掌握它之前可能会有些棘手.对数据重新排序的方法如下所示.请记住.我们拥有的是浮点数组,因此我们必须将这些顶点、纹理和法线数组转换为浮点数组.因此,这些数组的长度将是顶点列表的长度乘以数字3\(对于顶点和法线\),或者乘以2\(对于纹理坐标).

```java
private static Mesh reorderLists(List<Vector3f> posList, List<Vector2f> textCoordList,
    List<Vector3f> normList, List<Face> facesList) {

    List<Integer> indices = new ArrayList<>();
    // Create position array in the order it has been declared
    float[] posArr = new float[posList.size() * 3];
    int i = 0;
    for (Vector3f pos : posList) {
        posArr[i * 3] = pos.x;
        posArr[i * 3 + 1] = pos.y;
        posArr[i * 3 + 2] = pos.z;
        i++;
    }
    float[] textCoordArr = new float[posList.size() * 2];
    float[] normArr = new float[posList.size() * 3];

    for (Face face : facesList) {
        IdxGroup[] faceVertexIndices = face.getFaceVertexIndices();
        for (IdxGroup indValue : faceVertexIndices) {
            processFaceVertex(indValue, textCoordList, normList,
                indices, textCoordArr, normArr);
        }
    }
    int[] indicesArr = new int[indices.size()];
    indicesArr = indices.stream().mapToInt((Integer v) -> v).toArray();
    Mesh mesh = new Mesh(posArr, textCoordArr, normArr, indicesArr);
    return mesh;
}

private static void processFaceVertex(IdxGroup indices, List<Vector2f> textCoordList,
    List<Vector3f> normList, List<Integer> indicesList,
    float[] texCoordArr, float[] normArr) {

    // Set index for vertex coordinates
    int posIndex = indices.idxPos;
    indicesList.add(posIndex);

    // Reorder texture coordinates
    if (indices.idxTextCoord >= 0) {
        Vector2f textCoord = textCoordList.get(indices.idxTextCoord);
        texCoordArr[posIndex * 2] = textCoord.x;
        texCoordArr[posIndex * 2 + 1] = 1 - textCoord.y;
    }
    if (indices.idxVecNormal >= 0) {
        // Reorder normal vectors
        Vector3f vecNorm = normList.get(indices.idxVecNormal);
        normArr[posIndex * 3] = vecNorm.x;
        normArr[posIndex * 3 + 1] = vecNorm.y;
        normArr[posIndex * 3 + 2] = vecNorm.z;
    }
}
```

Another thing to notice is that texture coordinates are in UV format so y coordinates need to be calculated as 1 minus the value contained in the file.
另一个需要注意的是,纹理坐标是UV格式的,所以y坐标需要计算为 1 减去文件中包含的值.

Now, at last, we can render obj models. I’ve included an OBJ file that contains the textured cube that we have been using in previous chapters. In order to use it in the `init` method of our `DummyGame` class we just need to construct a `GameItem` instance like this.
现在,我们终于可以绘制obj模型了.我已经做好了一个OBJ文件,其中包含了我们在前几章中使用的纹理立方体.为了在`DummyGame`类的`init`方法中使用它,我们只需要像这样构造一个`GameItem`实例.

```java
Texture texture = new Texture("/textures/grassblock.png");
mesh.setTexture(texture);
GameItem gameItem = new GameItem(mesh);
gameItem.setScale(0.5f);
gameItem.setPosition(0, 0, -2);
gameItems = new GameItem[]{gameItem};
```

And we will get our familiar textured cube.
然后我们将得到我们熟悉的纹理立方体:

![Textured cube](textured_cube.png)

We can now try with other models. We can use the famous Stanford Bunny \(it can be freely downloaded\) model, which is included in the resources. This model is not textured so we can use it this way:

```java
Mesh mesh = OBJLoader.loadMesh("/models/bunny.obj");
GameItem gameItem = new GameItem(mesh);
gameItem.setScale(1.5f);
gameItem.setPosition(0, 0, -2);
gameItems = new GameItem[]{gameItem};
```

![Stanford Bunny](standford_bunny.png)

The model looks a little bit strange because we have no textures and there’s no light so we cannot appreciate the volumes but you can check that the model is correctly loaded. In the `Window` class when we set up the OpenGL parameters add this line.
我们现在可以试试其他型号.我们可以使用著名的斯坦福兔子模型\(可以免费下载\),它包含在参考资料中.此模型没有纹理,因此我们可以这样使用:

```java
glPolygonMode( GL_FRONT_AND_BACK, GL_LINE );
```

You should now see something like this when you zoom in.
现在,当你放大,应该看到这样的东西:

![Stanford Bunny Triangles](standford_bunny_triangles.png)

Now you can see all the triangles that compose the model.
现在你也可以看到组成模型的所有三角形.

With this OBJ loader class you can now use Blender to create your models. Blender is a powerful tool but it can be some bit of overwhelming at first, there are lots of options, lots of key combinations and you need to take your time to do the most basic things by the first time. When you export the models using Blender please make sure to include the normals and export faces as triangles.
有了这个obj装载类,您现在可以使用Blender来创建您的模型.Blender是一个强大的工具,但一开始可能有点让人不知所措,因为有太多功能可以选择,也有很多的组合键,如果是第一次使用,你需要花很多时间做一些最基本的事情.另外,使用Blender导出模型时,请确保包含法线,并将面导出为三角形.

![OBJ Export options](obj_export_options.png)

Remember to split edges when exporting, since we cannot assign several texture coordinates to the same vertex. Also, we need the normals to be defined per each triangle, not assigned to vertices. If you find light problems with some models \(next chapters\), you should verify the normals. You can visualize them inside Blender. In later chapters, we will use [Assimp](https://www.assimp.org/) to load the models, which will allow us to load other formats and remove the restrictions on the model structure mentioned above. In any case, If you cannot wait for that, you can check the [following code](https://gist.github.com/Nutriz/e473701b359487a606caf48465b2fb77) by Jérôme Gully, which will allow you to use models which reuse texture coordinates.
导出时请记住分割边,因为我们无法将多个纹理坐标指定给同一个顶点.另外,我们需要为每个三角形定义法线,而不是将其指定给顶点.如果你发现某些模型存在光照问题\(下一章\),你应该验证法线.你可以想象一下它们在Blender里会是什么样子的.在后面的章节中,我们将使用[Assimp](https://www.assimp.org/)加载模型,这将允许我们加载其他格式,并消除对上述模型结构的限制.在任何情况下,如果你不想等待,你可以看看[下面的代码](https://gist.github.com/Nutriz/e473701b359487a606caf48465b2fb77)作者：Jérôme Gully,这将允许你使用重用纹理坐标的模型.

![Edge split](edge_split.png)

