# [ R e C A ] :panda_face: r e a d   m e 

#### Installing AIToolbox (AITB)

##### 1. Pre-requisites
   * GCC 4.9 +
   * cmake [``cmake`` package]
   * Boost version 1.53+ [``libboost-dev`` package]
   * [Eigen 3.2+](http://eigen.tuxfamily.org/index.php?title=Main_Page) library
   * [lp_solve](http://lpsolve.sourceforge.net/5.5/) library [``lp-solve`` package]

##### 2. Build.
Once the pre-requisites are installed, clone the [AITB repository](https://github.com/Svalorzen/AI-Toolbox), then build it and test the installation with the following commands:

```bash
cd AIToolbox_root
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j
ctest -V
```


#### Generating the data (``Data/`` folder)

 * **1. Foodmart dataset.** Infers POMDP parameters from the Foodmart dataset. In aprticular, the reward function is 0 if the recommendation *y* does not match the user's choice *x*. Otherwise, it is the shop profit on selling item *x*.

     ``./prepare_foodmart.py -pl [1] -k [2] -ul [3] -a [4] -t [5] -d [6] -o [7] --norm --help``

     * ``[1]`` Product discretization level (Defaults to 4). Must be between 0 (*fine-grained, 1561 products*)  and 4 (*3 high-level categories*).
     * ``[2]`` History length (Defaults to 2). Must be strictly greater than 1.
     * ``[3]`` User discretization level. For now, only option 0 is implemented (6 user profiles).
     * ``[4]`` Scaling parameter for transition matching the recommended action (Defaults to 1.1). Must be greater than 1.
     * ``[5]`` Proportion of the dataset to keep for parameter inference (Defaults to 0.8).
     * ``[6]`` Path to the Foodmart dataset (either ``tar.gz`` archive or ``Foodmart/data`` directory).
     * ``[7]`` Path to the main output directory (Defaults to ``../Code/Models``).
     * ``[--norm]`` If present, normalize the transition probabilities. (*Note*: Transition probabilities are already normalized in the main C routines).
     * ``[--help]`` displays help about this command

**Example** *(6 environments, 3 actions, 13 states)* :  ``python prepare_foodmart.py -d Foodmart.tar.gz -pl 4 -k 2``





 * **2. Synthetic dataset.** Infers synthetic POMDP parameters to highlight the impact of using multiple environments. The model comprises as many environments as items (actions). More precisely, the *i*-th environment corresponds to users who choose item *i* with a high probability (80%) and the others items with a uniform probability in any circumstances.  he reward is 0 if the recommendation does not match the user's choice, and 1 otherwise.

     ``./prepare_synth.py -n [1] -k [2] -t [3] -o [4] --norm --help``

     * ``[1]`` Number of items/actions (Defaults to 3).
     * ``[2]`` History length (Defaults to 2). Must be strictly greater than 1.
     * ``[3]`` Number of test sessions to generate according to the synthetic distribution (Defaults to 2000).
     * ``[4]`` Path to the output directory (Defaults to ``../Code/Models``).
     * ``[--norm]`` If present, normalize the transition probabilities. (*Note*: Transition probabilities are already normalized in the main C routines).
     * ``[--help]`` displays help about this command

**Example** *(3 environments, 3 actions, 13 states)* :  ``python prepare_synth.py -n 3 -k 2``




#### Running the code (``Code/`` folder)
If needed, first set the correct library pathes in ``run.sh``. The script can then be used as follow:

``./run.sh -m [1] -d [2] -n [3] -k [4] -g [5] -s [6] -h [7] -e [8] -x [9] -b [10] -c -p -v``

   * ``[1]`` Model to use (Defaults to mdp). Available options are *mdp* (MDP model obtained by a weighted average of all the environments' transition probabilities and solved by Value iteration), *pbvi* (point-based value iteration optimized for the MEMDP structure), *pomcp* and *memcp* (Monte-carlo solver, respectively without and with optimization for the MEMDP structure).
   * ``[2]`` Dataset to use (Defaults to fm). Available options are *fm* (foodmart) and *rd* (synthetic data).
   * ``[3]`` Product discretization level if foodmart dataset, or the number of items if synthetic data.
   * ``[4]`` History length (Defaults to 2). Must be strictly greater than 1.
   * ``[5]`` Discount Parameter gamma (Defaults to 0.95). Must be strictly between 0 and 1.
   * ``[6]`` Number of iterations for mdp, and number of simulation steps for pomcp and memcp (Defaults to 1500).
   * ``[7]`` Horizon parameter for the POMDP solvers. Defaults to 2. Must be above 1 (strictly, for pbvi).
   * ``[8]`` Convergence criterion for mdp and ip. Defaults to 0.01.
   * ``[9]`` Exploration parameter for pomcp and memcp. Defaults to 10000 (high exploration). A high exploration parameter allows for less "Observation never seen in the simulation" during evaluation of a pomcp or memcp model. (*Note*: to see these errors, you need to run in verbose mode).
   * ``[10]`` Number of beliefs to use for PBVI, or number of particles for the belief approximation in pomcp and memcp. Defaults to  100.
   * ``[-c]`` If present, recompile the code before running (*Note*: this should be used whenever using a dataset with different parameters as the number of items, environments etc are determined at compilation time).
   * ``[-p]`` If present, use Kahan summation for more precision while handling small probabilities. Use this option if AIToolbox throws an ``Input transition table does not contain valid probabilities`` error.
   * ``[-v]`` If present, enables verbose output. In verbose mode, evaluation results per environments are displayed, and the std::cerr stream is eanbled during evaluation.

**Example** *(foodmart, 6 environments, 3 actions, 13 states)* :
```bash
cd Data/
python prepare_foodmart.py -d Foodmart.tar.gz -pl 4 -k 2
cd ../Code/
./run.sh -m mdp -d fm -n 4 -k 2 -c
./run.sh -m memcp -d fm -n 4 -k 2 -c
```


**Example** *(synthetic, 3 environments, 3 actions, 13 states)* :
```bash
cd Data/
python prepare_synth.py -n 3 -k 2
cd ../Code/
./run.sh -m mdp -d rd -n 3 -k 2 -c
./run.sh -m memcp -d rd -n 3 -k 2 -c
```
