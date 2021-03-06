Algebraic solutions
====================

#  Definitions

##  Matrices

**Matrix Definition** A  matrix is a linear transformation between finite dimensional vector spaces.

**Assembling a matrix**
Assembling a matrix means defining its action as entries stored in a **sparse** or **dense** format. For example, in the finite element context, the storage format is sparse to take advantage of the many zero entries.

**Symmetric matrix**
$$A = A^T$$


**Definite (resp. semi-definite) positive matrix**
All eigenvalue are 
 1. $$>0$$ (resp $$\geq 0$$) or 
 2. $$x^T\ A\ x >\ 0\, \forall\ x$$ (resp. $$x^T\ A\ x
\geq 0\, \forall\ x$$)

**Definite (resp. semi-negative) matrix**
All eigenvalue are 
 1. $$<0$$ (resp. $$\leq 0$$) or
 2. $$x^T\ A\ x < 0\ \forall\ x$$ (resp. $$x^T\ A\ x \leq 0\, \forall\ x$$)

**Indefinite matrix**
There exists 
 1. positive and negative eigenvalue (Stokes, Helmholtz) or
 2. there exists $$x,y$$ such that $$x^TAx > 0 > y^T A y$$

## Preconditioners

### Definition

Let $$A$$ be a $$\mathbb{R}^{n\times n}$$ matrix, $$x$$ and $$b$$ be $$\mathbb{R}^n$$ vectors, we wish to solve
$$A x = b.$$

**Definition**: A preconditioner $$\mathcal{P}$$ is a method for constructing a matrix (just a linear function, not assembled!)  $$P^{-1} = \mathcal{P}(A,A_p)$$ using a matrix $$A$$ and extra information $$A_p$$, such that the spectrum of $$P^{-1}A$$ (left preconditioning) or $$A P^{-1}$$ (right preconditioning) is well-behaved. The action of preconditioning improves the conditioning of the previous linear system. 

**Left preconditioning**:  We solve for
$$  (P^{-1} A) x = P^{-1} b $$
and we build the Krylov space 
$$\{ P^{-1} b, (P^{-1}A) P^{-1} b, (P^{-1}A)^2 P^{-1} b, \dots\}$$

**Right preconditioning**: We solve for
$$  (A P^{-1}) P x = b $$
and we build the Krylov space 
$$\{ b, (P^{-1}A)b, (P^{-1}A)^2b, \dotsc \}$$

Note that the product $$P^{-1}A$$ or $$A P^{-1}$$ is never assembled.

### Properties

Let us now describe some  properties of preconditioners

  - $$P^{-1}$$ is dense, $$P$$ is often not available and is not needed
  - $$A$$ is rarely used by $$\mathcal{P}$$, but $$A_p = A$$ is common
  - $$A_p$$ is often a sparse matrix, the \e preconditioning  \e matrix
  
Here are some numerical methods to solve the system $$A x = b$$
  - **Matrix-based**: Jacobi, Gauss-Seidel, SOR, ILU(k), LU
  - **Parallel**: Block-Jacobi, Schwarz, Multigrid, FETI-DP, BDDC
  - **Indefinite**: Schur-complement, Domain Decomposition, Multigrid

More details and strategies are available in the [Preconditioner section](Preconditioner.md).

# Principles 

  Feel++ abstracts the PETSc library and provides a subset (sufficient in most
  cases) to the PETSc features. It interfaces with the following PETSc
  libraries: `Mat` , `Vec` , `KSP` , `PC` , `SNES.` 
  - `Vec`  Vector handling library
  - `Mat`  Matrix handling library
  - `KSP`  Krylov SubSpace library implements various iterative solvers
  - `PC`  Preconditioner library implements various  preconditioning strategies
  - `SNES`  Nonlinear solver library implements various  nonlinear solve strategies

All linear algebra are encapsulated within backends using the command line option `--backend=<backend>` or config file option `backend=<backend>` which provide interface to several libraries


