# SafePlan

SafePlan is an optimal temporal logic planning tool for multi-robot systems in uncertain semantic maps [1]. SafePlan generates safe multi-robot plans for collaborative high-level tasks captured by global temporal logic specifications in the presence of uncertainty in the workspace. The workspace is modeled as a semantic map determined by Gaussian distributions over landmark positions and arbitrary discrete distributions over landmark classes. tTo specify mission and safety requirements, we employ co-safe Linear Temporal Logic defined over perception-based predicates. This allows us to incorporate uncertainty and probabilistic satisfaction requirements directly into the task specification. SafePlan is a highly scalable sampling-based approach that simultaneously searches the semantic map along with an automaton corresponding to the task and synthesizes paths that satisfy the assigned task specification. 
The paths returned by the planner [biased_TLRRT_star.py](rotors_simulator/rotors_gazebo/scripts/biased_TLRRT_star.py) can be visualized in the simulation environment. This simulation 
uses the RotorS ROS package [RotorS](https://github.com/ethz-asl/rotors_simulator) which is a MAV gazebo simulator and the BebopS ROS package 
[BebopS](https://github.com/gsilano/BebopS), which is an extension of the ROS package RotorS.
The simulation integrates feedback in the control to update the estimates of the landmark positions and thus reformulate the paths in real-time to unsure satisfaction of the task specifications.


[1] Y. Kantaros and G. J. Pappas, "Optimal Temporal Logic Planning for Multi-Robot Systems in Uncertain Semantic Maps," 2019 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS), Macau, China, 2019, pp. 4127-4132, doi: 10.1109/IROS40897.2019.8968547.


# Requirements for SafePlan
* [Python >=3.6](https://www.python.org/downloads/)
* [sympy](https://www.sympy.org/en/index.html)
* [re]()
* [Pyvisgraph](https://github.com/TaipanRex/pyvisgraph)
* [NetworkX](https://networkx.github.io)
* [Shapely](https://github.com/Toblerity/Shapely)
* [scipy](https://www.scipy.org)
* [matplotlib](https://matplotlib.org)
* [termcolor](https://pypi.org/project/termcolor/)
* [visilibity](https://github.com/tsaoyu/PyVisiLibity)

## Additional requirements for simulation
* [gazebo2rviz](https://github.com/andreasBihlmaier/gazebo2rviz)
* [pysdf](https://github.com/andreasBihlmaier/pysdf.git)

# Usage
## Structures
* Class [Task](rotors_simulator/rotors_gazebo/scripts/task.py) defines the task specified in LTL
* Class [Workspace](rotors_simulator/rotors_gazebo/scripts/workspace.py) define the workspace where robots reside
* Class [Landmark](rotors_simulator/rotors_gazebo/scripts/workspace.py) define the landmarks in the workspace
* Class [Buchi](rotors_simulator/rotors_gazebo/scripts/buchi_parse.py) constructs the graph of NBA from LTL formula
* Class [Geodesic](rotors_simulator/rotors_gazebo/scripts/geodesic_path.py) constructs geodesic path for given environment
* Class [BiasedTree](rotors_simulator/rotors_gazebo/scripts/biased_tree.py) involves the initialization of the tree and relevant operations
* Function [construction_biased_tree](rotors_simulator/rotors_gazebo/scripts/construct_biased_tree.py) incrementally grow the tree
* Script [biased_TLRRT_star.py](rotors_simulator/rotors_gazebo/scripts/biased_TLRRT_star.py) contains the main function
* Functions [path_plot](rotors_simulator/rotors_gazebo/scripts/draw_picture.py) and [path_print](draw_picture.py) draw and print the paths, respectively
* Functions [export_disc_to_txt](rotors_simulator/rotors_gazebo/scripts/export_disc_to_text.py) and [export_cov_to_txt](rotors_simulator/rotors_gazebo/scripts/export_cov_to_text.py) export discretized waypoints and covariance at waypoints, respectively
* Functions [kf_update](rotors_simulator/rotors_gazebo/scripts/kf.py) and [ekf_update](rotors_simulator/rotors_gazebo/scripts/ekf.py) update position and cavariance using kalman filter and extended kalman filter respectively

## Basic procedure
* First, in the class [Workspace](rotors_simulator/rotors_gazebo/scripts/workspace.py) specify the size of the workspace, the layout of landmarks and obstacles, and the covariance associated with each landmark. Also specify the number of classes and the class distribution.
* Then, specify the LTL task in the class [Task](rotors_simulator/rotors_gazebo/scripts/task.py), which mainly involves the assigned task, the number of robots, the initial locations of robots and the minimum distance between any pair of robots, and workspace in the class [Workspace](rotors_simulator/rotors_gazebo/scripts/workspace.py) that contains the information about the size of the workspace, the layout of regions and obstacles. If manual initiation is set to False, then the robots will be initated at random locations. 
* Select the desired dynamics in the [sample_control_to_target]() funtion in the class [BiasedTree](rotors_simulator/rotors_gazebo/scripts/biased_tree.py).
* Set the parameters used in the TL-RRT* in the script [biased_TLRRT_star.py](rotors_simulator/rotors_gazebo/scripts/biased_TLRRT_star.py), such as the maximum number of iterations, the step size, sensor range, sensor noise. 
* Specify location of folder where the waypoints should be saved (resources folder in the rotors_gazebo package).


Basic Usage of simulation
---------------------------------------------------------

Running the simulation is quite simple, so as customizing it: it is enough to run in a terminal the command

```console
$ roslaunch rotors_gazebo ltl_sim.launch 
```
> **Note** For the first run you will need to update the starting positions of the robots in the launch file. This position should be the same starting positions as mentioned in the LTL task in the class [Task](rotors_simulator/rotors_gazebo/scripts/task.py). Alternatively if you run the program [biased_TLRRT_star.py](rotors_simulator/rotors_gazebo/scripts/biased_TLRRT_star.py) from the terminal before launching the simulation, the launch file will be automatically updated.

The ltl_sim.launch file  launches a gazebo environment and an RViz environment. Once the Gazebo environment is unpaused, run the following line in the terminal.
```console
$ rosrun rotors_gazebo online_planner.py 
```
The [online_planner.py](rotors_simulator/rotors_gazebo/scripts/online_planner.py) file is used to calculate the path of the robots. It uses the feedback of the cameras mounted on each of the drones to update the estimates of the landmarks. It then determines if replanning is necessary to satisfy the LTL condition.
You can also visualize the simulation in RViz. Markers display the estimated positions of the landmark, the current position of each robot and the path traced by each robot. Add visualization_marker to display the Gazebo world in RViz. 



# Example
### Offline Input
1) Workspace estimate, robot dynamics, and algorithm parameters.
2) LTL formula
For example the task involving one robot is specified by 
```python
self.formula = '<> e2  && []!e1' 
self.subformula = {1: ['(l1_1)',0,0.8,1.5, 0],
                    2: ['(l2_2)',0,0.8,1.5, 0], 
                  }
robot_initial_pos = ((102,128),)  # in the form of ((x,y), (x,y), ...)
```
Refer [this README](rotors_simulator/rotors_gazebo/scripts/README.md) for more details on setting up the offline parameters for SafePlan algorithm.

### Online Input
1) Semantic map provided by SLAM algorithms.
2) Sensor feedback to determine landmark positions. 

### Output
Simulation (Gazebo and RViz) that uses feedback to update the estimates of the landmark positions and thus reformulate their trajectories in real-time to meet the task specifications.

[![SafePlan Simulation](Sim_image.png)](https://youtu.be/RCjGMC4ZuRk "SafePlan simulation")



# User-defined 
* User can replace the [generate_nn_output](rotors_simulator/rotors_gazebo/scripts/neural_net.py) function with their neural network. The output of this network is used to update the landmark estimates and class distributions. User can also change the [update_landmark_estimates](rotors_simulator/rotors_gazebo/scripts/online_planner.py) and the [update_class_distribution](rotors_simulator/rotors_gazebo/scripts/online_planner.py) functions as per requirement. The current fucntions use a kalman filter and a bayes filter respectively to update the probability distributions. 
* User can use any Gazebo world as per requirement. If a new world is used update the landmark estimates and class distributions in the class [Workspace](rotors_simulator/rotors_gazebo/scripts/workspace.py)




# Installation Instructions - Ubuntu 18.04 with ROS Melodic and Gazebo 9
To use the code developed and stored in this repository some preliminary actions are needed. They are listed below.

1. Install and initialize ROS Melodic desktop full, additional ROS packages, catkin-tools, and wstool:

```console
$ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
$ sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
$ sudo apt update
$ sudo apt install ros-melodic-desktop-full ros-melodic-joy ros-melodic-octomap-ros ros-melodic-mavlink
$ sudo apt install python-wstool python-catkin-tools protobuf-compiler libgoogle-glog-dev ros-melodic-control-toolbox
$ sudo rosdep init
$ rosdep update
$ echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
$ source ~/.bashrc
$ sudo apt install python-rosinstall python-rosinstall-generator build-essential
```

2. If you don't have ROS workspace yet you can do so by

```console
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws/src
$ catkin_init_workspace  # initialize your catkin workspace
$ cd ~/catkin_ws/
$ catkin init
$ cd ~/catkin_ws/src
$ git clone git@github.com:samarth-kalluraya/SafePlan_simulation-.git													
$ cd ~/catkin_ws
```

3. Build your workspace with `python_catkin_tools` (therefore you need `python_catkin_tools`)

```console
$ rosdep install --from-paths src -i
$ catkin build
```

4. Add sourcing to your `.bashrc` file

```console
$ echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
$ source ~/.bashrc
```

5. Update the pre-installed Gazebo version. This fix the issue with the `error in REST request for accessing api.ignition.org`

```console
$ sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
$ wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
$ sudo apt update
$ sudo apt install gazebo9 gazebo9-* ros-melodic-gazebo-*
$ sudo apt upgrade
```

> In the event that the simulation does not start, the problem may be related to Gazebo and missing packages. Therefore, run the following commands. 
```console
$ sudo apt-get remove ros-melodic-gazebo* gazebo*
$ sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
$ wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install gazebo9 gazebo9-* ros-melodic-gazebo-*
$ sudo apt upgrade
```







