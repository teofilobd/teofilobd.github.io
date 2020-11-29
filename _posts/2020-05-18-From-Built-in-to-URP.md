---
published: true
layout: post
title: From Built-in to URP 
date: '2020-05-18'
categories:
  - unity
  - shaders
  - urp
  - graphics
---

Unity's [Scriptable Render Pipeline](https://unity.com/srp){:target="_blank"} represents a great advance on the way that unity deals with graphics, giving more power to the users to customize the pipeline the way they want. I have started to use the [Universal Render Pipeline (URP)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.1/manual/index.html){:target="_blank"} recently and, despite all its advantages over the built-in pipeline, it still suffers of lack of documentation. I mean, you can find information of every function available in the package documentation, but it's still hard to find examples and translations from built-in to URP. Unity's docs are (were?) good because when you look for something, you usually find the explanation of that and sometimes it's followed by an example of how to use that. 

I am pretty sure that Unity is working on improving the current situation of the docs (specially regarding packages), but while that's not available I decided to write this article with a kind of translation from built-in to URP. I'm doing this not only to help others, but to help myself as well, so I can find all things in one place.

Before starting, here are some useful links to help you dive into URP stuff:
- [URP package docs](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.1/manual/index.html){:target="_blank"}
- [URP github](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal){:target="_blank"}
- [Official examples](https://github.com/Unity-Technologies/UniversalRenderingExamples){:target="_blank"}
- [Unlit template](https://gist.github.com/phi-lira/10159a824e4e522060c47e21762941bb){:target="_blank"}
- [Boat attack demo project](https://github.com/Verasl/BoatAttack){:target="_blank"}
- [Phil Lira's shader examples](https://github.com/phi-lira/UniversalShaderExamples){:target="_blank"}
- [Outline post-effect using ScriptableRendererFeature](https://alexanderameye.github.io/outlineshader){:target="_blank"}
- [Phil Lira's twitter](https://twitter.com/phi_lira){:target="_blank"}

## Summary

Okay! So let's go! All things here are mostly based on version 7.3 (version I'm using currently) sooo... you know... things might change. Some of the sections follow the built-in documentation order. The article is organized as follows:

1. <a href="#general-structure-">General Structure</a>
2. <a href="#shader-include-files-">Shader Include Files</a>
3. <a href="#light-modes-">Light Modes</a>
4. <a href="#variants-">Variants</a>
5. <a href="#predefined-shader-preprocessor-macros-">Predefined Shader Preprocessor Macros</a>
	1. <a href="#helpers-">Helpers</a>
	2. <a href="#shadow-mapping-">Shadow Mapping</a>
	3. <a href="#texturesampler-declaration-macros-">Texture/Sampler Declaration Macros</a>
6. <a href="#built-in-shader-helper-functions-">Built-in Shader Helper Functions</a>
	1. <a href="#vertex-transformation-functions-">Vertex Transformation Functions</a>
	2. <a href="#generic-helper-functions-">Generic Helper Functions</a>
	3. <a href="#forward-rendering-helper-functions-">Forward Rendering Helper Functions</a>
	4. <a href="#screen-space-helper-functions-">Screen-space Helper Functions</a>
	5. <a href="#vertex-lit-helper-functions-">Vertex-lit Helper Functions</a>
7. <a href="#built-in-shader-variables-">Built-in Shader Variables</a>
	1. <a href="#lighting-">Lighting</a>
8. <a href="#random-stuff-">Random Stuff</a>
	1. <a href="#shadows-">Shadows</a>
	2. <a href="#fog-">Fog</a>
	3. <a href="#depth-">Depth</a>
	4. <a href="#etc-">Etc.</a>
9. <a href="#post-processingvfx-">Post-processing/VFX</a>
10. <a href="#conclusion-">Conclusion</a>

## General Structure <a href="#summary">↑</a>

First of all, add `"RenderPipeline" = "UniversalPipeline"` to your tags. Next, all URP shaders are written using `HLSL` embraced by `HLSLPROGRAM/ENDHLSL/etc.` macros. To avoid headaches, use them as well.

{:.table}
Built-in              | URP                   
--------------------- | --------------------- 
CGPROGRAM<br>HLSLPROGRAM  | HLSLPROGRAM
ENDCG<br>ENDHLSL | ENDHLSL
CGINCLUDE<br>HLSLINCLUDE | HLSLINCLUDE

## Shader Include Files <a href="#summary">↑</a>

{:.table}
Content |Built-in              | URP                   
---------------------|--------------------- | --------------------- 
Core | Unity.cginc | [Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl){:target="_blank"}
Light | AutoLight.cginc | [Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl){:target="_blank"}
Shadows | AutoLight.cginc | [Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl){:target="_blank"}
Surface shaders | Lighting.cginc | None, but you can find a side project for this [here](https://github.com/phi-lira/UniversalShaderExamples/tree/master/Assets/_ExampleScenes/51_LitPhysicallyBased){:target="_blank"}

Other useful includes:
- [Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl){:target="_blank"}
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl){:target="_blank"}
- [Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl){:target="_blank"}
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl){:target="_blank"}
- [Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl){:target="_blank"}
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl){:target="_blank"}
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTextue.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTexture.hlsl){:target="_blank"}

## Light Modes <a href="#summary">↑</a>

{:.table}
 Built-in              | URP                   
---------------------|--------------------- 
ForwardBase | UniversalForward
ForwardAdd | Gone
Deferred and related | [UniversalGBuffer seems to have just been added to URP](https://github.com/Unity-Technologies/Graphics/pull/71){:target="_blank"}
Vertex and related | Gone
ShadowCaster | ShadowCaster
MotionVectors | Not suppoted yet

The other light modes supported are:
- DepthOnly
- Meta (for lightmap baking)
- Universal2D

## Variants <a href="#summary">↑</a>

URP support some variants, so depending on the things you are using, you might need to add some `#pragma multi_compile` for some of the following keywords:

- `_MAIN_LIGHT_SHADOWS`
- `_MAIN_LIGHT_SHADOWS_CASCADE`
- `_ADDITIONAL_LIGHTS_VERTEX` 
- `_ADDITIONAL_LIGHTS`
- `_ADDITIONAL_LIGHT_SHADOWS`
- `_SHADOWS_SOFT`
- `_MIXED_LIGHTING_SUBTRACTIVE`

## Predefined Shader Preprocessor Macros <a href="#summary">↑</a>

### Helpers <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   
--------------------- | --------------------- 
**UNITY_PROJ_COORD**(*a*) | Gone. Do **a.xy/a.w** instead
**UNITY_INITIALIZE_OUTPUT**(*type*, *name*) | **ZERO_INITIALIZE**(*type*, *name*)

### Shadow Mapping <a href="#summary">↑</a>

You must include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl){:target="_blank"}.

{:.table}
Built-in              | URP                   
--------------------- | --------------------- 
**UNITY_DECLARE_SHADOWMAP**(*tex*) | **TEXTURE2D_SHADOW_PARAM**(*textureName*, *samplerName*)
**UNITY_SAMPLE_SHADOW**(*tex*, *uv*) | **SAMPLE_TEXTURE2D_SHADOW**(*textureName*, *samplerName*, *coord3*)  
**UNITY_SAMPLE_SHADOW_PROJ**(*tex*, *uv*) | **SAMPLE_TEXTURE2D_SHADOW**(*textureName*, *samplerName*, *coord4.xyz/coord4.w*)

### Texture/Sampler Declaration Macros <a href="#summary">↑</a>

Unity has a bunch of texture/sampler macros to improve cross compatibility between APIs, but people are not used to use them. Those still exist in URP, but now with different names and new additions. I will not put all of them here because it's a lot, but you can check their definitions per platform in the [API includes](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/API){:target="_blank"}.

{:.table}
Built-in              | URP                   
--------------------- | --------------------- 
**UNITY_DECLARE_TEX2D**(*name*) | **TEXTURE2D**(*textureName*); **SAMPLER**(*samplerName*);  
**UNITY_DECLARE_TEX2D_NOSAMPLER**(*name*) | **TEXTURE2D**(*textureName*);
**UNITY_DECLARE_TEX2DARRAY**(*name*) | **TEXTURE2D_ARRAY**(*textureName*); **SAMPLER**(*samplerName*); 
**UNITY_SAMPLE_TEX2D**(*name*, *uv*) | **SAMPLE_TEXTURE2D**(*textureName*, *samplerName*, *coord2*)
**UNITY_SAMPLE_TEX2D_SAMPLER**(*name*, *samplername*, *uv*) | **SAMPLE_TEXTURE2D**(*textureName*, *samplerName*, *coord2*)
**UNITY_SAMPLE_TEX2DARRAY**(*name*, *uv*) | **SAMPLE_TEXTURE2D_ARRAY**(*textureName*, *samplerName*, *coord2*, *index*)
**UNITY_SAMPLE_TEX2DARRAY_LOD**(*name*, *uv*, *lod*) | **SAMPLE_TEXTURE2D_ARRAY_LOD**(*textureName*, *samplerName*, *coord2*, *index*, *lod*)

Important to note that `SCREENSPACE_TEXTURE` has become `TEXTURE2D_X`. If you are working on some screen space effect for VR in *Single Pass Instanced* or *Multi-view* modes, you must declare the textures used with `TEXTURE2D_X`. This macro will handle for you the correct texture (array or not) declaration. You also have to sample the textures using `SAMPLE_TEXTURE2D_X` and use `UnityStereoTransformScreenSpaceTex` for the uv.

## Built-in Shader Helper Functions <a href="#summary">↑</a>

You can find them all in ["Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl){:target="_blank"}.

### Vertex Transformation Functions <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   
--------------------- | --------------------- 
*float4* **UnityObjectToClipPos**(*float3 pos*) | *float4* **TransformObjectToHClip**(*float3 positionOS*)
*float3* **UnityObjectToViewPos**(*float3 pos*) | **TransformWorldToView**(**TransformObjectToWorld**(*positionOS*))

### Generic Helper Functions <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
*float3* **WorldSpaceViewDir** (*float4 v*) | *float3* **GetWorldSpaceViewDir**(*float3 positionWS*) | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl){:target="_blank"}
*float3* **ObjSpaceViewDir** (*float4 v*) | Gone. Do **TransformWorldToObject**(**GetCameraPositionWS()**) - *objectSpacePosition*; |
*float2* **ParallaxOffset** (*half h*, *half height*, *half3 viewDir*) | Gone? Copy from UnityCG.cginc |
*fixed* **Luminance** (*fixed3 c*) | *real* **Luminance**(*real3 linearRgb*) | Include ["Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl){:target="_blank"}
*fixed3* **DecodeLightmap** (*fixed4 color*) | *real3* **DecodeLightmap**(*real4 encodedIlluminance*, *real4 decodeInstructions*) | Include ["Packages/com.unity.render-pipelines.core/ShaderLibrary/EntityLighting.hlsl"](https://github.com/Unity-Technologies/Graphics/blob/master/com.unity.render-pipelines.core/ShaderLibrary/EntityLighting.hlsl){:target="_blank"} <br> `decodeInstructions` is used as `half4(LIGHTMAP_HDR_MULTIPLIER, LIGHTMAP_HDR_EXPONENT, 0.0h, 0.0h)` by URP
*float4* **EncodeFloatRGBA** (*float v*) | Gone? Copy from UnityCG.cginc |
*float* **DecodeFloatRGBA** (*float4 enc*) | Gone? Copy from UnityCG.cginc  | 
*float2* **EncodeFloatRG** (*float v*) | Gone? Copy from UnityCG.cginc |
*float* **DecodeFloatRG** (*float2 enc*) | Gone? Copy from UnityCG.cginc |
*float2* **EncodeViewNormalStereo** (*float3 n*) | Gone? Copy from UnityCG.cginc |
*float3* **DecodeViewNormalStereo** (*float4 enc4*) | Gone? Copy from UnityCG.cginc |
 
### Forward Rendering Helper Functions <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
*float3* **WorldSpaceLightDir** (*float4 v*) | *_MainLightPosition.xyz* - **TransformObjectToWorld**(*objectSpacePosition*) | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl){:target="_blank"}
*float3* **ObjSpaceLightDir** (*float4 v*) | **TransformWorldToObject**(*_MainLightPosition.xyz*) - *objectSpacePosition* | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl){:target="_blank"}
*float3* **Shade4PointLights** (*...*) | Gone. You can try to use `half3 VertexLighting(float3 positionWS, half3 normalWS)` | For `VertexLighting(...)` include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl){:target="_blank"}
 
### Screen-space Helper Functions <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
*float4* **ComputeScreenPos** (*float4 clipPos*) | *float4* **ComputeScreenPos**(*float4 positionCS*) | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl){:target="_blank"}
*float4* **ComputeGrabScreenPos** (*float4 clipPos*) | Gone. |

