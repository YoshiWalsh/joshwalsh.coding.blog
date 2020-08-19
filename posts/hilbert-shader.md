> :Hero src=/img/hilbert-curve.jpg,
>       leak=96px

> :Title shadow=0 0 8px black, color=white
>
> Making an animated Hilbert Curve using WebGL

> :Author src=github

<br>

This blog post details the process of creating my [animated Hilbert Curve WebGL toy](https://sandbox.ymindustries.com/pixelshader/hilbert.html).

*Note: For some reason the toy doesn't work on some mobile devices. I'm not yet sure why, I still need to look into it.*

_**SYNDICATED CONTENT:** This post originally appeared on blog.joshwalsh.me. You can find the original [here](https://blog.joshwalsh.me/hilbert-shader/). Some formatting and content has been removed in order to fit the `coding.blog` platform._

---

# Inspiration
I’ve been growing increasingly interested in shaders. I think I’ve created some nice looking visual effects using only the CPU, but performance has always been a key limitation. This was probably most evident when creating my [Moonlit Clouds](https://sandbox.ymindustries.com/clouds) toy. My original idea for how to fake volumetric lighting ended up very slow in Firefox Quantum, and completely non-functional in any other browser. Luckily I accidentally discovered a much cheaper method which looks almost as good. I knew I wanted to learn to use WebGL, I just needed an idea that would provide me with sufficient motivation to do so.

On 2018-04-21 I read [a blog post](https://blog.benjojo.co.uk/post/scan-ping-the-internet-hilbert-curve) about using Hilbert Curves to map the internet in a way that humans could visually understand. Within the post there’s an animation of a small section of the internet which was moving through the Hilbert Curve. As soon as I saw this I wondered what it would look like if a much bigger Hilbert Curve was animated. Given that a large number of pixels would have to be updated every single frame, this seemed like the perfect opportunity to try out WebGL.

# Research
I didn’t want to spoil the challenge for myself, so my research into this was pretty much entirely comprised of reading the Wikipedia article about Hilbert Curves. Fortunately the wikipedia article includes some sample code for transforming points in Cartesian Space into Hilbert Space, and vice-versa. Pixel Shaders work by running a piece of code against every single pixel that needs to be rendered, all in parallel. This means it’s important that we can find the distance along the Hilbert Curve that any pixel is, given only its coordinates.

The other piece of research I did was into WebGL itself. I tried a few tutorials, but was frustrated by something they all had in common: you needed to copy/paste large quantities of unexplained boilerplate code before you could get the simplest program working. I eventually cut out the middleman and simply downloaded a [boilerplate](https://github.com/paulirish/webgl-boilerplate), abandoning any tutorials.

# First steps
The boilerplate I used came with a [lovely sample shader](https://sandbox.ymindustries.com/pixelshader/example.html) to show that it was working, but I wanted to make my own simple shader program just to get used to it. I decided that a good Hello World program would be to just display a random colour for every pixel. The challenge I encountered with this is that WebGL doesn’t include any PRNG (Psueo-Random Number Generator) functions.

After a quick Google search I found a one-liner GLSL PRNG which is compatible with WebGL. I got it from [here](https://stackoverflow.com/a/4275343/674675) but it seems that the snippet is so ubiquitous that its exact origins are unknown. It’s not a particularly good PRNG (for reasons mentioned in [this blog post](http://byteblacksmith.com/improvements-to-the-canonical-one-liner-glsl-rand-for-opengl-es-2-0/)) but it’s easy and good enough for my Hello World program.

You can see my noise shader [here](https://sandbox.ymindustries.com/pixelshader/noise.html).

# Porting the Hilbert Curve algorithm
With my first shader done, I got to work on my real project. Before I could do anything else, like experimenting with animations and colours, I had to get the Hilbert Curve algorithm working within a shader. I copy/pasted the algorithm from Wikipedia, then quickly modified it to use valid GLSL syntax.

Most of these changes were simple. For example, WebGL1 (which I was using) doesn’t have a modulo/remainder operator (%), so any uses of that had to be converted to use the mod() function instead. One of the changes was a bit harder though...

# Bitwise functions
The example code from Wikipedia made use of a few bitwise operators. Bitwise operators aren’t supported in WebGL1. Researching this limitation, I found a [gist](https://gist.github.com/EliCDavis/f35a9e4afb8e1c9ae94cce8f3c2c9b9a) with some examples of functions to achieve bitwise behaviour in WebGL1. This worked well for AND, but sadly didn’t include XOR or even NOT, so I would have to build these myself.

To do this, I first looked into how the bitwise imitation functions worked. The gist doesn’t explain it, but fortunately the method is quite simple. There are a few components to how it works:
1. Getting the value of a bit: Without bitwise operators, GLSL doesn’t give us a good way to get the value of a bit. The function instead uses division and modulo in order to achieve this.
2. Comparing isolated bits using boolean operators.
3. Setting the output bits: The function uses multiplication and addition to achieve this.
4. Iterating over all bits in the number and performing these operations.

Once I understood this, it was straightforward to copy one of the functions and adapt it into a NOT function. Once I had my NOT function, I could create an XOR function by combining the other bitwise functions.

After implementing that naive solution, I realised that for unsigned integers, all a bitwise NOT does is subtract the integer from INT\_MAX. INT\_MAX in this case is $2^{32} - 1$, or $4,294,967,295$. Replacing my iterative NOT function with a simple subtraction didn’t noticeably change anything, but doubtless it improved performance by some degree.

But this got me thinking, what were my bitwise functions really doing?

# Replacing bitwise functions
Examining the code from Wikipedia, we can see that the xy2d function uses two bitwise operators. The first is the bitwise AND, which is used here:

```c
rx = (x & s) > 0;
ry = (y & s) > 0;
```

The `x` and `y` variables contain the coordinates of the current pixel. The value `s` is always a power of two. Because it’s always a power of two, this means the bitwise AND will only ever return 1 bit worth of data, the bit that represents a value of `s`. We can achieve the same thing using the modulo function:

```c
rx =x % s*2 >= s; 
ry = y % s*2 >= s;
```

The second bitwise operator is XOR, which is used here:

```c
d += s * s * ((3 * rx) ^ ry);
```

This one appears complicated at first, but it makes more sense when you consider that `rx` and `ry` will only ever have a value of `0` or `1`. Given that there are only four possible combinations of this values, I decided to understand them using a truth table:

| rx | ry | output |
|:-- |:-- |:------ |
| 0  | 0  | 0      |
| 0  | 1  | 1      |
| 1  | 0  | 3      |
| 1  | 1  | 2      |

I ended up just replacing this with if statements:

```glsl
int f;
if(rx) { 
    if(ry) { 
        f = 2; 
    } else { 
        f = 3; 
    } 
} else { 
    if(ry) { 
        f = 1; 
    } else {
        f = 0; 
    } 
} 
d = d + (s * s * f);
```

Not very elegant, but it works. Technically conditionals (and branching) are bad for performance in shaders, but it’s fast enough for my purposes.

And with that, the Hilbert Curve works. From here it was a simple matter of experimenting with colouring schemes and animation speeds until I found something that looked good to me.

---

<br>

> :Author src=github

> :ToCPrevNext
