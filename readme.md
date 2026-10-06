# OVERLAP TOOLKIT

This directory is a collection of scripts for computing different types of overlap (inner product) of many-body wavefunction. The scripts available are

- `pair_overlap.jl` -- compute the pair overlap |⟨ψ₁|ψ₂⟩| of two states |ψ₁⟩ and |ψ₂⟩
- `cumulative_overlap.jl` -- compute the cumulative overlap of a single state with another collection of states given in a directory
- `CHS_overlap.jl` -- compute the cumulative overlap of a single state with a given *conformal Hilbert space* (CHS) spanned by a collection of Jack polynomials

The toolkit requires the [QHE_Julia](https://github.com/hq-tr/QHE_Julia) library. All states must be given in terms of their monomial expansion in the standard format.

## Pair Overlap
In its most basic usage, the pair overlap of two states provided in files `state1` and `state2` can be computed as

```
julia pair_overlap.jl state1 state2
```

This usage requires that both states are normalized and stored in binary format. If this is not satisfied, additional flags can be specified *before* the file names (file names are positional arguments and must be supplied last). The available flags are

- `--decimal1` -- if the file storing the first state is stored in decimal format
- `--decimal2` -- if the file storing the second state is stored in decimal format
- `--n_orb` or `-o` -- number of orbital. If any `--decimal1` or `--decimal2` is used, this argument must be specified.
- `--normalize1` -- the geometry to normalize the first state in. Available options are `sphere` and `disk` (case insensitive)
- `--normalize2` -- the geometry to normalize the second state in. Available options are `sphere` and `disk` (case insensitive)
- `--quiet` or `-q` -- suppress intermediate printouts and only display the result.

The result is displayed on the terminal. To save it to file, parse the printout to a file, for example:

```
julia pair_overlap.jl state1 state2 >> output.txt
```

## Cumulative Overlap
The cumulative overlap of a state |ψ⟩ with a collection of states |ϕ₁⟩, |ϕ₂⟩,... is defined as

$$\left(\sum_{i}|\langle\psi|\phi_i\rangle|^2\right)^{1/2}$$

assuming all |ϕᵢ⟩ are orthonormal. The most basic usage of `cumulative_overlap.jl` is 

```
julia cumulative_overlap.jl -f state -d dir_name
```

where `state` is the name of the file storing |ψ⟩ and `dir_name` is the path to the directory storing all the states |ϕᵢ⟩. The directory must contain nothing but these states, and the names of the files in this directory does not matter. Again, basic use assume every wavefunction file is in binary format. Additional flags can be used for alternate formats:

- `--decimal` -- if |ψ⟩ is stored in decimal format
- `--decimal-directory` -- if every |ϕᵢ⟩ is in decimal format. Every |ϕᵢ⟩ must be stored in the same format whether binary or decimal.
- `--n_orb` or `-o` -- number of orbitals. Must be specified if any state is in decimal format.
- `--output` -- name of file to store the output. The result will be printed in the terminal regardless whether this argument is supplied. If it is supplied, the output file will contain the total cumulative overlap as well as overlap with every individual |ϕᵢ⟩.
- `--basis` or `-b` -- this is use in a special case where the basis and coefficients of |ψ⟩ are stored in two separate files. In this case, the argument of `-f` must indicate the coefficient file, while the argument of this flag indicates the basis file. Basis is assumed to be in decimal format, and therefore `-o` must also be specified.

## CHS Overlap
This is a special case of cumulative overlap where instead of the orthonormal basis |ϕᵢ⟩, a collection of Jack polynomials |Jₘ⟩ is supplied. Jack polynomials are not normalized and in general not orthogonal, so the script contains these two procedure. The spherical geometry is always used.

All the Jacks must be stored in files named `J_<root>` where `<root>` is the root configuration in binary format. For example, the Jack containing the Laughlin ground state for 3 particles may be stored in `J_1001001`. All the Jacks of a given phase must be stored in the same directory. The path to the directory containing the Jacks for a given phase is specified in the file `CHS.config`. This file maybe modified directly (using Julia syntax). Alternatively, a different config file may be created (if you do not wish to overwrite the default file), in which case the main script can be configured to refer to this file with

```
julia CHS_overlap.jl --configure
```

Then, the cumulative overlap of a state stored in file named `state` with a given FQH phase can be computed with

```
julia CHS_overlap.jl -f state -e <Ne> -o <No> --phase <phase>
```

The number of electrons and number of orbitals must be specified with the flags `-e` and `-o`, respectively. Additionally `<phase>` is the name of the FQH phase to compute the overlap in. Currently the available options are `"Laughlin"`, `"Gaffnian"`, and `"Moore-Read"` (case-sensitive).

The Jacks are always assumed to be stored in binary format. The input state is assumed to be in binary format by default, but decimal can also be used by the `--decimal` flag.