### Vertex-lit Helper Functions <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
*float3* **ShadeVertexLights** (*float4 vertex*, *float3 normal*) | Gone. You can try to use `UNITY_LIGHTMODEL_AMBIENT.xyz + VertexLighting(...)` | For `VertexLighting(...)` include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl){:target="_blank"}
 
A bunch of utilities can be found in ["Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl){:target="_blank"}.

## Built-in Shader Variables <a href="#summary">↑</a>

Most of the shader variables remains the same, except by lighting.

### Lighting <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
*_LightColor0* | *_MainLightColor* | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl){:target="_blank"}
*_WorldSpaceLightPos0* | *_MainLightPosition* | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl){:target="_blank"}
*_LightMatrix0* | Gone ? Cookies are not supported yet
*unity_4LightPosX0*, *unity_4LightPosY0*, *unity_4LightPosZ0* |  In URP, additional lights are stored in an array/buffer (depending on platform). Retrieve light information using `Light GetAdditionalLight(uint i, float3 positionWS)` | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl){:target="_blank"}
*unity_4LightAtten0* 	| In URP, additional lights are stored in an array/buffer (depending on platform). Retrieve light information using `Light GetAdditionalLight(uint i, float3 positionWS)` | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl){:target="_blank"}
*unity_LightColor* 	| In URP, additional lights are stored in an array/buffer (depending on platform). Retrieve light information using `Light GetAdditionalLight(uint i, float3 positionWS)` | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl){:target="_blank"}
*unity_WorldToShadow* | `float4x4 _MainLightWorldToShadow[MAX_SHADOW_CASCADES + 1]` or `_AdditionalLightsWorldToShadow[MAX_VISIBLE_LIGHTS]` | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl){:target="_blank"}

