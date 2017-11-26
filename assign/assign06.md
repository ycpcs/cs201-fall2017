---
layout: default
title: "Assignment 6: Mandelbrot Set Color Mapping"
---

**Due**: Tuesday, December, 12th by 11:59 PM


Getting Started
===============

Download [CS201\_Assign06.zip](CS201_Assign06.zip) and import it into your Eclipse workspace (**File&rarr;Import&rarr;General&rarr;Existing projects into workspace&rarr;Archive file**.)

You should see a project called **CS201\_Assign06** in the Package Explorer. Your will be making changes to the **ColorMappingColorChooser** class. You will also be adding test cases to the **ColorMappingColorChooserTest** class.  You should add any additional classes or methods needed to complete the assignment.

Your Task
=========

Your task is to implement a color chooser for the [Mandelbrot Set](http://en.wikipedia.org/wiki/Mandelbrot_set) renderer from [CS201 Assignment 5](assign05.html).

Here a couple of images captured from the Mandelbrot Set rendered using the color map you will be implementing for this assigmment (click to see larger version).

> <a href="img/assign06/MantaRayAndSeahorses.png"><img style="width: 300px; height: 300px;" src="img/assign06/MantaRayAndSeahorses.png" /></a>
> <a href="img/assign06/SpiralSeahorses.png"><img style="width: 300px; height: 300px;" src="img/assign06/SpiralSeahorses.png" /></a>

You will be creating a class that will be called from a supplied rendering program.  You will be able to use the rendering program to test your color mapping implementation, and explore the Mandelbrot Set rendering.  You will also be adding JUnit test cases to thoroughly test the color mapping class.  Expected values for each of the three methods you are to implement are supplied, along with an example JUnit test case for you to use as an example for the additional test cases you need to provide.

The initial rendering should look like this, based on an 800 x 800 image, and maximum iteration count of 2000 (click to see larger version):

> <a href="img/assign06/FullMandelbrotSet.png"><img style="width: 300px; height: 300px;" src="img/assign06/FullMandelbrotSet.png" /></a>

Refer back to [CS201 Assignment 5](assign05.html) for the description and details for the Mandelbrot Set.

Important Hint
--------------

Because the core computation is based on complex numbers, having a class to represent complex numbers will make implementing the computation much easier. The class should look something like this:

{% highlight java %}
public class Complex {
    // ...fields...

    // Constructor
    public Complex(double real, double imag) {
        // ...
    }

    // add given complex number to this one, returning the Complex result
    public Complex add(Complex other) {
        // ...
    }

    // multiply given complex number by this one, returning the Complex result
    public Complex multiply(Complex other) {
        // ...
    }

    // get the magnitude of this complex number
    public double getMagnitude() {
        // ...
    }
}
{% endhighlight %}

You will need to think about what fields to add and how to implement each operation.

Once your **Complex** class is ready, you can iterate the equation as follows:

{% highlight java %}
z = z.multiply(z).add(c);
{% endhighlight %}

This assumes that you have variables **z** and **c** that refer to instances of the **Complex** class.

Creating a task class
---------------------

You should create a task class that computes the iteration counts for a region of the image.

Here is a suggested skeleton:

{% highlight java %}
public class MandelbrotTask implements Runnable {
    private double x1, y1, x2, y2;
    private int startCol, endCol, startRow, endRow;
    private int[][] iterCounts;

    public MandelbrotTask(double x1, double y1, double x2, double y2,
                          int startCol, int endCol, int startRow, int endRow,
                          int[][] iterCounts) {
        this.x1 = x1;
        this.y1 = y1;
        this.x2 = x2;
        this.y2 = y2;
        this.startCol = startCol;
        this.endCol = endCol;
        this.startRow = startRow;
        this.endRow = endRow;
        this.iterCounts = iterCounts;
    }

    public void run() {
        for (int i = startRow; i < endRow; i++) {
            for (int j = startCol; j < endCol; j++) {
                Complex c = getComplex(i, j);
                int iterCount = computeIterCount(c);
                iterCounts[i][j] = iterCount;
            }
        }
    }

    // TODO: implement getComplex and computeIterCount methods
}
{% endhighlight %}

The idea is that a **MandelbrotTask** object will compute iteration counts for the region of the image with rows from **startRow** (inclusive) to **endRow** (exclusive) and columns from **startCol** (inclusive) to **endCol** (exclusive). The region of the complex plane is specified by **x1**, **y1**, **x2**, and **y2**.  The **iterCounts** array is the single global array used to store the iteration counts for the overall image (where the first dimension is rows and the second dimension is columns).

From the **main** method, you could use a **MandelbrotTask** object to compute all of the iteration counts as follows:

{% highlight java %}
int[][] iterCounts = new int[HEIGHT][WIDTH];
MandelbrotTask task = MandelbrotTask(x1, y1, x2, y2, 0, WIDTH, 0, HEIGHT, iterCounts);
task.run();
{% endhighlight %}

Note that this approach is purely sequential, and will not take advantage of multiple CPU cores.

Rendering the Mandelbrot Set
============================

Rendering the Mandelbrot set is done by assigning a color to sampled points in a region of the x/y plane.

Points that are in the Mandelbrot set should be rendered as black.

Points that are outside the Mandelbrot set should be rendered using a color that indicates how many times the equation was iterated before the magnitude of Z reached 2.0. In my implementation, purple is used for points where the magnitude of Z reached 2.0 in 1 iteration. Then, as higher numbers of iterations are needed for the magnitude of Z to reach 2.0, my renderer chooses colors that transition smoothly from purple, to blue, to green, to yellow, to orange, and last to red (for points that are very close to the set, but not within it.)

You may choose any assignment of colors to numbers of iterations, as long as the each color is based on the number of iterations.

So, the 600 by 600 image you render will pick sample points uniformly spaced in a 600 by 600 grid which overlays the region of the x/y plane specified by the user, and set an image color for each corresponding pixel based on whether or not the point is in the set, and if not, how many iterations were required to show that it is not in the set.

Rendering An Image, Saving It
=============================

The Java **BufferedImage** class allows you to render an image:

{% highlight java %}
BufferedImage bufferedImage = new BufferedImage(WIDTH, HEIGHT, BufferedImage.TYPE_INT_ARGB);
Graphics g = bufferedImage.getGraphics();

// ... use g to perform drawing operations ...

g.dispose();
{% endhighlight %}

Once the image has been rendered into the **BufferedImage** object, you can write it to a file as follows:

{% highlight java %}
OutputStream os = new BufferedOutputStream(new FileOutputStream(fileName));
try {
    ImageIO.write(bufferedImage, "PNG", os);
} finally {
    os.close();
}
{% endhighlight %}

Parallelism
===========

The computation performed by the program can take a fair amount of CPU time. However, the computation of the number of iterations for each point is *independent* of the computations for all other points. Therefore, you can speed the program up by using multiple threads to compute the number of iterations in different parts of the overall region.

For example, you might divide the overall region into quadrants, and use a separate thread to compute the points in each quadrant. Since there are four threads, if you run the program on a computer with 4 CPU cores, then you could see up to a 4 times speedup.  (Question: is this the best approach for dividing up the work?)

**Suggestion**: Create multiple **MandelbrotTask** objects, and execute each one in a separate thread.

Grading Criteria
================

Your submission will be graded according to the following criteria:

- Computation: 40%
- Basic rendering of image: 20%
  * Smooth interpolation of colors (extra credit): 5%
- Use of threads for parallelism: 30%
- Design, coding style: 10%

Insane Extra Credit
-------------------

For up to 50 points of extra credit, implement an interactive GUI.  The GUI should start by visualizing the region (-2,-2) to (2,2).  The user should be allowed to zoom in on an arbitrary square region.  (Make sure that it is only possible for the user to select a square region.)

**Important**: The GUI should do the computation in a separate thread, not in the event loop thread.  In other words, do *not* do the computation directly in an event handler.  Instead, when a new rendering needs to be performed, the event handler should start a thread to do the computation.  When the computation is done, it should use **SwingUtilities.invokeLater** to schedule some code to run in the event loop, which then should cause the view to be updated.

An example of a GUI program that uses a thread to schedule a long-running computation and rendering is available here:

> [GUIWithComputationThread.zip](GUIWithComputationThread.zip)

Feel free to use this program as a basis for your extra-credit GUI.  (Note that Google Chrome may spuriously report that this file is malicious; in that case you can use Firefox to download it.)

An example implementation that you can download and try out is here:

> [mandelbrot-obfuscated.jar](mandelbrot-obfuscated.jar)

This example implementation allows zooming in by dragging the mouse, and zooming out by clicking the right mouse button.  Your implementation doesn't need to work exactly the same way, but it should allow similar functionality.

<b>Important</b>: Make sure that you have the sequential and parallel computation working before you try creating a GUI.

Submitting
==========

Save the project (**CS201\_Assign05**) to a zip file by right-clicking it and choosing

> **Export...&rarr;Archive File**

Upload the saved zip file to the Marmoset server as **assign05**. The server URL is

> <https://cs.ycp.edu/marmoset/>
