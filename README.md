# EQeq

Charge equilibration method for crystal structures.

Modified version, which allows to specify additional parameters:

 * `lambda` (default: 1.2) The dielectric screening parameter. corresponds to eps_eff = 1.67
 * `hI0` (default: -2.0) The electron affinity of hydrogen
 * `chargePrecision` (default: 3) Number of digits to use for point charges
 * `method` (default: "ewald", alternative: "nonperiodic") Method to compute the Coulombic interaction
 * `mR` (default: 2) Number of "expansion" unit cells to consider in periodic calculation ("real space"). 2 => 5x5x5
 * `mK` (default: 2) Number of "expansion" unit cells to consider in periodic calculation ("frequency space"). 2 => 5x5x5
 * `eta` (default: 50) Ewald splitting parameter
 * `ionizationdata` (default: [ionizationdata.dat](data/ionizationdata.dat)) File with ionization potentials and electron affinities. Default data are  
   EA: experimental, [T.Andersen et al., 1999](http://aip.scitation.org/doi/10.1063/1.556047)  
   IP: experimental, [C.E.Moore, 1970](https://nvlpubs.nist.gov/nistpubs/Legacy/NSRDS/nbsnsrds34.pdf)
 * `chargecenters` (default: [chargecenters.dat](data/chargecenters.dat)) File with common oxidation states (lowered, if missing ionizationdata)

### Requirements

The package requires :
 - numpy
 - openbabel (tested with : openbabel==3.1.0)

> Note : do not use pybel from pip since it does not install the necessary methods from openbabel.

### Usage

To run the HKUST-1 example:

```bash
cd examples/HKUST1
./run.sh
```

### Summary

The source code in this program demonstrates the charge equilibration method described
in the accompanying paper. The purpose of the source code provided is to be
minimalistic and do "just the job" described. In practice, you may wish to add various
features to the source code to fit the particular needs of your project.

#### Major highlights of program:

 * Obtains charges for atoms in periodic systems without iteration
 * Can use non-neutral charge centers for more accurate point charges
 * Designed for speed (but without significant code optimizations)

#### Features not implemented but that you may want to consider adding:

 * Spherical cut-offs (for both real-space and reciprocal-space sums)
 * An iterative loop that guesses the appropriate charge center (so the user does not have to guess)
 * Ewald parameter auto-optimization
 * Various code optimizations

#### Running the program:

Program expects two input files `ionization.dat` and `chargecenters.dat`. Please
look at source code to see what the other optional inputs are for (should be
mostly self-explanatory). Compile with something like:

```
g++ main.cpp -O3 -o eqeq
```

and run with

```
./eqeq my_file.cif
```

#### Python bindings

To facilitate automation and scaling, this version of EQeq can be operated via
Python. To enable, you must build EQeq as a shared library:

```
g++ -c -fPIC main.cpp -O3 -o eqeq.o
g++ -shared -Wl,-soname,libeqeq.so -O3 -o libeqeq.so eqeq.o
sudo cp libeqeq.so /usr/lib
```

(for Macs, replace `-soname` with `-install_name`)
(if you don't have sudo access, `mkdir ~/lib; cp libeqeq.so ~/lib`, then change
the path at the top of `eqeq.py`)

From the command line, you can run `eqeq.py`:

```
python eqeq.py --help
python eqeq.py IRMOF-1.cif --output-type mol --method ewald
```

This includes extensive help documentation on running EQeq, and an easy
interface to change individual parameters.

To use globally with Python scripts, you must put the `EQeq` directory on your
PYTHONPATH:

```
export PYTHONPATH:/path/above/EQeq:$PYTHONPATH
```

Then, you can call EQeq from Python with

```python
import EQeq
EQeq.run("IRMOF-1.cif")
```

The input takes both filenames and actual data, and outputs either files or
strings. This change allows for streaming data, which enables EQeq to be used
as part of a broader code pipeline.

```python
import EQeq
# Load a file. In practice, this can come from any source, such as a database
with open("IRMOF-1.cif") as in_file:
    data = in_file.read()
charges = EQeq.run(data, output_type="list", method="ewald")
```