If you want to loop over all additional lights using `GetAdditionalLight(...)`, you can query the additional lights count by using `GetAdditionalLightsCount()`.

## Random Stuff <a href="#summary">↑</a>

### Shadows <a href="#summary">↑</a>

For more info about shadows, check ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl){:target="_blank"}.

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
**UNITY_SHADOW_COORDS**(*x*) | Gone? DIY, e.g. `float4 shadowCoord : TEXCOORD0;` |
**TRANSFER_SHADOW**(*a*) | *a.shadowCoord* = **TransformWorldToShadowCoord**(*worldSpacePosition*) | With cascades on, do this on fragment to avoid visual artifacts
**SHADOWS_SCREEN** | Gone. Not supported.

### Fog <a href="#summary">↑</a>

For more info about fog, check ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl){:target="_blank"}.

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
**UNITY_FOG_COORDS**(*x*) | Gone? DIY, e.g. `float fogCoord : TEXCOORD0;` |
**UNITY_TRANSFER_FOG**(*o*, *outpos*) | *o.fogCoord* = **ComputeFogFactor**(*clipSpacePosition.z*); | 
**UNITY_APPLY_FOG**(*coord*, *col*) | *color* = **MixFog**(*color*, *i.fogCoord*); |

### Depth <a href="#summary">↑</a>

