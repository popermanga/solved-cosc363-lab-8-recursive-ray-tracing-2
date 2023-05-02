Download Link: https://assignmentchef.com/product/solved-cosc363-lab-8-recursive-ray-tracing-2
<br>
In this lab, we will extend the ray tracer developed in lab-07 to generate planar surfaces,  reflections, patterns and textures.

I.  RayTracer.cpp  from Lab07

<ol>

 <li>In Lab07, we developed a basic ray tracer for rendering a scene containing a set of spheres, and implemented ambient, diffuse, and specular reflections and shadows (Fig. 1).</li>

 <li>We will now generate reflections on the blue sphere. Reflectivity is one of the attributes of a sceneObject. The material properties of scene objects such as color, reflectivity, and specularity are specified in the initialize() Please add the following statement in this function:</li>

</ol>

sphere1-&gt;setReflectivity(true, 0.8);

The second parameter in the above function provides the value of the coefficient of reflection <em><sub>r</sub></em> .  For scene objects that are marked as reflective, we need to generate secondary rays along the direction of reflection at points of intersection and recursively call the trace() function to get the reflected colour values.  This is done by including the following code segment in the trace() function (after computing the shadow color, before returning the final color value). See also Lec08-Slide 18:

if (obj-&gt;isReflective() &amp;&amp; step &lt; MAX_STEPS)

{

float rho = obj-&gt;getReflectionCoeff();         glm::vec3 normalVec = obj-&gt;normal(ray.hit);

glm::vec3 reflectedDir = glm::reflect(ray.dir, normalVec);           Ray reflectedRay(ray.hit, reflectedDir);            glm::vec3 reflectedColor = trace(reflectedRay, step + 1);           color = color + (rho * reflectedColor);

}







The variable MAX_STEPS is used to limit the depth of recursion to a prespecified value.  A sample output with reflections is shown in Fig. 2.  Note that with reflections, the program takes significantly more time to render a scene.

<strong>Fig. 2</strong>.

<h2>Plane.cpp</h2>

The Plane class is a subclass of SceneObject, and has a constructor that takes either three or four vertices that form the boundary vertices for a planar region. The header file Plane.h  and the implementation file Plane.cpp for this class, are provided.  Being a subclass of SceneObject, the Plane class must provide implementations for the functions intersect()  and  normal().

Let us take a look at the functions in Plane.cpp.   The intersect()  function uses the following equation to find the point of intersection between the ray and the plane  (Slides 27, 29), and returns the value of the ray parameter  <em>t</em>  at the point of intersection.

(<em>A</em> <em>p</em><sup>0</sup>)<strong><em>n</em></strong>

<em>t </em>                                                                                                  (1)

<h2><em>d.n</em></h2>

The surface normal vector <strong><em>n</em></strong> of the plane is computed as (<em>C</em><em>B</em>)(<em>A</em><em>B</em>)    (Fig. 3),  where <em>A</em>, <em>B</em>, <em>C</em>, <em>D</em> are the four vertices the plane defined in an anti-clockwise sense with respect to the required direction of the normal vector.

Even though the normal of a plane is independent of the point at which it is computed, we need to use the standard signature of the normal function (normal(p)) as specified in the SceneObject class.

The function isInside(q)  implements a point inclusion test to determine whether a point <em>q</em> is inside the (triangular or quadrilateral) region on the plane’s surface, given by the user specified (three or four) boundary points. This test uses a set of vectors computed using the boundary points, as shown on slide Lec-08Slide 29.

<ol start="3">

 <li>In the initialize() function of the ray tracer, create a pointer to a plane object as shown below. The four parameters define the vertices of the floor plane.</li>

</ol>




Plane *plane = new Plane (glm::vec3(-20., -15, -40),    //Point A                            glm::vec3(20., -15, -40),     //Point B                     glm::vec3(20., -15, -200),    //Point C

glm::vec3(-20., -15, -200));    //Point D and add this to the list of sceneObjects:

sceneObjects.push_back(plane);

Remember to add the statement   #include “Plane.h”  at the beginning of the program.  You should get an output similar to that shown in Fig. 4:

<strong>Fig. 4.</strong>

