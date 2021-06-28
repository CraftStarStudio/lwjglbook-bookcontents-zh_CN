# 学习光渲染和计算


在本章中,我们将学习如何添加光到我们的3D游戏引擎中.我们不会实现一个物理上完美的光照模型,因为抛开复杂性不谈,单单是它需要大量的计算机资源就够我们好受了.取而代之的是,我们将实现一个能提供不错结果的近似算法:我们将使用名为Phong shading\(由Bui Tuong Phong开发\)的算法.另一个需要指出的重要问题是,我们将只对灯光进行建模,但不会对那些灯光应生成的阴影进行建模\(这将在另一章中完成\).

PS:“碎片”指的是画面中的三角形们

在开始之前,让我们定义一些灯光类型:

* **点光源**:(Point light)这类光源模拟从空间中的一个点向各个方向均匀发射的光源.

* **聚光灯**:(Spot light)这种类型的光模拟了从空间中的某个点发射的光源,但不是向所有方向发射,而是局限于一个圆锥体,注意是圆锥.

* **平行光**:(Directional light)这种类型的光模拟我们从太阳接收到的光,3D空间中的所有物体都受到来自特定方向的平行光线的照射.无论对象是近还是远,所有光线都将以相同的角度照射对象.

* **环境光**:(Ambient light)这种光来自空间的任何地方,以同样的方式照亮所有物体.

![Light types](./light_types.png)


因此,要对光进行建模,我们需要考虑光的类型、位置和其他一些参数,如颜色.当然,我们还必须考虑受光线影响的物体吸收和反射光线的方式.


Phong shading 着色算法将为模型中的每个点(即每个顶点)建模灯光效果.这就是为什么它被称为局部照明模拟.那为什么这个算法不会计算阴影?原因:它只计算要应用到每个顶点的光,而不考虑顶点是否在阻挡光的对象后面.我们将在后面的章节中克服这个缺点.但是,正因为如此,它是一个简单而快速的算法,提供了非常好的效果.我们将在这里使用一个简化的版本,没有充分考虑材料的反射之类的东西。


Phong算法考虑了照明的三个组成部分:


* **环境光**:(Ambient light)模拟来自任何地方的光,这将有助于我们照亮\(以所需的强度\)没有受到任何光照射的区域,就像背景光一样.

* **漫反射**:(Diffuse reflectance)考虑到面向光源的表面更亮.

* **镜面反射**:(Specular reflectance)光在抛光或金属表面的反射模型.

最后,我们想要得到的是,乘以指定给三角形的颜色,根据它所接收到的光,使颜色变亮或变暗.我们将组件命名,$$A$$表示环境光,$$D$$表示漫反射,$$S$$表示镜面反射.这一因素将是这些组成部分的增加:

$$L = A + D + S$$

事实上,这些部件确实是颜色,这就是每个光分量贡献的颜色分量.这是因为光组件不仅提供一定程度的强度,而且可以改变模型的颜色.在我们的片段着色器中,我们只需要将该浅色乘以原始片段颜色\(从纹理或基色获得\).


我们还可以为相同的材质指定不同的颜色,这些颜色将用于环境光、漫反射和镜面反射组件.因此,这些成分将被与材质相关的颜色所调制.如果材质有纹理,我们只需为每个组件使用一个纹理.

因此,无纹理材质的最终颜色将是:$$L=a*氛围色+D*漫射色+S*镜面反射色$$.($$L = A * ambientColour + D * diffuseColour + S * specular Colour$$.)

纹理材料的最终颜色为：

$$L=A*纹理颜色+D*纹理颜色+S*纹理颜色$$.($$L = A * textureColour + D * textureColour + S * textureColour$$)


## 环境光组件

让我们想想怎么使用第一个组件,环境光组件.这只是一个常数,它会使我们所有的物体变得更亮或更暗.我们可以用它来模拟一天中某一特定时间的光线\(黎明、黄昏等\),也可以用它来添加一些光线到光线不直接照射的点,但我们还可以使用间接光,\(反射引起的\)以一种简单的方式照射光线.


