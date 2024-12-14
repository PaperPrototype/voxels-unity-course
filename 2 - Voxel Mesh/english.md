# Voxel Terrain
Why go through all the trouble of making a single Quad?!

If our whole world is going to be made out of "voxels", why not just use Unity's builtin cube, and make a million of them?

If we made a world out of millions of cubes, we are going to be rendering, well, millions of cubes... including cubes that we won't even be able to see!

Take for example a voxel terrain made of Unity's default cubes:

![2D voxel terrain unoptimized](/Assets/2D_voxel_terrain_unoptimized.png)

You can see that cubes that are not visible to the player are still going to get rendered, and this will give us horrible performance. To solve this we can split each cube into 6 individual quads (1 quad for each side of the cube), and then check if the current voxel has neighboring voxel on that side. If it does, then we don't create a quad on that side of the voxel.

![2D voxel terrain optimized](/Assets/2D_voxel_terrain_optimized.png)

By only creating quads where necessary we can prevent speding time rendering triangles that aren't visible, and get an optimized mesh! And as it turns out, this is what exaclty what Minecraft does!

# Voxel Mesh
For now we will build a single voxel using 6 individual quads (in the next section we will build terrain). 

Instead of hand coding each quad for each voxel, we can be smart and store all the possible vertices and triangle numbers. This is often refered to as creating "lookup tables".

