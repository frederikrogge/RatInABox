#  RatInABox 

`RatInABox` 🐀📦 (paper here) is a toolkit for simulating motion and various cell types found in the Hippocampal formation. `RatInABox` is fully continuous is space and time: position and neuronal firing rates are calculated rapidly online with float precision. With it you can:

* **Generate realistic trajectories** for rats exploring complex 1- and 2-dimensional environments under a random policy or using imported data
* **Generate artificial neuronal data** Simulate various location or velocity selective cells found in the Hippocampal-Entorhinal system, or build your own more complex cell type. 
* **Build complex networks** Build, train and analyse complex networks of cells, powered by `RatInABox`. 

<img src="./images/riab.gif" width=1000>

`RatInABox` contains three classes: 

1. `Environment`: The environment/maze (or "box") that the agent lives in. 1- or 2-dimensional.
2. `Agent`: The agent (or "rat") moving around the `Environment`. 
3. `Neurons`: A population of neurons with firing rates determined by the state (position and velocity) of the `Agent`. Make your own or use one of our premade cell types: 
    * `PlaceCells`
    * `GridCells`
    * `BoundaryVectorCells` (egocentric or allocentric)
    * `VelocityCells`
    * `SpeedCells`
    * `HeadDirectionCells`
    * `FeedForwardLayer`

The top animation shows the kind of simulation you can easily run using this toolbox. In it an `Agent` randomly explores a 2D `Environment` with a wall. Three populations of `Neurons` (`PlaceCells`, `GridCells`, `BoundaryVectorCells`) vary their activity and "fire" as the `Agent` explores. 

## Key features

* **Flexible**: Generate arbitrarily complex environments. 
* **Biological**: Simulate large populations of spatially and/or velocity modulated cell types. Neurons can be rate based or spiking. Motion model fitted to match real rodent motion. 
* **Fast**: Simulating 1 minute of exploration in a 2D environment with 100 place cells (dt=10 ms) take just 2 seconds on a laptop (no GPU needed).
* **Precise**: No more pre-discretised positions, tabular state spaces, or jerky movement policies. It's all continuous. 
* **Visual** Plot or animate trajectories, firing rate timeseries', spike rasters, receptive fields, heat maps, velocity histograms...using the plotting functions ([summarised here](./demos/list_of_plotting_fuctions.md)). 
* **Easy**: Sensible default parameters mean you can have realisitic simulation data to work with in ~10 lines of code.
* **General**: Build your own bespoke `Neurons` classes and combine them into complex networks of neurons (example scripts given).


## Get started 

At the bottom of this readme we list [example scripts](./demos/): one simple and one extensive. Reading through these  should be enough to get started. We also provide two case studies where `RatInABox` is used in a [reinforcement learning project](./demos/reinforcement_learning_example.ipynb) and a [path integration](./demos/path_integration_example.ipynb) project. Jupyter scripts reproducing all figures in the [paper](./demos/paper_figures.ipynb) and [readme](./demos/readme_figures.ipynb) are also provided.


## Requirements
* python 3.7+
* numpy
* scipy
* matplotlib
* jupyter (optional)
* tqdm (optional)

## Installing and Importing
Once you have cloned this repository either run 
```python
python setup.py install
```
or just point your code to this directory directly (recommended if you want to meddle with the code)
```python
sys.path.append("path/from/your/code/to/ratinabox/")
```

`RatInABox` also support pip installion
```python
pip install git+https://github.com/TomGeorge1234/ratinabox.git
```

Once installed, import using  
```python
import ratinabox
from ratinabox.Environment import Environment
from ratinabox.Agent import Agent
from ratinabox.Neurons import *
```

## Feature run-down
Here is a list of features loosely organised into three categories: those pertaining to (i) the `Agent`, (ii) the `Agent` and (iii) the `Neurons`. Specific details can be found in the paper, [here](link/to/paper/on/arxiv). 


### (i) `Environment` features
#### Walls 
Arbitrarily add walls to the environment to produce arbitrarily complex mazes:
```python 
Environment.add_wall([[x0,y0],[x1,y1]])
```
Here are some easy to make examples.

<img src="./images/walls.png" width=1000>

#### Boundary conditions 
Boundary conditions can be "periodic" or "solid". Place cells and the motion of the Agent will respect these boundaries accordingly. 
```python
Env = Environment(
    params = {'boundary_conditions':'periodic'} #or 'solid' (default)
) 
```
<img src="./images/boundary_conditions.png" width=600>