To use the camera depth texture, include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl){:target="_blank"} and the `_CameraDepthTexture` will be declared for you as well as helper the functions `SampleSceneDepth(...)` and `LoadSceneDepth(...)`.

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
**LinearEyeDepth**(*sceneZ*) | **LinearEyeDepth**(*sceneZ*, *_ZBufferParams*) | Include ["Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl){:target="_blank"}
**Linear01Depth**(*sceneZ*) | **Linear01Depth**(*sceneZ*, *_ZBufferParams*) | Include ["Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl){:target="_blank"}

### Etc. <a href="#summary">↑</a>

{:.table}
Built-in              | URP                   |
--------------------- | --------------------- | --------------------- 
**ShadeSH9**(*normal*) | **SampleSH**(*normal*) | Include ["Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl){:target="_blank"}
*unity_ColorSpaceLuminance* | Gone. Use `Luminance()` | Include ["Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl){:target="_blank"}

## Post-processing/VFX <a href="#summary">↑</a>

URP does not support `OnPreCull`, `OnPreRender`, `OnPostRender` and `OnRenderImage`. It does support `OnRenderObject` and `OnWillRenderObject`, but you might find issues depending on what you want to do. So, If you used to use those when creating your visual effects, I recommend learning how to use the new approaches available. The `RenderPipelineManager` provides the following injection points in the pipeline:
- `beginCameraRendering(ScriptableRenderContext context, Camera camera)`
- `endCameraRendering(ScriptableRenderContext context, Camera camera)`
- `beginFrameRendering(ScriptableRenderContext context,Camera[] cameras)`
- `endFrameRendering(ScriptableRenderContext context,Camera[] cameras)`

Example of usage:

```C#
void OnEnable()
{
	RenderPipelineManager.beginCameraRendering += MyCameraRendering;
}

void OnDisable()
{
	RenderPipelineManager.beginCameraRendering -= MyCameraRendering;
}

void MyCameraRendering(ScriptableRenderContext context, Camera camera)
{
	...
	if(camera == myEffectCamera)
	{
	...
	}
	...
}
```

Like I said, `OnWillRenderObject` is supported, however if you need to perform a render call inside of it (e.g. water reflection/refraction), it will not work. As soon as you call `Camera.Render()`, you will see the following message:

>Recursive rendering is not supported in SRP (are you calling Camera.Render from within a render pipeline?) 

In this case, replace the `OnWillRenderObject` by `begin/endCameraRendering` (like the example above) and call `RenderSingleCamera()` from URP instead of `Camera.Render()`. Changing the example above, you would have something like
```c#
void MyCameraRendering(ScriptableRenderContext context, Camera camera)
{
	...
	if(camera == myEffectCamera)
	{
	...
		UniversalRenderPipeline.RenderSingleCamera(context, camera);
	}
	...
}
```

The other approach to work with Post-processing is to use a `ScriptableRendererFeature`. [This post](https://alexanderameye.github.io/outlineshader){:target="_blank"} has a great explanation of an outline effect using this feature. A `ScriptableRendererFeature` allows you to inject `ScriptableRenderPass(es)` at different stages of the pipeline, thus being a powerful tool for creating post-processing effects. The injection places are the following:
- `BeforeRendering`
- `BeforeRenderingShadows`
- `AfterRenderingShadows`
- `BeforeRenderingPrepasses`
- `AfterRenderingPrePasses`
- `BeforeRenderingOpaques`
- `AfterRenderingOpaques`
- `BeforeRenderingSkybox`
- `AfterRenderingSkybox`
- `BeforeRenderingTransparents`
- `AfterRenderingTransparents`
- `BeforeRenderingPostProcessing`
- `AfterRenderingPostProcessing`
- `AfterRendering`

This is a simple example of a `ScriptableRendererFeature` performing a blit with a custom material:
```c#
public class CustomRenderPassFeature : ScriptableRendererFeature
{
    class CustomRenderPass : ScriptableRenderPass
    {
        CustomRPSettings _CustomRPSettings;
        RenderTargetHandle _TemporaryColorTexture;

        private RenderTargetIdentifier _Source;
        private RenderTargetHandle _Destination;

        public CustomRenderPass(CustomRPSettings settings)
        {
            _CustomRPSettings = settings;
        }

        public void Setup(RenderTargetIdentifier source, RenderTargetHandle destination)
        {
            _Source = source;
            _Destination = destination;
        }

        public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)
        {
            _TemporaryColorTexture.Init("_TemporaryColorTexture");
        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            CommandBuffer cmd = CommandBufferPool.Get("My Pass");

            if (_Destination == RenderTargetHandle.CameraTarget)
            {
                cmd.GetTemporaryRT(_TemporaryColorTexture.id, renderingData.cameraData.cameraTargetDescriptor, FilterMode.Point);
                cmd.Blit(_Source, _TemporaryColorTexture.Identifier());
                cmd.Blit(_TemporaryColorTexture.Identifier(), _Source, _CustomRPSettings.m_Material);
            }
            else
            {
                cmd.Blit(_Source, _Destination.Identifier(), _CustomRPSettings.m_Material, 0);
            }

            context.ExecuteCommandBuffer(cmd);
            CommandBufferPool.Release(cmd);
        }

        public override void FrameCleanup(CommandBuffer cmd)
        {
            if (_Destination == RenderTargetHandle.CameraTarget)
            {
                cmd.ReleaseTemporaryRT(_TemporaryColorTexture.id);
            }
        }
    }

    [System.Serializable]
    public class CustomRPSettings
    {
        public Material m_Material;
    }

    public CustomRPSettings m_CustomRPSettings = new CustomRPSettings();
    CustomRenderPass _ScriptablePass;

    public override void Create()
    {
        _ScriptablePass = new CustomRenderPass(m_CustomRPSettings);

        _ScriptablePass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        _ScriptablePass.Setup(renderer.cameraColorTarget, RenderTargetHandle.CameraTarget);
        renderer.EnqueuePass(_ScriptablePass);
    }
}
``` 

You can create a `ScriptableRendererFeature` by clicking on **"Create > Rendering > Universal Render Pipeline > Renderer Feature"**. The feature that you have created has to be added to your `ForwardRenderer`. To do so, select the `ForwardRenderer`, click on **"Add Renderer Feature"** and select the feature to be added. You can expose properties in the feature inspector, for example, if you add the example above, you will see a material slot available. 

## Conclusion <a href="#summary">↑</a>

That's it for now. I will try to keep this updated *(narrator: he won't)* according to the new things I'm learning or new features that I see added. If you have comments or suggestions, you can find me on [twitter](https://www.twitter.com/teodutra){:target="_blank"}.

