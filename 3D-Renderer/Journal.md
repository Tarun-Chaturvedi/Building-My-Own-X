
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

```
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

**Part 5 – Writing a Basic Raytracer**

So we’re now in the deep end of this journey of understanding raytracing — actually seeing how one works on our computer.

Before that, there’s an interesting side path about compressing program code into smaller and smaller forms, a style called *minimalistic programming*. In the vein of raytracing, a smart guy named **Paul Heckbert** wrote a tiny raytracer. The code is still above my understanding right now, but it’s cool to look at how it exists:

```cpp
// minray > minray.ppm
#include <stdlib.h>
#include <stdio.h>
#include <math.h>
typedef int i;typedef float f;struct v{f x,y,z;v operator+(v r){return v(x+r.x,y+r.y,z+r.z);}v operator*(f r){return v(x*r,y*r,z*r);}f operator%(v r){return x*r.x+y*r.y+z*r.z;}v(){}v operator^(v r){return v(y*r.z-z*r.y,z*r.x-x*r.z,x*r.y-y*r.x);}v(f a,f b,f c){x=a;y=b;z=c;}v operator!(){return*this*(1/sqrt(*this%*this));}};i G[]={247570,280596,280600,249748,18578,18577,231184,16,16};f R(){return(f)rand()/RAND_MAX;}i T(v o,v d,f&t,v&n){t=1e9;i m=0;f p=-o.z/d.z;if(.01<p)t=p,n=v(0,0,1),m=1;for(i k=19;k--;)for(i j=9;j--;)if(G[j]&1<<k){v p=o+v(-k,0,-j-4);f b=p%d,c=p%p-1,q=b*b-c;if(q>0){f s=-b-sqrt(q);if(s<t&&s>.01)t=s,n=!(p+d*t),m=2;}}return m;}v S(v o,v d){f t;v n;i m=T(o,d,t,n);if(!m)return v(.7,.6,1)*pow(1-d.z,4);v h=o+d*t,l=!(v(9+R(),9+R(),16)+h*-1),r=d+n*(n%d*-2);f b=l%n;if(b<0||T(h,l,t,n))b=0;f p=pow(l%r*(b>0),99);if(m&1){h=h*.2;return((i)(ceil(h.x)+ceil(h.y))&1?v(3,1,1):v(3,3,3))*(b*.2+.1);}return v(p,p,p)+S(h,r)*.5;}i main(){printf("P6 512 512 255 ");v g=!v(-6,-16,0),a=!(v(0,0,1)^g)*.002,b=!(g^a)*.002,c=(a+b)*-256+g;for(i y=512;y--;)for(i x=512;x--;){v p(13,13,13);for(i r=64;r--;){v t=a*(R()-.5)*99+b*(R()-.5)*99;p=S(v(17,16,8)+t,!(t*-1+(a*(R()+x)+b*(y+R())+c)*16))*3.5+p;}printf("%c%c%c",(i)p.x,(i)p.y,(i)p.z);}}
```

I tried running the raytracer in both C++ and Python, The c++ one was given by the article ,the python one is for more abstract understanding faster to read or recall.

C++
```cpp
#include <cstdlib>
#include <cstdio>
#include <cmath>
#include <fstream>
#include <vector>
#include <iostream>
#include <cassert>

#if defined __linux__ || defined __APPLE__
#else
#define M_PI 3.141592653589793
#define INFINITY 1e8
#endif

template<typename T>
class Vec3
{
public:
    T x, y, z;
    Vec3() : x(T(0)), y(T(0)), z(T(0)) {}
    Vec3(T xx) : x(xx), y(xx), z(xx) {}
    Vec3(T xx, T yy, T zz) : x(xx), y(yy), z(zz) {}
    Vec3& normalize()
    {
        T nor2 = length2();
        if (nor2 > 0) {
            T invNor = 1 / sqrt(nor2);
            x *= invNor, y *= invNor, z *= invNor;
        }
        return *this;
    }
    Vec3<T> operator * (const T &f) const { return Vec3<T>(x * f, y * f, z * f); }
    Vec3<T> operator * (const Vec3<T> &v) const { return Vec3<T>(x * v.x, y * v.y, z * v.z); }
    T dot(const Vec3<T> &v) const { return x * v.x + y * v.y + z * v.z; }
    Vec3<T> operator - (const Vec3<T> &v) const { return Vec3<T>(x - v.x, y - v.y, z - v.z); }
    Vec3<T> operator + (const Vec3<T> &v) const { return Vec3<T>(x + v.x, y + v.y, z + v.z); }
    Vec3<T>& operator += (const Vec3<T> &v) { x += v.x, y += v.y, z += v.z; return *this; }
    Vec3<T>& operator *= (const Vec3<T> &v) { x *= v.x, y *= v.y, z *= v.z; return *this; }
    Vec3<T> operator - () const { return Vec3<T>(-x, -y, -z); }
    T length2() const { return x * x + y * y + z * z; }
    T length() const { return sqrt(length2()); }
    friend std::ostream & operator << (std::ostream &os, const Vec3<T> &v)
    {
        os << "[" << v.x << " " << v.y << " " << v.z << "]";
        return os;
    }
};

