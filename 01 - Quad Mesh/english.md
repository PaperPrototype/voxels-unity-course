# Introduction

Pixel
> Picture Element

Voxel
> Volume Element

# Examples
A voxel is the futuristic version of a pixel. Voxels are essentially 3D pixels. The ultimate game worlds of the future are made out of "voxels". 

[Teardown](https://teardowngame.com/) is a game where everything is "voxels". The voxel system they built is fully destructible.

![Teardown preview](/Assets/teardown.jpg)

[Minecraft](https://minecraft.net) is one of the most popular voxel games. Although it doesn't really use "voxels" as you'll find out soon (PS: it uses "meshes" to fake it).

![Minecraft preview](/Assets/minecraft.jpg)

In order to have a world made of voxels we have to come up with a way of making the pixels on our screen have a picture that looks like... well voxels... on other words a method of "rasterizing".

# Voxel Rendering

Voxel Rasterizing
> Using voxel volume data (thik of a 3D pixel image format).
> Requires a custom renderer to convert the data into pixels on our screen, thus it is much harder to build.
>
> Can utilize Raytracing to speed things up, which I believe Teardown uses.

Triangle "mesh" Rasterizing
> Get 3 positions in 3D space. draw lines between them to build the edges of a triangle. Now figure out where to fill in the pixels that the triangle takes up on the screen.
>
> Built into almost all graphics cards, hence it is very fast. This is what minecraft does, and what we will be doing.

To "Rasterize"
> To take data, and use it to decide where to fill in the pixels on a screen to build an image.

The most widely used rendering method for games (by far) is Triangle Rasterization. Most likely your favorite game is made using a triangle based renderer of some sort. 

If you've ever zoomed really close to something in a game, you might have noticed that the object was not as smooth as you thought it was! That is because it was probably made of triangles, or a "mesh" of triangles! 

On the other hand a Voxel Rasterizer is fundamentally limited in that graphics cards don't have specialized hardware for filling in pixels based on volume data (remember, 3D pixel data) like they do for triangles. Graphics cards literally have a processor just for taking 3 positions on the X and Y on your screen, and filling those in with colored pixels!

So we either have to write our own voxel rasterizer/renderer that can somehow use what our graphics card gives us (probably some sort of hybrid system that still uses meshes), or make our "voxels" straight out of triangles like minecraft. Unity also has a `Mesh` class and renderer that cna render meshes that is really easy to use.

So triangles it is. Lets start at the beginning and make a single renderable triangle.

# Vertices
We group triangles together and call them a mesh. In Unity the only way to render a triangle is to build a "mesh" and put it into a `MeshFilter` component. 

We will start with making a square Mesh using 2 triangles.

To make a triangle in 3D space we have to define 3 positions that will become the 3 corners of the triangle. Each position is called a "Vertex" and when talking about meshes these are referred to as "Vertices". Even though this is a weird name, don't forget that they are really just a 3D position.  You may also hear the word "point" used, just remember these words are all referring to the same idea.

In code vertices for a triangle would look something like this.

(This is example code. No need to create a script in Unity yet.)
```cs
mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, -1, 0) }
```

A `Vector3` is how Unity represents positions. A `Vector3` just holds 3 `float`s.

If we make a simple graph and put the vertices from the example code onto it, you would see something like this.

![three vertices](/Assets/three_vertices.png)

# Triangles ("connecting" our vertices)
We have 3 vertices, now how do we tell Unity to render a triangle using those 3 vertices? 

In code, we can literally say get the first vertex, the second one, and the third one, and fill those 3 vertices in to make a triangle.

(Pseudo code)
```cs
mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, -1, 0) }
mesh.triangles = { 0, 1, 2 }
```

If you look at our code that is exactly what we did! These "triangle" numbers are just indices telling us where in the list we can find 3 vertices (the "first" vertex is at an offset of `0`, since, to access items in a list, we use a offsets).

These "triangle" numbers have to be in sets of 3. So if I had say `0, 1, 2, 1` the last number wouldn't belong to any "triangle". 

(If did try using 4 numbers Unity would give us an error)

The above mesh, if "rendered" by Unity's renderer, would yield a triangle much like this:

![triangle mesh](/Assets/triangle_mesh.png)

# Quad
A "quad" is just the same as a square, but, it lives in 3D space. A "square" is a 2D object that exists only within 2D space; hence we will be using the word Quad.

Currently we have 3 vertices, so to make a Quad we need 1 more vertex in the upper right corner.

```cs
mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, -1, 0), Vector3(1, 1, 0) }
```

![four vertices](/Assets/four_vertices.png)

Using the new vertex, and reusing 2 of the previous vertices, and adding 3 more numbers to our triangles list, we can make another "triangle".

```cs
mesh.vertices = 
{ 
	Vector3(-1, -1, 0), 
	Vector3(-1, 1, 0), 
	Vector3(1, -1, 0), 
	Vector3(1, 1, 0), // new vertex
}

mesh.triangles = { 0, 1, 2,  1, 3, 2 }
//                           \__second set of "triangle" numbers
```

If we rendered the resulting quad mesh it would look like this:

![quad mesh](/Assets/quad_mesh.png)

# Normals
In order for the renderer to know which direction the surface of our quad is facing we have to give each vertex a "normal".

A normal is the direction that is "upwards" from our surface (usually at a 90 degree angle).

![correct normal](/Assets/correct_normal.png)

You can also make the normals face away from the surface at a non-90 degree angle, which in Unity (and most game engines) would make the shading do a "smoothing" between the 2 normals.

![incorrect normal](/Assets/incorrect_normal.png)

This could be useful if we wanted simulate a smooth surface in our mesh.

In code we simply add a normal for each vertex.

```cs
mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, -1, 0), Vector3(1, 1, 0) }
mesh.triangles = { 0, 1, 2,  1, 3, 2 }

mesh.normals = { Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1), Vector3(0, 0, 1) }
```

The normals in the above code are all facing in the positive z direction.

One thing I should mention is that normals usually have a "mangitude" of 1.

![normal magnitude](/Assets/normal_magnitude.png)

This just means that the distance from the start to the end of the `Vector3` should be `1`. 

Any vector (Vector2 or Vector3) with a magnitude of 1 is called a *unit vector*. This definition is only useful if you get into deeper vector mathematics, but I'm telling you its meaning in case if you hear someone using the word.

Since normals represent a "direction" they don't really have a "position". Thus, mathematically it would be pretty silly to try to get a position out of them. Instead you would use the vertex of that normal if you wanted to get a position.

# mesh.RecalculateNormals
Rather than calculating all the normals ourselves we can use Unity's builtin method that generates normals for us.

```cs
mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, -1, 0), Vector3(1, 1, 0) }
mesh.triangles = { 0, 1, 2, 1, 3, 2 }

mesh.RecalculateNormals()
```

Don't worry this is still fake code so you don't have to worry about adding this to a script (yet). Although the `RecalculateNormals` method does exist like this in Unity.

To use the `RecalculateNormals` method it requires us to already have the `vertices` and `triangles` set in our mesh.

The `RecalculateNormals` method will use the "order" we put the triangles in to determine which side of the triangle the normals shoudl be on.

Clockwise will make the normals face us

(images taken from [forum.unity.com/threads/unity-has-a-clockwise-winding-order](https://forum.unity.com/threads/unity-has-a-clockwise-winding-order.129923/#post-3198466))

![clockwise triangle order](/Assets/clockwise_triangle.png)

... and counter-clockwise will make them face away from us.

![counter clockwise triangle](/Assets/counter_clockwise_triangle.png)

When a triangles normals are facing away from us, that side of the triangle doesn't get rendered. This is called "backface culling". 

Backface culling makes it so that only 1 side of a triangle gets rendered. Triangles with normals that face away from the player won't usually be seen (unless the game has mirrors in it), so the renderer only renders triangles whose normals *are* facing the player, thus reducing the number of triangles that have to be rendered, and giving you better performance. Most modern renderers have backface culling. 

Blender does not have backface culling on by default, so you can see both sides of the meshes while modeling in blender.

# Making the Quad
In our voxel project in Unity make a new Folder called "Quad". Then create a new Scene and Script both called "Quad".

![project assets quad](/Assets/project_assets_quad.png)

Open the Quad scene and create a new empty GameObject. Name it "Quad" and drag Quad.cs (our script) onto it.

![hierarchy quad](/Assets/hierarchy_quad.png)

Double click the Quad.cs script to open it.

> Why do I keep saying "Quad.cs" even though the script file is called "Quad"? First because it helps distinguish between the Scene and the Script and the GameObject (since we names them all the same). Also its because Unity (for some reason) hides the ".cs" in the names of files, but, if you open our script in Finder or File Explorer, our script does indead have ".cs" in its name!

The Quad GameObject will need a MeshRenderer component and a MeshFilter component. We can force Unity to have those on the Quad GameObject by adding a `RequireComponent` attribute above our `Quad` class in our Quad.cs script.

```cs
using UnityEngine;

[RequireComponent(typeof(MeshRenderer))] // added this
[RequireComponent(typeof(MeshFilter))] // added this
public class Quad : Monobehaviour
// ... unchanged code, just leave it alone for now
```

Always make sure to save the script by clicking `cmd + s` on mac or `ctrl + s` on windows (you can also go to `file > save` and click save).

Now in `Start()` in Quad.cs add the following code.

```cs
    private void Start() // leave this alone
    {
		// add this...
		var filter = gameObject.GetComponent<MeshFilter>();

		// and all of this...
        var mesh = new Mesh();
        mesh.vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, -1, 0), new Vector3(1, 1, 0) };
        mesh.triangles = new int[] { 0, 1, 2, 1, 3, 2 };

		// and this...
        mesh.RecalculateNormals();

		// and this...
        filter.mesh = mesh;
	}
```

As you can see it is almost identical to our original example code.

You may also notice the word `var`. Thats actually a shortcut for creating variables!

Instead of writing `MeshFilter` or `Mesh` in front of our variables, like this...

```cs
    private void Start()
    {
		MeshFilter filter = gameObject.GetComponent<MeshFilter>();

        Mesh mesh = new Mesh();
        mesh.vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, -1, 0), new Vector3(1, 1, 0) };
        mesh.triangles = new int[] { 0, 1, 2, 1, 3, 2 };

        mesh.RecalculateNormals();

        filter.mesh = mesh;
	}
```

We can just type `var` in instead and be done with it!

We also use Unity's builtin `RecalculateNormals` method, to calculate our normals, rather than manually doing them ourselves.

If you click play in Unity (make sure to save) then you should see a pink square!

![pink square](/Assets/pink_square.png)

Unity is making this square pink to let us know "hey, this thing needs a material before I can render it, so I'm gonna make it pink for now."

Make a new material in the Assets folder called Voxel.

![new material](/Assets/new_material.png)

Click the material and set "Metallic" and "Smoothness" to zero (this will get rid of any shiny-ness).

Now just drag it onto the Quad GameObject. And click play and the Quad should be white colored!

If you click on the Quad GameObject then click the `f` key (f stands for "focus"), you should then be able to hold the `option` key and orbit around it using the mouse.

# UV's
Almost every mesh in a game has a texture.

A texture is just an image. The renderer will take care of 'stretching' the textures onto our triangles, all we have to do is tell it which vertex a specific part of our texture should "map" to. 

A texture "UV size" in Unity is always in the range of 0 to 1, regardless of the texture's actual size in pixels.

![2D uv texture](/Assets/2D_uv_texture.png)

A "UV" in Unity is represented as a `Vector2`.

A UV of `Vector2(0, 0)` would get the bottom left corner of our texture.

![2D uv texture coordinate](/Assets/2D_uv_texture_coordinate.png)

How can we use this map texture coordinates to a vertex? Well, each vertex in our mesh needs a corresponding UV. In our example code, to texture the triangle it would look like this.

```cs
mesh.vertices = { Vector3(-1, -1, 0), Vector3(-1, 1, 0), Vector3(1, -1, 0) }
mesh.uv = { Vector2(0, 0), Vector2(0, 1), Vector2(1, 0) }
mesh.triangles = { 0, 1, 2 }
```

And the resulting triangle would look like this.

![2D textured triangle](/Assets/2D_textured_triangle.png)

Finally we can add 1 more UV (since there is 1 UV per vertex)...

```cs
mesh.uv = { Vector2(0, 0), Vector2(0, 1), Vector2(1, 0), Vector2(1, 1) }
```

...and we can fully textured the Quad!

![2D textured quad](/Assets/2D_textured_quad.png)

# Coding the UVs for the Quad
Open the course menu, and you'll find a resources section. Download the `Texture.png.zip` file into your project, and unzip it.

Now double click the Voxel material and drag and drop the texture onto the albedo property.

![material albedo property](/Assets/material_albedo_property.png)

Now we can add the code for the UVs in the Quad.cs script.

```cs
	private void Start()
    {
        var filter = gameObject.GetComponent<MeshFilter>();

        var mesh = new Mesh();
        mesh.vertices = new Vector3[] { new Vector3(-1, -1, 0), new Vector3(-1, 1, 0), new Vector3(1, -1, 0), new Vector3(1, 1, 0) };
        mesh.uv = new Vector2[] { new Vector2(0, 0), new Vector2(0, 1), new Vector2(1, 0), new Vector2(1, 1) }; // added this
        mesh.triangles = new int[] { 0, 1, 2, 1, 3, 2 };

        mesh.RecalculateNormals();

        filter.mesh = mesh;
    }
```

Now click play to see a fully texured quad!

![fully textured quad](/Assets/quad_textured.png)