| Library | Format  | Backend |
|---------|---------|---------|
| PETSc   | sparse  | `petsc` |
| Eigen   | sparse  | `eigen` |
| Eigen   | dense   | `eigen_dense` |



The default `backend` is `petsc.` 

# Examples

## Laplacian

We start with the quickstart Laplacian example, recall that we wish to, given a domain $$\Omega$$, find $$u$$ such that

$$-\nabla \cdot (k \nabla u) = f \mbox{ in } \Omega \subset \mathbb{R}^{2},\\
u = g \mbox{ on } \partial \Omega $$

Monitoring KSP solvers

```cpp
feelpp_qs_laplacian --ksp-monitor=true
```

Viewing KSP solvers


```sh
shell> mpirun -np 2 feelpp_qs_laplacian --ksp-monitor=1  --ksp-view=1
  0 KSP Residual norm 8.953261456448e-01
  1 KSP Residual norm 7.204431786960e-16
KSP Object: 2 MPI processes
  type: gmres
    GMRES: restart=30, using Classical (unmodified) Gram-Schmidt
     Orthogonalization with no iterative refinement
    GMRES: happy breakdown tolerance 1e-30
  maximum iterations=1000
  tolerances:  relative=1e-13, absolute=1e-50, divergence=100000
  left preconditioning
  using nonzero initial guess
  using PRECONDITIONED norm type for convergence test
PC Object: 2 MPI processes
  type: shell
    Shell:
  linear system matrix = precond matrix:
  Matrix Object:   2 MPI processes
    type: mpiaij
    rows=525, cols=525
    total: nonzeros=5727, allocated nonzeros=5727
    total number of mallocs used during MatSetValues calls =0
      not using I-node (on process 0) routines
  ```

### Solvers and preconditioners

## Stokes

  We now turn to the quickstart Stokes example, recall that we wish to, given a
  domain $$\Omega$$, find $$(\mathbf{u},p) $$ such that

  $$
  -\Delta \mathbf{u} + \nabla p = \mathbf{ f} \mbox{ in } \Omega,\\
  \nabla \cdot \mathbf{u} =    0 \mbox{ in } \Omega,\\
  \mathbf{u} = \mathbf{g} \mbox{ on } \partial \Omega
  $$

 This problem is indefinite. Possible solution strategies are
 - Uzawa, 
 - penalty(techniques from optimisation), 
 - augmented lagrangian approach (Glowinski,Le Tallec)

**Note** that The Inf-sup condition must be satisfied. In particular for a multigrid strategy, the smoother needs to preserve it.

### General approach for saddle point problems

 The Krylov subspace solvers for indefinite problems are MINRES, GMRES. As to preconditioning, we look first at the saddle point matrix $$M$$ and its block factorization $$M = LDL^T$$, indeed we have :
$$M =   \begin{pmatrix}
          A & B \\
          B^T & 0
        \end{pmatrix}
        =
        \begin{pmatrix}
          I & 0\\
          B^T C & I
        \end{pmatrix}
        \begin{pmatrix}
          A & 0\\
          0 & - B^T A^{-1} B
        \end{pmatrix}
        \begin{pmatrix}
          I & A^{-1} B\\
          0 & I
        \end{pmatrix}
$$
        
- Elman, Silvester and Wathen propose 3 preconditioners:

$$
P_1 =
\begin{pmatrix}
\tilde{A}^{-1} & B\\
B^T & 0
\end{pmatrix}, \quad
P_2 =
\begin{pmatrix}
\tilde{A}^{-1} & 0\\
0 & \tilde{S}
\end{pmatrix},\quad
P_3 =
\begin{pmatrix}
\tilde{A}^{-1} & B\\
0 & \tilde{S}
\end{pmatrix}
$$
where $$\tilde{S} \approx S^{-1} = B^T A^{-1} B$$ and  $$\tilde{A}^{-1}
    \approx A^{-1}$$

# Options

