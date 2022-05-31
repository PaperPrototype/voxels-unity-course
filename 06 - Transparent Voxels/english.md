I hope you enjoyed the last lecture! Cause this one is going to be even less coding! And more fun!

The last lecture gave us water colored blocks. The only problem is that water is transparent, while dirt is not. Our current material is not transparent. We are going to need 1 transparent material for the pransparent voxels, and 1 non-transparent material for dirt sand and grass. 

We are going to have 2 separate meshes, each with their own material which (in the end) will give us the following (awesome!) looking terrain

![](/Assets/chunk_water_final.png)

# Layers
Instead of storing the mesh data for both meshes in the Chunk.cs script, we are going to make a "layer" system. Each layer will have its own mesh and material. Each layer will also get its own gameObject.

In the Chunk folder, create a new script called "Layer". Open the script and edit the code to the following:

```cs
using UnityEngine;
using System;

[Serializable]
public class Layer
{
    public string name = "default";
    public bool useCollider = true;
    public Material material;
    public int atlasSize = 1; // x and y atlas size of texture

    // we add serialize field so we can debug... and feel satisfied :)
    [SerializeField] private GameObject gameObject;
    [SerializeField] private MeshFilter meshFilter;
    [SerializeField] private MeshRenderer meshRenderer;
    [SerializeField] private MeshCollider meshCollider;

    [SerializeField] private Vector3[] vertices;
    [SerializeField] private int[] triangles;
    [SerializeField] private Vector2[] uvs;

    [SerializeField] private int vertexOffset = 0;
    [SerializeField] private int triangleOffset = 0;
}
```

As you can see we make the Layer class serializable so that we can edit it in the inspector. Everything we need to build a mesh is in this class! There is something we haven't seen before. That is the `MeshCollider` component. If `useCollider` is true then we will apply a mesh collider to our terrain. This way the transparent layer (that will have water) won't have collision, but the terrain voxels can still have collision.

Now lets add a layers variable to the Chunk.cs class (as well as remove all uses of the previous the mesh data arrays). Update Chunk.cs to the following:

```cs
using UnityEngine;
using Unity.Mathematics;

public class Chunk : MonoBehaviour
{
    // size of the chunk
    public int ChunkResolution = 16;

    // voxel types and layers
    public VoxelType[] voxelTypes;
    public Layer[] layers;

    // noise
    private FastNoiseLite noise;

    private void Start()
    {
        noise = new FastNoiseLite();

        // TODO initialize layers

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

        // TODO complete layers
    }

	//...
}
```

We've removed the mesh vertices triangles etc. The `MeshVoxel` method is also broken. We'll fix this in a second.

First add an `Initialize`  method to the Layer.cs script:

```cs
using UnityEngine;
using System;

[Serializable]
public class Layer
{
    //...

	// ADD THIS
    public void Initialize(int chunkResolution)
    {
        // intialize gameObject for this layer
        gameObject = new GameObject();
        meshFilter = gameObject.AddComponent<MeshFilter>();
        meshRenderer = gameObject.AddComponent<MeshRenderer>();
        meshRenderer.material = material;

        if (useCollider)
        {
            meshCollider = gameObject.AddComponent<MeshCollider>();
        }

        // initialize mesh data arrays
        vertices = new Vector3[24 * chunkResolution * chunkResolution * chunkResolution];
        triangles = new int[36 * chunkResolution * chunkResolution * chunkResolution];
        uvs = new Vector2[24 * chunkResolution * chunkResolution * chunkResolution];
    }
}
```

This creates and sets up all of our variables. Go back to the Chunk.cs script and replace the comment `// TODO initialize layers` with the following code:

```cs
		//...

		// init mesh layers
        for (int i = 0; i < layers.Length; i++)
        {
            layers[i].Initialize(ChunkResolution);
        }

		//...
```

Once we have setup all the mesh variables in the layers, new we need to create the mesh for each layer! But first, each VoxelType will need to tell us which layer is is part of. For that update the VoxelType.cs script to the following:

```cs
using UnityEngine;
using System;

[Serializable]
public class VoxelType
{
    public string name = "default";
    public bool isSolid = false;
    public Vector2 atlasOffset = Vector2.zero;
    public int layer = 0; // add this
}
```

Now we can create a `MeshQuad` method in the `Layer` class (that will be used by `MeshVoxel`):

```cs
using UnityEngine;
using System;

[Serializable]
public class Layer
{
	//...

	// mesh quad on specific side of voxel
    public void MeshQuad(Vector3 offsetPos, VoxelType voxelType, int side)
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

        // apply offset and add uv corner offsets
        // then divide by the atlas size to get a smaller section (AKA our individual texture)
        uvs[vertexOffset + 0] = (voxelType.atlasOffset + new Vector2(0, 0)) / atlasSize;
        uvs[vertexOffset + 1] = (voxelType.atlasOffset + new Vector2(0, 1)) / atlasSize;
        uvs[vertexOffset + 2] = (voxelType.atlasOffset + new Vector2(1, 0)) / atlasSize;
        uvs[vertexOffset + 3] = (voxelType.atlasOffset + new Vector2(1, 1)) / atlasSize;

        triangleOffset += 6;
        vertexOffset += 4;
    }
}
```