#### 1- or 2-dimensions 
`RatInABox` supports 1- or 2-dimensional `Environment`s. Almost all applicable features and plotting functions work in both. The following figure shows 1 minute of exploration of an `Agent` in a 1D environment with periodic boundary conditions spanned by 10 place cells. 
```python 
Env = Environment(
    params = {'dimensionality':'1D'} #or '2D' (default)
) 
```
<img src="./images/one_dimension.png" width=800>



### (ii) `Agent` features

#### Random motion model
Random motion is stochastic but smooth. The speed (and rotational speed, if in 2D) of an Agent take constrained random walks governed by Ornstein-Uhlenbeck processes. You can change the means, variance and coherence times of these processes to control the shape of the trajectory. Default parameters are fit to real rat locomotion data from Sargolini et al. (2006): 

<img src="./images/riab_vs_sargolini.gif" width=600>

The default parameters can be changed to obtain different style trajectories. The following set of trajectories were generated by modifying the rotational speed parameter `Agent.rotational_velocity_std`:

```python
Agent.speed_mean = 0.08 #m/s
Agent.speed_coherence_time = 0.7
Agent.rotation_velocity_std = 120 * np.pi/180 #radians 
Agent.rotational_velocity_coherence_time = 0.08
```
<img src="./images/motion_model.png" width=800>


#### Importing trajectories
`RatInABox` supports importing external trajectory data (rather than using the in built random motion policy). Imported data can be of low temporal resolution. It will be smoothly upsampled using a cubic splines interpolation technique. We provide a 10 minute trajectory from the open-source data set of Sargolini et al. (2006) ready to import. In the following figure blue shows (low resolution) trajectory data imported into an `Agent` and purple shows the smoothly upsampled trajectory taken by the `Agent` during exploration. 
```python
Agent.import_trajectory(dataset='sargolini')
#or 
Agent.import_trajectory(times=array_of_times,
                        positions=array_of_positions)

```
<img src="./images/imported_trajectory.png" width=300>

#### Policy control 
By default the movement policy is an random and uncontrolled (e.g. displayed above). It is possible, however, to manually pass a "drift_velocity" to the Agent on each `update()` step. This 'closes the loop' allowing, for example, Actor-Critic systems to control the Agent policy. As a demonstration that this method can be used to control the agent's movement we set a radial drift velocity to encourage circular motion. We also use RatInABox to perform a simple model-free RL task and find a reward hidden behind a wall (the full script is given as an example script [here](./demos/reinforcement_learning_example.ipynb))
```python
Agent.update(drift_velocity=drift_velocity)
```
<img src="./images/motion.gif" width=600>

#### Wall repelling 
Under the random motion policy, walls in the environment mildly "repel" the agent. Coupled with the finite turning speed this replicates an effect (known as thigmotaxis and linked to anxiety) where the agent is biased to over-explore near walls and corners (as shown in these heatmaps) matching real rodent behaviour. It can be turned up or down with the `anxiety` parameter.
```python 
Αgent.thigmotaxis = 0.8 #1 = high thigmotaxis (left plot), 0 = low (right)
```
<img src="./images/wall_repel.png" height=300>


### (iii) `Neurons` features 

#### Multiple cell types: 
We provide a list of premade `Neurons` subclasses. These include: 

* `PlaceCells(Neurons)` 
* `GridCells(Neurons)`
* `BoundaryVectorCells(Neurons)` (can be egocentric or allocentric)
* `VelocityCells(Neurons)`
* `SpeedCells(Neurons)`
* `HeadDirectionCells(Neurons)`
* `FeedForwardLayer(Neurons)` - calculates activated weighted sum of inputs from a provide list of input `Neurons` layers.

This last class, `FeedForwardLayer` deserves special mention. Instead of its firing rate being determined explicitly by the state of the `Agent` it summates synaptic inputs from a provided list of input layers (which can be any `Neurons` subclass). This layer is the building block for how more complex networks can be studied using `RatInABox`. 

Place cells come in multiple types (give by `params['description']`):
* `"gaussian"`: normal gaussian place cell 
* `"gaussian_threshold"`: gaussian thresholded at 1 sigma
* `"diff_of_gaussian"`: gaussian(sigma) - gaussian(1.5 sigma)
* `"top_hat"`: circular receptive field, max firing rate within, min firing rate otherwise
* `"one_hot"`: the closest palce cell to any given location is established. This and only this cell fires. 