<ol start="4">

 <li>Please note that the plane shown in the above figure is an infinite plane. The intersect() method does not use the boundary vertices of the plane. We need to check if the point of intersection given by the ray parameter <em>t</em> computed by the intersect() method lies within the boundaries of the plane.</li>

</ol>

Please edit the file “Plane.cpp” and replace the last statement (return t;) of the intersect() method with the following code:




glm::vec3 q = p0 + dir*t;   //Point of intersection

if( isInside(q) ) return t;   //Inside the plane   else return -1;             //Outside

You should now get a finite plane bounded by the vertices, as shown in Fig. 5.




<h3>                                              Fig. 5                                                          Fig. 6</h3>

Specular reflections from the floor plane can be disabled (Fig. 6)  by setting the specularity property of the floor plane in the initialize() function:

plane-&gt;setSpecularity(false);

<ol start="5">

 <li>We will now generate a stripe pattern on the floor plane by modifying the plane’s material colour value at each point. We will use the method shown in Fig. 7 to generate the pattern.</li>

</ol>

0         20          40          60          80           100              <em><sup>x </sup></em>

<table width="116">

 <tbody>

  <tr>

   <td width="116">int  ix = x/20; int   k = ix%2;</td>

  </tr>

 </tbody>

</table>

ix=0          ix=1        ix=2       ix=3       ix=4

<strong>Fig. 7                </strong>k=0           k=1         k=0         k=1        k=0




A range of values along any coordinate axes (<em>x </em>in the above example) can be subdivided into sub-ranges of fixed widths (20 in the above example) by using an integer division of the coordinate value by the width. Using a modulo operation (2 in the above example) we can convert the range to a repeating pattern of integers corresponding to the number of colour values of a pattern.

We will use the above method to convert the z-coordinate values on the plane into values 0 or 1, and assign colour values to those points.  Include the following code segment in the trace() function after the statement “obj = sceneObjects[ray.index]”:

if (ray.index == 4)

{

//Stripe pattern    int stripeWidth = 5;    int iz = (ray.hit.z) / stripeWidth;

int k = iz % 2;                  //2 colors

if (k == 0) color = glm::vec3(0, 1, 0);    else color = glm::vec3(1, 1, 0.5);    obj-&gt;setColor(color);

}




The above code assumes that the index of the plane is 4.  Please use the correct index from the initialize() function based on the position of the plane in the sceneObjects list.   The output with the stripe pattern is shown in Fig. 8.

<h3> Fig. 8</h3>




<ol start="6">

 <li>Texture mapping requires a mapping of the coordinates (x, y, z) at the point of intersection (ray.hit) to a pair of texture coordinate values in the range 0-1. Regions on a two-dimensional plane can be easily mapped to this range using a simple linear transformation as shown in Fig. 9 below (see also Lec08-Slide33) .</li>

</ol>

texcoords = (ray.hit.x  x1)/(x2x1); texcoordt = (ray.hit.z  z1)/(z2z1);

<strong>Fig. 9 </strong>

The files “TextureBMP.h”, and  “TextureBMP.cpp” required for loading a texture in bitmap (.bmp;  24 bits per pixel) format, are provided.  Add the statement

#include “TextureBMP.h”  at the beginning of the program.  Also, add the declaration statement  for a texture object at the beginning of the program.

TextureBMP texture;

In the initialize()  function,  include the statement

texture = TextureBMP(“Butterfly.bmp”);

On page 4, we provided the code for modifying the colour values of the plane using an “if” statement to generate a stripe pattern.  Please modify this code block as follows.




if (ray.index == 4)

{

//Stripe pattern    …   obj-&gt;setColor(color);




//Add code for texture mapping here    <strong>float texcoords =     float texcoordt = </strong>

<strong>   if(texcoords &gt; 0 &amp;&amp; texcoords &lt; 1 &amp;&amp;      texcoordt &gt; 0 &amp;&amp; texcoordt &lt; 1) </strong>

<strong>   { </strong>

<strong>     color=texture.getColorAt(texcoords, texcoordt);      obj-&gt;setColor(color); </strong>

<strong>   } </strong>




}

In the space shown in the code snippet above, add statements for computing the texture coordinates as given in Fig. 9, for a region of the floor plane shown in Fig.

<ol start="10">

 <li>The expected output of the ray tracer is shown in Fig. 11.</li>

</ol>