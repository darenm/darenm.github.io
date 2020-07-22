---
layout: post
title:  "Win2D and Composition Geometry APIs"
date:   2018-05-02 12:00:00 +0700
categories: win2d uwp composition
---
# Win2D and Composition Geometry APIs

The Windows 10 April 2018 Update includes a new set of APIs in the Composition namespace that support various geometries such as lines, ellipses, rectangles and paths. However, during the insider cycle, it wasn’t possible to create arbitrary paths as there was a reliance on an implementation of IGeometrySource2D that could not be found. Fast forward until today when a [new version of the Win2D package has been released – V1.22.0](https://blogs.msdn.microsoft.com/win2d/2018/05/02/win2d-1-22-0-igeometrysource2d-bugfixes-and-removal-of-8-1-support/) and now we can make use of CanvasGeometry class to create our paths.

Here is quick walk-through on getting started with the Geometry APIs.

1. Create a blank UWP app.

1. Add the **Win2D.UWP** nuget package.

1. Update the MainPage.xaml so that the Grid has the name “RootGrid”:

    ```xml
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"
        x:Name="RootGrid">
    </Grid>
    ```

1. Switch to the **MainPage.xaml.cs** code behind and update the constructor to:

    ```csharp
    public MainPage()
    {
        InitializeComponent();

        Loaded += OnLoaded;
    }
    ```

1. Add the OnLoaded method:

    ```csharp
    private void OnLoaded(object sender, RoutedEventArgs routedEventArgs)
    {
        var c = Window.Current.Compositor;

        // Need this so we can add multiple shapes to a sprite
        var shapeContainer = c.CreateContainerShape();

        // Rounded Rectangle - just the rounded rect properties
        var roundedRectangle = c.CreateRoundedRectangleGeometry();
        roundedRectangle.CornerRadius = new Vector2(20);
        roundedRectangle.Size = new Vector2(400, 300);

        // Need to create a sprite shape from the rounded rect
        var roundedRectSpriteShape = c.CreateSpriteShape(roundedRectangle);
        roundedRectSpriteShape.FillBrush = c.CreateColorBrush(Colors.Red);
        roundedRectSpriteShape.StrokeBrush = c.CreateColorBrush(Colors.Green);
        roundedRectSpriteShape.StrokeThickness = 5;
        roundedRectSpriteShape.Offset = new Vector2(20);

        // Now we must add that share to the container
        shapeContainer.Shapes.Add(roundedRectSpriteShape);

        // Let's create another shape
        var roundedRectSpriteShape2 = c.CreateSpriteShape(roundedRectangle);
        roundedRectSpriteShape2.FillBrush = c.CreateColorBrush(Colors.Purple);
        roundedRectSpriteShape2.StrokeBrush = c.CreateColorBrush(Colors.Yellow);
        roundedRectSpriteShape2.StrokeThickness = 3;
        roundedRectSpriteShape2.Offset = new Vector2(90);
        roundedRectSpriteShape2.CenterPoint = new Vector2(200, 150);
        roundedRectSpriteShape2.RotationAngleInDegrees = 45;

        // Add it to the container - as it is added after the previous shape, it will appear on top
        shapeContainer.Shapes.Add(roundedRectSpriteShape2);

        // Create paths and animate them
        SetupPathAndAnimation(c, shapeContainer);

        // Now we need to create a ShapeVisual and add the ShapeContainer to it.
        var shapeVisual = c.CreateShapeVisual();
        shapeVisual.Shapes.Add(shapeContainer);
        shapeVisual.Size = new Vector2(1000, 1000);

        // Display the shapeVisual
        ElementCompositionPreview.SetElementChildVisual(RootGrid, shapeVisual);
    }
    ```

1. Add the empty (for now) SetupPathAndAnimation:

```csharp
private static void SetupPathAndAnimation(Compositor c, CompositionContainerShape shapeContainer)
{
    // Empty for now!
}
```

1. Compile and run this - you will see the following. I believe this code is pretty self-explanatory.

    ![image](images/image_636608892241803570.png)

1. Now we will add a path and animate it. Add the following classes and enum to the project (these are just some helpers I created):

    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Numerics;
    using Microsoft.Graphics.Canvas.Geometry;

    namespace YourNamespace
    {
        public static class PathBuilderExtensions
        {
            public static CanvasPathBuilder BuildPathWithLines(
                this CanvasPathBuilder builder,
                IEnumerable<Vector2> vectors,
                CanvasFigureLoop canvasFigureLoop)
            {
                var first = true;

                foreach (var vector2 in vectors)
                {
                    if (first)
                    {
                        builder.BeginFigure(vector2);
                        first = false;             }
                    else
                    {
                        builder.AddLine(vector2);
                    }
                }

                builder.EndFigure(canvasFigureLoop);
                return builder;
            }

            public static CanvasPathBuilder BuildPathWithLines(
                this CanvasPathBuilder builder,
                IEnumerable<(float x, float y)> nodes,
                CanvasFigureLoop canvasFigureLoop)
            {
                var vectors = nodes.Select(n => new Vector2(n.x, n.y));
                return BuildPathWithLines(builder, vectors, canvasFigureLoop);
            }
        }

        public class PathNode
        {
            private Vector2 _vector2;

            public PathNode(Vector2 vector2)
            {
                _vector2 = vector2;
            }
        }

        public enum NodeType
        {
            Line,
            Arc,
            CubicBezier,
            Geometry,
            QuadraticBezier
        }
    }
    ```

1. Replace the **SetupPathAndAnimation()** method with:

    ```csharp
    private static void SetupPathAndAnimation(Compositor c, CompositionContainerShape shapeContainer)
    {
        var startPathBuilder = new CanvasPathBuilder(new CanvasDevice());

        // Use my helper to create a W shaped path
        startPathBuilder.BuildPathWithLines(new(float x, float y)[]
            {
                (10, 10), (30, 80), (50, 30), (70, 80), (90, 10)
            },
            CanvasFigureLoop.Open);

        // Add another path
        startPathBuilder.BuildPathWithLines(new(float x, float y)[]
            {
                (105, 30), (105, 80)
            },
            CanvasFigureLoop.Open);

        // Create geometry and path that represents the start position of an animation
        var startGeometry = CanvasGeometry.CreatePath(startPathBuilder);
        var startPath = new CompositionPath(startGeometry);

        // Now create the end state paths
        var endPathBuilder = new CanvasPathBuilder(new CanvasDevice());
        endPathBuilder.BuildPathWithLines(new(float x, float y)[]
            {
                (10, 10), (30, 10), (50, 10), (70, 10), (90, 10)
            },
            CanvasFigureLoop.Open);

        endPathBuilder.BuildPathWithLines(new(float x, float y)[]
            {
                (105, 30), (105, 80)
            },
            CanvasFigureLoop.Open);

        var endGeometry = CanvasGeometry.CreatePath(endPathBuilder);
        var endPath = new CompositionPath(endGeometry);

        // Create a CompositionPathGeometery from the Win2D GeometeryPath
        var pathGeometry = c.CreatePathGeometry(startPath);

        // Create a CompositionSpriteShape from the path
        var pathShape = c.CreateSpriteShape(pathGeometry);
        pathShape.StrokeBrush = c.CreateColorBrush(Colors.Purple);
        pathShape.StrokeThickness = 5;
        pathShape.Offset = new Vector2(20);

        // Add the pathShape to the ShapeContainer that we used elsewhere
        // This will ensure it is rendered
        shapeContainer.Shapes.Add(pathShape);

        // Create an animation using the start and endpaths
        var animation = c.CreatePathKeyFrameAnimation();
        animation.Target = "Geometry.Path";
        animation.Duration = TimeSpan.FromSeconds(1);
        animation.InsertKeyFrame(0, startPath);
        animation.InsertKeyFrame(1, endPath);
        animation.IterationBehavior = AnimationIterationBehavior.Forever;
        animation.Direction = AnimationDirection.AlternateReverse;
        pathGeometry.StartAnimation(nameof(pathGeometry.Path), animation);
    }
    ```

1. Compile and run the code - you will notice that two paths are now being rendered and the W is animating.

    ![image](images/image_636608892246293316.png)

This is obviously a very simple example of using the new Composition Geometery APIs, but should be enough to get you started.