环境光是最容易计算的部分:我们只需要传递一种颜色,因为它将乘以我们的基色,它要做的也只是调制基色.假设我们已经确定片段的颜色是$$（1.0，0.0，0.0）$$,即红色.如果没有环境光,它将显示为一个完全红色的片段.如果我们将环境光设置为$$（0.5，0.5，0.5）$$,则最终颜色将为$$（0.5，0，0）$$,即红色的较暗版本.这种光会以同样的方式使所有的片段变暗\(谈论使物体变暗的光似乎有点奇怪,但事实上这就是我们得到的效果\).另外,如果RGB分量不相同,它可以增加一些颜色.所以我们只需要一个矢量,来调节环境光的强度和颜色.

## 漫反射


现在我们来谈谈漫反射.这模拟了这样一个现实中的现象:垂直于光源的表面,要看起来比以更间接的角度接收光的表面更亮.\(这么说吧\),这些物体接收到的光越多,光密度就越高.

![Diffuse Light](./diffuse_light.png)

但是,我们要怎么计算呢?你还记得上一章我们介绍了法线的概念吗?法线是垂直于长度等于1的曲面的向量,让我们在上图中画三个点的法线，如你所见，每个点的法线是垂直于每个点的切面的向量.我们将不绘制来自光源的光线，而是绘制从每个点到光点的矢量\(即相反方向\).

![Normals and light direction](./diffuse_light_normals.png)

如你所见，与$$P1$$关联的法线(名为$$N1$$)与指向光源的向量平行,该向量模拟光线的相反方向\($$N1$$已绘制置换,因此你可以看到它,但在数学上是相等的\)$$P1$$与指向光源的矢量的角度等于$$0$$.它的表面垂直于光源,所以$$P1$$将是最亮的点.


与$$P2$$相关联的法线名为$$N2$$,与指向光源的向量的角度约为30度,因此它应该是深褐色$$P1$$.最后,与$$P3$$关联的法线\(名为$$N3$$\)也与指向光源的向量平行.但两个向量方向相反,$$P3$$与指向光源的矢量成180度角,所以完全不应获得任何光.


所以,我们似乎有一个很好的方法来确定到达某一个点的光强度,这与形成法线的角度有关,用一个指向光源的向量形成法线.我们该怎么计算这个？

我们可以用一个数学运算方式————点积.此操作获取两个向量并生成一个数字\(标量\),如果它们之间的角度是锐角,则该数字为正;如果它们之间的夹角很大,较宽,则该数字为负.如果两个向量都是标准化的,即两者的长度都等于1,则点积将在$$-1$$和$$1$$之间.如果两个向量的方向完全相同\(角度$$0$$\),则点积为1;如果两个向量形成一个直角,则为$$0$$;如果两个向量指向相反方向,则为$$-1$$.


让我们定义两个向量,$$v1$$和$$v2$$,并让$$alpha$$成为它们之间的角度变量.点积由以下公式定义:

![Dot product](./dot_product.png)


如果两个向量都是标准化的,它们的长度,它们的模将等于一,因此点积等于余弦,判断这种“如果”它们之间的角度.我们将使用下述操作来计算漫反射量:


所以我们需要计算指向光源的向量.这要怎么做到?我们有每个点的位置\(顶点位置\)和光源的位置.首先,两个坐标必须在同一个坐标空间.为了简化,我们假设它们都在世界坐标空间中:那么这些位置就是指向顶点位置\($$VP$$\)和光源\($$VS$$\)的向量的坐标,如下图所示.

![Diffuse Light calculation I](./diffuse_calc_i.png)

如果我们从$$VP$$中减去$$V$$S,我们就得到了我们要寻找的向量,它叫做$$L$$.


现在,我们可以计算指向光源的向量和法线之间的点积.这个产出的结果被称为兰伯特项(Lambert term),因为约翰兰伯特是第一个提出如何计算这种关系模型的亮度表面的人.