typedef Vec3<float> Vec3f;

class Sphere
{
public:
    Vec3f center;                           /// position of the sphere
    float radius, radius2;                  /// sphere radius and radius^2
    Vec3f surfaceColor, emissionColor;      /// surface color and emission (light)
    float transparency, reflection;         /// surface transparency and reflectivity
    Sphere(
        const Vec3f &c,
        const float &r,
        const Vec3f &sc,
        const float &refl = 0,
        const float &transp = 0,
        const Vec3f &ec = 0) :
        center(c), radius(r), radius2(r * r), surfaceColor(sc), emissionColor(ec),
        transparency(transp), reflection(refl)
    { /* empty */ }
    bool intersect(const Vec3f &rayorig, const Vec3f &raydir, float &t0, float &t1) const
    {
        Vec3f l = center - rayorig;
        float tca = l.dot(raydir);
        if (tca < 0) return false;
        float d2 = l.dot(l) - tca * tca;
        if (d2 > radius2) return false;
        float thc = sqrt(radius2 - d2);
        t0 = tca - thc;
        t1 = tca + thc;
        
        return true;
    }
};

#define MAX_RAY_DEPTH 5

float mix(const float &a, const float &b, const float &mix)
{
    return b * mix + a * (1 - mix);
}

Vec3f trace(
    const Vec3f &rayorig,
    const Vec3f &raydir,
    const std::vector<Sphere> &spheres,
    const int &depth)
{
    float tnear = INFINITY;
    const Sphere* sphere = NULL;
    for (unsigned i = 0; i < spheres.size(); ++i) {
        float t0 = INFINITY, t1 = INFINITY;
        if (spheres[i].intersect(rayorig, raydir, t0, t1)) {
            if (t0 < 0) t0 = t1;
            if (t0 < tnear) {
                tnear = t0;
                sphere = &spheres[i];
            }
        }
    }
    if (!sphere) return Vec3f(2);
    Vec3f surfaceColor = 0; // color of the ray/surfaceof the object intersected by the ray
    Vec3f phit = rayorig + raydir * tnear; // point of intersection
    Vec3f nhit = phit - sphere->center; // normal at the intersection point
    nhit.normalize(); // normalize normal direction
    float bias = 1e-4; // add some bias to the point from which we will be tracing
    bool inside = false;
    if (raydir.dot(nhit) > 0) nhit = -nhit, inside = true;
    if ((sphere->transparency > 0 || sphere->reflection > 0) && depth < MAX_RAY_DEPTH) {
        float facingratio = -raydir.dot(nhit);
        float fresneleffect = mix(pow(1 - facingratio, 3), 1, 0.1);
        Vec3f refldir = raydir - nhit * 2 * raydir.dot(nhit);
        refldir.normalize();
        Vec3f reflection = trace(phit + nhit * bias, refldir, spheres, depth + 1);
        Vec3f refraction = 0;
        if (sphere->transparency) {
            float ior = 1.1, eta = (inside) ? ior : 1 / ior; // are we inside or outside the surface?
            float cosi = -nhit.dot(raydir);
            float k = 1 - eta * eta * (1 - cosi * cosi);
            Vec3f refrdir = raydir * eta + nhit * (eta *  cosi - sqrt(k));
            refrdir.normalize();
            refraction = trace(phit - nhit * bias, refrdir, spheres, depth + 1);
        }
        surfaceColor = (
            reflection * fresneleffect +
            refraction * (1 - fresneleffect) * sphere->transparency) * sphere->surfaceColor;
    }
    else {
        for (unsigned i = 0; i < spheres.size(); ++i) {
            if (spheres[i].emissionColor.x > 0) {
                Vec3f transmission = 1;
                Vec3f lightDirection = spheres[i].center - phit;
                lightDirection.normalize();
                for (unsigned j = 0; j < spheres.size(); ++j) {
                    if (i != j) {
                        float t0, t1;
                        if (spheres[j].intersect(phit + nhit * bias, lightDirection, t0, t1)) {
                            transmission = 0;
                            break;
                        }
                    }
                }
                surfaceColor += sphere->surfaceColor * transmission *
                std::max(float(0), nhit.dot(lightDirection)) * spheres[i].emissionColor;
            }
        }
    }
    
    return surfaceColor + sphere->emissionColor;
}

