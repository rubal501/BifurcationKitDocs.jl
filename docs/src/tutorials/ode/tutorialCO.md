# 🟡 CO oxidation (codim 2)

```@contents
Pages = ["tutorialCO.md"]
Depth = 3
```

In this tutorial, we study the Bykov–Yablonskii–Kim
model of CO oxidation (see [^Govaerts]). The goal of the tutorial is to show a simple example of how to perform codimension 2 bifurcation detection.

$$\left\{\begin{array}{l}\dot{x}=2 q_{1} z^{2}-2 q_{5} x^{2}-q_{3} x y \\ \dot{y}=q_{2} z-q_{6} y-q_{3} x y \\ \dot{s}=q_{4} z-k q_{4} s\end{array}\right.\tag{E}$$

where $z=1-x-y-s$.

We start with some imports:

```@example TUTCO
using Revise, Parameters, Plots
using BifurcationKit
const BK = BifurcationKit

nothing # hide
```

## Problem setting

We can now encode the vector field (E) in a function and use automatic differentiation to compute its various derivatives.

```@example TUTCO
# vector field of the problem
function COm(u, p)
	@unpack q1,q2,q3,q4,q5,q6,k = p
	x, y, s = u
	z = 1-x-y-s
	out = similar(u)
	out[1] = 2q1 * z^2 - 2q5 * x^2 - q3 * x * y
	out[2] = q2 * z - q6 * y - q3 * x * y
	out[3] = q4 * z - k * q4 * s
	out
end

# parameters used in the model
par_com = (q1 = 2.5, q2 = 0.6, q3 = 10., q4 = 0.0675, q5 = 1., q6 = 0.1, k = 0.4)

recordCO(x, p) = (x = x[1], y = x[2], s = x[3])

# initial condition
z0 = [0.07, 0.2, 05]

# Bifurcation Problem
prob = BifurcationProblem(COm, z0, par_com, (@lens _.q2); record_from_solution = recordCO)
nothing # hide
```

## Continuation and codim 1 bifurcations

Once the problem is set up, we can continue the state w.r.t. $q_2$ and detect codim 1 bifurcations. This is achieved as follows:

```@example TUTCO
# continuation parameters
opts_br = ContinuationPar(p_min = 0., p_max = 1.9, ds = 0.002, dsmax = 0.01)

# compute the branch of solutions
br = continuation(prob, PALC(), opts_br; plot = true, verbosity = 2, normC = norminf)
```

```@example TUTCO
# plot the branch
scene = plot(br, xlims = (0.8,1.8))
```

## Continuation of Fold points

We follow the Fold points in the parameter plane $(q_2, k)$. We tell the solver to consider `br.specialpoint[2]` and continue it.

```@example TUTCO
sn_codim2 = continuation(br, 2, (@lens _.k),
	ContinuationPar(opts_br, p_max = 2.2, ds = -0.001, dsmax = 0.05);
	normC = norminf,
	# detection of codim 2 bifurcations
	detect_codim2_bifurcation = 2,
	# update the Fold problem at every continuation step
	update_minaug_every_step = 1,
	# compute both sides of the initial condition
	bothside = true,)

scene = plot(sn_codim2, vars = (:q2, :x), branchlabel = "Fold")
plot!(scene, br, xlims=(0.8, 1.8))
```

## Continuation of Hopf points

We tell the solver to consider `br.specialpoint[1]` and continue it.

```@example TUTCO
hp_codim2 = continuation(br, 1, (@lens _.k),
	ContinuationPar(opts_br, p_max = 2.8, ds = -0.001, dsmax = 0.05) ;
	normC = norminf,
	# detection of codim 2 bifurcations
	detect_codim2_bifurcation = 2,
	# tell to start the Hopf problem using eigen elements: compute left eigenvector
	#start_with_eigen = true,
	# we update the Hopf problem at every continuation step
	update_minaug_every_step = 1,
	# compute both sides of the initial condition
	bothside = true,
	)

# plotting
scene = plot(sn_codim2, vars = (:q2, :x), branchlabel = "Fold")
plot!(scene, hp_codim2, vars = (:q2, :x), branchlabel = "Hopf")
plot!(scene, br, xlims = (0.6, 1.5))
```


## References

[^Govaerts]: > Govaerts, Willy J. F. Numerical Methods for Bifurcations of Dynamical Equilibria. Philadelphia, Pa: Society for Industrial and Applied Mathematics, 2000.