让我们总结一下如何计算它.我们定义以下变量:

* $$vPos$$: 我们的顶点在模型视图空间坐标中的位置.

* $$lPos$$:灯光在视图空间坐标中的位置。

* $$intensity$$: 灯光的强度\(从0到1\).

* $$lColor$$: 灯光的颜色.

* $$normal$$: 顶点法线.


首先,我们需要计算从当前位置指向光源的向量:$$toLightDirection=lPos-vPos$$.这次操作的结果需要规范化。


然后,我们需要计算漫射因子\(标量\):$$diffuseFactor=normal\cdot toLightDirection$$.它被计算为两个向量之间的点积,因为我们希望它在$$-1$$和$$1$$之间,所以两个向量都需要标准化.颜色需要介于$$0$$和$$1$$之间,因此如果值低于$$0$$,我们会将其设置为0.

最后我们只需要通过漫反射因子和光强度,来调节光的颜色:

$$颜色=漫射颜色*L颜色*漫射因子*强度$$($$colour = diffuseColour * lColour * diffuseFactor * intensity$$)


## 镜面反射部件

在考虑镜面反射量之前,我们首先需要检查光是如何反射的.当光照射到物体表面时,一部分被吸收,另一部分则被反射,如果你还记得物理课上讲的东西的话，反射就是光从物体上反射回来.

![Light reflection](./light_reflection.png)


当然,表面并不是完全光滑的,如果你看近一点,你会看到很多不完美的地方.除此之外,还有许多光线\(事实上是光子\),它们会影响到表面,并在很大的角度范围内反射.因此,我们看到的就像一束光从表面反射过来.也就是说,光在撞击表面时会发生漫反射,这就是我们之前讨论过的漫射组件.

![Surface](./surface.png)

但是当光照射到抛光的表面时,例如金属,光的扩散便会降低,当光照射到表面时,大部分光会向相反的方向反射出去.

![Polished surface](./polished_surface.png)


要采用什么镜面反射组件模型取决于材料特性.关于镜面反射，重要的是,要注意只有当相机处于适当的位置才有用,也就是说,在反射光发射到的区域,反射光才会可见.

![Specular lighting](./specular_lightining.png)


既然镜面反射背后的机制已经解释了,我们就可以计算这个部件了.首先我们需要一个从光源指向顶点的向量.当我们计算漫射部件时,我们计算的正好相反,一个指向光源的向量$$toLightDirection$$,因此我们将其计算为$$fromLightDirection = -(toLightDirection)$$.



然后我们需要计算$$fromLightDirection$$撞击到表面时产生的反射光,并考虑其法线.GLSL函数`reflect`就是这样做的.所以,$$reflectedLight=反射（来自光源,法线)$$. 原文算式:($$reflectedLight = reflect(fromLightSource, normal)$$).


我们还需要一个指向摄影机的向量,我们将其命名为$$cameraDirection$$,它将被计算为摄影机位置和顶点位置之间的差:$$cameraDirection=cameraPos-vPos$$.摄像机位置矢量和顶点位置需要在同一个坐标系中,得到的矢量需要归一化.下图概述了我们迄今为止计算的主要组件:

![Specular lighting calculation](./specular_lightining_calc.png)


现在我们需要计算我们看到的光强度,我们称之为$$specularFactor$$.如果$$cameraDirection$$和$$reflectedLight$$矢量平行且指向同一方向,则该分量将更高;如果它们指向相反方向,则该分量将取其较低的值.为了计算这个点积,又来解救。所以$$specularFactor=cameraDirection\cdot reflectedLight$$.我们只希望此值介于$$0$$和$$1$$之间,因此如果它低于$$0$$,则将其设置为0.


我们还需要考虑到,如果相机指向反射光锥,这种光必须更强烈.这将通过将$$specularFactor$$提高到名为$$specularPower$$的参数来实现.

$$specularFactor = specularFactor^{specularPower}$$.