void render(const std::vector<Sphere> &spheres)
{
    unsigned width = 640, height = 480;
    Vec3f *image = new Vec3f[width * height], *pixel = image;
    float invWidth = 1 / float(width), invHeight = 1 / float(height);
    float fov = 30, aspectratio = width / float(height);
    float angle = tan(M_PI * 0.5 * fov / 180.);
    for (unsigned y = 0; y < height; ++y) {
        for (unsigned x = 0; x < width; ++x, ++pixel) {
            float xx = (2 * ((x + 0.5) * invWidth) - 1) * angle * aspectratio;
            float yy = (1 - 2 * ((y + 0.5) * invHeight)) * angle;
            Vec3f raydir(xx, yy, -1);
            raydir.normalize();
            *pixel = trace(Vec3f(0), raydir, spheres, 0);
        }
    }
    std::ofstream ofs("./untitled.ppm", std::ios::out | std::ios::binary);
    ofs << "P6\n" << width << " " << height << "\n255\n";
    for (unsigned i = 0; i < width * height; ++i) {
        ofs << (unsigned char)(std::min(float(1), image[i].x) * 255) <<
               (unsigned char)(std::min(float(1), image[i].y) * 255) <<
               (unsigned char)(std::min(float(1), image[i].z) * 255);
    }
    ofs.close();
    delete [] image;
}
int main(int argc, char **argv)
{
    srand48(13);
    std::vector<Sphere> spheres;
    spheres.push_back(Sphere(Vec3f( 0.0, -10004, -20), 10000, Vec3f(0.20, 0.20, 0.20), 0, 0.0));
    spheres.push_back(Sphere(Vec3f( 0.0,      0, -20),     4, Vec3f(1.00, 0.32, 0.36), 1, 0.5));
    spheres.push_back(Sphere(Vec3f( 5.0,     -1, -15),     2, Vec3f(0.90, 0.76, 0.46), 1, 0.0));
    spheres.push_back(Sphere(Vec3f( 5.0,      0, -25),     3, Vec3f(0.65, 0.77, 0.97), 1, 0.0));
    spheres.push_back(Sphere(Vec3f(-5.5,      0, -15),     3, Vec3f(0.90, 0.90, 0.90), 1, 0.0));
    spheres.push_back(Sphere(Vec3f( 0.0,     20, -30),     3, Vec3f(0.00, 0.00, 0.00), 0, 0.0, Vec3f(3)));
    render(spheres);
    
    return 0;
}
```
Python
```python
import math
import struct

# ---------------------------
# Vec3 class
# ---------------------------
class Vec3:
    def __init__(self, x=0, y=0, z=0):
        self.x = float(x)
        self.y = float(y)
        self.z = float(z)

    def __add__(self, v):
        return Vec3(self.x + v.x, self.y + v.y, self.z + v.z)

    def __sub__(self, v):
        return Vec3(self.x - v.x, self.y - v.y, self.z - v.z)

    def __mul__(self, f):
        if isinstance(f, Vec3):
            return Vec3(self.x * f.x, self.y * f.y, self.z * f.z)
        return Vec3(self.x * f, self.y * f, self.z * f)

    __rmul__ = __mul__

    def dot(self, v):
        return self.x * v.x + self.y * v.y + self.z * v.z

    def length(self):
        return math.sqrt(self.x*self.x + self.y*self.y + self.z*self.z)

    def normalize(self):
        l = self.length()
        if l > 0:
            self.x /= l
            self.y /= l
            self.z /= l
        return self

    def __repr__(self):
        return f"[{self.x} {self.y} {self.z}]"


# ---------------------------
# Sphere class
# ---------------------------
class Sphere:
    def __init__(self, center, radius, surfaceColor, reflection=0.0, transparency=0.0, emissionColor=Vec3(0,0,0)):
        self.center = center
        self.radius = radius
        self.radius2 = radius * radius
        self.surfaceColor = surfaceColor
        self.emissionColor = emissionColor
        self.transparency = transparency
        self.reflection = reflection

    # Ray-sphere intersection
    def intersect(self, rayorig, raydir):
        L = self.center - rayorig
        tca = L.dot(raydir)
        if tca < 0:
            return False, None, None
        d2 = L.dot(L) - tca * tca
        if d2 > self.radius2:
            return False, None, None
        thc = math.sqrt(self.radius2 - d2)
        return True, tca - thc, tca + thc


MAX_RAY_DEPTH = 5