This last place cell type, `"one_hot"` is particularly useful as it essentially rediscretises space and tabularises the state space (gridworld again). This can be used to  contrast and compare learning algorithms acting over continuous vs discrete state spaces. 

#### Geometry of `PlaceCells` 
Choose how you want `PlaceCells` to interact with walls in the `Environment`. We provide three types of geometries.  
<img src="./images/wall_geometry.png" width=900>

#### Spiking 
All neurons are rate based. However, at each update spikes are sampled as though neurons were Poisson neurons. These are stored in `Neurons.history['spikes']`. The max and min firing rates can be set with `Neurons.max_fr` and  `Neurons.min_fr`.
```
Neurons.plot_ratemap(spikes=True)
```
<img src="./images/spikes.png" width="1000">


#### Rate maps 
`PlaceCells`, `GridCells` and allocentric `BoundaryVectorCells` (among others) have firing rates which depend exclusively on the position of the agent. These rate maps can be displayed by querying their firing rate at an array of positions spanning the environment, then plotting. This process is done for you using the function `Neurons.plot_rate_map()`. 

More generally, however, cells firing is not only determined by position but potentially other factors (e.g. velocity or historical effects if the layer is part of a recurrent network). In these cases the above method of plotting rate maps will fail. A more robust way to display the receptive field is to plot a heatmap of the positions of the Agent has visited where each positions contribution to a bin is weighted by the firing rate observed at that position. Over time, as coverage become complete, the firing fields become visible.
```
Neurons.plot_rate_map() #attempted to plot "ground truth" rate map 
Neurons.plot_rate_map(method="history") #plots rate map by firing-rate-weighted position heatmap
``` 
<img src="./images/rate_map.png" width=600>

#### More complex Neuron types and networks of Neurons
We encourage users to create their own subclasses of `Neurons`. This is easy to do, see comments in the `Neurons` class within the [code](./ratinabox/Neurons.py) for explanation. By forming these classes from the parent `Neurons` class, the plotting and analysis features described above remain available to these bespoke Neuron types. Additionally we provide a `Neurons` subclass called `FeedForwardLayer`. This neuron sums inputs from any proived list of other `Neurons` classes and can be used as the building block for constructing complex multilayer networks of `Neurons`, as we do [here](./demos/path_integration_example.ipynb) and [here](./demos/reinforcement_learning_example.ipynb). 

## Example Scripts
In the folder called [demos](./demos/) we provide numerous script and demos which will help when learning `RatInABox`. In approximate order of complexity, these include:
* [simple_example.ipynb](./demos/simple_example.ipynb): a very simple tutorial for importing RiaB, initilising an Environment, Agent and some PlaceCells, running a brief simulation and outputting some data.
* [extensive_example.ipynb](./demos/extensive_example.ipynb): a more involved tutorial. More complex enivornment, more complex cell types and more complex plots are used. 
* [list_of_plotting_functions.md](./demos/list_of_plotting_fuctions.md): All the types of plots available for are listed and explained. 
* [readme_figures.ipynb](./demos/readme_figures.ipynb): (Almost) all plots/animations shown in the root readme are produced from this script (plus some minor formatting done afterwards in powerpoint).
* [paper_figures.ipynb](./demos/paper_figures.ipynb): (Almost) all plots/animations shown in the paper are produced from this script (plus some major formatting done afterwards in powerpoint).
* [reinforcement_learning_example.ipynb](./demos/reinforcement_learning_example.ipynb): RatInABox is use to construct, train and visualise a small two-layer network capable of model free reinforcement learning in order to find a reward hidden behind a wall. 
* [path_integration_example.ipynb](./demos/path_integration_example.ipynb): RatInABox is use to construct, train and visualise a large multi-layer network capable of learning a "ring attractor" capable of path integrating a position estimate using only velocity inputs.

## Contribute 
`RatInABox` is an open source project, and we actively encourage community contributions. These can take various forms, such as new movement policies, new cells types, new plotting functions, new geometries, bug fixes, documentation, citations of relevant work, or additional experiment notebooks. If there is a small contribution you would like to make, please feel free to open a pull request, and we can review it. If you would like to add a new `Neurons` class please pull request it into the [contribs](./ratinabox/contribs/) directory. If there is a larger contribution you are considering please contact the correponding author at `tomgeorge1@btinternet.com`. 

## Cite
If you use `RatInABox` in your research or educational material, please cite the work as follows: `my wicked bibtex citation`
The research paper corresponding to the above citation can be found [here](link/to/my/paper).


