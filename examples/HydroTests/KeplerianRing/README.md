Keplerian Ring Test
===================

This test uses the '2D Keplerian Ring' to assess the accuracy of SPH
calculations. It is known to fail with all schemes in SWIFT. For more
information on the test, please see Cullen & Dehnen 2009
(http://arxiv.org/abs/1006.1524) Section 4.3.

It is key that for this test in particular that the initial conditions are
perfectly arranged (here we use the same methodology as described in the paper
referenced above).

The test uses:

+ $M = 10^10$ for a central point mass
+ $r$ up to 5 (particles created to edge of box when relevant)
+ Approximately 16000 particles
+ $c_s = 0.01 \ll v_\phi = 10$, enforced by giving them a mass of 1 unit and an
  internal energy of 0.015.


Code Setup
----------

You will need to recompile SWIFT with an external potential. This can be done
by using

    ./configure --with-ext-potential=point-mass

in the root directory of the project. We suggest leaving all other code options
as their defaults.

Please also consider using:

    ./configure --with-ext-potential=point-mass-softened

if you are running with the initial conditions generated with the script and
using a nonzero softening.


Initial Conditions Generation
-----------------------------

The initial condition generation is handled by ```makeIC.py```, for which
the options are available with

    python3 makeIC.py --help

however the defaults are very sensible. If you wish to change the central mass,
you will need to change the external point mass in ```keplerian_ring.yml``` as
well.

There are three main types of generation, handled through `--generationmethod`:

+ `spiral`: using the original spiral method with a density distribution
            created using different mass particles.
+ `gaussian`: a guassian density profile with density distribution changed
              through the _distribution_ of equal mass particles.
+ `grid`: a grid with a flat density profile (similar to Hopkins 2013) imposed
          on it through different particle masses.

All of them have their benefits and drawbacks, so give each a go.

Plotting
--------

Once you have ran swift (we suggest that you use the following)

    ../../../swift --external-gravity --stars --hydro --threads=16 keplerian_ring.yml 2>&1 | tee output.log

there will be around 350 ```.hdf5``` files in your directory. To check out
the results of the example use the plotting script:

    python3 plotSolution.py

which will produce a png file in the directory. Check out `plotSolution.py
--help` for more options.

You can also try the `make_movie.py` script which should make a movie.


Theory
------

We use the 'spiral' technique to generate the initial conditions. To do this,
we first calculate
$$
    m_i = \frac{i - \frac{1}{2}}{N}
$$
for each particle $i$ with $N$ the total number of required particles. Then the
radius of each particle is given by
$$
    r_i = f^{-1}(m_i)
$$
where for us the PDF $f$ is a Gaussian.

To choose the radial positions, we then choose an initial angle $\theta_1$, and
from there
$$
    \delta \theta_i = \left[2\pi\left( 1 - \frac{r_{i-1}}{r_i} \right)\right]~.
$$

### Inverse CDF of Gaussian

We need:
$$
    f^{-1}(m_i) = r_i = r_{central} + \sqrt{2}\sigma \mathrm{erf}^{-1}(2m_i - 1),
$$
and thankfully scipy has an inverse error function built in.


Implementation Details
----------------------

+ The $m_i$ are generated by ```generate_m_i```.
+ The inverse Gaussian PDF is implemented as ```inverse_gaussian```.
+ The $\theta_i$ are generated by ```generate_theta_i```.


Useful Information
------------------

$G$ in simulation units (by default using this yaml) is given by:
$$
    4.300970\times10^{-6}
$$
This is used to convert the mass of the central potential (used in the
parameterfile) to the $GM$ required in the initial conditions generator.