最后,我们需要对材料的反射率进行建模,如果光被反射,反射率也会调节强度,这将通过另一个名为reflection的参数完成.所以镜面反射组件的颜色将是: 
$$specularColour * lColour * reflectance * specularFactor * intensity$$. 


## 衰减

现在我们知道了如何计算三个组件,这三个组件将为我们模拟点光源和环境光提供帮助.但是我们的光模型仍然不完整,因为在现在的计算中,物体反射的光与光源的距离无关.我们要完善这一点,也就是说,我们需要模拟光的衰减.


衰减计算是做一个距离和光的函数.光的强度与距离的平方成反比.这一事实很容易想象,因为光沿着球体表面传播能量,其半径等于光传播的距离,球体表面与其半径的平方成正比.我们可以用这个公式计算衰减因子: $$1.0 / (atConstant + atLinear * dist + atExponent * dist^{2})$$.

为了模拟衰减,我们只需要将衰减因子乘以最终颜色.

## 实施阶段


现在我们可以开始为上面描述的所有概念编程,我们要先从着色器开始.大部分工作将在片段着色器中完成,但我们需要将一些数据,从顶点着色器传递给它.在前面的章节中,片段着色器刚刚接收到纹理坐标,现在我们还要传递另外两个参:


* 顶点法线转换为\(标准化的\)模型视图空间坐标。

* 转换为模型视图空间坐标的顶点位置.


这是顶点着色器的代码:

```glsl
#version 330

layout (location=0) in vec3 position;
layout (location=1) in vec2 texCoord;
layout (location=2) in vec3 vertexNormal;

out vec2 outTexCoord;
out vec3 mvVertexNormal;
out vec3 mvVertexPos;

uniform mat4 modelViewMatrix;
uniform mat4 projectionMatrix;

void main()
{
    vec4 mvPos = modelViewMatrix * vec4(position, 1.0);
    gl_Position = projectionMatrix * mvPos;
    outTexCoord = texCoord;
    mvVertexNormal = normalize(modelViewMatrix * vec4(vertexNormal, 0.0)).xyz;
    mvVertexPos = mvPos.xyz;
}
```


在我们继续使用片段着色器之前,必须强调一个非常重要的概念.从上面的代码可以看出,包含顶点法线的变量`mvVertexNormal`被转换为模型视图空间坐标.这是通过将`vertexNormal`与`modelViewMatrix`相乘 (如顶点位置) 来实现的.但是有一个微妙的区别，顶点法线的w分量被设置为0,然后再乘以矩阵：`vec4(vertexNormal，0.0)`.我们为什么要这样做?因为我们确实希望法线被旋转和缩放,但我们不希望它被平移,所以我们只关心它的方向,而不关心它的位置,这是通过将is w component(w分量)设置为0来实现的,这是使用齐次坐标的优点之一,通过设置w component,我们可以控制应用了哪些变换.你可以试试手工做下矩阵乘法,看看为什么会这样.


现在我们可以开始在片段着色器中进行真正的工作了,除了声明来自顶点着色器的值作为输入参数之外,我们还将定义一些有用的结构来建模灯光和材质特性.首先,我们将定义光的模型结构:

```glsl
struct Attenuation
{
    float constant;
    float linear;
    float exponent;
};

struct PointLight
{
    vec3 colour;
    // Light position is assumed to be in view coordinates
    vec3 position;
    float intensity;
    Attenuation att;
};
```

点光源由一种颜色、一个位置、一个介于$$0$$和$$1$$之间的数字和一组参数来定义,这些参数将模拟其强度和衰减方程.

模拟材料特性的结构是:

```glsl
struct Material
{
    vec4 ambient;
    vec4 diffuse;
    vec4 specular;
    int hasTexture;
    float reflectance;
};
```


一种材料是由一组颜色定义的 \(如果我们不使用纹理给碎片上色\)

* 用于环境组件的颜色.

* 用于漫反射组件的颜色.

* 用于镜面反射组件的颜色.


材质还由一个标志定义,该标志控制材质是否具有关联的纹理和反射率索引.我们将在片段着色器中使用统一方法.

