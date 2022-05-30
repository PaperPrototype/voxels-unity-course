TODO
- update the voxel type system to have "layers" or "groups"

- voxels in same layer will consider eachother solid and combine with eachother, but consider other layers to be air 
	- 

- layers decide if their voxels will get a collision mesh or not?
	- by consequence each layer will have its own gameObject/layer per chunk

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

As you can see we make the Layer class serializable so that we can edit it in the inspector. Everything we need to build a mesh is in this class! There is something we haven't seen before. That is the `MeshCollider` component. If `useCollider` is true then we will apply a mesh collider to our terrain.

