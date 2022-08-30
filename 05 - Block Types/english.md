The last lecture we used noise to figure out where we want and don't want voxels to be. In this lecture we will decide what *type* of voxels we want!

In the end we will have the following:

![voxel types](/Assets/voxel_types.png)

The water isn't transparent, but we will handle that in the next lecture with "layers".

# Voxel Type
In the Chunk folder, create a new script called "VoxelType".

Change the VoxelType.cs script to the following:

```cs
using System;

[Serializable]
public class VoxelType
{
    public string name = "default";
    public bool isSolid = false;
}
```

We add the `[Serializable]` attribute so that it can be "serialized" and edited in the inspector window. To use the serializable attribute we include `using System` at the top.

Open up the Chunk.cs script and add the following:

```cs
//... unchanged code
public class Chunk : MonoBehaviour
{
    public int ChunkResolution = 16;
    public VoxelType[] voxelTypes; // add this

	//... unchanged code
}
```

Save the scripts and go back to unity and add some voxel types to our awesome inspector!

![](/Assets/voxel_types_inspector.png)

Make sure to create the voxel types I created. Open the Chunk.cs script and now lets get these voxel types to work! Under the `IsSolid` method add the following method:

```cs
    private VoxelType GetVoxelType(int x, int y, int z)
    {
        float terrainHeight = GetNoiseHeight(8, 1, x, z);

        if (y > terrainHeight)
        {
            return voxelTypes[0]; // air
        }

        return voxelTypes[1]; // dirt
    }
```

The usage of noise us exaclty the same as what we did for `IsSolid`...

```cs
    private bool IsSolid(int x, int y, int z)
    {
        //...

        float terrainHeight = GetNoiseHeight(8, 1, x, z);

        if (y > terrainHeight)
        {
            return false;
        }

        return true;
    }
```

But we are giving back a `VoxelType` instead of true or false.

Now change the `IsSolid` method to the following:

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

        return GetVoxelType(x, y, z).isSolid; // add this
    }
```

We get the voxel type and return the `isSolid` variable!

Save the script and click play. Tada!

![](/Assets/voxel_types_chunk3.png)

Still kinda boring though. We only have 1 voxel type. Dirt! And even if we added more, we still only have 1 texture!

# Error Handling
Before we start adding bunch of voxel types with different textures, we have 1 very big problem.

The `GetVoxelType` method...

```cs
        if (y > terrainHeight)
        {
            return voxelTypes[0]; // air
        }

        return voxelTypes[1]; // dirt
```

...assumes we have the correct voxels setup in our inspector:

![voxel types](/Assets/voxel_types_inspector.png)

But what if we don't?! Welp, you can find out by removing the dirt voxel type. Go ahead and do just that, then come back.

Finished? Alright, you probably got the following error `IndexOutOfRangeException: Index was outside the bounds of the array`. Basically we tried to offset into our `voxelTypes` array and we offsetted "outside the bounds of the array".

Lets fix this so we can test if an error happened, and then *catch* that error and report it with a mroe helpful message.

Change the code in `GetVoxelType` to the following:

```cs
	private VoxelType GetVoxelType(int x, int y, int z)
    {
        float terrainHeight = GetNoiseHeight(8, 1, x, z);

        try
        {
            if (y > terrainHeight)
            {
                return voxelTypes[0]; // air
            }

            return voxelTypes[1]; // dirt
        }
        catch
        {
            Debug.LogError("That voxel type does not exist. You are offsetting outside of the voxelTypes array.");

            // give back new VoxelType (will use defaults we set in the VoxelType class)
            return new VoxelType();
        }
    }
```

We add a `try` block and put all our unsafe offsetting code in it. If the code in the `try` block fails, we "catch" the error in a `catch` block and log the error with a more helpful message.

If you click play, now you should see our custom error message!

# Texture Atlas
Open the course menu at the top and download the file called "atlas.png.zip" and unzip into the Assets folder. Delete the zip file but keep the png.

Since we are doing minecraft style pixel art, we don't want Unity to blend our texture colors and make it look "blurred".

![](/Assets/atlas_texture_blurred.png)

Change the filter mode to "Point (no filter)" in the inspector of atlas.png and click "Apply" to apply the changes.

![](/Assets/atlas_import_settings.png)

This will make our texture keep its pixels nice and sharp.

![](/Assets/atlas_texture_sharp.png)

The texture you just downloaded is called a Texture Atlas.

> In computer graphics, a "texture atlas" is an image containing multiple smaller images. By using custom texture coordinates a sub-image of the atlas is drawn. [see wikipedia](https://en.wikipedia.org/wiki/Texture_atlas)

# Voxel Type UVs
Each voxel type is going to need to access one of the sub-texture's in the texture atlas. To access the dirt sub-texture the UVs would be the following:

![](/Assets/atlas_highlight_dirt.png)

We are going to write code that will let us have a UV offset for each voxel type. An offset of `x = 0, y = 0` will get the dirt sub-texture of the atlas. And given an offset of `x = 1, y = 0` would get the grass sub-texture.

Open the VoxelType.cs script and change it to the following:
```cs
using UnityEngine; // add this
using System;

[Serializable]
public class VoxelType
{
    public string name = "default";
    public bool isSolid = false;
    public Vector2 atlasOffset = Vector2.zero; // add this
}
```

Now open the Chunk.cs script and add a special new variable:
```cs
//...
public class Chunk : MonoBehaviour
{
    public int ChunkResolution = 16;
    public int atlasSize = 1; // add this
    public VoxelType[] voxelTypes;