```glsl
uniform sampler2D texture_sampler;
uniform vec3 ambientLight;
uniform float specularPower;
uniform Material material;
uniform PointLight pointLight;
uniform vec3 camera_pos;
```

现在我们创建新统一方法以设置以下变量:


* 环境光:它包含一种颜色,以同样的方式影响每一个碎片.

* 镜面反射率\(当用到镜面反射光时，方程式中使用的指数\).

* 点光源.

* 材料特性.

* 摄影机在视图空间坐标中的位置.


我们还将定义一些全局变量,这些变量将保存要在环境光、漫反射和镜面反射组件中使用的材质颜色组件.我们使用这些变量,是因为如果组件具有纹理,我们将对所有组件使用相同的颜色,并且我们不希望执行冗余的纹理查找,变量定义如下:

```glsl
vec4 ambientC;
vec4 diffuseC;
vec4 speculrC;
```

我们现在可以定义一个函数,根据材料特性设置这些变量:

```glsl
void setupColours(Material material, vec2 textCoord)
{
    if (material.hasTexture == 1)
    {
        ambientC = texture(texture_sampler, textCoord);
        diffuseC = ambientC;
        speculrC = ambientC;
    }
    else
    {
        ambientC = material.ambient;
        diffuseC = material.diffuse;
        speculrC = material.specular;
    }
}
```

现在我们将定义一个函数,它以点光源、顶点位置及其法线作为输入,返回为前面描述的漫射光和镜面反射光组件计算的颜色量.

```glsl
vec4 calcPointLight(PointLight light, vec3 position, vec3 normal)
{
    vec4 diffuseColour = vec4(0, 0, 0, 0);
    vec4 specColour = vec4(0, 0, 0, 0);

    // 漫射光
    vec3 light_direction = light.position - position;
    vec3 to_light_source  = normalize(light_direction);
    float diffuseFactor = max(dot(normal, to_light_source ), 0.0);
    diffuseColour = diffuseC * vec4(light.colour, 1.0) * light.intensity * diffuseFactor;

    // 镜面反射光
    vec3 camera_direction = normalize(-position);
    vec3 from_light_source = -to_light_source;
    vec3 reflected_light = normalize(reflect(from_light_source, normal));
    float specularFactor = max( dot(camera_direction, reflected_light), 0.0);
    specularFactor = pow(specularFactor, specularPower);
    specColour = speculrC * specularFactor * material.reflectance * vec4(light.colour, 1.0);

    // 衰减
    float distance = length(light_direction);
    float attenuationInv = light.att.constant + light.att.linear * distance +
        light.att.exponent * distance * distance;
    return (diffuseColour + specColour) / attenuationInv;
}
```

前面的代码相对简单,它只计算漫反射组件的颜色,另一个计算镜面反射组件的颜色,并通过光在传播到我们正在处理的顶点时所遭受的衰减,对它们进行调制.

请注意,顶点坐标在在可视视野的空间中.在计算镜面反射部件时,我们必须得到到视点的方向,即相机.可以这样做:

```glsl
 vec3 camera_direction = normalize(camera_pos - position);
```

但是,由于`position`在视野空间中,所以相机的位置总是在原点,即$$(0, 0, 0)$$,所以我们这样计算它;

```glsl
 vec3 camera_direction = normalize(vec3(0, 0, 0) - position);
```

还可以这样简化:

```glsl
 vec3 camera_direction = normalize(-position);
```

在前面的函数中,顶点着色器的主函数非常简单.

```glsl
void main()
{
    setupColours(material, outTexCoord);

    vec4 diffuseSpecularComp = calcPointLight(pointLight, mvVertexPos, mvVertexNormal);

    fragColor = ambientC * vec4(ambientLight, 1) + diffuseSpecularComp;
}
```


