# MultiScaleArrays.jl: High-Performance Matrix Exponentiation and Products

MultiScaleArrays.jl allows you to easily build multiple scale models which are
fully compatible with native Julia scientific computing packages like
DifferentialEquations.jl or Optim.jl. These models utilize
a tree structure to describe phenomena of multiple scales, but the interface allows
you to describe equations on different levels, using aggregations from lower
levels to describe complex systems. Their structure allows for complex and dynamic
models to be developed with only a small performance difference. In the end, they present
themselves as an `AbstractArray` to standard solvers, allowing them to be used
in place of a `Vector` in any appropriately made Julia package.

## Installation

To install MultiScaleArrays.jl, use the Julia package manager:

```julia
using Pkg
Pkg.add("MultiScaleArrays")
```

## Example

The usage is best described by an example. Here we build a hierarchy where
Embryos contain Tissues which contain Populations which contain Cells, and the
cells contain proteins whose concentrations are modeled as simply a vector
of numbers (it can be anything linearly indexable).

```julia
using MultiScaleArrays
struct Cell{B} <: AbstractMultiScaleArrayLeaf{B}
    values::Vector{B}
end
struct Population{T<:AbstractMultiScaleArray,B<:Number} <: AbstractMultiScaleArray{B}
    nodes::Vector{T}
    values::Vector{B}
    end_idxs::Vector{Int}
end
struct Tissue{T<:AbstractMultiScaleArray,B<:Number} <: AbstractMultiScaleArray{B}
    nodes::Vector{T}
    values::Vector{B}
    end_idxs::Vector{Int}
end
struct Embryo{T<:AbstractMultiScaleArray,B<:Number} <: AbstractMultiScaleArrayHead{B}
    nodes::Vector{T}
    values::Vector{B}
    end_idxs::Vector{Int}
end
```

This setup defines a type structure which is both a tree and an array. A picture of a possible
version is the following:

![](https://user-images.githubusercontent.com/1814174/27211626-79fe1b9a-520f-11e7-87f1-1cb33da91609.PNG)

Let's build a version of this. Using the constructors we can directly construct leaf types:

```julia
cell1 = Cell([1.0; 2.0; 3.0])
cell2 = Cell([4.0; 5.0])
```

and build types higher up in the hierarchy by using the `constuct` method. The method
is `construct(T::AbstractMultiScaleArray, nodes, values)`, though if `values` is not given it's
taken to be empty.

```julia
cell3 = Cell([3.0; 2.0; 5.0])
cell4 = Cell([4.0; 6.0])
population  = construct(Population, deepcopy([cell1, cell3, cell4]))
population2 = construct(Population, deepcopy([cell1, cell3, cell4]))
population3 = construct(Population, deepcopy([cell1, cell3, cell4]))
tissue1 = construct(Tissue, deepcopy([population, population2, population3])) # Make a Tissue from Populations
tissue2 = construct(Tissue, deepcopy([population2, population, population3]))
embryo = construct(Embryo, deepcopy([tissue1, tissue2])) # Make an embryo from Tissues
```

## Idea

The idea behind MultiScaleArrays is simple. The `*DiffEq` solvers (OrdinaryDiffEq.jl,
StochasticDiffEq.jl, DelayDiffEq.jl, etc.) and native optimization packages like
Optim.jl in their efficient in-place form all work with any Julia-defined
`AbstractArray` which has a linear index. Thus, to define our multiscale model,
we develop a type which has an efficient linear index. One can think of representing
cells with proteins as each being an array with values for each protein. The linear
index of the multiscale model would be indexing through each protein of each cell.
With proper index overloads, one can define a type such that `a[i]` does just that,
and thus it will work in the differential equation solvers. MultiScaleArrays.jl
takes that further by allowing one to recursively define an arbitrary `n`-level
hierarchical model which has efficient indexing structures. The result is a type
which models complex behavior, but the standard differential equation solvers will
work directly and efficiently on this type, making it easy to develop novel models
without having to re-develop advanced adaptive/stiff/stochastic/etc. solving
techniques for each new model.

## Contributing

- Please refer to the
  [SciML ColPrac: Contributor's Guide on Collaborative Practices for Community Packages](https://github.com/SciML/ColPrac/blob/master/README.md)
  for guidance on PRs, issues, and other matters relating to contributing to ModelingToolkit.
- There are a few community forums:
    - the #diffeq-bridged channel in the [Julia Slack](https://julialang.org/slack/)
    - [JuliaDiffEq](https://gitter.im/JuliaDiffEq/Lobby) on Gitter
    - on the [Julia Discourse forums](https://discourse.julialang.org)
    - see also [SciML Community page](https://sciml.ai/community/)
