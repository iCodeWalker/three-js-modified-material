# Three.js Modified Materials

1. How to improve three.js built-in materials.

2. If we want to use MeshStandardMaterial, but we also want to apply a vertex animation, we need a way to improve that material. There are two ways using which we can do this.

   1. With a Three.js hook that let us play with the shaders and inject our code.
      We will us the given materials but at specific moments we will put our code in the material.

   2. By recreating the material, but following what is done in three.js code.
      In this we will copy a large chunk of code from the three.js and put in our custom shader, and than write our own code for the changes we need.
      2nd option is acceptable but the material can be really complex which includes, extensions, defines, uniform merging etc.

3. We will use the hooks to inject our code inside the material before it gets compiled.

4. Hooking the materials :

   1. We can hook the material compilation with the 'onBeforeCompile' property.
      The function is called by the three.js before the material is compiled.

      material.onBeforeCompile = function(shader){
      console.log(shader);
      }
