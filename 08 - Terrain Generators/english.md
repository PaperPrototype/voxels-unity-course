Currently if we want to change how the terrain looks we have to update the `GetVoxelType` method. We also may need to update the `voxelTypes` in the inspector. But the worst part? Each time we update the `GetVoxelType` method to make a new terrain, we lose the old one!

This lecture is going to make our code more *reusable*. Basically we will be able to create a new "terrain script" that has its own `GetVoxelType` method drag and drop that into our `Chunk` script, and vuala! New terrain! Want the old terrain? Just drag and drop that terrain into the `Chunk` script!

# Reusable?
Lets look at the variables and constants. I'm not talking about a *variable* as in a variable in code, I mean theorically if we make a new terrain, what will stay the same in our code (and the inspector), and what will change?

I've already figured this out for you and there are 3 things that we might change to make a new terrain.

1. the Voxel Types
2. the Layers
3. the `GetVoxelType` method

To make our code reusable for many different terrains (while being able to keep old terrains that we make), we must separate the above three things, so that they can be seamlessly swapped out and replaced!

# Terrain Generator
For now lets move all the "variable" functionality for creating a new terrain (the stuff that will change) into its own script called `TerrainGenerator`, and we leave the "constants" (unchanging) code in the Chunk.cs script.

Create a new script called TerrainGenerator and write the following code:

```cs
using UnityEngine;

public class TerrainGenerator : MonoBehaviour
{
    public int ChunkResolution = 16;
    public int Seed = 1337;

    public VoxelType[] VoxelTypes;
    public Layer[] Layers;

    private FastNoiseLite noise;

    public VoxelType GetVoxelType(int x, int y, int z)
    {
        float caves = GetNoiseCaves(5f, x, y, z);

        if (caves > 0.3)
        {
            return VoxelTypes[0]; // air
        }

        return VoxelTypes[1]; // dirt
    }
}
```

We also obviously need to include the `GetNoise` methods:

```cs
using UnityEngine;

public class TerrainGenerator : MonoBehaviour
{
    //...

    // get noise height in range of floorHeight to maxHeight
    public float GetNoiseHeight(float maxHeight, float floorHeight, float x, float z)
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

    // gives numbers in range of 0 to 1
    public float GetNoiseCaves(float scale, int x, int y, int z)
    {
        return (noise.GetNoise(x * scale, y * scale, z * scale) + 1) / 2;
    }
}
```

Now lets update the Chunk.cs script and remove the layers and voxel types and noise (phew), and switch to using the ones from our `TerrainGenerator`:

```cs
using UnityEngine;
using Unity.Mathematics;

public class Chunk : MonoBehaviour
{
    // replace all variables with a terrain generator
    public TerrainGenerator terrainGenerator;

    private void Start()
    {
        // initialize layers
        // TODO

        for (int x = 0; x < terrainGenerator.ChunkResolution; x++)
        {
            for (int y = 0; y < terrainGenerator.ChunkResolution; y++)
            {
                for (int z = 0; z < terrainGenerator.ChunkResolution; z++)
                {
                    if (IsSolid(x, y, z))
                    {
                        MeshVoxel(x, y, z);
                    }
                }
            }
        }

        // complete and set the mesh for each layer
        // TODO
    }

    //...
}
```

We also updated the `Start` method and added some `TODO` comments since we will need to handle layers through the terrain generator.

Add the following 2 methods to the `TerrainGenerator` script:

```cs
using UnityEngine;

public class TerrainGenerator : MonoBehaviour
{
    //...

    public void Initialize()
    {
        noise = new FastNoiseLite(Seed);

        // init mesh layers
        for (int i = 0; i < Layers.Length; i++)
        {
            Layers[i].Initialize(ChunkResolution);
        }

    }

    public void Complete()
    {
        // complete and set mesh
        for (int i = 0; i < Layers.Length; i++)
        {
            Layers[i].Complete();
        }
    }

    //...
}
```

Now you can go back and replace the `TODO` comments with the `Initialize` and `Complete` methods:

```cs
using UnityEngine;
using Unity.Mathematics;

public class Chunk : MonoBehaviour
{
    //...
    
    private void Start()
    {
        // initialize layers
        terrainGenerator.Initialize();

        for (int x = 0; x < terrainGenerator.ChunkResolution; x++)
        {
            for (int y = 0; y < terrainGenerator.ChunkResolution; y++)
            {
                for (int z = 0; z < terrainGenerator.ChunkResolution; z++)
                {
                    if (IsSolid(x, y, z))
                    {
                        MeshVoxel(x, y, z);
                    }
                }
            }
        }

        // complete and set the mesh for each layer
        terrainGenerator.Complete();
    }

    //...
}
```

## Clean up the Chunk.cs script
Our Chunk.cs script is very broken right now and we need to fix it!

Start by deleting the `GetVoxelType` and `GetNoiseHeight` and `GetNoiseCaves` methods since they are no longer needed, and are now in the `TerrainGenerator` script.

We also broke the `MeshVoxel` and `IsNeighborSolid` and `IsSolid` methods, so we need to fix them to use our `TerrainGenerator` (we basically just have to add a `terrainGenerator.` in front of everything that is broken):

