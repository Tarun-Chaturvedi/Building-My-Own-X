
I’m just setting up the folder and path for an almost non-distracting environment. I’ll give some time each day to build a 3D renderer.

**What do I know about 3D renderers?**
Nothing. From the name, I can guess that they render an illusion of three-dimensional objects on a two-dimensional screen.
The math for this can’t be too complicated, so I can start writing again soon.

As of **4 October 2025**, there are **11 links** provided for understanding how to build a 3D renderer.

Starting with:

1. [**C++: *Introduction to Ray Tracing: A Simple Method for Creating 3D Images***](https://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-ray-tracing/how-does-it-work)

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