	//...
}
```

Update the `MeshVoxel` method so we apply our UVs based on the `VoxelType.uvOffset`:
```cs
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

				// get the voxel type
                var voxelType = GetVoxelType(x, y, z);

                // first add offset and uv corners
                // then divide by the atlas size to get a smaller section 
				// (AKA our sub-texture in the atlas)
                uvs[vertexOffset + 0] = (voxelType.atlasOffset + new Vector2(0, 0)) / atlasSize;
                uvs[vertexOffset + 1] = (voxelType.atlasOffset + new Vector2(0, 1)) / atlasSize;
                uvs[vertexOffset + 2] = (voxelType.atlasOffset + new Vector2(1, 0)) / atlasSize;
                uvs[vertexOffset + 3] = (voxelType.atlasOffset + new Vector2(1, 1)) / atlasSize;

                triangleOffset += 6;
                vertexOffset += 4;
            }
        }
    }
```

We are adding the old corner UVs (for accessing each corner of a texture) and the atlasOffset of each voxelType, then dividing it by `atlasSize`.

We divide by `atlasSize` to shrink our atlasOffset and corner UVs back down in the range of 0 to 1, giving us our desired sub-texture UV coordinates.

Our texture atlas is a 4x4 texture atlas, so we need to set the `atlasSize` to 4.

![](/Assets/atlas_size_inspector.png)

Now lets setup a new Material for our texture atlas. Create a new material called "Atlas" drag the atlas.png texture onto its albedo property.

![](/Assets/atlas_material_settings.png)

Also set the metallic and smoothness to 0, cause we don't really want our terrain to be shiny... do we?

Aaaaand... now drag and drop the new atlas material onto the Chunk GameObject in the scene.

Now all that is left for us to do is click play!

![](/Assets/voxel_types_chunk_dirt.png)

# Grass, Dirt, Sand, and Water!
We got VoxelType UVs working. All that is left is to add more voxel types!

Open the Chunk.cs script and change the code in `GetVoxelType` to the following:

```cs
 	private VoxelType GetVoxelType(int x, int y, int z)
    {
        float terrainHeight = GetNoiseHeight(8, 1, x, z);

        try
        {
            if (y == 5)
            {
                return voxelTypes[3]; // sand
            }

            if (y < terrainHeight)
            {
                return voxelTypes[1]; // dirt
            }

            if (y < (terrainHeight + 1))
            {
                return voxelTypes[2]; // grass
            }
            else if (y < 8)
            {
                return voxelTypes[4]; // water
            }

            return voxelTypes[0]; // air
        }
        catch
        {
            Debug.LogError("That voxel type does not exist. You are offsetting outside of the voxelTypes array.");

            // give back new VoxelType (will use defaults we set in the VoxelType class)
            return new VoxelType();
        }
    }
```

It's very important that you experiment with the above code and get a handle of how it works. But, before you go and experiment with it, I recommend you finish this lecture first.

For the above code to work, we need to add some more voxel types in the inspector, and then we can click play! (See below picture for voxel types)

![](/Assets/voxel_types.png)

# (BONUS) Side specific texturing
Currently our voxels have the same texure on each side. In Minecraft you can have a different texture for the sides, bottom, and top.

To accomplish this we will add top and bottom `atlasOffset` variables to our VoxelType.

```cs
using UnityEngine;
using System;

[Serializable]
public class TestVoxelTypeBONUS
{
    public string name = "default";
    public bool isSolid = true;

    [SerializeField]
    private Vector2 atlasOffsetTop = Vector2.zero;
    [SerializeField]
    private Vector2 atlasOffsetSides = Vector2.zero;
    [SerializeField]
    private Vector2 atlasOffsetBottom = Vector2.zero;

    // side order
    // right, left, up, down, front, back
    public Vector2 GetAtlasOffset(int side)
    {
        // top
        if (side == 2)
        {
            // give back atlasOffsetTop and stop here
            return atlasOffsetTop;
        }
        // bottom
        else if (side == 3)
        {
            // give back atlasOffsetBottom and stop here
            return atlasOffsetBottom;
        }

        // if above fails then this gets returned
        return atlasOffsetSides;
    }
}
```

We also make the atlasOffsets private... but also make sure they show up in the inspector. This way we can edit them in the inpector, but not access them in the chunk code. Instead we have a new method called `GetAtlasOffset` which gives us the atlas offset for the side we are looking at.

Now just update the UV code in the `MeshVoxel` method in the Chunk.cs script...

```cs
private void MeshVoxel(int x, int y, int z)
{
    Vector3 offsetPos = new Vector3(x, y, z);

    for (int side = 0; side < 6; side++)
    {
        if (!IsNeighborSolid(x, y, z, side))
        {
            //...

            // apply offset and add uv corner offsets
            // then divide by the atlas size to get a smaller section (AKA our sub-texture in the atlas)
            uvs[vertexOffset + 0] = (voxelType.GetAtlasOffset(side) + new Vector2(0, 0)) / atlasSize;
            uvs[vertexOffset + 1] = (voxelType.GetAtlasOffset(side) + new Vector2(0, 1)) / atlasSize;
            uvs[vertexOffset + 2] = (voxelType.GetAtlasOffset(side) + new Vector2(1, 0)) / atlasSize;
            uvs[vertexOffset + 3] = (voxelType.GetAtlasOffset(side) + new Vector2(1, 1)) / atlasSize;

            //...
        }
    }
}
```

Now update the atlasOffsets, and you should be able to have grass blocks with dirt on the bottom and sides.

![](/Assets/voxel_types_side_specific_texturing.png)

Feel free to create your own texture atlas with better textures. Mine are very simple.