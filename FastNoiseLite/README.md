## Quickstart
Here's an example for creating a 128x128 array of OpenSimplex2 noise

```cs
// Create and configure FastNoise object
FastNoiseLite noise = new FastNoiseLite();
noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);

// Gather noise data
float[] noiseData = new float[128 * 128];
int index = 0;

for (int y = 0; y < 128; y++)
{
    for (int x = 0; x < 128; x++)
    {
        noiseData[index++] = noise.GetNoise(x, y);
    }
}

// Do something with this data...
```

## Info

See repository on github [https://github.com/Auburn/FastNoiseLite](https://github.com/Auburn/FastNoiseLite)

See Authors Github [https://github.com/Auburn](https://github.com/Auburn)

[![discord](https://img.shields.io/discord/703636892901441577?style=flat-square&logo=discord "Discord")](https://discord.gg/SHVaVfV)
[![npm](https://img.shields.io/npm/v/fastnoise-lite)](https://www.npmjs.com/package/fastnoise-lite)

# FastNoise Lite

FastNoise Lite is an extremely portable open source noise generation library with a large selection of noise algorithms. This library focuses on high performance while avoiding platform/language specific features, allowing for easy ports to as many possible languages.

This project is an evolution of the [original FastNoise](https://github.com/Auburn/FastNoise/tree/FastNoise-Legacy) library and shares the same goal: An easy to use library that can quickly be integrated into a project and provides performant modern noise generation. See a breakdown of changes from the transition to FastNoise Lite [here](https://github.com/Auburn/FastNoise/pull/49)

If you are looking for a more extensive noise generation library consider using [FastNoise2](https://github.com/Auburn/FastNoise2). It provides large performance gains thanks to SIMD and uses a node graph structure to allow complex noise configurations with lots of flexibility.

## Features

- 2D & 3D
- OpenSimplex2 Noise
- OpenSimplex2S Noise
- Cellular (Voronoi) Noise
- Perlin Noise
- Value Noise
- Value Cubic Noise
- OpenSimplex2-based Domain Warp
- Basic Grid Gradient Domain Warp
- Multiple fractal options for all of the above
- Supports floats and/or doubles

### Supported Languages

- [C#](https://github.com/Auburn/FastNoiseLite/blob/master/CSharp/README.md)
- [C++98](https://github.com/Auburn/FastNoiseLite/blob/master/Cpp/README.md)
- [C99](https://github.com/Auburn/FastNoiseLite/blob/master/C/README.md)
- [Java](https://github.com/Auburn/FastNoiseLite/blob/master/Java/README.md)
- [JavaScript](https://github.com/Auburn/FastNoiseLite/blob/master/JavaScript/README.md)
- [HLSL](https://github.com/Auburn/FastNoiseLite/blob/master/HLSL/README.md)

### [Getting Started](https://github.com/Auburn/FastNoiseLite/wiki#getting-started)
### [Documentation](https://github.com/Auburn/FastNoiseLite/wiki/Documentation)