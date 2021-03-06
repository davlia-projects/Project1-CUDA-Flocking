**University of Pennsylvania, CIS 565: GPU Programming and Architecture,
Project 1 - Flocking**

* David Liao
* Tested on: Windows 7, Xeon E5-1630 v4 @ 3.70GHz 32GB, GTX 1070 4096MB (SigLab MOR 103-57)


# Flocking Simulation
![boids](/images/5000boids.gif)

# Performance Analysis
In the following section, we analyze the performance of our 3 kernel functions and algorithms. We use framerate and timers for velocity updates to demonstrate how the three different approaches differ. We take average samples over 1000 iterations for velocity update timers. Velocity calculation timings are highlighted specifically since algorithmic differences are present. The other kernel functions in Coherent and Scattered grid approaches are trivial and embarassingly parallel thus they provide no interesting insight. 

![Table and Charts of Data](/images/tables.png)

##### Naive Implementation
At an algorithmic viewpoing, each boid requires an additional check for N-1 other boids , thus increasing the number of boids will increase computation type by a strictly N^2 factor. From a low-level perspective, the number of boids that can be computed in parallel is limited by the number of threads that the hardware has (any more would require boids than the number of threads would require multiple stages to calculate position updates all of them). Since the access for each boid would be all the other boids, the locality of the data is poor leading to access time issues since data access is wide ranged. Both the algorithmic and low level issues are reflected in the data as expected. We can see that the velocity calculation takes up the majority of the time per boid and grows by the same factor that the frame rate drops. 

##### Scattered with uniform grid
The algorithmic speed up is constant but still polynomial. At a low-level, the boid look-up is significantly reduced but once again the locality of data is poor. In addition, the data access is quite "jumpy" and would result in a lot of fetching (if there's some form of pagination in the GPU) once the number of boids increases. Though it is interesting to note that from 5k to 10k boids there is an increase in frames but also an increase in the time it took to calculate velocity updates. The performance drop past that point is expected as we can see the toll that unstructured data takes. Beyond 80k we see a sharp drop in performance. Some low-level swapping must occur at this point and causes a performance drop. From the combined point of view, the more dense the grids are, the more boids we have to inspect (thus further emphasizing the importance of spatial locality). 

##### Coherent with uniform grid
The algorithmic speed up is constant but still polynomial. At a low-level, the boid look-up is significantly reduced and the locality of data is much better than the scattered uniform grid approach. The Coherent grid approach has a linear growth rate w.r.t. the velocity update timings since it is not affected by these locality issues. However, increasing the number of boids beyond a certain point might cause buffer issues as a block might require to fetch vector buffers that exceed its memory capacity. This problem is similar to the one faced by the Scattered grid approach. The performance improvements were expected since the structured placement of data meant less fetching. Though I was quite surprised by the magnitude of improvement since we went from exponential increase in velocity update timings to a linear increase (wow).

##### On the topic of block count and block sizes
Changing the block count and size did not affect performance. This is due to the fact that the problems we are trying to solve is embarassingly parallel. Changing the block size should not change the fact that each thread has to perform a velocity update, position update, etc. In other words, no task would block another task and thus no computational latency occurs (yay no stalls!). In this case, changing block size or count does not affect performance as expected.
