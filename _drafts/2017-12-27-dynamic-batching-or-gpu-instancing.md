---
published: false
---
When you are developing a game, no matter what is your target device, you'll want to save draw calls in order to be able to keep pushing more stuff into your game using all the budget you have. 

There are some ways of reducing the number of draw calls in a game. Usually, people resort either to combining meshes manually or following engine's rules to creating batches. In Unity, you can combine meshes using their utility classes ([Unity's combine meshes utility](https://docs.unity3d.com/ScriptReference/Mesh.CombineMeshes.html)) or configure your meshes and materials in order to make them batch together ([Unity's draw call batching doc.](https://docs.unity3d.com/Manual/DrawCallBatching.html)). 

For static meshes, this kind of setup is easy and straightforward. However, for dynamic meshes, the things start to get a bit more complicated. These are the rules to be followed according to Unity's documentation:

- *Batching dynamic GameObjects has certain overhead per vertex, so batching is applied only to Meshes containing fewer than 900 vertex attributes in total.*
  - *If your Shader is using Vertex Position, Normal and single UV, then you can batch up to 300 verts, while if your Shader is using Vertex Position, Normal, UV0, UV1 and Tangent, then only 180 verts.*
  - *Note: attribute count limit might be changed in future.*
- *GameObjects are not batched if they contain mirroring on the transform (for example GameObject A with +1 scale and GameObject B with –1 scale cannot be batched together).*
- *Using different Material instances causes GameObjects not to batch together, even if they are essentially the same. The exception is shadow caster rendering.*
- *GameObjects with lightmaps have additional renderer parameters: lightmap index and offset/scale into the lightmap. Generally, dynamic lightmapped GameObjects should point to exactly the same lightmap location to be batched.*
- *Multi-pass Shaders break batching.*
  - *Almost all Unity Shaders support several Lights in forward rendering, effectively doing additional passes for them. The draw calls for “additional per-pixel lights” are not batched.*
  - *The Legacy Deferred (light pre-pass) rendering path has dynamic batching disabled, because it has to draw GameObjects twice. *

If you have several dynamic meshes in your game, it might be hard to follow theses rules, specially the one about the number of vertex attributes. However, there is another technique, called GPU instancing, that can be used for this purpose as well. 

Let's say you have many dynamic meshes into your scene, but theses meshes are all instances of a same mesh. If this is the case, I have good news for you. By using GPU instancing you can draw all of those instances with one draw call, and they can even have different properties in their material.

> GPU instancing can be seen as an alternative to the use of batches when you have several instances of a same mesh in a scene. 

# But how does GPU instancing work?

In short:
1. Send the mesh data to GPU (triangles, vertices, indices, etc.).
2. Send to GPU a list of properties per instance (position, scale, rotation, etc.)
3. Draw mesh x times at once, where x is the number of instances.

What you actually have to do in Unity:
1. Create a shader with support for GPU instancing and use it in the mesh's material (don't forget to enable it).
2. Add instances of your mesh into the scene.
3. (Optional) Create a script to setup different material properties per instance.

# Pros and Cons

Pros:
1. You're sending less data to GPU.
2. Your mesh is not limited by 900 vertex attributes.
3. Despite being the same mesh, you can have some variety by playing with the material properties.

Cons: 
1. No lightmap (do not forget to disable lightmap static).
2. Always the same mesh.
3. You have to create scripts to setup different material properties.

# Let's see in practice

## Basic example

In this first example, let's create a simple shader with GPU instancing capabilities and use it to replicate several cubes into a scene. The scene has a container made with boxes and a spawning point that creates cubes at according to a interval specified. This is the script responsible for spawning cubes:

```C#
using System.Collections;
using UnityEngine;

public class ObjectSpawnerBasic : MonoBehaviour
{
    public GameObject m_ObjectPrefab;
    public float m_SpawningInterval = 1f;

	void Start ()
    {
        StartCoroutine(SpawObjects());
	}

    IEnumerator SpawObjects()
    {
        WaitForSeconds waitForInterval = new WaitForSeconds(m_SpawningInterval);
        while(true)
        {
            GameObject go = Instantiate(m_ObjectPrefab, transform);
            go.transform.parent = transform;
            go.GetComponent<Rigidbody>().AddForce(new Vector3(Random.Range(0f, 100f), 
            												  Random.Range(0f, 100f), 
                                    						  Random.Range(0f, 100f)));

            yield return waitForInterval;
        }
    }
}

```

