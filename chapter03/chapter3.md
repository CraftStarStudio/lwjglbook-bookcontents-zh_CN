# A brief about coordinates

In this chapter we will talk a little bit about coordinates and coordinate systems trying to introduce some fundamental mathematical concepts in a simple way to support the techniques and topics that we will address in subsequent chapters. We will assume some simplifications which may sacrifice preciseness for the sake of legibility.
在本章中，我们将讨论一点坐标和坐标系的内容，以简单的方式介绍一些基本的数学概念，以支持我们将在后续章节中讨论的技术和主题。我们会将假设一些简化，为了易读性而牺牲一些精确性。

We locate objects in space by specifying their coordinates. Think about a map. You specify a point on a map by stating its latitude or longitude. With just a pair of numbers a point is precisely identified. That pair of numbers are the point coordinates (things are a little bit more complex in reality, since a map is a projection of a non perfect ellipsoid, the earth, so more data is needed but it’s a good analogy).
我们通过指定物体的坐标来定位空间中的物体。想象一下画一张地图，在地图上指定一个点是通过说明它的纬度或经度，只要一对数字，一个点就可以被精确地识别出来；这对数字就是点坐标（实际上对会情况稍微复杂一点，因为地图是一个非完美椭球体地球的投影，所以需要更多的数据，但这依然是一个很好的类比）。

A coordinate system is a system which employs one or more numbers, that is, one or more components to uniquely specify the position of a point. There are different coordinate systems (Cartesian, polar, etc.) and you can transform coordinates from one system to another. We will use the Cartesian coordinate system.
坐标系是使用一个或多个数字，即一个或多个分量来唯一指定点位置的系统。有不同的坐标系（笛卡尔坐标系、极坐标系等），可以将坐标从一个坐标系转换到另一个坐标系。我们将会使用笛卡尔坐标系。

In the Cartesian coordinate system, for two dimensions, a coordinate is defined by two numbers that measure the signed distance to two perpendicular axes, x and y.
在笛卡尔坐标系中，对于二维，坐标是由两个数字定义对，这两个数字表示测量到的两个垂直轴符号距离（x和y）。

![Cartesian Coordinate System](cartesian_coordinate_system.png) 

Continuing with the map analogy, coordinate systems define an origin. For geographic coordinates the origin is set to the point where the equator and the zero meridian cross. Depending on where we set the origin, coordinates for a specific point are different. A coordinate system may also define the orientation of the axis. In the previous figure, the x coordinate increases as long as we move to the right and the y coordinate increases as we move upwards. But we could also define an alternative Cartesian coordinate system with different axis orientation in which we would obtain different coordinates.
让我们继续用地图来类比坐标系定义原点，对于地理坐标，原点设置为赤道和零子午线相交的点。根据我们设置原点的位置，特定点的坐标是不同的，坐标系也可以定义轴的方向。在上一个图中，只要向右移动，x坐标就会增加，而向上移动，y坐标就会增加。但我们也可以定义一个不同轴方向的笛卡尔坐标系，在这个坐标系中我们便可以得到不同的坐标。

![Alternative Cartesian Coordinate System](alt_cartesian_coordinate_system.png)

As you can see we need to define some arbitrary parameters, such as the origin and the axis orientation in order to give the appropriate meaning to the pair of numbers that constitute a coordinate. We will refer to that coordinate system with the set of arbitrary parameters as the coordinate space. In order to work with a set of coordinates we must use the same coordinate space. The good news is that we can transform coordinates from one space to another just by performing translations and rotations.
如你所见，我们需要定义一些任意参数，例如原点和轴方向，以便为构成坐标的一对数字赋予适当的含义。我们将引用具有任意参数集的坐标系作为坐标空间。为了使用一组坐标，我们必须使用相同的坐标空间。好消息是我们可以通过执行平移和旋转将坐标从一个空间转换到另一个空间。

If we are dealing with 3D coordinates we need an additional axis, the z axis. 3D coordinates will be formed by a set of three numbers (x, y, z). 
 
![3D Cartesian Coordinate System](3d_cartesian_coordinate_system.png)

As in 2D Cartesian coordinate spaces we can change the orientation of the axes in 3D coordinate spaces as long as the axes are perpendicular. The next figure shows another 3D coordinate space.
 
![Alternative 3D Cartesian Coordinate System](alt_3d_cartesian_coordinate_system.png)

3D coordinates can be classified in two types: left handed and right handed. How do you know which type it is? Take your hand and form a “L” between your thumb and your index fingers, the middle finger should point in a direction perpendicular to the other two. The thumb should point to the direction where the x axis increases, the index finger should point where the y axis increases and the middle finger should point where the z axis increases. If you are able to do that with your left hand, then its left handed, if you need to use your right hand is right-handed.

![Right Handed vs Left Handed](righthanded_lefthanded.png) 

2D coordinate spaces are all equivalent since by applying rotation we can transform from one to another. 3D coordinate spaces, on the contrary, are not all equal. You can only transform from one to another by applying rotation if they both have the same handedness, that is, if both are left handed or right handed.

Now that we have defined some basic topics let’s talk about some commonly used terms when dealing with 3D graphics. When we explain in later chapters how to render 3D models we will see that we use different 3D coordinate spaces, that is because each of those coordinate spaces has a context, a purpose. A set of coordinates is meaningless unless it refers to something. When you examine this coordinates (40.438031, -3.676626) they may say something to you or not. But if I say that they are geometric coordinates (latitude and longitude) you will see that they are the coordinates of a place in Madrid.

When we will load 3D objects we will get a set of 3D coordinates. Those coordinates are expressed in a 3D coordinate space which is called object coordinate space. When the graphics designers are creating those 3D models they don’t know anything about the 3D scene that this model will be displayed in, so they can only define the coordinates using a coordinate space that is only relevant for the model.

When we will be drawing a 3D scene all of our 3D objects will be relative to the so called world space coordinate space. We will need to transform from 3D object space to world space coordinates. Some objects will need to be rotated, stretched or enlarged and translated in order to be displayed properly in a 3D scene.

We will also need to restrict the range of the 3D space that is shown, which is like moving a camera through our 3D space. Then we will need to transform world space coordinates to camera or view space coordinates. Finally these coordinates need to be transformed to screen coordinates, which are 2D, so we need to project 3D view coordinates to a 2D screen coordinate space.

The following picture shows OpenGL coordinates, (the z axis is perpendicular to the screen) and coordinates are between -1 and +1.

![OpenGL coordinates](opengl_coordinates.png) 

Don’t worry if you don’t have a clear understanding of all these concepts. They will be revisited during next chapters with practical examples. 

