---
layout: post
title: "Silverlight 3D: Part Two"
date: 2011-06-11 02:47
comments: true
categories: [silverlight, 3D, XNA] 
---
{% img  https://dl.dropbox.com/u/10021156/blog/Ball2.png %}
##Why should you care?
3D Photorealism is nothing new. Plenty of companies have been substituting photos of their product line for creating photo-realistic 3D renditions instead (big cost savings — Apple is a great example of this). The only issue is that these products are typically displayed in 2D on a company website. This means real-time 3D manipulation is not possible outside of a QuickTime plug-in (which usually displays 2D images) or a 2D flipbook of 3D images where all positions are predefined. Enter Silverlight 5 and Babylon Toolkit. Instead of rendering out content as 2D images, the 3D file format (.obj in our case) can be maintained and manipulated during the web app’s runtime thanks to the new 3D features of Silverlight 5 and the extensibility of the Babylon Toolkit.

##Importing 3D Models into your Silverlight 5 project
In our previous post we went through a high-level overview of how to manually implement 3D graphics in Silverlight 5 as well as a discussion about how to program against the GPU. We concluded that there are three ways to implement 3D effects in Silverlight 5.

1 – Plot all your vertices individually (by hand).
2 – Plot your vertices algorithmically.
3 – Import a 3D model into your Silverlight application via a third-party API.

For this next example we will use a 3D modeling app named “Modo” to create a soccer ball model. This model will then be used as 3D content in our Silverlight App.

##3D Apps
Back in the day I used to Modo quite a bit (Here is the only thing I ever made that was worth looking at). For those of you who don’t know Modo, it’s a next-gen 3D modeling, painting, animating and rendering package that is used by many professionals in the 3D industry including Pixar. (Note: Any 3D application that is capable of producing the .OBJ file format will suffice for our purposes:

- 3D Studio Max
- Blender
- Maya
- Sketchup
all seem to be just as capable). If you want to go ahead and create the Soccer Ball using Modo or another 3D program click here for a walkthrough or you can download the whole project here instead.

Final Result
{% img https://dl.dropbox.com/u/10021156/blog/Modo-1024x626.jpg %}

##Babylon
After you have created the model, you will need to download the Babylon Importer Toolkit (big thanks to David Catuhe). The Babylon Toolkit gives us access to XNA-like functionality that would only be accessible in the full version of the XNA game studio. This means we don’t have to go about the process of plotting our vertices individually or even worrying about pixel shaders because Catuhe’s Babylon Toolkit provides abstractions on top of these granular aspects of Silverlight 3D.

##Putting it all together
After you download the toolkit, create a new project and add the assemblies found inside the toolkit to your Silverlight application. Next, import your .OBJ, .MTL (material) files and .JPG (texture files – if necessary) into your silverlight project and set their build action to “resource.”

{% img https://dl.dropbox.com/u/10021156/blog/Build-Action-Resource.png %}

Now you should be able to grab these items as streams and use the Babylon API to call the Import function asynchronously. For a fuller demonstration of this process check out the Babylon sample project of a 3D Viper from Battlestar Galactica and see the code below.

{% codeblock lang:xml XAML Side %}
<Grid x:Name="LayoutRoot" Background="Gray" MouseMove="drawingSurface_MouseMove" MouseLeftButtonDown="LayoutRoot_MouseLeftButtonDown" MouseLeftButtonUp="LayoutRoot_MouseLeftButtonUp" MouseEnter="LayoutRoot_MouseEnter" MouseWheel="LayoutRoot_MouseWheel">
        <DrawingSurface  Name="drawingSurface" Draw="DrawingSurface_Draw" SizeChanged="drawingSurface_SizeChanged"/>
        <StackPanel Name="waitBar" VerticalAlignment="Center" Visibility="Collapsed">
            <TextBlock Text="Loading...Please wait" HorizontalAlignment="Center"/>
            <ProgressBar Height="10" Minimum="0" Maximum="100" Margin="20, 10" Name="waitProgressBar"/>
        </StackPanel>
    </Grid>
{% endcodeblock %}

{% codeblock lang:csharp C# Side %}
namespace Footballs
{
    public partial class MainPage
    {
        Model model;
        bool mouseLeftDown;
        Point startPosition;
        readonly OrbitCamera camera = new OrbitCamera { Radius = 20 };
        StatesManager statesManager;
        GraphicsDevice device;

        public MainPage()
        {
            InitializeComponent();
        }

        private void DrawingSurface_Draw(object sender, DrawEventArgs e)
        {
            device = e.GraphicsDevice;
            if (model == null)
            {  //This will spawn a new thread to grab and initialize our model
                Dispatcher.BeginInvoke(() => InitModel(e.GraphicsDevice));
                statesManager = new StatesManager(e.GraphicsDevice);
            }

            if (model == null)
                return;

            DateTime start = DateTime.Now;

            // States
            statesManager.DepthBufferEnable = true;
            statesManager.CullMode = CullMode.None;
            statesManager.ApplyDepthStencilState();
            statesManager.ApplyRasterizerState();
            e.GraphicsDevice.BlendState = BlendState.AlphaBlend;

      // Draw
     e.GraphicsDevice.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, new Color(0, 0, 0, 0), 1.0f, 0);

          //We use BasicEffect instead of manually using pixel/vertex shader
            foreach (ModelMesh mesh in model.Meshes)
            {
                foreach (BasicEffect basicEffect in mesh.Effects)
                {
                    basicEffect.SceneAmbientColor = new Color(0.3f, 0.3f, 0.3f, 0);
                    basicEffect.World = Matrix.Identity;
                    basicEffect.View = camera.View;
                    basicEffect.Projection = camera.Projection;
                    basicEffect.LightPosition = camera.Position + new Vector3(1, 5, 3);
                    basicEffect.CameraPosition = camera.Position;
                }
            }
            model.Draw();
            camera.ApplyInertia();

            double duration = (double)DateTime.Now.Subtract(start).Ticks / TimeSpan.TicksPerMillisecond;
            int verticesCount = model.Meshes.Sum(v => v.VerticesCount);
            e.InvalidateSurface();
        }

        private void InitModel(GraphicsDevice device)
        {
Stream objStream = Application.GetResourceStream(new Uri("/Footballs;component/Models/Brazile.obj", UriKind.Relative)).Stream;

            ObjImporter importer = new ObjImporter();

            importer.OnImportCompleted += m =>
            {
                waitBar.Visibility = Visibility.Collapsed;
                model = m;

                drawingSurface.Invalidate();
            };

            // Report loading progress
            importer.OnImportProgressChanged += p =>
            {
                waitProgressBar.Value = p;
            };
            waitBar.Visibility = Visibility.Visible;

            //THIS IS WHERE ALL THE MAGIC HAPPENS
 importer.ImportAsync(objStream, GetResourceStream, device, ImportationOptions.Optimize);
        }

        Stream GetResourceStream(string name)
        {
            try
            {
                return Application.GetResourceStream(new Uri(string.Format("/Footballs;component/Models/{0}", name), UriKind.Relative)).Stream;
            }
            catch
            {
                Dispatcher.BeginInvoke(() => MessageBox.Show("Unable to find " + name, "Error", MessageBoxButton.OK));
                return null;
            }
        }

//This is the resizing section
        #region Aspect ratio
        private void drawingSurface_SizeChanged(object sender, SizeChangedEventArgs e)
        {
            ComputeAspectRatio();
        }

        void ComputeAspectRatio()
        {
            camera.AspectRatio = (float)(drawingSurface.ActualWidth / drawingSurface.ActualHeight);
        }

        private void UserControl_Loaded(object sender, RoutedEventArgs e)
        {
            ComputeAspectRatio();
        }
        #endregion

//This is the mouse section
        #region Mouse
        private void drawingSurface_MouseMove(object sender, System.Windows.Input.MouseEventArgs e)
        {
            if (!mouseLeftDown)
                return;

            Point currentPosition = e.GetPosition(drawingSurface);
            camera.InertialAlpha += (float)(currentPosition.X - startPosition.X) * camera.AngularSpeed;
            camera.InertialBeta -= (float)(currentPosition.Y - startPosition.Y) * camera.AngularSpeed;
            startPosition = currentPosition;
        }

        private void LayoutRoot_MouseLeftButtonDown(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            mouseLeftDown = true;
            startPosition = e.GetPosition(drawingSurface);
        }

        private void LayoutRoot_MouseLeftButtonUp(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            mouseLeftDown = false;
        }

        private void LayoutRoot_MouseEnter(object sender, System.Windows.Input.MouseEventArgs e)
        {
            mouseLeftDown = false;
        }

        private void LayoutRoot_MouseWheel(object sender, System.Windows.Input.MouseWheelEventArgs e)
        {
            camera.Radius -= e.Delta * camera.Radius / 1000.0f;
        }
        #endregion
{% endcodeblock %}

## Conclusion
In this tutorial we had a brief overview of how 3D content is traditionally served on the web. We then discussed how Silverlight 5 combined with the Babylon toolkit expedites the process of dynamic content delivery while proceeding to create our own 3D model and host it in a new Silverlight 5 App. Hope this series was helpful. Questions/Comments/Concerns/Cries of Outrage? Feel free to comment.