And this is the simplest shader with GPU instancing that you can have. It basically enables GPU instancing. I added simple light interaction so the cubes don't look flat.

```ShaderLab
Shader "Unlit/BasicInstancing"
{
	Properties
	{
		_Color ("Color", Color) = (1,1,1,1)
	}

	SubShader
	{
		Tags { "RenderType"="Opaque" }
		LOD 100

		Pass
		{
			Tags { "LightMode" = "ForwardBase"}

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			// Enable gpu instancing variants.
			#pragma multi_compile_instancing

			#include "UnityCG.cginc"

			struct appdata
			{
				float4 vertex : POSITION;
				float3 normal : NORMAL;

				// Need this for basic functionality.
				UNITY_VERTEX_INPUT_INSTANCE_ID
			};

			struct v2f
			{
				float4 vertex : SV_POSITION;
				float3 normal : TEXCOORD01;
				float3 worldPos : TEXCOORD02;
			};

			fixed4 _Color;
			
			v2f vert (appdata v)
			{
				v2f o;

				// Need this for basic functionality.
				UNITY_SETUP_INSTANCE_ID(v);

				o.normal = UnityObjectToWorldNormal(v.normal);
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
				o.vertex = UnityObjectToClipPos(v.vertex);
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				float3 normalDir = normalize(i.normal);
				float3 lightDir = _WorldSpaceLightPos0;

				// Simple light interaction.
				float3 diffuse = clamp(dot(normalDir, lightDir), 0.5, 1);

				return fixed4(diffuse * _Color.rgb, 1);
			}
			ENDCG
		}
	}
}

```

The cube prefab material uses the shader above and has the option 'Enable GPU Instancing' ticked. The scene is composed only by objects using this same material. As result you can see that we have only one draw for the whole scene below.

![Basic Example]({{site.baseurl}}/_drafts/BasicExample.JPG)


## Playing with properties

Now, let's create a script to change the properties of each cube by using material property blocks. In this case, each cube instance will have a different color.

To help managing each cube's color property, we are going to use the following script:

```C#
using UnityEngine;

public class ObjectPropertyHandler : MonoBehaviour
{
    public Color m_Color;

    private void Start()
    {
        // Create property block and set to the mesh.
        MaterialPropertyBlock propertyBlock = new MaterialPropertyBlock();
        propertyBlock.SetColor("_Color", m_Color);
        GetComponent<MeshRenderer>().SetPropertyBlock(propertyBlock);        
    }
}
```

It basically allows you to set the instance color property. Now, our cube spawner will be slightly different, I added a line that sets a random color to the cube instance as soon as it is created:

```C#

    IEnumerator SpawObjects()
    {
        WaitForSeconds waitForInterval = new WaitForSeconds(m_SpawningInterval);
        while (true)
        {
            GameObject go = Instantiate(m_ObjectPrefab, transform);
            go.transform.parent = transform;
            go.GetComponent<Rigidbody>().AddForce(new Vector3(Random.Range(0f, 100f), 
                                                              Random.Range(0f, 100f), 
                                                              Random.Range(0f, 100f)));
            ObjectPropertyHandler oph = go.AddComponent<ObjectPropertyHandler>();

            // Choose random color.
            oph.m_Color = new Color(Random.Range(0f, 1f), Random.Range(0f, 1f), Random.Range(0f, 1f), 1f);
            yield return waitForInterval;
        }
    }
```


## Playing with texture

As seen in the previous example, we can change properties of instances while keeping the instancing working. However, we cannot have different textures per instance, what would demand a lot of memory by the way. But, there is a workaround for this: we can set an UV offset per instance as well as different scale and translation for the local UV space. In other words, we can have an texture atlas and sample it differently per instance.

## Changing mesh with noise

This last example shows another way of varying your instances. In this case, I'm using the instance ID to control the speed of the waves on the sphere surfaces.

# Conclusion

GPU instancing is a powerful technique that you could be using in your game. This was just a simple and brief introduction to it, for more information about the capabilities and limitations of this technique you can check [Unity's documentation](https://docs.unity3d.com/Manual/GPUInstancing.html).