def mix(a, b, m):
    return b * m + a * (1 - m)


# ---------------------------
# Main raytrace() function
# ---------------------------
def trace(rayorig, raydir, spheres, depth):
    tnear = float("inf")
    sphere_hit = None

    # Find intersection
    for sphere in spheres:
        hit, t0, t1 = sphere.intersect(rayorig, raydir)
        if hit:
            if t0 < 0:
                t0 = t1
            if t0 < tnear:
                tnear = t0
                sphere_hit = sphere

    if sphere_hit is None:
        return Vec3(2, 2, 2)  # background color

    hitPoint = rayorig + raydir * tnear
    nhit = (hitPoint - sphere_hit.center).normalize()
    bias = 1e-4
    inside = False

    if raydir.dot(nhit) > 0:
        nhit = nhit * -1
        inside = True

    # Reflection + Refraction
    if (sphere_hit.transparency > 0 or sphere_hit.reflection > 0) and depth < MAX_RAY_DEPTH:
        facingratio = -raydir.dot(nhit)
        fresneleffect = mix(pow(1 - facingratio, 3), 1, 0.1)

        # reflection
        reflDir = (raydir - nhit * 2 * raydir.dot(nhit)).normalize()
        reflection = trace(hitPoint + nhit * bias, reflDir, spheres, depth + 1)

        refraction = Vec3(0,0,0)

        if sphere_hit.transparency:
            ior = 1.1
            eta = ior if inside else 1/ior
            cosi = -nhit.dot(raydir)
            k = 1 - eta*eta * (1 - cosi*cosi)
            refrDir = (raydir * eta + nhit * (eta * cosi - math.sqrt(k))).normalize()
            refraction = trace(hitPoint - nhit * bias, refrDir, spheres, depth + 1)

        return (reflection * fresneleffect +
                refraction * (1 - fresneleffect) * sphere_hit.transparency) * sphere_hit.surfaceColor

    # Diffuse shading
    surfaceColor = Vec3(0,0,0)
    for sph in spheres:
        if sph.emissionColor.x > 0:
            lightDir = (sph.center - hitPoint).normalize()
            transmission = Vec3(1,1,1)

            # Shadow ray
            for s in spheres:
                if s is not sph:
                    hit, t0, t1 = s.intersect(hitPoint + nhit*bias, lightDir)
                    if hit:
                        transmission = Vec3(0,0,0)
                        break

            surfaceColor += sphere_hit.surfaceColor * transmission * \
                            max(0.0, nhit.dot(lightDir)) * sph.emissionColor

    return surfaceColor + sphere_hit.emissionColor


# ---------------------------
# Rendering
# ---------------------------
def render(spheres):
    width, height = 640, 480
    image = [Vec3(0,0,0)] * (width * height)
    invW = 1 / width
    invH = 1 / height
    fov = 30
    aspect = width / height
    angle = math.tan(math.pi * 0.5 * fov / 180.0)

    idx = 0
    for y in range(height):
        for x in range(width):
            xx = (2*((x + 0.5)*invW) - 1) * angle * aspect
            yy = (1 - 2*((y + 0.5)*invH)) * angle

            raydir = Vec3(xx, yy, -1).normalize()
            image[idx] = trace(Vec3(0,0,0), raydir, spheres, 0)
            idx += 1

    # Save PPM
    with open("untitled.ppm", "wb") as f:
        f.write(bytearray(f"P6\n{width} {height}\n255\n", "ascii"))
        for p in image:
            r = int(min(1.0, p.x) * 255)
            g = int(min(1.0, p.y) * 255)
            b = int(min(1.0, p.z) * 255)
            f.write(struct.pack("BBB", r, g, b))


# ---------------------------
# Main scene
# ---------------------------
if __name__ == "__main__":
    spheres = [
        Sphere(Vec3(0, -10004, -20), 10000, Vec3(.2,.2,.2)),
        Sphere(Vec3(0, 0, -20), 4, Vec3(1, .32, .36), 1, 0.5),
        Sphere(Vec3(5, -1, -15), 2, Vec3(.9, .76, .46), 1, 0.0),
        Sphere(Vec3(5, 0, -25), 3, Vec3(.65, .77, .97), 1, 0.0),
        Sphere(Vec3(-5.5, 0, -15), 3, Vec3(.9, .9, .9), 1, 0.0),
        Sphere(Vec3(0, 20, -30), 3, Vec3(0,0,0), 0, 0, Vec3(3,3,3)),
    ]

    render(spheres)

```

This article was fun and easy to understand, and now I at least understand what I *don’t* know better than before. I’ll continue feeding my curiosity — until then, stay tuned.


