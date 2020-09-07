---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.5.2
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

# Time integration - Part 1

In this part of the course we discuss how to solve ordinary differential equations (ODE). Although their numerical resolution is not the main subject of this course, their study nevertheless allows to introduce very important concepts that are essential in the numerical resolution of partial differential equations (PDE).

The ODEs we consider can be written in the form:

\begin{equation}
  y^{(n)}=f(t, y, \dots, y^{(n-1)}) \label{eq:ODE},
\end{equation}

where $y=y(t)$ is a function of the variable $t$ and $y^{(n)}$ represents the n-th derivative of $y$ with respect to $t$: 

\begin{equation}
  y^{(n)}=\frac{d^n y}{dt^n}.
\end{equation}

When $f$ does not depend explicitely on time, we qualify the problem as *autonomous*.

Note that we have used $t$ as the variable on which the unkown function $y$ depends and we will usually refer to it as *time*. However, all the methods we describe in this chapter also apply to other problems in which a given function depends on an independant variable and the corresponding ODE have the form \eqref{eq:ODE}.

As an example and toy problem, let us consider radioactive decay. Imagine we have a sample of material containing $N$ unstable nuclei at a given initial time $t_0$. The time evolution of $N$ then follows an exponential decay law:

\begin{equation}
  \frac{dN(t)}{dt}=-\alpha N(t).
  \label{eq:decay}
\end{equation}

where $\alpha>0$ is a constant depending on the type of nuclei present in the material. Of course, we don't really need a computer to solve this equation as its solution is readilly obtained and reads:

\begin{equation}
  N(t)=N(t_0)e^{-\alpha t} \label{eq:expDecay}
\end{equation}

However, our objective here is to obtain the above time evolution using a numerical scheme.

## The forward (explicit) Euler method

The most elementary time integration scheme - we also call these 'time advancement schemes' - is known as the Euler method, which is actualluy a family of the numerical methods for ordinary differential equations. In order to introduce several fundamental concepts that will pop up frequently in the rest of course, we refer to the *forward (or explicit) Euler method*. This scheme is based on computing an approximation of the unknown function at time $t+dt$ from its known value at time $t$ using the Taylor expansion limited to the first two terms. For radioactive decay, we then have:

\begin{align}
   & N(t+dt) \equiv N(t) + N'(t)dt & \textrm{Forward Euler method} \label{eq:ForwardEuler}
\end{align}

From this equation, we note that the forward Euler method is second order for going from $t$ to $t+dt$ (the dropped term in the Taylor expansion is $O(dt^2))$. Once the value of $N$ is known at time $t+dt$, one can re-use \eqref{eq:ForwardEuler} to reach time $t+2dt$ and so on.

Schematically, we therefore start the time marching procedure at the initial time $t_0$ and make a number of steps (called time steps) of size $dt$ until we reach the final desired time $t_f$. In order to do this, we need $n_t = (t_f - t_i)/dt$ steps.

By convention, we denote the different intermediate times as $t^n = t+ndt$ and the corresponding values of $N$ as $N^n = N(t+ndt)$ so that $N^n = N(t^n)$.

The forward Euler scheme is then alternatively written as:

