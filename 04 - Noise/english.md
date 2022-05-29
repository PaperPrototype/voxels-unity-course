If we want to be able to optimize our voxel chunk mesh, we need a way of knowing where a voxel should and shouldn't be.

![](/Assets/2D_terrain_solid_yes_or_no.png)

For now we can make a flat terrain by using a simple if statement:

```
if y > 12
	return air;
else
	return dirt;
```

Once we get flat terrain working we will transition to using something called "noise" that will give us hills and mountains.

# Flat terrain
Open the Chunk.cs script and add the following method under `MeshVoxel`:

```cs
    private bool IsSolid(int x, int y, int z)
    {
        if (y > 8)
        {
            return false;
        }

        return true;
    }
```

This code says if the voxel y position is greate than 8, then that voxel is an "air" voxel (AKA not solid).

In the nested loop in `Start` we can use `IsSolid` to determine if we should mesh a voxel or not:

```cs
    void Start()
    {
		//...

		for (int x = 0; x < ChunkResolution; x++)
        {
            for (int y = 0; y < ChunkResolution; y++)
            {
                for (int z = 0; z < ChunkResolution; z++)
                {
                    if (IsSolid(x, y, z)) // added
                    {
                        MeshVoxel(x, y, z);
                    }
                }
            }
        }

		//...
	}
```

Now if you click play we should have flat terrain!

![chunk flat](/Assets/chunk_flat.png)

But there is still a problem! Voxels are not optimizing with eachother!

![chunk flat unoptimized](/Assets/chunk_flat_unoptimized.png)

Whats happending is illustrated by the following diagram:

![2D voxel terrain flat unoptimized](/Assets/2D_voxel_terrain_flat_unoptimized.png)

To solve this, for every voxel that `IsSolid`, we need to also check its neighbors... more on this in a second.

We are going to need to download a Math library first. Open Unity's package manager window. Click the plus with a dropdown in the top left. Select "add package from git URL..." and type in `com.unity.mathematics` then click enter.

Now open the Tables.cs script and add the following code:

```cs
using UnityEngine;
using Unity.Mathematics; // added

public static class Tables
{
	//...

    // offset to neighboring voxel position
    public static readonly int3[] NeighborOffsets = new int3[6]
    {
        new int3( 1,  0,  0), // right
        new int3(-1,  0,  0), // left
        new int3( 0,  1,  0), // up
        new int3( 1, -1,  0), // down
        new int3( 0,  0,  1), // front
        new int3( 0,  0, -1), // back
    };
}
```

