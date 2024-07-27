---
name: Flocking Simulation
tools: [CUDA, C++]
image: ../img/project_img/flocking_simulation/preface.gif
description: This project implemented a flocking simulation based on the Reynolds Boids algorithm with two levels of optimization - scattered uniform grid, and coherent uniform grid optimization.
---

# Flocking Simulation

[GitHub repo](https://github.com/TongHuoAo/Project1-CUDA-Flocking)

![](../img/project_img/flocking_simulation/preface.gif)

This project implemented a flocking simulation based on the Reynolds Boids algorithm with two levels of optimization: scattered uniform grid, and coherent uniform grid optimization.

All features are tested on: **Windows 11, Ryzen 7 1700X @ 3.4GHz 16GB, GTX 1080 16GB**


## Boids Simulation
-------
### Basic Implementation
In the flocking simulation, boids (which represent birds or fish) move within the space should follow these 3 basic rules:

* Cohesion: Boids move toward the perceived center of mass of their neighbors.
* Separation: Boids maintain a certain distance to avoid getting too close to their neighbors.
* Alignment: Boids generally align their direction and speed with those of their neighbors.

These rules combined together determine the velocity change for each boid at every timestep. In other words, each boid needs to refer to nearby boids within specified distance to calculate its velocity change at every timestep.

### Optimization
#### Uniform Grids
In the basic implementation, every boid has to check each other boid in each timestep, which is very inefficient. 

Uniform grids now introduced to make it more efficient. A uniform grid divided the entire simulation domain into cells with side as the neighborhood distance. Then, each boid can only check boids in neighbor grids instead of traversing all other boids. 

![](../img/project_img/flocking_simulation/Boids Ugrid neighbor search shown.png)

This can be easily done on CPU, but doesn't transfer well on GPU since:
* We don't have resizeable arrays on the GPU
* Naively parallelizing the iteration may lead to race conditions, where two particles need to be written into the same bucket on the same clock cycle.

Instead, we will construct the uniform grid by sorting. If we label each boid with an index representing its enclosing cell and then sort the list of boids by these indices, we can ensure that pointers to boids in the same cells are contiguous in memory.

![](../img/project_img/flocking_simulation/Boids Ugrids buffers naive.png)

Then, we can rearrange the boid data so that all velocities and positions of boids in the same cell are also continuous in memory.

![](../img/project_img/flocking_simulation/Boids Ugrids buffers data coherent.png)

## Result Visualization
-------
Figure 1. Flocking simulation with 5000 boids.
![Flocking simulation with 5000 boids](../img/project_img/flocking_simulation/boid5000.gif)

Figure 2. Flocking simulation with 20000 boids.
![Flocking simulation with 20000 boids](../img/project_img/flocking_simulation/boid20000.gif)

Figure 3. Flocking simulation with 50000 boids.
![Flocking simulation with 50000 boids](../img/project_img/flocking_simulation/boid50000.gif)

## Performance Analysis
-------
### For each implementation, how does changing the number of boids affect performance? Why do you think this is?
Figure 4. Framerate vs. Number of boids.
![Framerate vs. Number of boids](../img/project_img/flocking_simulation/framerate_vs_number__of_boids.png)

This analysis is done with block size as 128. 

As shown in Figure 4, we can tell that for each implementation, with or without visualization, when the number of boids increased, the framerate decreased. This make sense since when the number of boids increased, the number of neighbors used for updating velocities and positions increased as well, which cost more time to calculate.

Also, we can notice from the figure that when the number of boids is small (smaller than 5000), the performance of naive implementation is better than uniform grid. Since there are not much boids and distribution of boids is very scattered, it is not necessary to find out which grid should be included when calculating velocities and positions, which introduced unnecessary overhead. 

As the number of boids increased (greater than 5000), the naive implementation have to traverse all boids and compare the distance to determine which boid should be included. This calculation and comparisons are not necessary for most boids, while split the spaces into grid can prevent doing these unnecessary work, which can improve the performance of simulation. 


### For each implementation, how does changing the block count and block size affect performance? Why do you think this is?
Figure 5. Framerate vs. Block size.
![Framerate vs. Block size](../img/project_img/flocking_simulation/framerate_vs_block_size.png)

This analysis is done in number of boids as 20000.

From Figure 5 we can tell that when the block size is smaller than 32, increasing block size improved the performance for all three implementations, while the block size is greater than 32, the performance became stable (almost not change) as the block size increased. 

This is related to the warp size, which defines the maximum number of threads can be contained in a warp, and a warp is the smallest unit for scheduling. In my GPU, the warp size is 32 threads. If the block size is smaller than 32, the remaining threads in the warp remain idle but occupy scheduling resources, which cause the whole process take more time. When the block size is greater than 32 threads, all threads are in use, and it will not be a bottleneck of the performance.


### For the coherent uniform grid: did you experience any performance improvements with the more coherent uniform grid? Was this the outcome you expected? Why or why not?

Based on Figure 4, we can tell that the performance improved with more coherent uniform grid compared with scattered uniform grid, especially when the number of boids is large. This improvement matched the expectation since instead of getting velocity and position data by randomly access buffers, we rearranged the buffer so that the data of boids which located closely in the same grid are put together. Hence the memory access is more efficient due to spatial locality, which improve the performance by reducing the number of random access to memory.


### Did changing cell width and checking 27 vs 8 neighboring cells affect performance? Why or why not?

I expect checking 27 neighboring cells with cell width as the distance for checking rules will improve the performance of simulation, especially when the number of boids is large. When we are checking 8 neighboring cells with cell width as $2$ times the distance, we are checking the boids within $(4)^3 = 64$ unit space. But if we changing the cell width to $r$ and check 27 cells, we are checking the boids within $(3)^3 = 27$ unit space, which is actually smaller than previous. Although checking more cells will introduce some extra overheads (sorting, determine cells and some other overheads), they are negligible compared with the improvement. Thus, I expect checking 27 neighboring cells will improve the performance.