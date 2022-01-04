# Shader ~~stealing~~ porting 101

## 1- Adding this to FNF:

**(THIS MIGHT BREAK ANY SHADERS YOU USED BEFORE)**

- Copy the source and assets folder to your fnf source folder.

- Add this in your `Paths.hx` 
```haxe
	inline static public function shaderFragment(key:String, ?library:String)
	{
		return getPath('shaders/$key.frag', TEXT, library);
	}
```

- Add those to your `PlayState.hx`

    - In the variables:

```haxe
	public static var animatedShaders:Map<String, DynamicShaderHandler> = new Map<String, DynamicShaderHandler>();
```

   - In update function:
```haxe
		for (shader in animatedShaders)
		{
			shader.update(elapsed);
		}
```

---

## 2- Porting Shaders from shadertoy:

**(PLEASE MAKE SURE TO READ THE LICENSE OF THE SHADER YOU ARE PORTING)**
- First of all, you need to know that shadertoy automatically uses the inputs below:

 ```cpp
 uniform vec3      iResolution;           // viewport resolution (in pixels)
 uniform float     iTime;                 // shader playback time (in seconds)
 uniform float     iTimeDelta;            // render time (in seconds)
 uniform int       iFrame;                // shader playback frame
 uniform float     iChannelTime[4];       // channel playback time (in seconds)
 uniform vec3      iChannelResolution[4]; // channel resolution (in pixels)
 uniform vec4      iMouse;                // mouse pixel coords. xy: current (if MLB down), zw: click
 uniform samplerXX iChannel0..3;          // input channel. XX = 2D/Cube
 uniform vec4      iDate;                 // (year, month, day, time in seconds)
 uniform float     iSampleRate;           // sound sample rate (i.e., 44100)
 ```

Currently, my implementation only supports the following inputs: (iResolution, iTime, iChannel0).

Support for iMouse input is planned for the future.
 - For this instance we will be porting https://www.shadertoy.com/view/lddXWS
 
 The shader code is the following:
 ```cpp
 const float RADIUS	= 100.0;
 const float BLUR	= 200.0;
 const float SPEED   = 2.0;
 void mainImage( out vec4 fragColor, in vec2 fragCoord )
 {
     vec2 uv = fragCoord.xy / iResolution.xy;
     vec4 pic = texture(iChannel0, vec2(uv.x, uv.y));
     
     vec2 center = iResolution.xy / 2.0;
     float d = distance(fragCoord.xy, center);
     float intensity = max((d - RADIUS) / (2.0 + BLUR * (1.0 + sin(iTime*SPEED))), 0.0);
     
     fragColor = vec4(intensity + pic.r, intensity + pic.g, intensity + pic.b, 1.0);
 }
 ```
 We need to modify some stuff, 
 - main function header `void mainImage( out vec4 fragColor, in vec2 fragCoord )` should be changed to `void main()` 
    and add this new line at the start of function: `vec2 fragCoord = openfl_TextureCoordv * iResolution;`
 - The shader outputs to `fragColor`, this should be changed to `gl_FragColor`
 - at the very start of the shader add those two uniforms:
     `uniform vec2 iResolution;`
     `uniform float iTime;`
 - if your shader makes use of `iChannel0` sampler, change that to `bitmap`, other channels are not supported (unless replacing them all with bitmap still serves your purpose, use common sense)
 - if your shader outputs alpha pixels and they're black (Black instead of transparent), Make sure to use\change texture function to `texture2D` instead of `texture`

- The final result should be like this:
```cpp
    uniform vec2 iResolution;
    uniform float iTime;

    const float RADIUS	= 200.0;
    const float BLUR	= 500.0;
    const float SPEED   = 2.0;

    void main()
    {
        vec2 fragCoord = openfl_TextureCoordv * iResolution;

        vec2 uv = fragCoord.xy / iResolution.xy;
        vec4 pic = texture2D(bitmap, vec2(uv.x, uv.y));
        
        vec2 center = iResolution.xy / 2.0;
        float d = distance(fragCoord.xy, center);
        float intensity = max((d - RADIUS) / (2.0 + BLUR * (1.0 + sin(iTime*SPEED))), 0.0);

        gl_FragColor = vec4(intensity + pic.r, intensity + pic.g, intensity + pic.b, 0.2);
    }
```
---
## 3- Usage:

-	Shaders should be placed in /shaders folder, with `.frag` extension, 
	See shaders folder for more examples.
- Params are the file name and `optimize` bool, this might help with heavy shaders **but only makes a difference on decent Intel CPUs (ones with SSE Instructions support)**.
 ```haxe
 new DynamicShaderHandler("Example");
FlxG.camera.setFilters([new ShaderFilter(animatedShaders["Example"].shader)]);
 ```
 or
 ```haxe
 var spr:FlxSprite = new ShaderSprite("Example");
 ```

 ---
 
 ## Credits:
- [SqirraRNG](https://github.com/gedehari): Runtime shaders workaround.
- Kemo (me): Handler, Improvements, Guide.

    - If you are going to port this to any engine, please make sure to leave a *noticable* credit and respect the effort, this was planned to be nexus engine exclusive but there ya :).

    - If you run into any issues (that aren't common sense), feel free to DM me on discord `kemo#1337`.