When building the voxel, we can access the vertices and triangles based on the quad for the corresponding side we want to have a mesh on. Obviously we can check for neighboring voxels before deciding what sides of the voxel should have a quad mesh... but thats for the Noise section (where we decide where we want, and don't want voxels to be).

Create a new script in the Assets folder called "Tables.cs".

Open the script and write the following code into it.

```cs
using UnityEngine;

public static class Tables
{
    // all 8 possible vertices for a voxel
    public static readonly Vector3[] Vertices = new Vector3[8]
    {
        new Vector3(-0.5f, -0.5f, -0.5f),
        new Vector3(0.5f, -0.5f, -0.5f),
        new Vector3(0.5f, 0.5f, -0.5f),
        new Vector3(-0.5f, 0.5f, -0.5f),
        new Vector3(-0.5f, -0.5f, 0.5f),
        new Vector3(0.5f, -0.5f, 0.5f),
        new Vector3(0.5f, 0.5f, 0.5f),
        new Vector3(-0.5f, 0.5f, 0.5f),
    };

    // vertices to build a quad for each side of a voxel
    public static readonly int[,] QuadVertices = new int[6, 4]
    {
        // quad order
        // right, left, up, down, front, back

        // 0 1 2 2 1 3 <- triangle numbers

        // quads
        {1, 2, 5, 6}, // right quad
        {4, 7, 0, 3}, // left quad

        {3, 7, 2, 6}, // up quad
        {1, 5, 0, 4}, // down quad

        {5, 6, 4, 7}, // front quad
        {0, 3, 1, 2}, // back quad
    };
}
```

In the above script we have a `Vertices` table that contains all 8 possible vertices needed to make a voxel.

Then we have a `QuadVertices` table has indexes for finding vertices in the `Vertices` table to build 6 individual quads.

You'll also notice I added comment with the triangle numbers we will need for each quad, namely `0 1 2 2 1 3`. Because of how the table has been set up we can use the same triangle numbers for all the quads.

Now lets make a voxel. Create a new folder called "Voxel", and in it a new scene and script called voxel.

![project assets voxel](/Assets/project_assets_voxel.png)

Open the Voxel scene and create a new empty GameObject called "Voxel". Add the Voxel.cs script to the GameObject.

Open the Voxel.cs script and write the following code into it.

```cs
using UnityEngine;

[RequireComponent(typeof(MeshFilter))]
[RequireComponent(typeof(MeshRenderer))]
public class Voxel : MonoBehaviour
{
    void Start()
    {
        // number of quads in a voxel
        const int quadsNum = 6;

        // vertices and triangles for a single voxel
        var vertices = new Vector3[4 * quadsNum];
        var triangles = new int[6 * quadsNum];

        int vertexOffset = 0;
        int triangleOffset = 0;

        // for each side of the voxel
        for (int side = 0; side < quadsNum; side++)
        {
            // create mesh for quad on this side of the voxel
			// TODO
        }

        var mesh = new Mesh();
        mesh.vertices = vertices;
        mesh.triangles = triangles;

        mesh.RecalculateNormals();
        mesh.RecalculateBounds();

        var filter = gameObject.GetComponent<MeshFilter>();
        filter.mesh = mesh;
    }
}

```

We first create a variable called `quadsNum` that tells us how many quads to draw for a single voxel. Obviously this number is 6, but we make the variable so the code reads nicer.

We then create 2 arrays. The first being called `vertices`, and it will hold the number of vertices needed to make a voxel (which is 4 vertices per quad times 6 sides of a voxel). The second array, called `triangles`, will hold the number of triangles needed for the mesh of a single voxel and it is 6 triangle numbers per quad times the number of quads.

## Vertices
Now remove the `TODO` comment and replace it with the following code.

```cs
            // offset using currentVertex
            vertices[vertexOffset + 0] = Tables.Vertices[Tables.QuadVertices[side, 0]];
            vertices[vertexOffset + 1] = Tables.Vertices[Tables.QuadVertices[side, 1]];
            vertices[vertexOffset + 2] = Tables.Vertices[Tables.QuadVertices[side, 2]];
            vertices[vertexOffset + 3] = Tables.Vertices[Tables.QuadVertices[side, 3]];

			// TODO triangles

            vertexOffset += 4;
```

We use the `vertexOffset` as an offset to know where in the vertices array to place the vertices for the current quad. We then increase the `vertexOffset` after adding the vertices, so that the next loop will add vertices at the correct offset for the next quad.

When we are offsetting into the `Tables.QuadVertices` with the `side` variable (the first offset slot has the `side` variable in it), we are accessing the quads vertex index for that specific side:

```cs
Tables.Vertices[Tables.QuadVertices[0, ?]];

// is getting...

QuadVertices {
	{1, 2, 5, 6}, // <- this
	{4, 7, 0, 3},

	{3, 7, 2, 6},
	{1, 5, 0, 4}, 

	{5, 6, 4, 7},
	{0, 3, 1, 2},
};
```

and the number in the second slot is getting the index where we can find the specific vertex for that corner of the quad:

```cs
Tables.Vertices[Tables.QuadVertices[0, 0]];
Tables.Vertices[Tables.QuadVertices[0, 1]];
Tables.Vertices[Tables.QuadVertices[0, 2]];
Tables.Vertices[Tables.QuadVertices[0, 3]];

// is getting...

QuadVertices {

	{1, 2, 5, 6},
//   \__\__\__\___these

	{4, 7, 0, 3},

	{3, 7, 2, 6},
	{1, 5, 0, 4}, 

	{5, 6, 4, 7},
	{0, 3, 1, 2},
};

// which help us get...

Vertices = {
	new Vector3(-0.5f, -0.5f, -0.5f), // 0
	new Vector3(0.5f, -0.5f, -0.5f),  // 1 <- this
	new Vector3(0.5f, 0.5f, -0.5f),   // 2 <- this
	new Vector3(-0.5f, 0.5f, -0.5f),  // 3
	new Vector3(-0.5f, -0.5f, 0.5f),  // 4
	new Vector3(0.5f, -0.5f, 0.5f),   // 5 <- this
	new Vector3(0.5f, 0.5f, 0.5f),    // 6 <- this
	new Vector3(-0.5f, 0.5f, 0.5f),   // 7
};
```

Take a minute to make sure you understand how this is working.

We do this process 6 times, once for each quad:

```cs
        // for each side of the voxel
        for (int side = 0; side < quadsNum; side++)
        {
            // offset using currentVertex
            vertices[vertexOffset + 0] = Tables.Vertices[Tables.QuadVertices[side, 0]];
            vertices[vertexOffset + 1] = Tables.Vertices[Tables.QuadVertices[side, 1]];
            vertices[vertexOffset + 2] = Tables.Vertices[Tables.QuadVertices[side, 2]];
            vertices[vertexOffset + 3] = Tables.Vertices[Tables.QuadVertices[side, 3]];

			// TODO triangles

            vertexOffset += 4;
        }
```

## Triangles
The triangles are easy, since all the quads have been set up to use the exact same triangle numbers.

Replace `TODO triangles` with the following code.

```cs
            // 0 1 2 2 1 3 <- triangle numbers
            triangles[triangleOffset + 0] = vertexOffset + 0;
            triangles[triangleOffset + 1] = vertexOffset + 1;
            triangles[triangleOffset + 2] = vertexOffset + 2;
            triangles[triangleOffset + 3] = vertexOffset + 2;
            triangles[triangleOffset + 4] = vertexOffset + 1;
            triangles[triangleOffset + 5] = vertexOffset + 3;

			// TODO UV's

            triangleOffset += 6;
```

Now can click play and you should see a pink voxel being rendered!

![pink voxel](/Assets/pink_voxel.png)

We offset into the triangles array using the `triangleOffset`:


```cs
			triangles[triangleOffset + 0] =
            triangles[triangleOffset + 1] =
            triangles[triangleOffset + 2] =
            triangles[triangleOffset + 3] =
            triangles[triangleOffset + 4] =
            triangles[triangleOffset + 5] =
```

... as well as offset the triangle numbers themselves so that they point to the correct vertex in the `vertices` array:

```cs
            = vertexOffset + 0;
            = vertexOffset + 1;
			= vertexOffset + 2;
			= vertexOffset + 2;
			= vertexOffset + 1;
			= vertexOffset + 3;
```

## Explaining the rest
For reference the code in `Start` should now look like the following:

```cs
	void Start()
    {
        // number of quads in a voxel
        const int quadsNum = 6;

        // vertices and triangles for a single voxel
        var vertices = new Vector3[4 * quadsNum];
        var triangles = new int[6 * quadsNum];

        int vertexOffset = 0;
        int triangleOffset = 0;

        // for each side of the voxel
        for (int side = 0; side < quadsNum; side++)
        {
            // create mesh for quad on this side of the voxel

            // offset using currentVertex
            vertices[vertexOffset + 0] = Tables.Vertices[Tables.QuadVertices[side, 0]];
            vertices[vertexOffset + 1] = Tables.Vertices[Tables.QuadVertices[side, 1]];
            vertices[vertexOffset + 2] = Tables.Vertices[Tables.QuadVertices[side, 2]];
            vertices[vertexOffset + 3] = Tables.Vertices[Tables.QuadVertices[side, 3]];

            // 0 1 2 2 1 3 <- triangle numbers
            triangles[triangleOffset + 0] = vertexOffset + 0;
            triangles[triangleOffset + 1] = vertexOffset + 1;
            triangles[triangleOffset + 2] = vertexOffset + 2;
            triangles[triangleOffset + 3] = vertexOffset + 2;
            triangles[triangleOffset + 4] = vertexOffset + 1;
            triangles[triangleOffset + 5] = vertexOffset + 3;

			// TODO UV's

            triangleOffset += 6;
            vertexOffset += 4;
        }

        var mesh = new Mesh();
        mesh.vertices = vertices;
        mesh.triangles = triangles;

        mesh.RecalculateNormals();
        mesh.RecalculateBounds();

        var filter = gameObject.GetComponent<MeshFilter>();
        filter.mesh = mesh;
    }
```

Now that we are creating 6 quads for our voxel we can focus on the code I haven't quite explained yet (but you probably figured out).

We create a new mesh as usual, and then set its vertices and triangles to our `vertices` and `triangles` array:

```cs
		var mesh = new Mesh();
        mesh.vertices = vertices;
        mesh.triangles = triangles;
```

We then use the builtin `RecalculateNormals` method, as well as the `RecalculateBounds` method. We haven't seen `RecalculateBounds` before, but it lets the renderer know what area in 3D space the mesh is taking up, so it can do shadows and other rendering stuff correctly.

## UV's
The UV's are pretty straight forward.

Edit the code in Start to the following:

```cs
        const int quadsNum = 6;

        var vertices = new Vector3[4 * quadsNum];
        var triangles = new int[6 * quadsNum];
        var uvs = new Vector2[4 * quadsNum]; // added this

        int vertexOffset = 0;
        int triangleOffset = 0;

        for (int side = 0; side < quadsNum; side++)
        {
            vertices[vertexOffset + 0] = Tables.Vertices[Tables.QuadVertices[side, 0]];
            vertices[vertexOffset + 1] = Tables.Vertices[Tables.QuadVertices[side, 1]];
            vertices[vertexOffset + 2] = Tables.Vertices[Tables.QuadVertices[side, 2]];
            vertices[vertexOffset + 3] = Tables.Vertices[Tables.QuadVertices[side, 3]];

            triangles[triangleOffset + 0] = vertexOffset + 0;
            triangles[triangleOffset + 1] = vertexOffset + 1;
            triangles[triangleOffset + 2] = vertexOffset + 2;
            triangles[triangleOffset + 3] = vertexOffset + 2;
            triangles[triangleOffset + 4] = vertexOffset + 1;
            triangles[triangleOffset + 5] = vertexOffset + 3;

            uvs[vertexOffset + 0] = new Vector2(0, 0); // added this
            uvs[vertexOffset + 1] = new Vector2(0, 1); // added this
            uvs[vertexOffset + 2] = new Vector2(1, 0); // added this
            uvs[vertexOffset + 3] = new Vector2(1, 1); // added this

            triangleOffset += 6;
            vertexOffset += 4;
        }

        var mesh = new Mesh();
        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.uv = uvs; // added this

        mesh.RecalculateNormals();
        mesh.RecalculateBounds();

        var filter = gameObject.GetComponent<MeshFilter>();
        filter.mesh = mesh;
```

Per usuall we just insert the UV's and make sure to offset with `vertexOffset` so the UV's correspond with the correct vertex. Let me know in the course chat if you aren't able to understand this.

If you drag the material onto the voxel and click play, you should see a fully textured voxel!

![uv voxel](/Assets/uv_voxel.png)