# Continuation of Neimark-Sacker (NS) bifurcations of periodic orbits

!!! warning "Work in progress"
    Continuation of NS of POs is a recent addition. It has not been thoroughly tested.

In this page, we explain how to perform continuation of NS points of periodic orbits and detect the associated codim 2 bifurcations.

### List of detected codim 2 bifurcation points
|Bifurcation|symbol used|
|---|---|
| Cusp | cusp |
| Strong resonance 1:1 bifurcation | R1 |
| Fold / Flip| foldFlip |
| Fold / Neimark-Sacker | foldNS |
| Double Niemark-Sacker | nsns |
| Chenciner | ch |

In a nutshell, all you have to do (see below) is to call `continuation(br, ind_bif, lens2)` to continue the bifurcation point stored in `br.specialpoint[ind_bif]` and set proper options.

## Neimark-Sacker continuation

The continuation of Neimark-Sacker bifurcation points is based on a **Minimally Augmented**[^Govaerts] formulation which is an efficient way to detect singularities (see [Fold / Hopf Continuation](@ref)). All the methods (see [Periodic orbits computation](@ref)), except the Trapezoid one, for computing periodic orbits are compatible with this algorithm. In particular, you can perform these computations in large dimensions.

### Detection of codim 2 bifurcation points

You can detect the following codim 2 bifurcation points by using the keyword argument `detect_codim2_bifurcation` in the method `continuation` 

- the detection of Chenciner bifurcation is done by the the computating the NS normal form
- the detection the above bifurcation points is done by monitoring the number of eigenvalues $\lambda$ such that $\Re\lambda > \min\limits_{\nu\in\Sigma(dF)}|\Re\nu|$ and $\Im\lambda > \epsilon$ where $\epsilon$ is the Newton tolerance.

## Setting the jacobian

In order to apply the newton algorithm to the PD functional, one needs to invert the jacobian. This is not completely trivial as one must compute this jacobian and then invert it. You can select the following jacobians for your computations (see below):

- [Default] for `jacobian_ma = :autodiff`, automatic differentiation is applied to the PD functional and the matrix is then inverted using the provided linear solver. In particular, the jacobian is formed. This is very well suited for small dimensions  (say < 100)
- for `jacobian_ma = :finiteDifferences`, same as `jacobian_ma = :autodiff` but the jacobian is computed using finite differences.

## Codim 2 continuation

To compute the codim 2 curve of Fold points of periodic orbits, one can call [`continuation`](@ref) with the following options

```@docs
 continuation(br::BifurcationKit.AbstractBranchResult, ind_bif::Int64,
				lens2::Lens, options_cont::ContinuationPar = br.contparams ;
				kwargs...)
```

where `br` is a branch of periodic orbits and the options are as above except with have an additional parameter axis `lens2` which is used to locate the bifurcation points.

## Algorithmic details

The continuation of NS points is based on the formulation

$$G(u,p,\omega) = (F_{po}(u,p,\omega), \Re\sigma(u,p,\omega), \Im\sigma(u,p,\omega))\in\mathbb R^{n+2}\quad\quad (\mathcal F_{ns})$$

where $F_{po}$ is the functional for locating periodic orbits and the test function $\sigma$ is solution of

$$\left[\begin{array}{cc}
N(u,p,\omega) & w \\
v^{\top} & 0
\end{array}\right]\left[\begin{array}{c}
r \\
\sigma(u,\omega,p)
\end{array}\right]=\left[\begin{array}{c}
0_{n} \\
1
\end{array}\right].$$

In the case of Multiple Standard Shooting, the matrix $N$ is based on the monodromy $M(x_i,T_i)$

$$N:=\left(\begin{array}{cccccc}
{M_1} & -I & {0} & {\cdots} & 0 \\
0 & {M_2} & -I & {\cdots} & {0} \\
{\vdots} & {\cdots} & {\ddots} & {\ddots} & {\vdots} \\
{0} & {\cdots} & {\cdots} & {\ddots} & -I \\
e^{i\omega}I & {\cdots} & {\cdots} & 0 & {M_{m}} \\
\end{array}\right).$$

The jacobian of the NS functional to use for the Newton algorithm

$$\left[\begin{array}{ccc}
\partial_{u}F_{po} & \partial_pF_{po} & 0 \\
\partial_u\sigma_r & \partial_p\sigma_r & \partial_\omega\sigma_r\\
\partial_u\sigma_i & \partial_p\sigma_i & \partial_\omega\sigma_i
\end{array}\right].$$

## References

[^Govaerts]:> Govaerts, Willy J. F. Numerical Methods for Bifurcations of Dynamical Equilibria. Philadelphia, Pa: Society for Industrial and Applied Mathematics, 2000.