
I’m just setting up the folder and path for an almost non-distracting environment. I’ll give some time each day to build a 3D renderer.

**What do I know about 3D renderers?**
Nothing. From the name, I can guess that they render an illusion of three-dimensional objects on a two-dimensional screen.
The math for this can’t be too complicated, so I can start writing again soon.

As of **4 October 2025**, there are **11 links** provided for understanding how to build a 3D renderer.

Starting with:

1. [**C++: *Introduction to Ray Tracing: A Simple Method for Creating 3D Images***](https://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-ray-tracing/how-does-it-work)

---
---
### Part 1 – How Does It Work?

Since it’s an introductory course on a subset of 3D rendering (which itself is a subset of CGI programming), it feels like being thrown into a jungle and having to familiarize yourself with the environment.
But the goal is to have something to play with by the end of the lesson.

Ray tracing works on the principles of optical physics — how we perceive light and how it interacts with surfaces.
It projects the three-dimensionality of objects using **perspective projection**.

I imagine it as a technique that evolved from the art of eyeballing perspective, to using strings on a canvas, to using rays of light on a digital canvas to make believable 3D images.

Each light ray that bounces off a surface can go in any direction, but we only see what enters our eyes and hits the back of the eyeball on the cone and rod cells.
The resolution of the image formed depends on how many rays enter the eye and whether they hit the right spots.

An image can be made to look a certain way by adjusting the **angle of incoming light** and the **properties of the surface** — how it absorbs and reflects certain wavelengths.

---

**Fun Fact:**
The theory that objects emit light and that we see them because light from the object enters our eyes is called the *intromission theory*. It was believed by thinkers like **Aristotle**, and later proven experimentally by **Ibn al-Haytham** (also the name of a Genshin Impact character I like).

Ibn al-Haytham can be considered the first true scientist by modern standards — and is also known as the **father of optics**.

---

## Part 2: The Ray Tracing Algorithm in a Nutshell

From his studies, we can conclude that the interaction of light with objects is necessary for visibility — either one alone (light or object) would be invisible.

There are mainly **two types of ray tracing**:

1. **Forward Ray Tracing:**
   This method aligns closely with how light works in real life. Light from a source shoots out in all directions, reflects off surfaces in all directions, and only a minuscule fraction of that light actually enters the camera or the eye — which is how we see things.
   However, it’s extremely resource-intensive and inefficient for computation.

2. **Backward Ray Tracing:**
   This is the reverse of how light behaves in real life. Here, we trace rays from the camera (or eye) into the scene to see how they interact with objects. When a ray hits an object, we cast another ray from the object toward the light source to determine whether it’s illuminated or in shadow.
   This method gives us the exact necessary light paths, making it far more efficient.

3. **Hybrid Tracing:**
   *(Note: not explained in the article — I just came across it online.)*
   It combines rasterization and ray tracing techniques for better performance and realism.

Like a Scooby-Doo reveal, it turns out ray tracing is **path tracing all along** — whether it starts from the light source or the eye, it falls under the broader umbrella of **path tracing** in computer graphics.

---

## Part 3: Implementing the Ray Tracing Algorithm

The fundamentals of the ray tracing algorithm are **Primary Rays** and **Shadow Rays**.

As I understand it, when an image is rendered pixel by pixel:

* For each pixel:

  * The algorithm launches a **primary ray** into the scene.
    The direction of this ray is determined by drawing a line from the eye (camera) to the center of the pixel.
  * The **primary ray’s path** is tracked to see if it intersects with any object.
    If multiple intersections occur, we choose the nearest one to the eye.
  * After finding the nearest intersection point, we launch a **secondary ray** (or shadow ray) from that point toward the light source.
  * If the shadow ray reaches the light source unobstructed, the object is illuminated; otherwise, something is casting a shadow on it.
* Repeating this for all pixels gives us a 2D image (a pixel grid) of a 3D scene.

---

### Pseudocode

```python
def renderScene(imageWidth, imageHeight, objects, eyePosition, lightPosition, light, pixels):
    for j in range(imageHeight):
        for i in range(imageWidth):
            # Determine the direction of the primary ray
            primRay = computePrimRay(i, j)

            # Search for intersections
            closestObject = None
            minDist = math.inf
            pHit = None
            nHit = None

            for obj in objects:
                hit, p, n = intersect(obj, primRay)
                if hit:
                    dist = distance(eyePosition, p)
                    if dist < minDist:
                        minDist = dist
                        closestObject = obj
                        pHit = p
                        nHit = n

            if closestObject is not None:
                # Compute shadow ray
                shadowRay = Ray(origin=pHit, direction=(lightPosition - pHit))
                is_in_shadow = False

                for obj in objects:
                    hit, _, _ = intersect(obj, shadowRay)
                    if hit:
                        is_in_shadow = True
                        break

                # Illuminate the pixel
                if not is_in_shadow:
                    pixels[i][j] = closestObject.color * light.brightness
                else:
                    pixels[i][j] = 0
            else:
                # Background color
                pixels[i][j] = 0
```

---

As I thought, the ray tracing algorithm is **simple yet elegant**.

**Arthur Appel** introduced ray tracing in his 1969 paper *“Some Techniques for Shading Machine Renderings of Solids.”*

Despite its straightforward and nature-inspired approach, ray tracing wasn’t widely used for decades for one main reason — **computers were too slow**.

But with advances in computational power, we’re starting to see this technique appear more frequently — not just for shadows and reflections, but for full, physically accurate rendering.

---

### Part 4: Adding Reflection and Refraction

To get more natural and authentic images of a 3D object on a 2D screen, we need to use a combination of **reflection** and **refraction**.

A paper on this topic titled *"An Improved Illumination Model for Shaded Display"* by **Turner Whitted** was an improvement upon **Appel’s ray tracing algorithm**. Whitted essentially introduced calculations for both reflection and refraction.

For **reflection**, the important values to know are:

* The **surface normal** at the incident point
* The **incident ray’s angle of approach**

For **refraction**, the same applies, but it also depends on the **material’s index of refraction**, which determines the angle at which light is transmitted through the material.

Since energy can neither be created nor destroyed, the percentage of light that is reflected and refracted should add up to **100%**. The exact ratio between reflected and refracted light is calculated using the **Fresnel equation**, given by the French physicist **Augustin-Jean Fresnel** (a cool name — you can call him Augustin, Jean, or Fresnel depending on the weather). He was the first to understand that light is a **transverse wave**.

Whitted’s algorithm comprises three main components:

1. **Reflection calculation** – A light ray is sent from the eye to the object. From the point of incidence, a shadow ray is drawn toward the light source to check for shadows, and the intensity of the light is factored in.
2. **Refraction calculation** – The same primary ray from the eye, upon hitting the surface, generates a **transmission ray** according to the material’s refractive index. This ray travels inside the object or exits through the other side (for translucent objects), where refraction is calculated again.
3. **Fresnel equation** – Determines the proportions of reflection and refraction using the surface normal at the point of incidence, the refractive index, and the angle of incidence.

---

#### Pseudo-code for Whitted’s Algorithm:

```cpp
color reflectionColor = computeReflectionColor(); 
color refractionColor = computeRefractionColor(); 

float Kr; // reflection mix value
float Kt; // refraction mix value

fresnel(refractiveIndex, normalHit, primaryRayDirection, &Kr, &Kt);

// Note: Kt = 1 - Kr
glassBallColorAtHit = Kr * reflectionColor + Kt * refractionColor;
```

---

The problem with reflective or refractive rays is the possibility of an **infinite mirror loop** — when reflections within an object go on forever. This is solved by introducing a **depth limiter** that stops recursion after a certain number of bounces.

This restriction helps make rendering faster .

---




