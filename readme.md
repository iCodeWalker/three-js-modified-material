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
   float angle = position.y\*0.9;

7. Animating the rotation :

   1. In the onBeforeCompile, we have access to 'uniforms'.
   2. Add a 'uTime' uniform.
   3. Retrieve 'uTime' uniform in the common chunk.
   4. Use it on the angle.
      angle = (position.y + uTime)\*0.9;

   5. We cannot update the uTime in tick function because we cannot access the uTime outside the 'onBeforeCompile' function.
      To access it outside we can create a 'customUniforms' object outside of 'onBeforeCompile', and update it inside the 'onBeforeCompile'.

   6. Fixing the shadows :

      To handle shadows, Three.js do renders from lights point of view called shadow maps. When these renders of lights occurs, all the materials are replaced by the another set of materials called depth material. These depth materials does not twist.

      Create a plane behind the model to view this effect.

      const plane = new THREE.Mesh(
      new THREE.PlaneGeometry(15, 15, 15),
      new THREE.MeshStandardMaterial()
      );

      plane.rotation.y = Math.PI;
      plane.position.y = -5;
      plane.position.z = 5;
      scene.add(plane);

      We need to twist depth material too.

      The material used for the shadows is a 'MeshDepthMaterial', we cannot access it easily, but we can override it with the property 'customDepthMaterial'.

      Create a 'depthMaterial' with the 'MeshDepthMaterial' class. We can use THREE.RGBADepthPacking as 'depthPacking'

      Use this 'depthMaterial' on the 'customDepthMaterial' property.

      The drop shadow is fixed using this, but the core shadow (the shadow on the model) still has some issues.

      It can be done using fixing the normal.
      The normals are the data associated with the vertices that lets us know which direction is the outward direction.
      This outward direction is used for lights, shadows, reflection, and stuff like that.
      We rotated the vetices but we did not rotated the normals.

      The chunk handling the normals is called 'beginnormal_vertex'.
