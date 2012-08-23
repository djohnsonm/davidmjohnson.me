---
layout: post
title: "Silverlight 3D: Part One"
date: 2011-05-26 16:17
comments: true
categories: [silverlight, 3D, XNA, .NET]
---
{% img https://dl.dropbox.com/u/10021156/blog/Ball.png %}
##The Silverlight 5 Beta
It’s no secret that Microsoft has released the beta version of Silverlight 5 with many new and surprising features. But what might be unfamiliar territory for some is the inclusion of a low-level 3D API based on the XNA stack. That’s right, the core of the XNA Game Studio is now built into Silverlight (with some caveats of course). “But isn’t XNA just for games?” Mostly, yes. But that hasn’t stopped Microsoft from allowing developers to get creative. Check out John Papa’s demo at MIX11 of a 3D House Builder Simulator. 3D Product Visualization is now a reality and is a viable solution for any developer wanting to spice up content delivery. If you aren’t convinced take a look at the SLARToolkit (Silverlight Augemented Reality) for more cool effects, or the 3D rendition of Scott Guthrie way back at the firestarter event. If that still hasn’t caught your attention then stop reading this post. But for the rest of you, we’ll go over what it takes to build and display 3D graphics in SL5.

##Three ways to build 3D graphics in Silverlight 5:
Create your models by hand (by plotting each individual vertex) this can be is very tedious.
Create your model algorithmically (Check out the Solar Winds Project)
Import your model from a third-party modeling tool

If you have already developed XNA applications/games you will find that XNA for Silverlight is very similar, albeit there are some differences. Silverlight 5 provides a new User Control named “Drawing Surface” which includes the “Draw” event. This renders all 3D content on a separate thread known as the “composition” thread. This also updates all projection, view and world matrices which are responsible for positioning things like the camera and the model in 3D space. So let’s get started creating our own polygon in 3D space. Note: This is part one of a two-part series. In this series we will focus on the theory and abstract concepts behind the creation of 3D graphics in Silverlight 5 to produce a simple triangle. Part two will be much cooler.

##Essential Resources
In order to complete this tutorial you will need to have the following on your machine:
DirectX SDK
Visual Studio 2010
Silverlight 5 Beta
Microsoft.Xna.Framework.Math.dll

## Five Steps
At a high level there are 5 basic steps to follow in order to render 3D graphics in Silverlight 5 (Straight from the .CHM).
1. Setup the initial project
2. Create a structure to hold vertex data (includes VertexDeclaration), instantiate the struct, and pass it into your Vertex Buffer.
3. Create World view, Camera view, and Projection space matrices.
4. Create PixelShader and VertexShader objects.
5. Raise the Draw Event.

This may all sound complicated initially, but as you continue to read it will begin to make sense. Also, this is meant to be a crash-course guide to XNA and SL5. If you want real depth I recommend you check out this book. So let’s get started.

##Getting Started with the 3D API – Your first triangle.
Create a new Silverlight Project named “FirstTriangle” make sure you check that it is a Silverlight 5 application. Make sure to add a web project as well.

###Step 1: Turn it on
By default GPU Acceleration is not enabled, so go ahead and include this extra param tag into your Triangle web application. You will be unable to render graphics without it.

{% codeblock lang:xml Turn it on %}
<param name="EnableGPUAcceleration" value="true" />
    <!--  Set other parameters here  -->
{% endcodeblock %}

{% codeblock lang:xaml Add the control, Add the Event Handler, Declare your variables %}
<Grid x:Name="LayoutRoot" Background="Black">
    <DrawingSurface Name="surface" Draw="surface_Draw"  Loaded="surface_Loaded" />
</Grid>
{% endcodeblock %}

The two events we will be focusing on in this application primarily are the “Draw” event and the “Loaded” event. Also, declare these variables in your MainPage.xaml.cs class. We’ll dive into what they mean in a little later.

{% codeblock lang:c# Shaders and Buffers %}
VertexShader vs;
PixelShader ps;
VertexBuffer vb;
Matrix viewproj;
{% endcodeblock %}

###Step 2: Create Structure to hold Vertex Data
Before we can go any further it’s important to note exactly what we’re doing when we program against the GPU. Check out the image below to see the pipeline of just how the vertices we describe get displayed up on the screen.

{% img https://dl.dropbox.com/u/10021156/blog/XNA-explanation-227x300.png %}

{% blockquote %}
“Points in three-dimensional space are represented as a 3-component vector called a vertex. A vertex contains an x, y, and z component. Vertices are connected together to form primitives, such as triangles, which are defined with three vertices. Multiple triangles can be combined together into lists or stripes which from a mesh. Meshes of triangles are combined together to create complex shapes. For example, a square can be drawn using two triangles. ”
{% endblockquote %}

###Vertex Information
Vertex information is typically stored in a structure which may include data such as position, color, and texture coordinates. So in your MainPage.xaml.cs add this struct below our MainPage class. This struct defines the type of content our vertices will hold (in this case only Position and Color data). Our struct will expose a static, read-only, Vertex Declaration that will be passed into the VertexBuffer. A VertexDeclaration defines the layout of the vertex data.

{% codeblock lang:c# VertexDeclaration %}
public struct VertexPositionColor
    {
        public Vector3 Position;
        public Color Color;
 
        public VertexPositionColor(Vector3 Position, Color Color)
        {
            this.Position = Position;
            this.Color = Color;
        }
 
        public static readonly VertexDeclaration VertexDeclaration = new VertexDeclaration(new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),new VertexElement((sizeof(float) * 3), VertexElementFormat.Color, VertexElementUsage.Color, 0)); }
{% endcodeblock %}

The VertexDeclaration takes in four parameters. The first corresponds to the offset. Your first VertexElement should not contain an offset since it is the first element. The next VertexElement should be offset by 12 bytes, or by the size of VertexElement (which was a Vector3 in our case). Note: Vector3 contains 3 floats. One float is 4 bytes, Therefore, 4 bytes * 3 floats = 12 bytes… that’s where the 12 is coming from. (Note: Vector3 is in the Microsoft.XNA.Framework.Math.dll, hopefully this will be included in Silverlight 5 too). So, each vertex will contain information about position as well as color. (You can also map textures to 3D objects as well using another VertexElementFormat of type “Texture” – We will dive into this in Part 2).

You can think of this struct as your data model, describing how your vertex data will be interpreted by the GPU. You can also think of the instantiation of this struct as your view model. So we will instantiate this struct inside of our surface_loaded event handler to begin the initialization process.

###Vertex Buffer
{% codeblock lang:c# Vertex Buffer %}
private void surface_Loaded(object sender, RoutedEventArgs e)
        {
//get device
GraphicsDevice device = GraphicsDeviceManager.Current.GraphicsDevice;
 
 //Declare struct
 VertexPositionColor[] vertices = new VertexPositionColor[3];
 
//Plot your verts
vertices[0] = new VertexPositionColor(new Vector3(0, 1, 0) /*top*/, new Color(255, 0, 0, 0) /*red*/);
vertices[1] = new VertexPositionColor(new Vector3(-1, -1, 0) /*left*/, new Color(0, 255, 0, 0) /*green*/);
vertices[2] = new VertexPositionColor(new Vector3(1, -1, 0) /*right*/, new Color(0,0, 255, 0) /*blue*/);
 
//Assign it to buffer
vb = new VertexBuffer(device, VertexPositionColor.VertexDeclaration, 0, BufferUsage.WriteOnly);
vb.SetData(0, vertices, 0, vertices.Length, 0);
 
//Set Camera
view = Matrix.CreateLookAt(new Vector3(0, 0, 5), Vector3.Zero, Vector3.Up); //defines the position of the camera in 3D space
projection = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, 1.667f, 1, 100); //viewing angle, aspect ratio, near and far axis
viewproj = view * projection; //This represents our world space which will be passed into our vertex shader.
 
//Set up Shaders
ps = PixelShader.FromStream(device, Application.GetResourceStream(new Uri(@"FirstTriangle;component/Triangle.ps", UriKind.Relative)).Stream);
vs = VertexShader.FromStream(device, Application.GetResourceStream(new Uri(@"FirstTriangle;component/Triangle.vs", UriKind.Relative)).Stream);
        }
{% endcodeblock %}

###Graphics Device
The GraphicsDevice is a working representation of your GPU, so grab it. Next, you will create a vertex array containing three vertices to represent your triangle. The code should be self-explanatory. The next two lines of code create a new VertexBuffer and assign the structure of our triangle to it. A VertexBuffer is a data buffer that contains a VertexDeclaration which describes the buffer layout to the graphics device. The vertex buffer also contains all the vertex data.

{% blockquote %}
In a 3D scene there are typically multiple views that must be maintained. Three common views are the world space, view space, and projection space. These are defined as 4×4 matrices. The world space is the main point of reference. It is the space that objects are placed in. The view space is composed of three components. The position of the camera in world space, the coordinates in world space that the camera is looking at, and the direction that is up. For example, if the first component was 0,0,5, then this indicates that camera is at position (0,0,5). If the second component was 0,0,0, (Vector3.Zero) then this indicates that the camera is pointing towards the origin of world space. Finally, if the final component is 0,1,0, (Vector.Up) then this indicates that the up direction of the camera is along the positive y-axis. The final space is the projection matrix. This is the actual region, within the view space, that the camera sees. The space is defined as a frustum. A frustum is like a pyramid that has the pointed portion cut off leaving a flat plane. Everything in front of the front plane of the frustum is clipped and everything behind the back plane of the frustum is clipped. So, only the objects within the region between the front and back planes are rendered.”
{% endblockquote %}

The first parameter, MathHelper.PiOver4, corresponds to .785 radians which is roughly 45 degree field of view, The next parameter is 1.667f which corresponds to a 4:3 aspect ratio. 1 represents the near distance plane and 10 the far distance plane. Only vertices between these two numbers will be viewable.

{% img https://dl.dropbox.com/u/10021156/blog/viewingFrustrum.png An illustrated look at the viewing frustrum  %}

###Step 4: Working with shaders
After we multiply our view and projection matrices we yield world space. This space will then be passed into what is known as a vertex shader. A vertex shader is a graphics processing function used to add special effects to objects in a 3D environment. Vertex shaders are run once for each vertex given to the graphics processor. The purpose is to transform each vertex’s 3D position in virtual space to the 2D coordinate at which it appears on the screen (as well as a depth value for the Z-buffer). Vertex shaders can manipulate properties such as position, color, and texture coordinate, but cannot create new vertices. The output of the vertex shader goes to the next stage in the pipeline, which is either a geometry shader if present or the rasterizer otherwise. Similarly, A pixel shader is a computation kernel function that computes color and other attributes of each pixel. Pixel shaders range from always outputting the same color, to applying a lighting value, to doing bump mapping, shadows, specular highlights, translucency and other phenomena. For our purposes we will define a basic vertex and pixel shader using HLSL (High Level Shader Language) that will not perform any additional manipulations but simply pass our data through for rendering. Here is the code below:

{% codeblock lang:hlsl Triangle.vs.hlsl %}
// transformation matrix provided by the application
float4x4 WorldViewProj : register(c0);
 
// vertex input to the shader
struct VertexData
{
  float3 Position : POSITION;
  float4 Color : COLOR;
};
 
// vertex shader output passed through to geometry
// processing and a pixel shader
struct VertexShaderOutput
{
  float4 Position : POSITION;
  float4 Color : COLOR;
};
 
// main shader function
VertexShaderOutput main(VertexData vertex)
{
  VertexShaderOutput output;
 
  // apply standard transformation for rendering
  output.Position = mul(float4(vertex.Position,1), WorldViewProj);
 
  // pass the color through to the next stage
  output.Color = vertex.Color;
 
  return output;
}

{% endcodeblock %}

{% codeblock lang:hlsl Triangle.pl.hlsl %}
struct VertexShaderOutput
{
  float4 Position : POSITION;
  float4 Color : COLOR;
};
 
// main shader function
float4 main(VertexShaderOutput vertex) : COLOR
{
  return vertex.Color;
}

{% endcodeblock %}

###Compile
These shaders will then need to be compiled using fxc.exe (part of the DirectX SDK) and imported into your silverlight application. Make sure you set them with a build action of “resource.”
Next, find your fxc.exe application. Mine is located here: C:\Program Files (x86)\Microsoft DirectX SDK (June 2010)\Utilities\bin\x64.
so open up a command prompt and type

{% codeblock %}
cd C:\Program Files (x86)\Microsoft DirectX SDK (June 2010)\Utilities\bin\x64
{% endcodeblock %}

This will navigate you to the fxc.exe directory.
Now in order to compile your vertex shader you should execute this command

{% codeblock %}
Fxc.exe /T ps_2_0 /O3 /Zpr /Fo 
C:\users\djohnson\desktop\Triangle.ps 
C:\users\djohnson\desktop\Triangle.ps.hlsl
{% endcodeblock %}

###Note
Note: Since my HLSL files are located on my desktop I am referencing them there as well as projecting a Triangle.ps file for output. (This is the pixel shader extension and the file that should be created if you executed the command properly. Do the same with the vertex shader).
Command to compile a Vertex Shader using FXC.exe

{% codeblock %}
C:\Program Files (x86)\Microsoft DirectX SDK (June 2010)\Utilities\bin\x64\Fxc.exe /T vs_2_0 /O3 /Zpr /Fo 
C:\users\djohnson\desktop\Triangle.vs 
C:\users\djohnson\desktop\Triangle.vs.hlsl
{% endcodeblock %}

(Note: the vs_2_0 in this command is differen than the ps_2_0 in the previous)
After they are built import both the .ps and .vs files into your silverlight project and make sure their build action is set to resource. But if you are really lazy and don’t want to compile your own shaders you can just grab my pre-compiled shaders and import them here.

###Step 5
{% codeblock lang:csharp The Draw Event - Bringing it all together %}
private void surface_Draw(object sender, DrawEventArgs e)
     {
         GraphicsDevice device = e.GraphicsDevice;
         // Clear the GraphicsDevice
         device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Transparent, 10.0f, 0);
         //Set up device device
         device.SetVertexBuffer(vb);
         device.SetVertexShader(vs);
         device.SetPixelShader(ps);
         device.SetVertexShaderConstantFloat4(0, ref viewproj);
         device.DrawPrimitives(PrimitiveType.TriangleStrip, 0, vb.VertexCount / 3);
         e.InvalidateSurface();
     }
{% endcodeblock %}

The Draw event handler is where all action happens. In this handler we are once again grabbing the current graphics device, but this time we are invoking the clear method before we call our Draw Primitives. (Note: The Draw method runs in a loop similar to an XNA game loop rendering out each frame individually). Next, we pass in our VertexBuffer (which contains the vertex data) into our device. We then set both the vertex shader and pixel shader (which describe how our content is displayed pre-rasterization and post-rasterization). Finally, we pass in our view projection matrix into our vertex shader and call the DrawPrimitives method. In the Draw primitives method we are using “Triangle Strip.” This will create a polygon from the last vertice that was added in 3D space. This is accomplished by analyzing the two previous vertex positions and attempting to create a triangular polygon utilizing those positions. Invalidating the surface will invoke the draw method to be called again and will re-render out the content with our without any geometry transformations. (Geometry transformations can be accomplished through manipulation of the view and projection matrices, please see the third tutorial for more information on that).

If all goes according to plan you should have your first Triangle mesh drawn into 3D space.


{% img https://dl.dropbox.com/u/10021156/blog/TriangleResult-300x261.png All that hard work for a simple triangle %}

If you have any issues here is the source code.

Let me know if you have any questions comments concerning any of the info above. Cya in Part 2.