> If Visual Studio is giving you an error that it can't find `Unity.Mathematics` then you need to update the `Visual Studio Editor` package in Unity's package manager. You can find a reddit post about that [here](https://www.reddit.com/r/Unity3D/comments/fvnke9/visual_studio_code_doesnt_recognize/). Essentially in the package manager find the package called "Visual Studio Editor" and click update.

We create a new table called `NeighborOffsets` that will take in the `side` variable from `MeshVoxel`...

```cs
	private void MeshVoxel(int x, int y, int z)
    {
		//...

        for (int side = 0; side < 6; side++)

		//...
	}
```

... and give us an offset to the neighboring voxel's position on that side.


Above the `IsSolid` method add the following method:

```cs
	//...

    private bool IsNeighborSolid(int x, int y, int z, int side)
    {
        int3 offset = Tables.NeighborOffsets[side];

        return IsSolid(x + offset.x, y + offset.y, z + offset.z);
    }

	//...
```

You also need to include `using Unity.Mathematics` at the top of the Chunk.cs script.

Now we can use `IsNeighborSolid` in `MeshVoxel`:

```cs
	//...

	private void MeshVoxel(int x, int y, int z)
    {
        Vector3 offsetPos = new Vector3(x, y, z);

        for (int side = 0; side < 6; side++)
        {
            if (!IsNeighborSolid(x, y, z, side)) // added
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
    }

	//...
```

We are checking if the neighbor on that side is air (not solid). If the neighbor is air, *then* we can draw a Quad on that side.

For reference the Chunk.cs script should now look like this:

```cs
using UnityEngine;
using UnityEngine.Rendering;
using Unity.Mathematics;

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
        vertices = new Vector3[24 * ChunkResolution * ChunkResolution * ChunkResolution];
        triangles = new int[36 * ChunkResolution * ChunkResolution * ChunkResolution];
        uvs = new Vector2[24 * ChunkResolution * ChunkResolution * ChunkResolution];

        for (int x = 0; x < ChunkResolution; x++)
        {
            for (int y = 0; y < ChunkResolution; y++)
            {
                for (int z = 0; z < ChunkResolution; z++)
                {
                    if (IsSolid(x, y, z))
                    {
                        MeshVoxel(x, y, z);
                    }
                }
            }
        }

        Mesh mesh = new Mesh();
        mesh.indexFormat = IndexFormat.UInt32;
        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.uv = uvs;

        mesh.RecalculateNormals();
        mesh.RecalculateBounds();

        var filter = gameObject.GetComponent<MeshFilter>();
        filter.mesh = mesh;
    }

    private void MeshVoxel(int x, int y, int z)
    {
        Vector3 offsetPos = new Vector3(x, y, z);

        for (int side = 0; side < 6; side++)
        {
            if (!IsNeighborSolid(x, y, z, side))
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
    }

    private bool IsNeighborSolid(int x, int y, int z, int side)
    {
        int3 offset = Tables.NeighborOffsets[side];

        return IsSolid(x + offset.x, y + offset.y, z + offset.z);
    }

    private bool IsSolid(int x, int y, int z)
    {
        if (y > 8)
        {
            return false;
        }

        return true;
    }
}
```

Click play and you should see a super optimized flat terrain!

![](/Assets/chunk_optimized.png)

In our diagram it would look like this:

![](/Assets/2D_voxel_terrain_flat_optimized.png)

The red top are the quads we decided to addd to the mesh. And the greyed out is the quads we decided not to add to the mesh.

# Noisy Terrain
Alrighty, we have the optimization in place. Now lets upgrade to "noise" to have more interesting terrain.

What is noise? To put it simply, it is a smoothed random-like number, that given a "seed" will always give back the same "wave". We are getting a "height" number along the wave and using it for our terrain. Each seed produces unique waves.

We call this "pseudo-random" because if you access the noise at a certain X or Y, you will get the same number every time (given the seed hasn't changed)! It will make more sense once we use noise and see the results.

In the course menu download the file called "FastNoiseLite.zip" into your unity project. Unzip the file. Now we can use Jordan Peck's awesome noise library.

Open the Chunk.cs script and add the following:

```cs
	//...

    private FastNoiseLite noise; // added

    void Start()
    {
        noise = new FastNoiseLite(); // added
	
		//...
    }

	//...
```

(PST: you can change the default seed by changing the code to `FastNoiseLite(13123)`, 13123 would be the new seed. I recommend keeping the same seed until you finish this lecture. Once you finish the lecture you can experiment)

Now under `IsSolid` add the following method:

```cs
	//...

    // get noise height in range of floorHeight to maxHeight
    private float GetNoiseHeight(float maxHeight, float floorHeight, float x, float z)
    {
        return // floorHeight to maxHeight
        (
            (
                (
                    noise.GetNoise(x, z)    // range -1           to 1
                    + 1                     // range  0           to 2
                ) / 2                       // range  0           to 1
            ) * (maxHeight - floorHeight)   // range  0           to (maxHeight - floorHeight)
        ) + floorHeight;                    /* range  floorHeight to ((maxHeight - floorHeight) + floorHeight) 
                                             * simplifies ===> floorHeight to maxHeight 
                                             */
    }

	//...
```

I get some noise, but the noise is given in the range of -1 to 1. We do a bunch of math until we get a range of floorHeight to maxHeight. I did my best to illustrate the math with comments, and it really isn't complicated at all.

With this awesome method you can now edit the `IsSolid` method to the following:

```cs
	//...

    private bool IsSolid(int x, int y, int z)
    {
        float terrainHeight = GetNoiseHeight(8, 1, x, z); // added

        if (y > terrainHeight) // changed
        {
            return false;
        }

        return true;
    }

	//...
```

And now if you click play you should see some awesome terrain!

![chunk_noise_optimized.png](/Assets/chunk_noise_optimized.png)

# Edge of chunk detection
Currently our chunk resembles the following center chunk:

![](/Assets/2D_noise_voxel_terrain_neighbors.png)

We are drawing as if we would have neighboring chunks, and this is perfect!... except we are only going to have 1 single chunk in our world. So we need to NOT optimize the sides or bottom of our chunk. And remember that we will not be having any neighboring chunks.

![](/Assets/2D_noise_voxel_terrain_no_neighbors.png)

To do this, all we have to do is update the `IsSolid` method to return `false` (air) if the neighbor voxel we are checking is outside of the chunk.

Add the following to `IsSolid`:

```cs

    private bool IsSolid(int x, int y, int z)
    {
        // if outside of the chunk
        if (x < 0 || x >= ChunkResolution ||
            y < 0 || y >= ChunkResolution ||
            z < 0 || z >= ChunkResolution)
        {
            return false; // air
        }

		//...
	}
```

Now click play and you should see our beautiful chunk! With its bottom and sides on :P

![](/Assets/chunk_no_neighbors.png)
