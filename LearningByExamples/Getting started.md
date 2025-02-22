---
author: Luke Chern
created: 2025-02-20
tags:
---
For a given function $f(x,y)$ , find a function satisfying $u(x,y)$ :

Here is the boundary of the bounded open set and .

We will compute with and the unit disk. The boundary is defined as:

Note

In **FreeFEM**, the domain is assumed to be described by the left side of its boundary.

The following is the **FreeFEM** program which computes :

```c++
// Define mesh boundary
border C(t=0, 2*pi){x=cos(t); y=sin(t);}
// The triangulated domain Th is on the left side of its boundary
mesh Th = buildmesh(C(50));
// The finite element space defined over Th is called here Vh
fespace Vh(Th, P1);
Vh u, v;// Define u and v as piecewise-P1 continuous functions

// Define a function f
func f= x*y;

// Get the clock in second
real cpu=clock();

// Define the PDE
solve Poisson(u, v, solver=LU)
    = int2d(Th)(    // The bilinear part
          dx(u)*dx(v)
        + dy(u)*dy(v)
    )
    - int2d(Th)(    // The right hand side
          f*v
   )
    + on(C, u=0);   // The Dirichlet boundary condition

// Plot the result
plot(u);

// Display the total computational time
cout << "CPU time = " << (clock()-cpu) << endl;
```

