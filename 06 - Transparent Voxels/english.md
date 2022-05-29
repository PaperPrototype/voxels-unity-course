TODO
- update the voxel type system to have "layers" or "groups"

- voxels in same layer will consider eachother solid and combine with eachother, but consider other layers to be air 
	- 

- layers decide if their voxels will get a collision mesh or not?
	- by consequence each layer will have its own gameObject/layer per chunk

I hope you enjoyed the last lecture! Cause this one is going to be even less coding! And more fun!

The last lecture gave us water colored blocks. The only problem is that water is transparent. Well, we are going to fix that in this lecture by making a separate mesh for transparent voxels, and a separate mesh for non transparent voxels, which will give us the following (awesome!) looking terrain.

![](/Assets/chunk_water_final.png)

# Layers
TODO