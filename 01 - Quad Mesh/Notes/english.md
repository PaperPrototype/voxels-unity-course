# Definitions

Vertex (singular) - Vertices (plural)
> A 3D position used to tell where a mesh's triangle corners will be.

Backface culling
> When a renderer doesn't render both sides of a triangle.

Index (singular) - indices? Indices? (plural)
> An index is a number that tells us the offset to find an item in a list or array.

Triangle(s)
> A set of 3 numbers telling us which vertices to use to make one corner of a triangle.
> The order that you put these ints affects the direction of the normals.
> [Triangle Mesh Wikipedia](https://en.wikipedia.org/wiki/Triangle_mesh)
> [Polygon Mesh Wikpedia](https://en.wikipedia.org/wiki/Polygon_mesh)

Normal(s)
> The direction that is 90 degrees "upwards" from a surface, for lighting purposes. Also the side of a triangle to render. Each vertex has a normal.
> 
> [Normals Wikpiedia](https://en.wikipedia.org/wiki/Normal_(geometry))
> [Vertex Normal Wikipedia](https://en.wikipedia.org/wiki/Vertex_normal)

Local (positioning and measuring)
> The positon of something relative to itself or its parent object.

Global (positioning and measuring)
> The position of something relative to the "world" or global position.

Vector
> 3 numbers used to represent a position or direction in space.

Bounds
> the area or volume something takes up in 3D, usally this the area is calculated with a box or cube shape, since the algorithm for calculating them is very simple.

UV(s)
> The position on a texture to map to a specific vertex of a triangle. A UV of `Vector2(0, 0)` gets the bottom left of a texture. 
> 
> ![2D uv texture coordinate](/Assets/2D_uv_texture_coordinate.png)

Unit Vector
> A vector who's magnitude is 1. The megnitude of a vector is just the distance from the start, or origin, to the "end" position that the vector is giving.
> 
> ![magnitude of a vector](/Assets/normal_magnitude.png)

Voxel
> Analogous to the word "pixel", vo standing for "volume" (instead of pixel's "picture") and el representing "element". Similar formations with el for "element" include the words "pixel" and "texel".
> [Voxel Wikipedia](https://en.wikipedia.org/wiki/Voxel)


# Keywords

`static`
> Marks variable as a "global" and stroes it ina special part of our program.

`readonly`
> Marks variables as "immutable". External classes can only read and not write to it.