```cs
using UnityEngine;
using Unity.Mathematics;

public class Chunk : MonoBehaviour
{
    //...

    private void MeshVoxel(int x, int y, int z)
    {
        Vector3 offsetPos = new Vector3(x, y, z);

        // update this
        var voxelType = terrainGenerator.GetVoxelType(x, y, z);

        for (int side = 0; side < 6; side++)
        {
            if (!IsNeighborSolid(x, y, z, side, voxelType))
            {
                try
                {
                    // update this
                    terrainGenerator.Layers[voxelType.layer].MeshQuad(offsetPos, voxelType, side);
                }
                catch
                {
                    Debug.LogError("Voxel type is accessing layer that does not exist and is out of bounds of the layers array.");
                }
            }
        }
    }

    private bool IsNeighborSolid(int x, int y, int z, int side, VoxelType selfVoxelType)
    {
        int3 offset = Tables.NeighborOffsets[side];

        // update this
        var neighborVoxelType = terrainGenerator.GetVoxelType(x + offset.x, y + offset.y, z + offset.z);

        // if from different layers
        if (neighborVoxelType.layer != selfVoxelType.layer)
        {
            return false; // treat eachother as air
        }

        return IsSolid(x + offset.x, y + offset.y, z + offset.z);
    }

    private bool IsSolid(int x, int y, int z)
    {
        // update this
        // if outside of the chunk
        if (x < 0 || x >= terrainGenerator.ChunkResolution ||
            y < 0 || y >= terrainGenerator.ChunkResolution ||
            z < 0 || z >= terrainGenerator.ChunkResolution)
        {
            return false; // air
        }

        // update this
        return terrainGenerator.GetVoxelType(x, y, z).isSolid;
    }
}
```

That was a lot of code to update and move around! 

We separated all the code for generating different terrains into its own script called `TerrainGenerator` that the `Chunk` script uses when creating terrain:

![](/Assets/terrain_generator_code_diagram.png)

Create a new empty GameObject called `TerrainGenerator` and add the `TerrainGenerator` script to it.

![](/Assets/terrain_generator_add_component.png)

Now drag the TerrainGenerator gameObject into the `terrainGenerator` slot in the Chunk script:

![](/Assets/terrain_generator_add_to_chunk.png)

Now just set up the voxel types and layers needed by `GetVoxelType` and run this code!

![](/Assets/terrain_generator_layers_and_voxel_types.png)

TADA!

# Abstract Classes
But there is 1 problem. We still are stuck with 1 single version of `GetVoxelType`! And we will lose any old versions of it if we update the `GetVoxelType` method in the `TerrainGenerator` script!

Lets fix this. Edit the `TerrainGenerator` class to be `abstract` and then also mark the `GetVoxelType` method to be abstract (we'll explain what this does in a second):

```cs
// update this
public abstract class TerrainGenerator : MonoBehaviour
{
    //...

    // update this
    public abstract VoxelType GetVoxelType(int x, int y, int z);

    //...
}
```

We have to remove the body of the `GetVoxelType` method, and only the `GetVoxelType(int x, int y, int z)` part of it exists. The code that will actually "implement" it will be another script...

Lets make a new script (that will implement `GetVoxelType`) called `CavesTerrainGenerator` and make it inherit from `TerrainGenerator`:

```cs
//                            inherits from
public class CavesGenerator : TerrainGenerator
{
    public override VoxelType GetVoxelType(int x, int y, int z)
    {
        float caves = GetNoiseCaves(5f, x, y, z);

        if (caves < 0.3)
        {
            return VoxelTypes[0]; // air
        }

        return VoxelTypes[1]; // dirt
    }
}
```

As you can see we inherit from `TerrainGenerator` and we `override` its `GetVoxelType` method!

## What just happened??
If you are a little confused by the above code thats ok.

The `TerrainGenerator` class is now "abstract" so it is only allowed to acts like a *template*.

![](/Assets/terrain_generator_code_diagram_abstract.png)

And now we can create endless scripts that inherit from the `TerrainGenerator` abstract class and all we have to do is write the `GetVoxelType` method!

## Test it out
Now go back to Unity and remove `TerrainGenerator` from the TerrainGenerator gameObject, and add `CavesGenerator` script instead (the `TerrainGenerator` is now abstract so it cannot live on its own but must live through another class that implements it).

![](/Assets/terrain_generator_caves_generator.png)

Now create the voxel types for Air and Dirt, and create a layer.

Got it? Done? OK!

Before we hit play you need to re-add the `TerrainGenerator` to the Chunk:

![](/Assets/terrain_generator_add_caves_to_chunk.png)

And click play to see our caves!

![](/Assets/terrain_generator_caves.png)

Pretty cool eh? Go ahead and make a few different terrains!

Also, rename the "TerrainGenerator" gameobject to "CavesGenerator" since it has our `CavesGenerator` script attatched to it.

# Teacher Showcase
Who said only students could showcase their work?! Not me!

I created a new `BeachGenerator` script (added it to a new gameobject called `BeachGenerator`) then dragged and dropped it into the "Terrain Generator" slot in the `Chunk` script! And Vuala! Beaches!

![](/Assets/terrain_generator_beach_generator.png)

(PS: I also added a "rock" texture to the atlas, and created a rock voxel type for the beach terrain)