\begin{align}
    & N^{n+1} \equiv N^n + N^{'n} dt & \textrm{Forward Euler method} \label{eq:ForwardEuler2}
\end{align}

Let's write a Python code for that. First, we perform necessary imports.
```python
import numpy as np
```

Now let's set some constant parameters of our problem. In a real-world codes constant parameters are usually separated from the main code. They are either put into separate module, or set in the inputfile. At this stage, let's just isolate them in a separate cell.

```python
alpha = 0.25 # Exponential law coeffecient
ti = 0.0     # Initial time
tf = 5.0     # Final time
dt = 0.5     # Time step
Ni = 100     # Initial condition
```

Now we can write a code for the actual numerical procedure.

```python
# First, we compute number of steps.
# Note that the number of steps must
# be an integer, but the timedata
# we construct it from is of float type.
# It is the reason, why we use int() funtion.
# It attempts conversion of input data to
# integer. If float is provided as an input,
# it disregards the decimals. For example,
# int(5.0/2.0) returns 2.
nt = int((tf-ti)/dt)

# Create an empty numpy array to contain
# the intermediate values of N, including
# those at ti and tf.
#
# You may wonder, how piece of numerical
# data can be empty or not empty? But here
# it is rather the conventional term used
# to indicate that the values have not been
# initialized - set to 0. It means that they
# can have any value in a range allowed by
# numerical precision. Why to do so? Well,
# initializing takes time. So, unless, you'll
# need array of zeros, np.empty is preferable
# over np.zeros.
N = np.empty(nt+1)

# We pass initial condition to the N array,
N[0] = Ni

# And perform the time stepping.
for i in range(nt):
    N[i+1] = N[i] - alpha*N[i]*dt
```

Done! The last entry in the array N now contains an estimate for $N(t_f)$.


### Numerical accuracy of the forward Euler method

Let us compare graphically our numerical values with the exact solution \eqref{eq:expDecay}. For that purpose we again use the matplotlib package:

```python
import matplotlib.pyplot as plt

%matplotlib inline

plt.style.use('../styles/mainstyle.use')
```

```python
# Cell, where we perform computation to
# build exact solution of differential
# equation.
#
# numpy.arange intends to build a nume-
# rical sequence. It is similar to the
# Python's standard range function, BUT,
# unlike it, it can operate not only the
# integers, but also floats, and its return
# type is numpy native array.
#
# For more info:
# https://numpy.org/doc/stable/reference/generated/numpy.arange.html
t = np.arange(nt+1) * dt

# We are all set to build exact solution array.
Nexact = Ni * np.exp(-alpha*t)
```

When you're *debugging* - developing, testing and optimizing your code, it is always a good idea to have your imports and setup of *constant parameters* separated from the code you're working on. The same stands for the actual computations and visualization. Imaging, you build your arrays of data in a same cell, as you plot it. You see a plot and you don't like the font, you rerun the cell, and then you think that it might be a good idea to cut the x-axis, you rerun the cell again. In such a way, each time you update your plot, you will recompute absolutely the same numpy array, which is *not catastrophic, but considered to be a poor organization of a code*.

```python
# Create a figure with a single subplot
# in it.
fig, ax = plt.subplots()

# Plot the exact solution. Set the linestyle.
# Matplotlib supports multiple line- and
# marker styles. For the linestyles see
# https://matplotlib.org/3.1.0/gallery/lines_bars_and_markers/linestyles.html
# For the markers see
# https://matplotlib.org/3.3.1/api/markers_api.html
#
# Note, though, that here we specify linestyle
# as '-' (equivalent to 'solid') for the infor-
# mative purposes. Solid linestyle is a default one,
# so, if you remove the linestyle specification here,
# the plot won't change.
#
# We also assign a label to the curve. Label is
# a string, which we want to be displayed in a
# legend.
ax.plot(t, Nexact, linestyle='-', label='Exact solution')

# Plot the approximate solution. You can see that
# here we specify the appearance of the markers
# without using the keyword 'marker'. We let
# Python figure out which argument we are aiming
# for by its position. There are POSITIONAL and
# KEYWORD arguments in Python functions. Posi-
# tional arguments must obey certain order, so
# that it is clear, which of the parameters
# stands for the certain argument. Keyword argu-
# ments are passed with the keywords. Like here,
# for example, we specify, that we want color='green',
# where color is a keyword argument. Sometimes
# keyword argumets can be passed as positional ones
# if you follow the order, provided in the implemen-
# tation of a funtion E X A C T L Y. 
# For more info
# https://problemsolvingwithpython.com/07-Functions-and-Modules/07.07-Positional-and-Keyword-Arguments/
ax.plot(t, N, '^', color='green', label='Forward Euler method')

# We set lables for the axes and title of a subplot.
ax.set_xlabel('$t$')
ax.set_ylabel('$N$')
ax.set_title('Radioactive decay')

# Make the legend visible.
ax.legend()

# And we save the whole figure to the specified
# location in png format. Png format is a default
# one, though. If you don't put .png suffix, it'll
# still be saved as a png image. Keyword argument
# dpi defines resolution of your image. It is lite-
# rally 'dots per image' - 300 is good enough even for
# the scientific paper, noo need to go to the extremes.
#
# Btw, as you've saved your figure once, it is a good
# idea to comment the line, so that, you don't save
# the same image over and over. Unless you modify the
# plot.
fig.savefig('../figures/radioactiveDecay.png', dpi=300)
```

The agreement between exact solution and an approximate one is good, but not precise. Moreover, it degrades with time. Why so? The answer, of course, comes from the error introduced by cutting the Taylor series in the definition of the forward Euler scheme, and we know things should improve, if we reduce $dt$ but this will come at the expense of increasing the total number of time steps and the computational cost. To get an idea about this, run the previous code with a smaller and smaller time step and see what happens to the curves.

To analyze this from the quantitative point of view, let us redo the computation using several values of $dt$ and compare the error made in estimating $N(t_f)$. In the following piece of code, we only store the value of $N$ at $t_f$.

```python
# Create a list containing the set of the
# timesteps, so that each timestep is a half
# of a previous one.
dt_list = np.asarray([0.5/2**k for k in range(5)])

# Create an array to store the values of
# N(tf) for the different time steps.
# numpy.empty_like returns the array with
# values non-initialized, just like numpy.empty,
# but, unlike numpy.empty, it takes as a para-
# meter not an integer, but either a sequence,
# or the numpy array - array-like in the termi-
# nology of numpy. The output array will have
# the same shape and type as an input data.
values = np.empty_like(dt_list)

# Now we want to loop over all values in
# dt_list. We also want to count our itera-
# tions, as if we were extracting indices of
# the elements of dt_list. We could create some
# variable i=0 (indexing in Python and most of
# the others programming languages starts with
# 0), and then increase it by 1 at each iteration, 
# like this:
#
# i = 0
# for dt in dt_list:
#    (do something...)
#    i =+ 1
#
# But the moral here is WHY BOTHER. In this case,
# and in many others, Python developers already
# have implemented what we need - enumerate fun-
# ction. It counts elements in an iterable object
# and returns the counter at each iteration.

for i, dt in enumerate(dt_list):
    # Copy initial condition into the variable,
    # which we are going to advance for each size
    # of dt.
    N = Ni
    
    nt = int((tf-ti)/dt)

    # Be careful not to shadow the variables of
    # the exteriour loop - i and dt - with the
    # variables of the interiour loop. We set it
    # to be j.
    for j in range(nt):
        N = N - alpha*N*dt

    # Save N final subsequently to the values array
    # each time it gets computed.
    values[i] = N
```

Let's now compute and plot the difference between the computed $N(t_f)$ and the exact solution.

```python
# We construct the array containg the difference
# between approximated and exact final solutions
# for each size of timestep considere in dt_list.
error = np.abs(values-Ni*np.exp(-alpha*tf))
```

```python
fig, ax = plt.subplots()

# Plot the error in logarifmic scale and see
# that it grows as timestep increases.
ax.loglog(dt_list, error, '*', label='Error')

# Fit a slope to the previous curve and see
# that as they are parallel, the error after
# nt timesteps is propotional to dt (not dt**2).
ax.loglog(dt_list, dt_list, color='green', label='$dt$')

ax.set_xlabel('$dt$')
ax.set_ylabel('Error')
ax.set_title('Accuracy')

ax.legend()

# fig.savefig('../figures/eulerSlope.png', dpi=300)
```

Do you notice something 'surprising' in this plot? Earlier we mentioned an accuracy of second order for the forward Euler method but here we observe an accuracy of first order. In fact, there is a straightforward explanation for this. We said "...that the forward Euler method is second order for going from $t$ to $t+dt$". Here we are comparing values after $N$ timesteps with $\displaystyle N=\frac{t_f-t_i}{dt}$. The total error is proportional to the product of the error made at each time step multiplied by the number of steps. As the latter scales as $dt^{-1}$, the total error scales like $dt^2 / dt = dt$. One says that **the error made during one time step accumulates during the computation**.


### Numerical stability of the forward Euler method

For the radioactive decay equation, the forward Euler method does a decent job: when reducing the time step, the solution converges to the exact solution, albeit only with first order accuracy. Let us now focus on another crucial property of numerical schemes called *numerical stability*. 

For the problem of radioactive decay, we first observe that, according to equation \eqref{eq:ForwardEuler2}, it is fair that:

\begin{equation}
    N^{n} = (1-\alpha dt)N^{n-1}  = (1-\alpha dt)^2 N^{n-2}= \dots = (1-\alpha dt)^{n}N_{0}
    \label{eq:demo_stability}
\end{equation}

\ref{eq:demo_stability} implies that $N^n\to \infty$ if $\vert 1-\alpha dt \vert \to \infty$. In such a case numerical scheme is called *unstable* - when solution grows unbounded (blows up in the jargon). On the other hand, in the case when $\vert 1-\alpha dt \vert \le 1$, Euler scheme is considered to be stable. This requirement limits a timestep allowed, when performing the numerical integration.

In many problems, the coefficients of the equations considered are complex (e.g. Schrödinger equation). If we generalise our radioactive decay problem to allow for complex valued coefficients $\alpha=\alpha_r + i\alpha_i$, the criteria for stability of the forward Euler scheme becomes,

\begin{equation}
  \vert 1-\alpha dt \vert \le 1 \Leftrightarrow (1-\alpha_rdt)^2+(\alpha_idt)^2 \le 1.
  \label{eq:complex_stability}
\end{equation}

Given this, one can then draw a stability diagram indicating the region of the complex plane $(\alpha_rdt , \alpha_idt)$, where the forward Euler scheme is stable. As it is obvious from \ref{eq:complex_stability}, the bounded region of stability *is a circle*.

```python
# Let's configure the size of the figure
# (in inches) to make it a square and bigger.
fig, ax = plt.subplots(figsize=(6, 6))

# We draw circle and customize it a bit. You
# can see that because of some reason the fun-
# ction name here starts with a capital letter -
# Circle. That's because matplotlib.pyplot.Circle
# is not really a function. It is an object called
# c l a s s in Python. We won't dig into classes 
# at this stage, but what it means for us here?
# Well, when we call the class by its name, we
# are actually calling its c o n s t r u c t o r -
# in-class function (method), which returns the
# i n s t a n c e of a class.
# In this way, we have the instance of a class -
# circle, which we have to add to the subplot
# somehow. Otherwise, the figure and circle are
# fully separated of each other, circle s t a n d s
# a l o n e from the whole drawing.
#
# For more info on the Circle class
# https://matplotlib.org/3.3.1/api/_as_gen/matplotlib.patches.Circle.html
circle = plt.Circle((-1, 0), 1, ec='k', fc='green', alpha=0.5, hatch='/')

# There is a method of Axes object, which is res-
# ponsible for exactly what we need - adding
# Artist object to the plots. Yes, Circle ori-
# ginate from the other generic object - Artist.
# Generraly speaking most of drawable object in
# Matplotlib originate from Artist. So, if we
# wanted to go exotic, we could have even added
# our lines through add_artist, instead of doing
# plot.
#
# Consider scheme on
# https://matplotlib.org/3.1.1/api/artist_api.html#matplotlib.artist.Artist
ax.add_artist(circle)

# We make sure, that scaling for the x-axis
# is the same, as for the y_axis.
# For more info
# https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.axes.Axes.set_aspect.html
ax.set_aspect(1)

# We move the axes drawn on the left and
# on the bottom of the subplot to the center,
ax.spines['left'].set_position('center')
ax.spines['bottom'].set_position('center')
# And hide the pieces of the subplot's frame
# which are on the right and on top.
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)

# See, that Python's syntax allows creation
# of few variables at a time.
xmin, xmax = -2.3, 2.3
ymin, ymax = -2., 2.

ax.set_xlim(xmin ,xmax)
ax.set_ylim(ymin, ymax)

# Let's complement our plot with arrows. We won't
# dig into details of how arrows are configured, as
# you already have enough knowledge to figure it out
# yourself. 
#
# For more info
# https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.pyplot.arrow.html
ax.arrow(xmin, 0., xmax-xmin, 0., fc='k', ec='k', lw=0.5,
         head_width=1./20.*(ymax-ymin), head_length=1./20.*(xmax-xmin),
         overhang = 0.3, length_includes_head= True, clip_on = False)

ax.arrow(0., ymin, 0., ymax-ymin, fc='k', ec='k', lw=0.5,
         head_width=1./20.*(xmax-xmin), head_length=1./20.*(ymax-ymin),
         overhang = 0.3, length_includes_head= True, clip_on = False)

# Let's set location for the axes labels, and
# change rotation rate for the label of the y-
# axis - by default it is 90.
ax.set_xlabel(r'$\alpha_r dt$')
ax.set_ylabel(r'$\alpha_i dt$', rotation=0)

ax.yaxis.set_label_coords(0.6, 0.95)
ax.xaxis.set_label_coords(1.05, 0.475)

ax.set_title('Stability of forward Euler scheme', y=1.01)

# Let's configure the ticks of the axes. The
# most straightforward way of doing it is to
# pass locations of the ticks explicitely.
#
# You also can pass the ticklabels you want
# using set_ticklabels method.
ax.set_xticks((-2, 2))
ax.set_yticks((-2, -1, 1, 2))

# Consider these 2 commented lines of code. 
# When cutomizing axes' ticks, Matplotlib pro-
# vides Locator and Formatter objects - you
# can customize the ticks in basically one
# simple call without accessing the ticklabels
# data.
# For more info
# https://jakevdp.github.io/PythonDataScienceHandbook/04.10-customizing-ticks.html
#
# ax.xaxis.set_major_locator(plt.MaxNLocator(2))
# ax.yaxis.set_major_locator(plt.MaxNLocator(4))

# Though, sometimes, Locator will not be as
# flexible as you need it to be. For example,
# imagine that here, on top of hiding some ticks,
# we also want to hide some specific tick - with
# a zero label. Consider the following way of
# doing it - by accessing indices of ticks in
# a tuple* returned by get_ticklabels function,
# and checkinf if it satisfies certain condition.
# Such method is certainly not as graceful as
# the one which goes with Locators, but it is
# quite u n i v e r s a l.
#
# *tuple in Python is a standard type of a sequence,
# which, unlike a sequence is unchangable.
# for i, label in enumerate(ax.yaxis.get_ticklabels()):
#     if i % 2 != 0 or i == 4:
#         label.set_visible(False)

# As the width of the axes became twice wider,
# since we've drawn arrows, let's adjust the width
# of the ticks.
ax.tick_params(width=2, pad=10)

# fig.savefig('../figures/eulerStabilityMap.png', dpi=300)
```

 If $dt$ is chosen sufficiently small, so that both $\alpha_rdt$ and $\alpha_i dt$ are inside a circle, then the forward Euler scheme will be stable. We see in particular that the forward Euler scheme cannot be made stable if $\alpha$ is purely imaginary, however small we choose the time step (we will consider a consequence of this below).


### Multi-dimensional example

So far we have only considered a simple one-dimensional example. In pratice, many problems are modelled with a series of coupled variables, which makes the corresponding equation multi-dimensional. Multi-dimensional equations also arise when the starting equations contain higher-order derivatives. They are converted to a system of first-order differential equations. Consider scalar third-order differential equation for $y=y(x)$:

\begin{equation}
    \frac{d^3 y(x)}{dx} = f(y, x).
    \label{eq:high_order_eq}
\end{equation}

Let us introduce new variables:

\begin{align}
    & y_0 = y(x), \\
    & y_1 = \frac{d y(x)}{dx}, \\
    & y_2 = \frac{d^2 y(x)}{dx}, \\
    & y_3 = \frac{d^3 y(x)}{dx}.
\end{align}

\ref{eq:high_order_eq} then transforms into the system of 3 first-order differential equations:

\begin{cases}
& \displaystyle\frac{d y_0}{dx} = y_1, \\
& \displaystyle\frac{d y_1}{dx} = y_2, \\
& \displaystyle\frac{d y_2}{dx} = f(y, x).
\end{cases}

This has been generic. Consider down-to-earth trivial example of equation of motion for a body in free fall:

\begin{equation}
    \frac{d^2 h}{d t^2}=-g,
\end{equation}

where $g$ is accelaretion due to gravity, $h$ is the height of the object with respect to the ground.

We introduce new variable $\displaystyle v = \frac{dh}{dt}$, which has physical meaning of velocity, and obtain the system of 2 first-order differential equations:

\begin{cases}
    & \displaystyle \frac{dh}{dt}=v,\\
    & \displaystyle \frac{dv}{dt}=-g. 
\end{cases}

If we apply the forward Euler scheme to this system, we get:

\begin{align}
    & h^{n+1}=h^n + v^n dt,\label{eq:grav_first}\\
    & v^{n+1}=v^n  - g dt.\label{eq:grav_second}
\end{align}

We can also write the system of equation in matrix form:

\begin{align}
\begin{pmatrix}
    h^{n+1} \\
    v^{n+1}
\end{pmatrix}
=
\begin{pmatrix}
    h^{n} \\
    v^{n}
\end{pmatrix}
+&
\begin{pmatrix}
    0 & 1 \\
    0 & 0
\end{pmatrix}
\begin{pmatrix}
    h^{n} \\
    v^{n}
\end{pmatrix}
dt
+
\begin{pmatrix}
    0 \\
    -g
\end{pmatrix}
dt \\
&\Leftrightarrow \nonumber \\
y^{n+1} &= y^n + Ly^n dt + bdt
\end{align}

with $y=(h\;\; v)$, $b=(0\;\; -g)$ and

\begin{align}
L=
\begin{pmatrix}
    0 & 1 \\
    0 & 0
\end{pmatrix}
\end{align}

Let's solve this system numerically and use numpy array functionalities to write our solution in a more compact way. As initial condition, we choose $h_0=100\,\textrm{m}$ and $v_0=0\,\textrm{m/s}$.

```python
g = 9.81  # ms^-2, gravitational constant
h0 = 100. # initial height
v0 = 0.   # initial velocity

ti = 0.   # initial time
tf = 4.0  # final moment of time to seek solution at
dt = 0.1  # timestep
```

```python
nt = int((tf-ti)/dt)

# We create a numpy array to contain the
# intermediate values of y, including
# those at ti and tf.
# You can see that, instead of passing
# single integer number to numpy.empty,
# we have passed a tuple of two integers.
# This is a way to create 2D numpy array.
#
# First integer defines number of rows in
# array, while the second integer defines
# number of columns (obvious analogy with
# matrices, BUT better not to call numpy
# array matrices, as there are also
# n u m e r i c a l objects in numpy called
# matrices. They differ. And to be honest,
# used quite poorly - numpy developers in-
# tend to deprecate them.).
y = np.empty((nt+1, 2))

# Store initial condition h0, v0 in the first
# row of the array. 
#
# Here some words about array indexinf must be
# said. The right way to index 1D array is, ob-
# viously, to pass single integer to it. It is
# a bit more complicated with 2D arrays. Gene-
# ric way to go, which always works, is to pass
# 2 integer numbers, first of which denote num-
# ber of a row, and the second - number of a co-
# lumn. But numpy developers implemented ways to
# easen the life of programmers. Below you see
# one of the examples. When you pass s i n g l e
# index to the numpy array, it is being interpreted
# as a row index. In this way you access a l l
# columns in first row, which spares you the nece-
# ssity to loop over all of them.
y[0] = h0, v0

# Create vector b.
b = np.asarray([0., -g])

# Create matrix L. Note that default type of
# values in numpy array is float of double
# precision. So, it does not make principal
# difference, whether you pass elements as
# floats (by putting floating point), or as
# integers. We prefer it like that to be 100%
# explicit. But after all, it is rather a per-
# sonal choice.
L = np.asarray([[0., 1.], [0., 0.]])

# Perform the time stepping. numpy.dot is a
# very useful function providing various func-
# tionality. It can do vector product, matrix
# multiplication, sum product over axes. You
# will always have to take care of compatibi-
# lity of the shapes of input data.
# For more info
# https://numpy.org/doc/stable/reference/generated/numpy.dot.html
for i in range(nt):
    y[i+1] = y[i] + np.dot(L, y[i])*dt + b*dt
```

Let's now display obtained results graphically. 

We shall also demonstrate an interesting feature of `matplotlib`. We will create multiple subplots and store them all in *one variable*. One could expect that this variable would become then some standard Python sequence (like tuple or list). But in the reality it will have a type of `numpy.ndarray`. Why is this so curious? Because it will be so, **even if `numpy` is not imported**. As `matplotlib` [developers claim][1]:

> If matplotlib were limited to working with lists, it would be fairly useless for numeric processing. Generally, you will use numpy arrays. In fact, all sequences are converted to numpy arrays internally. The example below illustrates plotting several lines with different format styles in one function call using arrays.

Indeed, we already spoke of the fact, that *numpy arrays are faster than lists*. But let's get back to our plot.

[1]: <https://matplotlib.org/tutorials/introductory/pyplot.html> "Internal conversion"

```python
# Let's create some sample array which
# will store discrete time data for nt
# timesteps.
t = np.arange(nt+1) * dt
```

```python
fig, ax = plt.subplots(1, 2, figsize=(9, 4))

# We, of course, now acces different sub-
# plots as the elements of numpy array.
ax[0].plot(t, y[:, 1], '-k', lw=0.8)

# Here we limit the x-axis strictly to
# the domain in which t is defined, and
# demonstrate VERY IMPORTANT FEATURE OF
# SEQUENCES IN PYTHON. It is the possi-
# bility of negative indexing, which is 
# absent in many other programming langu-
# ages. When the negative indexes are
# provided to the sequence in Python, then
# the elemnts are counted from the end 
# of the array. t[-1] refers to the last
# element of t. 
# This is a very useful feature, which
# spares you the necessity to even care
# about how many elements your sequence
# contains.
#
# For more info
# http://wordaligned.org/articles/negative-sequence-indices-in-python
ax[0].set_xlim(t[0], t[-1])

ax[0].set_xlabel('$t$')
ax[0].set_ylabel('$v$')
ax[0].set_title('Speed vs time (m/s)')

ax[1].plot(t, y[:, 0], '-k', lw=0.8)

ax[1].set_xlim(t[0], t[-1])

ax[1].set_xlabel('$t$')
ax[1].set_ylabel('$h$')
ax[1].set_title('Height vs time (m)')
```

### Numerical stability of the forward Euler method revisited

Let's consider another two dimensional example and analyze the motion of an object attached to a spring. The equation of motion reads:

\begin{equation}
    m\frac{d^2 x}{d t^2}=-kx,
    \label{eq:spring}
\end{equation}

where $x$ is the position of the object with respect to its equilibrium position and $k>0$ is the spring constant. Introducing the velocity $v=dx/dt$, this equation is equivalent to the following system:

\begin{align}
    & \frac{dx}{dt}=v,\\
    & \frac{dv}{dt}=-\gamma^2 x,
\end{align}
with $\gamma =\sqrt{k/m}$.

For the forward Euler scheme we have:

\begin{align}
\begin{pmatrix}
    x^{n+1} \\
    v^{n+1}
\end{pmatrix}
=
\begin{pmatrix}
    x^{n} \\
    v^{n}
\end{pmatrix}
+&
\begin{pmatrix}
    0 & 1 \\
    -\gamma^2 & 0
\end{pmatrix}
\begin{pmatrix}
    x^{n} \\
    v^{n}
\end{pmatrix}
dt
\end{align}

It does not seem very different from the previous problem so let's implement this. 

```python
k = 2.    # spring constant
m = 1.    # object's mass
x0 = 0.75 # initial position
v0 = 0.   # initial velocity
ti = 0.   # initial time
tf = 40.0 # final moment of time at which search for solution
dt = 0.15  # timestep
```

```python
# Let's first compute gamma and number
# of timesteps.
gamma = np.sqrt(k/m)
nt = int((tf-ti)/dt)

# Create a numpy array to contain the
# intermediate values of y, including
# those at ti and tf.
y = np.empty((nt+1, 2))

# Store initial condition in a first row
# of y.
y[0] = x0, v0

# Create matrix L.
L = np.asarray([[0., 1.], [-gamma**2, 0.]])

# Perform the time stepping.
for i in range(nt):
    y[i+1] = y[i] + np.dot(L, y[i])*dt
```

```python
# Store nt timesteps.
t = np.arange(nt+1) * dt
```

```python
fig, ax = plt.subplots(1, 2, figsize=(9, 4))

ax[0].plot(t, y[:, 1], '-k', lw=0.8)

ax[0].set_xlabel('$t$')
ax[0].set_ylabel('$v$')
ax[0].set_title('Speed vs time (m/s)')

ax[1].plot(t, y[:, 0], '-k', lw=0.8)

ax[1].set_xlabel('$t$')
ax[1].set_ylabel('$x$')
ax[1].set_title('Position vs time (m)')

# Here we take advantage of that we
# store both axes objects in one vari-
# ables - we don't have to restrict the
# limits for each of them separately, 
# as we can iterate over the members
# of the sequence - makes the code shorter.
for axis in ax:
    axis.set_xlim(0, 40.)
```

Don't you see something strange? We know, a fritionless harmonic oscillator, like the one we are considering, must oscillate back and forth with a constant amplitude. So, what exactly went wrong?

Let's inspect forward Euler scheme for stability for the system we are solving. In order to decouple equations for $x^{n+1}$ and $v^{n+1}$, we compute the eigenvalues $\lambda_i$ and eigenvectors $v_i$ of the matrix

\begin{align}
L=
\begin{pmatrix}
    0 & 1 \\
    \gamma^2 & 0
\end{pmatrix},
\end{align}

and get:

\begin{align}
\lambda_1 = i\gamma,\;\; \lambda_2=-i\gamma,\;\;
v_1 =
\begin{pmatrix}
    1 \\
    i\gamma
\end{pmatrix},
\;\;
v_2 =
\begin{pmatrix}
    1 \\
    -i\gamma
\end{pmatrix}.
\end{align}

The matrix $L$ can then be decomposed as,

\begin{align}
L=Q\Lambda Q^{-1}\; \hbox{with,} \;\;
\Lambda = 
\begin{pmatrix}
    i\gamma & 0 \\
    0 & -i\gamma
\end{pmatrix} \;\; \hbox{and} \;\;
Q=
\begin{pmatrix}
   1 & 1 \\
    i\gamma & -i\gamma
\end{pmatrix}.
\end{align}

Using the vector notation $y=(x\;\; v)$, we can then reformulate our time advancement scheme as,

\begin{align}
y^{n+1} = y^{n}+ Ly^{n}dt\;\; & \Leftrightarrow \;\; Q^{-1}y^{n+1} = Q^{-1}y^{n+1} + Q^{-1}Ly^{n}dt \\
& \Leftrightarrow \;\; z^{n+1} = z^{n+1} + \Lambda z^{n}dt. \label{eq:eigenCoor}
\end{align}

In $\eqref{eq:eigenCoor}$, $z=(z_1\;\; z_2)$ are the coordinates in the eigenvector basis $y=z_1(t) x + z_2(t) v$. In this basis, the system of equation is decoupled and reads:

\begin{align}
    & z_1^{n+1} = z_1^{n} + i\gamma z_1^{n} dt\\
    & z_2^{n+1} = z_2^{n} - i\gamma z_2^{n} dt
\end{align}

It is now clear why the forward Euler scheme displays the diverging behaviour observed in the plots. The coefficients present in the advancement scheme are both purely imaginery and we have seen above that their product with $dt$ necessarily lie outside of the domain of stability of the scheme. Therefore, we cannot avoid the divergence of our solution by taking even a very small time step. The forward Euler scheme is, therefore, not adapted to the simulation of a simple harmonic oscillator.


## The backwards (implicit) Euler method


We inspected the usefullness of the explicit Euler method for solving different differential equations, and discovered that while it gives a decent approximation in the certain cases (\ref{eq:decay}), it as absolutely unapplicable in the other cases, since it simply blows up for *any* timestep (\ref{eq:spring}). It urges us to search for different ways to approximate solution. Such as *implicit* Euler method, for example. 

As well as explicit Euler can also be referred as the forward Euler, implicit Euler is being sometimes called the backwards Euler. Generally speaking, the difference between explicit and implicit numerical schemes is that in the first case the solution at latter point of the dependant variable (at the latter moment of time, for example) is built from the solution found at the previous points; in the second case solution is seeked by solving the equation, which involves *both* the solutions at the latter and previous points. Consider schmetic notations:

\begin{align}
& y^{n+1}(t) = f\Big(y^n(t\Big),\;\; & \hbox{Explicit scheme}, \\
& g\Big(y^{n+1}(t), y^n(t) \Big) = 0,\;\; & \hbox{Implicit scheme}.
\end{align}

Time advancement with implicit Euler scheme is as follows:
\begin{equation}
y^{n+1} = y^n + dt f(y^{n+1}, t^{n+1}).
\end{equation}

To investigate the accuracy of implicit Euler against those of explicit Euler, let's go back to \ref{eq:decay}. In this simple case the one-step adavncement is quite trivial to derive, and is given as:
\begin{equation}
N^{n+1} = (1+\alpha dt)^{-1}N^n.
\end{equation}

Let us now estimate the acuracy of explicit scheme against the accuracy of implicit scheme for this model problem.

```python
# We redefine all the constants. Eventhough,
# they were defined in the above cell, some
# got overwritten when solving other problems.
#
# We increase the timestep to get noticable
# decrease in accuracy of the explicit scheme
# and see, how implicit scheme compares with
# it. We also increase the time domain to see
# how solution converges far away from the
# initial moment of time.
alpha = 0.25 # Exponential law coeffecient
ti = 0.0     # Initial time
tf = 15.0    # Final time
dt = 1.2     # Time step
Ni = 100     # Initial condition
```

```python
nt = int((tf-ti)/dt)

# Let us also redefine N array, so that we
# store both the solution predicted by the
# implicit and explicit schemes in one array
# - in different columns.
N = np.empty((nt+1, 2))

# We copy initial condition into both columns.
N[0] = Ni, Ni

# Define one-timestep advancement coefficient
# assumed by the implicit Euler outside of the
# loop, as it is independant of t.
# If some computation you ought to perform is
# independant of the iteration index, try to
# ALWAYS take it out of the loop. Otherwise, 
# you are performing useless repetitive steps,
# and simply waste your time.
coef_imp = (1.+alpha*dt)**(-1)

# Advance solution both with the implicit and
# explicit schemes.
for i in range(nt):
    N[i+1, 0] = N[i, 0] - alpha*N[i, 0]*dt
    
    N[i+1, 1] = coef_imp*N[i, 0]
```

```python
# t and Nexact have to be recomputed, as we
# have newly defined tf.
t = np.arange(nt+1) * dt

Nexact = Ni * np.exp(-alpha*t)
```

```python
fig, ax = plt.subplots(figsize=(8, 6))

ax.plot(t, Nexact, linestyle='-', color='k', lw=0.8, label='Exact solution')

ax.plot(t, N[:, 0], '^', color='green', label='Forward Euler method')
ax.plot(t, N[:, 1], '^', color='blue', label='Backwards Euler method')

ax.set_xlim(-0.1, 14.5)

# We set lables for the axes and title of a subplot.
ax.set_xlabel('$t$')
ax.set_ylabel('$N$')
ax.set_title('Radioactive decay')

# Make the legend visible.
ax.legend()
```

We are able to observe from the figure that implicit Euler is more accurate with the same timestep. This is no surprise. Explicit schemes are always easier to implement, but you will surely encounter situations when the timestep, required to achieve high accuracy with explicit scheme, has to be really small. In this way you loose a lot of computational time. Sometimes the small timestep is not even the matter of accuracy, like in the case above, but the matter of *stability*. It takes much less computational time to achieve desired accuracy with the implicit schemes.

But we still advice you against sticking *only* to implicit methods. There are also a lot of real-world problems, for which you are totally good to go with explicit schemes - for example, boundary-layer equation solved for the problem of hydrodynamic stability. So, you have always to be sure, that your reasons to use implicit scheme are solid, and you don't complicate your life for nothing


Let's go back now to the equations \ref{eq:spring}. As we have proved, that forward Euler is unstable for this case, it is a natural move to test, what is the outcome of the backwards Euler.

If applied to \ref{eq:spring},
\begin{align}
\begin{pmatrix}
    x^{n+1} \\
    v^{n+1}
\end{pmatrix}
=
\begin{pmatrix}
    x^{n} \\
    v^{n}
\end{pmatrix}
+&
\begin{pmatrix}
    0 & 1 \\
    \gamma^2 & 0
\end{pmatrix}
\begin{pmatrix}
    x^{n+1} \\
    v^{n+1}
\end{pmatrix}
dt.
\end{align}

After a little rearrangement:
\begin{align}
\begin{pmatrix}
1 & -dt\\
-\gamma^2 dt & 1
\end{pmatrix}
\begin{pmatrix}
    x^{n+1} \\
    v^{n+1}
\end{pmatrix}
= \begin{pmatrix}
    x^{n} \\
    v^{n}
\end{pmatrix},
\end{align}

and finally:
\begin{align}
\begin{pmatrix}
    x^{n+1} \\
    v^{n+1}
\end{pmatrix}
= \begin{pmatrix}
1 & -dt\\
-\gamma^2 dt & 1
\end{pmatrix}^{-1}
\begin{pmatrix}
    x^{n} \\
    v^{n}
\end{pmatrix}.
\end{align}

You see that things got a little more complicated - we had to make a little effort to express $y^{n+1}=(x^{n+1}\;\; v^{n+1})$ in terms of $y^{n}=(x^{n}\;\; v^{n})$, and we will, obviously, have to implement iverse of a matrix, but at least we are lucky that the right-hand side of \ref{eq:spring} does not contain explicit dependance on $t$.

So, let's implement implicit Euler in a code:

```python
# We have to redefine time parameters
# again, as they got overwritten.
ti = 0.    # initial time
tf = 40.0  # final moment of time at which search for solution
dt = 0.15  # timestep
```

```python
# Recompute number of timesteps and 
# the time array.
nt = int((tf-ti)/dt)

t = np.arange(nt+1) * dt
```

```python
y_imp = np.empty((nt+1, 2))

# Store initial condition in a first row
# of y.
y_imp[0] = x0, v0

# As matrix which advances the solution
# does not depend on t, we compute it
# right away, and find an inverse.
L_imp = np.linalg.inv(np.asarray([[1., -dt], [gamma**2*dt, 1.]]))

# Perform the time stepping. dt is hidden
# in L_imp, so, no need to multiply by it.
for i in range(nt):
    y_imp[i+1] = np.dot(L_imp, y_imp[i])
```

```python
fig, ax = plt.subplots(1, 2, figsize=(12, 6))

ax[0].plot(t, y_imp[:, 1], '-k', lw=0.8, label='Backwards Euler')
ax[0].plot(t, y[:, 1], '--k', lw=0.8, label='Forward Euler')

ax[0].set_xlabel('$t$')
ax[0].set_ylabel('$v$')
ax[0].set_title('Speed vs time (m/s)')

ax[1].plot(t, y_imp[:, 0], '-k', lw=0.8, label='Backwards Euler')
ax[1].plot(t, y[:, 0], '--k', lw=0.8, label='Forward Euler')

ax[1].set_xlabel('$t$')
ax[1].set_ylabel('$x$')
ax[1].set_title('Position vs time (m)')

for axis in ax:
    axis.set_xlim(0, 15)
    axis.set_ylim(-10, 10)
    axis.legend(loc='upper center')
```

As we see, implicit Euler, obviously, does not blow up, but the solution damps quickly, which is the consequence of the total error being accumulated after $N$ timesteps.


## Summary

In this notebook we have described the forward and the backwards Euler schemes, and how we can discretize an ordinary differential equation (or a system of ODE) to compute the time evolution of the physical quantities under consideration.

We gave the estimates for the accuracies of the forward and backwards Euler for the model problems, and introduced the concept of stability of the numerical scheme. The former results directly from the number of terms retained in the Taylor expansion of the variables, while the latter originates from the structure of the time advancement scheme and the eigenvalues of the rhs linear operator appearing the discretized equations.

In the next notebook, we introduce some more efficient time advancement schemes which have both better accuracy and larger domains of stability. They are know as Runge-Kutta schemes and we will use them extensively when analysing partial differential equations later on in the course.


## Exercices


**Exercise 1.** Write a Python code and perform the corresponding visualisation showing that for one time step, the forward Euler method is indeed of second order accuracy. 

**Exercise 2.** In the case of the body in free fall, compare the solution obtained with the forward Euler and the backwards schemes to the exact solution. Check again that both methods are of first-order accuracy for a finite time interval.

**Exercise 3.** Find and plot the regions of stability for both methods for the problem of a body in free fall.

```python

```