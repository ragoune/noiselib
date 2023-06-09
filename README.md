# noiselib
An implementation of libnoise in Go
This is a fork of the [worldsproject/noiselib](https://github.com/worldsproject/noiselib) project.
The objective here is to first fix the remaining bugs & then refactorize the package to highly improve the performances (using goroutines ?) and have a more "Go" approach.

# Goals
The goal of this library is to provide a complete reimplementation of http://libnoise.sourceforge.net/ in Go.
It should be functionally equivilent, and provide very similar outputs as compared to the original.

# What is to come
There still is some known bugs waiting to be resolved:
- The Rotatepoint module does not always seem to be computed, when it is the input of another module. The initial noise sometime returns to its initial axis.
- The Cylinder is vertically aligned by default, and it is not documented as so in the C libnoise. 

Next, this package is a 1:1 implementation of the C library. Some patterns are not "Go like" and will need some refactorization: 
a better usage of interface, a default Module structure (having the default methods, like SetSourceModule).

The performances could also be highly improved, using goroutines, as it takes 1 or 2 seconds sometime to generate an image.

# Examples

First, let's generate a simple constant module noise:
```go
    constant := noiselib.Constant{Value: 0.0}
```
Remember that values in libNoise go between [-1;1] so creating a constant with 0 value will give us an in-between result.

Now let's dive in a more complex example with a perlin noise:
```go
    perlin := noiselib.DefaultPerlin()
    perlin.Seed = 101
    perlin.Frequency = 3.0
```
All the modules properties are accessible, so we can set our parameters easily (refer to C libnoise documentation to see all the properties). Most of the modules come with a `Default` function to generate a standard value.
You want to use it as a constructor most of the time.

This last example is also valid if you want to create a Voronoi, a Billow, a RidgedMulti etc.

Now, the whole point of libnoise is that modules can be joined to create a complex texture:
```go
    seed := int(time.Now().Unix())
    baseVoronoi := noiselib.DefaultVoronoi()
    baseVoronoi.Seed = seed
    baseVoronoi.Frequency = 4.0
    baseVoronoi.EnableDistance = false
    
    basePerlin := noiselib.DefaultPerlin()
    basePerlin.Seed = seed
    basePerlin.Frequency = 2.0

    addModule := noiselib.Add{SourceModule: make([]noiselib.Module, noiselib.AddModuleCount)}
    addModule.SetSourceModule(0, baseVoronoi)
    addModule.SetSourceModule(1, basePerlin)
```
Now we can use the Add module as our primary source.

The package simplifies the output of this addModule to an image file :
``` go
    gradient := noiselib.GradientColor{}
    gradient.ClearGradient()
    gradient.AddGradientPoint(-1.0, color.RGBA{0.0, 0.0, 0.0, 1})
    gradient.AddGradientPoint(1.0, color.RGBA{1.0, 1.0, 1.0, 1})

	var img = image.NewRGBA(image.Rectangle{
		Min: image.Point{0, 0},
		Max: image.Point{256, 256},
	})

    isSeamless := false
	noisemap := noiselib.NoiseMapPlane(0.0, 1.0, 0.0, 1.0, 256, 256, isSeamless, addModule)
	renderImage := noiselib.RenderImage{
		DestinationImage: *img,
		NoiseMap:         noisemap,
	}

	renderImage.Gradient = gradient

	renderImage.RenderFile("test.png")
```
A gradient requires at least 2 parameters.