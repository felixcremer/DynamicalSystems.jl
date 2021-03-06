# Recurrence Plots
## Recurrence Matrices

A [Recurrence plot](https://en.wikipedia.org/wiki/Recurrence_plot) (which refers to the plot of a matrix) is a way to quantify *recurrences* that occur in a trajectory. A recurrence happens when a trajectory visits the same neighborhood on the phase space that it was at some previous time.

The central structure used in these recurrences is the (cross-) recurrence matrix:
```math
R[i, j] = \begin{cases}
1 \quad \text{if}\quad d(x[i], y[j]) \le \varepsilon\\
0 \quad \text{else}
\end{cases}
```
where $d(x[i], y[j])$ stands for the _distance_ between trajectory $x$ at point $i$ and trajectory $y$ at point $j$. Both $x, y$ can be single timeseries, full trajectories or embedded timeseries (which are also trajectories).

If $x\equiv y$ then $R$ is called recurrence matrix, otherwise it is called cross-recurrence matrix. There is also the joint-recurrence variant, see below.
With `RecurrenceAnalysis` you can use the following functions to access these matrices
```@docs
RecurrenceMatrix
CrossRecurrenceMatrix
JointRecurrenceMatrix
```

## Simple Recurrence Plots
The recurrence matrices are internally stored as sparse matrices with boolean values. Typically in the literature one does not "see" the matrices themselves but instead a plot of them (hence "Recurrence Plots"). By default, when a Recurrence Matrix is created we "show" a mini plot of it which is text-based scatterplot.

Here is an example recurrence plot/matrix of a full trajectory of the Roessler system:
```@example recurrence
using DynamicalSystems
ro = Systems.roessler(ones(3), a=0.15, b=0.20, c=10.0)
N = 2000; dt = 0.05
tr = trajectory(ro, N*dt; dt = dt, Ttr = 10.0)

R = RecurrenceMatrix(tr, 5.0; metric = "euclidean")
using UnicodePlots # hide
textrecurrenceplot(R; canvas = DotCanvas) # hide
```
```@example recurrence
typeof(R)
```
```@example recurrence
summary(R)
```

---

The above simple plotting functionality is possible through the package [`UnicodePlots`](https://github.com/Evizero/UnicodePlots.jl). The following function creates the plot:
```@docs
textrecurrenceplot
```

---

Here is the same plot but using Unicode Braille characters
```@example recurrence
textrecurrenceplot(R; ascii = false)
```

As you can see, the Unicode based plotting doesn't display nicely everywhere. It does display perfectly in e.g. Juno, which is where it is the default printing type. Here is how it looks like in a dark background:

![](rqaplot in Juno.PNG)

## Advanced Recurrence Plots
A text-based plot is cool, fast and simple. But often one needs the full resolution offered by the data of a recurrence matrix. This functionality is supported by the following function:
```@docs
recurrenceplot
```

---

Example:

```@example recurrence
Rp = recurrenceplot(R)
```

So let's plot this thing now:
```@example recurrence
using PyPlot; figure(figsize = (6, 8))
ax1 = subplot2grid((3,1), (0,0))
plot(0:dt:N*dt, tr[:, 2], "k"); xlim(0, N*dt); PyPlot.ylabel("\$y(t)\$")
ax2 = subplot2grid((3,1), (1, 0), rowspan = 2)

imshow(Rp, cmap = "binary_r", extent = (0, N*dt, 0, N*dt))
PyPlot.xlabel("\$t\$"); PyPlot.ylabel("\$t\$"); tight_layout()
subplots_adjust(hspace = 0.2)
savefig("rmatrix.png"); nothing # hide
```
![](rmatrix.png)

and here is exactly the same process, but using the embedded trajectory instead
```@example recurrence
using PyPlot # hide
y = tr[:, 2]
τ = estimate_delay(y, "mi_min")
m = reconstruct(y, 2, τ)
R = RecurrenceMatrix(m, 5.0; metric = "euclidean")
Rp = recurrenceplot(R)
figure(figsize = (6, 6));
imshow(Rp, cmap = "binary_r", extent = (0, N*dt, 0, N*dt))
PyPlot.xlabel("\$t\$"); PyPlot.ylabel("\$t\$"); tight_layout()
savefig("rmatrix2.png"); nothing # hide
```
![](rmatrix2.png)

which justifies why recurrence plots are so fitting to be used in embedded timeseries.

## Distances
The distance function used in [`RecurrenceMatrix`](@ref) and co. can be specified either as a string or as any `Metric` instance from [`Distances`](https://github.com/JuliaStats/Distances.jl). In addition, the following function returns a matrix with the cross-distances across all points in one or two trajectories:
```@docs
distancematrix
```
