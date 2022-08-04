If we want to make an entire voxel world using a single mesh, we are going to have a problem. 

Most mobile graphics cards use 16 bit indices (which allow you to have meshes with up to 65k vertices). In order to access a vertex the graphics card has to index into our array of vertices, and with 16 bits you can only count up to 65,535.

So the solution is to split up our world into "chunks" of voxels. Each chunk with however many voxels will give us less than 65,535 vertices.

You may recall Minecrafts "chunk distance" setting that lets you decide how far to load chunks around the player. Well, that is because Minecraft does exaclty what we just came up with! It splits the world up into chunks of blocks, and whenever we edit the terrain, it simply redoes that entire chunks mesh (at first it seems weird, why not update only that single voxels mesh? Well, its because making a system that would do that requires a lot more work, and just recreating an entire chunk's mesh is relatively fast and has worked quite well).

In this course we **won't** be builing a large world with many chunks, but instead will just be focusing on making a single "chunk" of voxels. That way you can nail the basics and in a later course make terrain or a building system that uses multiple chunks.

# Chunk
Create a new folder in Assets called "Chunk". Create a new Scene and Script called Chunk.

![](/Assets/project_assets_chunk.png)

Open the Chunk scene and create a new empty GameObject called "Chunk". Add the Chunk.cs script to it.

Now open the Chunk.cs script by double clicking it. Edit the code in Chunk.cs to be the following:

```cs
using UnityEngine;

[RequireComponent(typeof(MeshFilter))]
[RequireComponent(typeof(MeshRenderer))]
public class Chunk : MonoBehaviour
{
    public int ChunkResolution = 16;

    private Vector3[] vertices;
    private int[] triangles;
    private Vector2[] uvs;

    private int vertexOffset = 0;
    private int triangleOffset = 0;

    void Start()
    {
        // TODO create chunk mesh
    }
}
```

We first make sure the Chunk GameObject will have a MeshRenderer and MeshFilter.

Then we have a ChunkResolution number, that we will use to set the number of voxels in the X Y and Z axis of the chunk.

Then we have our Vertex Triangle and UV arrays, as well as a `vertexOffset` and `triangleOffset`.

Now lets make this chunk! First we need a method that we can use to generate the voxels given an X Y and Z grid number. For that we create the `MeshVoxel` method right below `Start`:

```cs
	private void MeshVoxel(int x, int y, int z)
    {
        Vector3 offsetPos = new Vector3(x, y, z);

        for (int side = 0; side < 6; side++)
        {
            vertices[vertexOffset + 0] = Tables.Vertices[Tables.QuadVertices[side, 0]] + offsetPos;
            vertices[vertexOffset + 1] = Tables.Vertices[Tables.QuadVertices[side, 1]] + offsetPos;
            vertices[vertexOffset + 2] = Tables.Vertices[Tables.QuadVertices[side, 2]] + offsetPos;
            vertices[vertexOffset + 3] = Tables.Vertices[Tables.QuadVertices[side, 3]] + offsetPos;

            triangles[triangleOffset + 0] = vertexOffset + 0;
            triangles[triangleOffset + 1] = vertexOffset + 1;
            triangles[triangleOffset + 2] = vertexOffset + 2;
            triangles[triangleOffset + 3] = vertexOffset + 2;
            triangles[triangleOffset + 4] = vertexOffset + 1;
            triangles[triangleOffset + 5] = vertexOffset + 3;

            uvs[vertexOffset + 0] = new Vector2(0, 0);
            uvs[vertexOffset + 1] = new Vector2(0, 1);
            uvs[vertexOffset + 2] = new Vector2(1, 0);
            uvs[vertexOffset + 3] = new Vector2(1, 1);

            triangleOffset += 6;
            vertexOffset += 4;
        }
    }
```

`MeshVoxel` is almost exaclty the same as the previous lecture, with 1 important difference. We create an offset position (in the `offsetPos` variable) from the `x` `y` and `z` parameters given to the method. We then add the `offsetPos` to each quads vertices so that the entire voxel is offsetted.

```cs
// we offset each vertex position by the voxels offsetPos
= Tables.Vertices[Tables.QuadVertices[side, 0]] + offsetPos;
= Tables.Vertices[Tables.QuadVertices[side, 1]] + offsetPos;
= Tables.Vertices[Tables.QuadVertices[side, 2]] + offsetPos;
= Tables.Vertices[Tables.QuadVertices[side, 3]] + offsetPos;
```

Lets now use this to build an entire chunk of voxels! 

In `Start` replace `// TODO create chunk mesh` with the following code:

```cs
		vertices = new Vector3[24 * ChunkResolution * ChunkResolution * ChunkResolution];
        triangles = new int[36 * ChunkResolution * ChunkResolution * ChunkResolution];
        uvs = new Vector2[24 * ChunkResolution * ChunkResolution * ChunkResolution];

        for (int x = 0; x < ChunkResolution; x++)
        {
            for (int y = 0; y < ChunkResolution; y++)
            {
                for (int z = 0; z < ChunkResolution; z++)
                {
                    MeshVoxel(x, y, z);
                }
            }
        }

		// TODO create new Mesh and set MeshFilter.mesh
```

We initialize our `vertices` array and set the length to 24 times the number of voxels in the chunk, since 24 (4 * 6) is the number of vertices needed for 1 voxel. For triangles we set the length to 36 times the number of voxels in the chunk, since 1 voxel takes 36 triangles (6 * 6 was the previous number, but I simplified it to 36 to make the code easier to read). The UV's needed are the same as the vertices.

We then have 3 nested loops:

```cs
for (int x = 0; x < ChunkResolution; x++)
	for (int y = 0; y < ChunkResolution; y++)
		for (int z = 0; z < ChunkResolution; z++)
```

These combined with `MeshVoxel` will fill our mesh arrays (namely `vertices` `uvs` and `triangles`) with the mesh data for an entire "chunk" of voxels.

After the nested loops replace the comment `// TODO create new Mesh and set MeshFilter.mesh` with the following code:

```cs
        Mesh mesh = new Mesh();
        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.uv = uvs;

        mesh.RecalculateNormals();
        mesh.RecalculateBounds();

        var filter = gameObject.GetComponent<MeshFilter>();
        filter.mesh = mesh;
```

We create a new mesh, set the mesh.vertices mesh.triangles and mesh.uv to our `vertices` `triangles` and `uvs` arrays. We then use `RecalculateNormals` to generate the normals for our mesh, as well as use `RecalculateBounds` to tell the renderer what area in 3D our mesh is taking up. Finally we set the mesh in the `MeshFilter` component to our new mesh. 

If you run this code you should see a pink chunk of voxels!

![](/Assets/pink_chunk.png)

Its pink because there is no material on it. Add our Voxel material and it should now look like this:

![](/Assets/chunk_incomplete.png)

There is 2 problems currently.

Our chunk seems to be missing half of its voxels!

And, our voxels are not optimized. Currently they are creating a quad on all 6 sides even if they don't need to be:

![](/Assets/2D_voxel_terrain_unoptimized.png)

We will be optimizing the voxels in the next lecture. For now we can fix the first problem so that all the voxels meshes get created. 

Currently our mesh has maxed out the number of vertices it can hold. To fix this we have to make a small change to the Chunk.cs script.

```cs
using UnityEngine;
using UnityEngine.Rendering; // added

[RequireComponent(typeof(MeshFilter))]
[RequireComponent(typeof(MeshRenderer))]
public class Chunk : MonoBehaviour
{
	//...

    void Start()
    {
		//...
        Mesh mesh = new Mesh();
        mesh.indexFormat = IndexFormat.UInt32; // added
        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.uv = uvs;
		//...
    }

	//...
}
```

You'll notice we add `using UnityEngine.Rendering` at the top. This gives us access to `IndexFormat.UInt32` and we set the meshes indexFormat to use a `UInt32`, which lets us have twice as many vertices in our mesh.

After making those changes and click play, the chunk's mesh should now be fixed!

![](/Assets/chunk_first.png)

Once we optimize our voxels the number of vertices should be reduced significantly and we can revert back to using a 16 bit indexFormat. I would encourage this because some mobile devices don't support 32 bit index formats.

And now onto optimizing our voxel mesh!