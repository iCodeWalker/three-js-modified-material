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

5. Adding content to the vertex shader :
   We will use #include ... to inject our code with a native Javascript replace(...)

   'begin_vertex' is handling the position, by creating a variable named 'transformed'
   we can replace #include <begin_vertex> inside the 'onBeforeCompile' function

   // Move the head by changing the y of transformed
   transformed.y += 3.0;

   // Twisting
   To twist the model, we need to do a rotation that vary depending on the elevation.

   Create a 'angle' variable
   float angle = 0.3;

   We are going to use a matrix. The rotation will only occur on x and z which is why we need a 2D rotation matrix.
   mat2 get2dRotateMatrix(float_angle)
   {
   return mat2(cos(\_angle), -sin(\_angle), sin(\_angle), cos(\_angle));
   }

   inclide inside common replace.

   create the 'rotateMatrix' variable using the 'get2dRotateMatrix' function
   mat2 rotateMatrix = get2dRotateMatrix(angle);

   apply this matrix to the x and z properties together.
   transformed.xz = rotateMatrix\*transformed.xz;

6. Make angle vary according to elevation (height) :