对`setupColours`函数的调用将用适当的颜色设置`ambientC`, `diffuseC` 和 `speculrC`变量.然后,我们要计算漫反射和镜面反射部件,并考虑到衰减.为了方便起见,我们将使用一个函数调用来实现这一点,上面已经解释过了.最后的颜色是通过添加环境光部件\(将`ambientC`乘以环境光\)来计算的.如你所见,环境光不受衰减的影响.


我们在着色器中引入了一些需要进一步解释的新概念.我们正在定义结构并将其用作统一结构.我们如何构建这些结构？首先,我们将定义两个新类来模拟点光源和材质的属性,它们分别命名为`PointLight`和`Material`.它们只是普通的POJOs,所以你可以在本书附带的源代码中看到它们.然后,我们需要在`ShaderProgram`类中创建新的方法,让其首先能够为点光源和材质结构创建一致性.

```java
public void createPointLightUniform(String uniformName) throws Exception {
    createUniform(uniformName + ".colour");//统一名称
    createUniform(uniformName + ".position");
    createUniform(uniformName + ".intensity");
    createUniform(uniformName + ".att.constant");
    createUniform(uniformName + ".att.linear");
    createUniform(uniformName + ".att.exponent");
}

public void createMaterialUniform(String uniformName) throws Exception {
    createUniform(uniformName + ".ambient");//统一名称
    createUniform(uniformName + ".diffuse");
    createUniform(uniformName + ".specular");
    createUniform(uniformName + ".hasTexture");
    createUniform(uniformName + ".reflectance");
}
```

如你所见,这非常简单,我们只是为构成结构的所有属性创建一个单独的统一名称.现在我们需要创建另外两个方法来设置这些统一结构的值,它们将是参数`PointLight`和`Material`实例.

```java
public void setUniform(String uniformName, PointLight pointLight) {
    setUniform(uniformName + ".colour", pointLight.getColor() );
    setUniform(uniformName + ".position", pointLight.getPosition());
    setUniform(uniformName + ".intensity", pointLight.getIntensity());
    PointLight.Attenuation att = pointLight.getAttenuation();
    setUniform(uniformName + ".att.constant", att.getConstant());
    setUniform(uniformName + ".att.linear", att.getLinear());
    setUniform(uniformName + ".att.exponent", att.getExponent());
}

public void setUniform(String uniformName, Material material) {
    setUniform(uniformName + ".ambient", material.getAmbientColour());
    setUniform(uniformName + ".diffuse", material.getDiffuseColour());
    setUniform(uniformName + ".specular", material.getSpecularColour());
    setUniform(uniformName + ".hasTexture", material.isTextured() ? 1 : 0);
    setUniform(uniformName + ".reflectance", material.getReflectance());
}
```


在本章源代码中,你还将看到,我们还修改了`Mesh`类以保存材质实例,并创建了一个简单的示例,该示例创建了一个点光源,该点光源可以通过使用“N”和“M”键进行移,以显示反射比值大于0的,网格上聚焦的点光源的外观.


让我们回到片段着色器,如前所述,我们需要另一个包含摄像机位置的均匀坐标,摄像机位置.这些坐标必须在视野空间中.通常我们会在世界空间坐标中设置灯光坐标,因此我们需要将它们与视图矩阵相乘,以便能够在着色器中使用它们.因此，我们需要在`Transformation`类中创建一个返回视图矩阵的新方法,以便我们可以转换灯光坐标.

```java
// 获取灯光对象的副本,并将其位置转换为视图坐标.
PointLight currPointLight = new PointLight(pointLight);
Vector3f lightPos = currPointLight.getPosition();
Vector4f aux = new Vector4f(lightPos, 1);
aux.mul(viewMatrix);
lightPos.x = aux.x;
lightPos.y = aux.y;
lightPos.z = aux.z; 
shaderProgram.setUniform("pointLight", currPointLight);
```

我们将不把整个源代码包含进来,因为如果这样做,这一章会太长,并且它对澄清这里解释的概念没有多大用处.如果你想查看完整版本的话,那么可以在本书附带的源代码中查看它.

![Lighting results](./lightning_result.png)

