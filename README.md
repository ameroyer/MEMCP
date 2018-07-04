![license](https://img.shields.io/github/license/ameroyer/ReCA.svg)
![GitHub repo size in bytes](https://img.shields.io/github/repo-size/ameroyer/ReCA.svg)
![GitHub top language](https://img.shields.io/github/languages/top/ameroyer/ReCA.svg)
![Maintenance](https://img.shields.io/maintenance/no/2017.svg)


# [ R e C A ]  r e a d   m e

# Installation

#### requirements
   * python (the scripts were tested with versions 2.7 and 3.2)
   * GCC 4.9 +
   * cmake [``cmake`` package]
   * Boost version 1.53+ [``libboost-dev`` package]
   * [Eigen 3.2+](http://eigen.tuxfamily.org/index.php?title=Main_Page) library
   * [lp_solve](http://lpsolve.sourceforge.net/5.5/) library [``lp-solve`` package]

#### installing AIToolbox
Clone the [AIToolbox repository](https://github.com/Svalorzen/AI-Toolbox), then build and test the installation with the following commands

**NOTE**: This was implemented with AIToolox in mid-year 2016. It is possile that more recent versions have different structures and import path mayb need to be changed.

```bash
cd AIToolbox_root
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j
ctest -V
```

# Dataset generation

#### synthetic recommandation dataset
Generate synthetic POMDP parameters to highlight the impact of using multiple environments. The model comprises as many environments as possible recommandations. The *i*-th environment corresponds to users choosing item *i* with a high probability (``p=0.8``) and uniform preference towards other recommandations.  The reward is 0 if the recommendation does not match the user's choice, and 1 otherwise.

  ```bash
  cd Data/
  ./prepare_synth.py -n [1] -k [2] -a [3] -t [4] -o [5] --norm --help
  ```

  * ``[1]`` Number of items (Defaults to 3).
  * ``[2]`` History length (Defaults to 2). Must be strictly greater than 1.
  * ``[3]`` Positive scaling parameter for correct recommandation. Must be greater than 1. Defaults to 1.1.
  * ``[4]`` Number of test sessions to generate following the generated distribution. Defaults to 2000.
  * ``[5]`` Path to the output directory (Defaults to ``../Code/Models``).
  * ``[--norm]`` If present, normalize the output transition probabilities.
  * ``[--zip]`` If present, transitions are stored in an archive. Recommended for large state spaces.
  * ``[--help]`` displays help about the script.

#### Foodmart dataset
 Estimate a POMDP model parameters and test sequences from the [Foodmart](https://github.com/neo4j-examples/neo4j-foodmart-dataset) dataset (.csv dataset files are included in the ``Data/`` directory).

 ```bash
  cd Data/
  ./prepare_foodmart.py -p [1] -k [2] -u [3] -a [4] -t [5] -d [6] -o [7] -D [8] --norm --help
```

  * ``[1]`` Items discretization level. Must be between 0 (*1561 fine-grained products*)  and 4 (*3 high-level categories*). Defaults to 4.
  * ``[2]`` History length, > 1. Defaults to 2.
  * ``[3]`` Number of profiles to generate. Defaults to 5.
  * ``[4]`` Positive scaling parameter for correct recommandation. Must be greater than 1. Defaults to 1.1.
  * ``[5]`` Number of test sequences to generate. Defaults to 2000.
  * ``[6]`` Path to the Foodmart dataset. Defaults to ``Data/Foodmart.gz``.
  * ``[7]`` Path to the output directory. Defaults to ``../Code/Models``.
  * ``[8]`` Number of sequences to isolate to estimate each environment's transition probabilities.
  * ``[--norm]`` If present, output transition probabilities are normalized.
  * ``[--zip]`` If present, transitions are stored in an archive. Recommended for large state spaces.
  * ``[--help]`` displays help about the script.

#### maze dataset
Generating POMDP parameters for a typical maze/path finding problem with multiple environments.

  ```bash
  cd Data/
  ./prepare_maze.py -i [1] -n [2] -s [3] -t [4] -w [5] -g [6] -e [7] -wf [8] -o [9] --rdf --help
  ```

  * ``[1]`` If given, load the maze structure from a file (see toy examples in the ``Mazes`` subdirectory). if not, the mazes are generated randomly with the following parameters.
  * ``[2]`` Maze width and height.
  * ``[3]`` Number of initial states in each maze. Defaults to 1.
  * ``[4]`` Number of trap states in each maze (non-rewarding absorbing states). Defaults to 0.
  * ``[5]`` Number of obstacles in each maze. Defaults to 0.
  * ``[6]`` Number of goal states in each maze. Defaults to 1.
  * ``[7]`` Number of mazes (environments) to generate. Defaults to 1.
  * ``[8]`` Failure rate (equivalent to falling in a trap state) when going forward in the direction of an obstacle. Defaults to 0.05.
  * ``[9]`` Path to the output directory (Defaults to ``../Code/Models``).
  * ``[--norm]`` If present, normalize the output transition probabilities.
  * ``[--rdf]`` If present, the failure rates (probability of staying put instead of realizing the intended action) for each environment are sampled uniformly over [0; 0.5[
  * ``[--help]`` displays help about the script.

# Building and evaluating the MEMDP-based models

#### set-up
The following variables can be configured at the beginning of the ``run.sh`` script (e.g. if some libaries are installed locally and not globally)
  * ``AIROOT``: path to the AIToolbox installation directory.
  * ``EIGEN``: path to the Eigen library installation directory.
  * ``LPSOLVE``: path to the lpsolve library installation directory.
  * ``GCC``: path to the g++ binary.
  * ``STDLIB``: path to the stdlib matching the given gcc compiler.

#### run
```bash
  cd Code/
./run.sh -m [1] -d [2] -n [3] -k [4] -u [5] -g [6] -s [7] -h [8] -e [9] -x [10] -b [11] -c -p -v
```

   * ``[1]`` Model to use. Defaults to mdp. Available options are
      * *mdp*. MDP model obtained by a weighted average of all the environments' transition probabilities and solved by Value iteration. The solver can be configured with
        * ``[7]`` Number of iterations. Defaults to 1000.
      * *pbvi*. point-based value iteration optimized for the MEMDP structure with options
        * ``[8]`` Horizon parameter. Must be greater than 1. Defaults to 2.
        * ``[11]`` Belief size. Defaults to  500.
      * *pomcp*, *pomcpex*, *pamcpex*, *pamcp*. Monte-carlo solvers. *pamcp* and *pamcpex* implement the past-aware graph initialization. *pomcpex* and *pamcpex* implement the exact belief computation. *pomcp* is the vanilla POMCP with MEMDP-optimized sampling (POMCP\*)
        * ``[7]`` Number of simulation steps. Defaults to 1000.
        * ``[8]`` Horizon parameter. Must be greater than 1. Defaults to 2.
        * ``[10]`` Exploration parameter. Defaults to 10000 (high exploration).
        * ``[11]`` Number of particles for the belief approximation. Defaults to  500.
   * ``[2]`` Dataset to use. Defaults to rd. Available options are
     * *fm* (foodmart recommandations) with following options
       * ``[3]`` Product discretization level. Defaults to 4.
       * ``[4]`` History length. Must be strictly greater than 1. Defaults to 2.
       * ``[5]`` User discretization level. Defaults to 0.
     * *mz* (maze solving problem) with following options
       * ``[3]`` Base name for the directory containing the corresponding MEMDP model parameters.
     * *rd* (synthetic data recommandations) with following options
       * ``[3]`` Number of actions. Defaults to 4.
       * ``[4]`` History length. Must be strictly greater than 1. Defaults to 2.
   * ``[6]`` Discount Parameter. Must be strictly between 0 and 1. Defaults to 0.95.
   * ``[9]`` Convergence criterion. Defaults to 0.01.
   * ``[-c]`` If present, recompile the code before running (*Note*: this should be used whenever using a dataset with different parameters as the number of items, environments etc are determined at compilation time).
   * ``[-p]`` If present, normalize the transition and use Kahan summation for more precision while handling small probabilities. Use this option if AIToolbox throws an ``Input transition table does not contain valid probabilities`` error.
   * ``[-v]`` If present, enables verbose output. In verbose mode, evaluation results per environments are displayed, and the std::cerr stream is eanbled during evaluation.

# examples

#### maze solving, 60 environments, 3 actions, ~100 states
  * if needed, generate the data (already available on the repository)
  ```bash
   cd Data/
   python prepare_maze.py --norm --zip -n 5 -s 1 -g 1 -w 0 -t 0 -e 60 --rdf
  ```

  * run the code (assuming the output directory is the default ``ROOT/Code/Models/``)
  ```bash
  cd ../Code/
  ./run.sh -m pbvi -d mz -n gen_5x5_101_60 -h 20 -b 100 -c
  ./run.sh -m pamcp -d mz -n gen_5x5_101_60 -h 10 -c	
  ```

#### synthetic recommandations, 10 environments, 10 actions, ~100 states

  * if needed, generate the data (already available on the repository)
  ```bash
   cd Data/
   python prepare_synth.py --norm --zip -n 10 -k 2
  ```

  * run the code (assuming the output directory is the default ``ROOT/Code/Models/``)
  ```bash
  cd ../Code/
  ./run.sh -m mdp -d rd -n 10 -k 2 -c
  ./run.sh -m pamcp -d rd -n 10 -k 2 -c	
  ./run.sh -m pomcpex -d rd -n 10 -k 2 -c	
  ```

#### Foodmart recommandations, 5 environments, 22 actions, ~500 states

  * if needed, generate the data (already available on the repository)
  ```bash
   cd Data/
   python prepare_foodmart.py --norm --zip -u 5 -p 3 -k 2
  ```

  * run the code (assuming the output directory is the default ``ROOT/Code/Models/``)
  ```bash
  cd ../Code/
  ./run.sh -m mdp -d fm -n 3 -k 2 -u 5 -c
  ./run.sh -m pamcp -d rd -n 3 -k 2 -u 5 -c
  ./run.sh -m pbvi -d rd -n 3 -k 2 -u 5 -c	
  ```


# Known issues
  * When using the ``--zip`` option for data generation, it might be necessary to run the script with ``python3`` due to an [issue](https://bugs.python.org/issue23306) with the gzip library in python < 3.