Now lets update the broken `MeshVoxel` method in the Chunk.cs script to use layers:

It is a pretty big function, but it is no different that what `MeshVoxel` is currently using to create each quad! Now lets update `MeshVoxel` (in Chunk.cs) to use `MeshQuad`:

```cs
	//...

	private void MeshVoxel(int x, int y, int z)
    {
        Vector3 offsetPos = new Vector3(x, y, z);

        var voxelType = GetVoxelType(x, y, z); // add this

        for (int side = 0; side < 6; side++)
        {
            if (!IsNeighborSolid(x, y, z, side))
            {
                // and this
                try
                {
                    layers[voxelType.layer].MeshQuad(offsetPos, voxelType, side);
                }
                catch
                {
                    Debug.LogError("Voxel type is accessing layer that does not exist and is out of bounds of the layers array.");
                }
            }
        }
    }

	// ...
```

As you can see we first get the current voxel type. Each voxel type has a `layer` variable that tells us which layer it is part of. We use the `layer` variable to offset into the `layers` array, and then we call `MeshQuad` on that layer!

Now that our meshes are being created for each layer we can finally "apply" the meshes to each layer by creating a `Complete` method in the Layer.cs script...

```cs
	//...

    public void Complete()
    {
        // instantiate new mesh and set arrays
        Mesh mesh = new Mesh();
        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.uv = uvs;

        mesh.RecalculateNormals();
        mesh.RecalculateBounds();

        // set the mesh to the mesh filter
        meshFilter.mesh = mesh;

        // if we want the mesh to be able to collide
        if (useCollider)
        {
            // set the mesh collider
            meshCollider.sharedMesh = mesh;
        }
    }

	//...
```

...and then replace the comment `// TODO complete layers` from the Chunkcs. script with the following code:

```cs
	//...

	private void Start()
    {
        noise = new FastNoiseLite();

        // init mesh layers
        for (int i = 0; i < layers.Length; i++)
        {
            layers[i].Initialize(ChunkResolution);
        }

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

		// ADD THIS
        // complete and set mesh
        for (int i = 0; i < layers.Length; i++)
        {
            layers[i].Complete();
        }
    }

	//...
```

# Layers in the Inspector
All the coding is done! All we have left to do is Setup a transparent material for water, create the layers, and update the water voxel type to use the transparent layer!

First create a new material exaclty the same as the Atlas material (you can just duplicate the old material). Make sure you set the new material's Rendering Mode to "Transparent". Also set the albedo property to be our atlas.png texture:

![](/Assets/transparent_voxels_material.png)

Create 2 new layers.

![](/Assets/transparent_voxels_layers.png)

The first layer should use the regular Atlas material, and the second should use the transparent material.

Update the water VoxelType layer to 1 (which will offset by 1 and get us the second layer):

![](/Assets/transparent_voxels_water_layer_1.png)

Make sure to set the Atlas Size of each layer to 4!

And now you should have a nice mesh with transparent water!

![](/Assets/transparent_voxels_result1.png)

...wait, something wrong. It's pretty obvious, but layers are optimizing with eachother! Thats not what we want!

# Layers are not solid to eachother
We can fix this by making layers not be solid with other layers.

We just need to update the `IsNeighborSolid` method to the following:

```cs
	//...

	private bool IsNeighborSolid(int x, int y, int z, int side, TestVoxelType5 selfVoxelType)
    {
        int3 offset = Tables.NeighborOffsets[side];

        var neighborVoxelType = GetVoxelType(x + offset.x, y + offset.y, z + offset.z);

        // if from different layers
        if (neighborVoxelType.layer != selfVoxelType.layer)
        {
            return false; // treat eachother as air
        }

        return IsSolid(x + offset.x, y + offset.y, z + offset.z);
    }

	//...
```

And then we also need to update the `MeshVoxel` method to use our new `IsNeighborSolid` function:

```cs
	//...

	private void MeshVoxel(int x, int y, int z)
    {
        Vector3 offsetPos = new Vector3(x, y, z);

        var voxelType = GetVoxelType(x, y, z);

        for (int side = 0; side < 6; side++)
        {
            if (!IsNeighborSolid(x, y, z, side, voxelType)) // update this
            {
                try
                {
                    layers[voxelType.layer].MeshQuad(offsetPos, voxelType, side);
                }
                catch
                {
                    Debug.LogError("Voxel type is accessing layer that does not exist and is out of bounds of the layers array.");
                }
            }
        }
    }

	//...
```

And vuala!

![](/Assets/transparent_voxels_result2.png)