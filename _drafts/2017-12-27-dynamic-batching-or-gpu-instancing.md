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
                go.GetComponent<Rigidbody>().AddForce(new Vector3(Random.Range(0f, 100f), Random.Range(0f, 100f), Random.Range(0f, 100f)));

                yield return waitForInterval;
            }
        }

        // Show spawning point.
        private void OnDrawGizmos()
        {
            Gizmos.DrawWireSphere(transform.position, 0.1f);
        }
    }


## Playing with properties

Now, let's create a script to change the properties of each sphere by using property blocks.

## Playing with texture

As seen in the previous example, we can change properties of instances while keeping the instancing working. However, we cannot have different textures per instance, what would demand a lot of memory by the way. But, there is a workaround for this: we can set an UV offset per instance as well as different scale and translation for the local UV space. In other words, we can have an texture atlas and sample it differently per instance.

## Changing mesh with noise

This last example shows another way of varying your instances. In this case, I'm using the instance ID to control the speed of the waves on the sphere surfaces.

# Conclusion

GPU instancing is a powerful technique that you could be using in your game. This was just a simple and brief introduction to it, for more information about the capabilities and limitations of this technique you can check [Unity's documentation](https://docs.unity3d.com/Manual/GPUInstancing.html).