As illustrated in [Fig. 2](https://doc.freefem.org/tutorials/#figpoissonu), we can see the isovalue of by using **FreeFEM** `plot` command (see line 29 above).

![firstTh](https://doc.freefem.org/_images/firstTh.png)

Fig. 1 Mesh Th by `buildmesh(C(50))`![firstU](https://doc.freefem.org/_images/firstU.png)

Fig. 2 Isovalue by `plot(u)`Poisson’s equation

Note

The qualifier `solver=LU` (line 18) is not required and by default a multi-frontal `LU` is used.

The lines containing `clock` are equally not required.

Tip

Note how close to the mathematics **FreeFEM** language is.

Lines 19 to 24 correspond to the mathematical variational equation:

for all which are in the finite element space and zero on the boundary .

Tip

Change `P1` into `P2` and run the program.

This first example shows how **FreeFEM** executes with no effort all the usual steps required by the finite element method (FEM). Let’s go through them one by one.

**On the line 2**:

The boundary is described analytically by a parametric equation for and for . When then each curve must be specified and crossings of are not allowed except at end points.

The keyword `label` can be added to define a group of boundaries for later use (boundary conditions for instance). Hence the circle could also have been described as two half circle with the same label:

```freefem
1border Gamma1(t=0, pi){x=cos(t); y=sin(t); label=C};
2border Gamma2(t=pi, 2.*pi){x=cos(t); y=sin(t); label=C};
```

Boundaries can be referred to either by name (`Gamma1` for example) or by label (`C` here) or even by its internal number here 1 for the first half circle and 2 for the second (more examples are in [Meshing Examples](https://doc.freefem.org/examples/mesh-generation.html#examplemeshgeneration)).

**On the line 5**

The triangulation of is automatically generated by `buildmesh(C(50))` using 50 points on `C` as in [Fig. 1](https://doc.freefem.org/tutorials/#figpoissonmesh).

The domain is assumed to be on the left side of the boundary which is implicitly oriented by the parametrization. So an elliptic hole can be added by typing:

```freefem
1border C(t=2.*pi, 0){x=0.1+0.3*cos(t); y=0.5*sin(t);};
```

If by mistake one had written:

```freefem
1border C(t=0, 2.*pi){x=0.1+0.3*cos(t); y=0.5*sin(t);};
```

then the inside of the ellipse would be triangulated as well as the outside.

Note

Automatic mesh generation is based on the Delaunay-Voronoi algorithm. Refinement of the mesh are done by increasing the number of points on , for example `buildmesh(C(100))`, because inner vertices are determined by the density of points on the boundary.

Mesh adaptation can be performed also against a given function f by calling `adaptmesh(Th,f)`.

Now the name (`Th` in **FreeFEM**) refers to the family of triangles shown in [Fig. 1](https://doc.freefem.org/tutorials/#figpoissonmesh).

Traditionally refers to the mesh size, to the number of triangles in and to the number of vertices, but it is seldom that we will have to use them explicitly.

If is not a polygonal domain, a “skin” remains between the exact domain and its approximation . However, we notice that all corners of are on .

**On line 8:**

A finite element space is, usually, a space of polynomial functions on elements, triangles here only, with certain matching properties at edges, vertices etc. Here `fespaceVh(Th,P1)` defines to be the space of continuous functions which are affine in on each triangle of .

As it is a linear vector space of finite dimension, basis can be found. The canonical basis is made of functions, called the *hat function* , which are continuous piecewise affine and are equal to 1 on one vertex and 0 on all others. A typical hat function is shown on [Fig. 4](https://doc.freefem.org/tutorials/#figpoissonhat).

![meshTh2](https://doc.freefem.org/_images/meshTh_2.png)

Fig. 3 `meshTh`![HatFunctions](https://doc.freefem.org/_images/hat_functions.png)

Fig. 4 Graph of (left) and (right)Hat functions

Note

The easiest way to define is by making use of the *barycentric coordinates* of a point , defined by where are the 3 vertices of . Then it is easy to see that the restriction of on is precisely .

Then:

where is the dimension of , i.e. the number of vertices. The are called the *degrees of freedom* of and the number of degree of freedom.

It is said also that the *nodes* of this finite element method are the vertices.

**Setting the problem**

On line 9, `Vhu,v` declares that and are approximated as above, namely:

On the line 12, the right hand side `f` is defined analytically using the keyword `func`.

Line 18 to 26 define the bilinear form of equation [(1)](https://doc.freefem.org/tutorials/#equation-eqn-poisson) and its Dirichlet boundary conditions.

This *variational formulation* is derived by multiplying [(1)](https://doc.freefem.org/tutorials/#equation-eqn-poisson) by and integrating the result over :

Then, by Green’s formula, the problem is converted into finding such that

with:

In **FreeFEM** the **Poisson** problem can be declared only as in:

```freefem
1Vh u,v; problem Poisson(u,v) = ...
```

and solved later as in:

```freefem
1Poisson; //the problem is solved here
```

or declared and solved at the same time as in:

```freefem
1Vh u,v; solve Poisson(u,v) = ...
```

and [(4)](https://doc.freefem.org/tutorials/#equation-eqn-weakform) is written with `dx(u)` , `dy(u)` and:

`int2d(Th)(dx(u)*dx(v)+dy(u)*dy(v))`

`int2d(Th)(f*v)` (Notice here, is unused)

Warning

In **FreeFEM** **bilinear terms and linear terms should not be under the same integral** indeed to construct the linear systems **FreeFEM** finds out which integral contributes to the bilinear form by checking if both terms, the unknown (here `u`) and test functions (here `v`) are present.

**Solution and visualization**

On line 15, the current time in seconds is stored into the real-valued variable `cpu`.

Line 18, the problem is solved.

Line 29, the visualization is done as illustrated in [Fig. 2](https://doc.freefem.org/tutorials/#figpoissonu).

**(see** [Plot](https://doc.freefem.org/documentation/visualization.html#plot) **for zoom, postscript and other commands).**

Line 32, the computing time (not counting graphics) is written on the console. Notice the C++-like syntax; the user needs not study C++ for using **FreeFEM**, but it helps to guess what is allowed in the language.

**Access to matrices and vectors**

Internally **FreeFEM** will solve a linear system of the type

which is found by using [(3)](https://doc.freefem.org/tutorials/#equation-defu) and replacing by in [(4)](https://doc.freefem.org/tutorials/#equation-eqn-weakform). The Dirichlet conditions are implemented by penalty, namely by setting and if is a boundary degree of freedom.

Note

The number is called `tgv` (*très grande valeur* or *very high value* in english) and it is generally possible to change this value, see the item :freefem

The matrix is called *stiffness matrix*. If the user wants to access directly he can do so by using (see section [Variational form, Sparse matrix, PDE data vector](https://doc.freefem.org/documentation/finite-element.html#variationalformsparsematrixpde) for details).

```freefem
1varf a(u,v)
2    = int2d(Th)(
3          dx(u)*dx(v)
4        + dy(u)*dy(v)
5    )
6    + on(C, u=0)
7    ;
8matrix A = a(Vh, Vh); //stiffness matrix
```

The vector in [(5)](https://doc.freefem.org/tutorials/#equation-eqn-equation) can also be constructed manually:

```freefem
1varf l(unused,v)
2    = int2d(Th)(
3          f*v
4    )
5    + on(C, unused=0)
6    ;
7Vh F;
8F[] = l(0,Vh); //F[] is the vector associated to the function F
```

The problem can then be solved by:

```freefem
1u[] = A^-1*F[]; //u[] is the vector associated to the function u
```

Note

Here `u` and `F` are finite element function, and `u[]` and `F[]` give the array of value associated (`u[]` and `F[]` ).

So we have:

where are the basis functions of Vh like in equation :eq: equation3, and is the number of degree of freedom (i.e. the dimension of the space Vh).

The linear system [(5)](https://doc.freefem.org/tutorials/#equation-eqn-equation) is solved by `UMFPACK` unless another option is mentioned specifically as in:

```freefem
1Vh u, v;
2problem Poisson(u, v, solver=CG) = int2d(...
```

meaning that `Poisson` is declared only here and when it is called (by simply writing `Poisson;`) then [(5)](https://doc.freefem.org/tutorials/#equation-eqn-equation) will be solved by the Conjugate Gradient method.