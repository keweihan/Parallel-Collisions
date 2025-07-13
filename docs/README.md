# Parallelization Schemes for Collision Detection
**Team**: Kewei Han (keweihan), Jiya Zhang (jiyaz)

# Reports
- [Final Report](./Final_Report.pdf)
- [Final Poster](./Final_Poster.pdf)

# Overview
## Summary
Exploration of parallelizing collision detection schemes using CUDA.

## Background
Our project focuses on parallelizing a compute-intensive 2D collision detection application involving simulation of 15,000+ moving particles and their collisions. 

In the most basic implementation of this scheme, in every frame the boundaries of each particle can be checked against every other particle in the entire scene to determine which particles collide.

As an optimization of this process, a fixed-grid spatial structure can be constructed every frame such that a 2D scene is divided evenly into coarse grid cells. Each grid cell represents an area in the 2D scene and is a container to references to particles in that area. After such a structure is constructed, the amount of collision checks needed can be drastically reduced to the amount of collision checks needed by only checking collisions between particles in a given cell. 

<p align="center">
  <img src="https://github.com/user-attachments/assets/86e3e9a5-7d4a-462c-b5a5-b26f15899953" alt="FA604B35-8263-480C-8785-D32AC5C731B9" width="250">
</p>
<p align="center"><em>Figure 1: visualization of fixed spatial grid structure</em></p>

Below gives pseudocode for a sequential version of the entire resolution process, showing the grid construction step as well as the collision resolution step.

```C++
// Construct grid 
void ColliderGrid::updateGrid() {
   for (int i = 0; i < entities.size(); ++i) {
       // Determine grid cells entity overlaps with
       // Insert entity into respective cell
   }
}

void resolveCollisions(ColliderGrid& colliderGrid) {
 // update grid with collider/entity references
 colliderGrid.updateGrid();
 
 // Iterate through every cell and resolve collisions
 for (int i = 0; i < colliderGrid.size(); ++i)
 {
    // Only brute force check colliders contained within each cell
    for (auto& colliderA : colliderGrid.getCellContents(i))
    {
       for (auto& colliderB : colliderGrid.getCellContents(i))
       {
          // Resolve collision and entity velocities
       }
    }
   }
}

int main() {
    Entity[] entities;
    ColliderGrid colliderGrid(entities);
  
    // Main simulation loop
    while(true)
    {
        // resolve collisions for this frame
        resolveCollisions(colliderGrid);
    }
}
```
Since this method splits a single space into multiple spaces that are, with exceptions, largely independent of each other, there is opportunity for significant parallelization with this spatial grid. 

Our project proposes exploring parallelizing over the **resoloution** step where checks are conducted within each cell in order to improve the performance of this application. From a high level, we assume that we are provided with some preconstructed grid structure, and to parallelize detection each thread can be assigned a cell in which to conduct checks and update velocities if a collision occurs. 

## Challenge
Successful parallelization of the collision detection step for good speedup can be challenging for a number of reasons. We anticipate the following key issues:
- Workload Imbalance: Unequal particle distribution across grid cells will result in some threads having more work than others. 
- Memory Access Patterns: Parallel updates to the entity velocities introduces risks of data races and conflicts when particles span multiple grid cells or quadrants. For instance, a particle can be colliding with two other particles each in different cells resulting in conflicting accesses to the original particle velocity.

One preliminary solution to the issue of workload imbalance is replacing a fixed grid with a quadtree - a spatial structure that maintains variable size grid cells such that each cell contains at most a fixed number of particles. This way we can parallelize over cells that have similar number of internal entities for a more balanced workload. This will also be a key exploration topic of our project.

<p align="center">
  <img src="https://github.com/user-attachments/assets/60e0c373-51bf-45dc-9d7e-0ca33fbfa80f" alt="FA604B35-8263-480C-8785-D32AC5C731B9" width="250">
</p>

However, this solution presents it's own challenges.
- Algorithmic Complexity with Quadtrees: While quadtree integration could reduce collision detection costs with unbalanced particle distribution, a quadtree must be implemented such that parallelization over execution of its elements are possible and efficient. 

By exploring parallelization of collision resolution with different spatial structures we hope to gain better understanding for how to implement more generalized solutions that perform well for different workload scenarios.

## Resources
Some basic resources we are considering are
- Hardware: GHC Machines (GPUs and multicore CPUs); personal Windows operating system.
- Starter Code: https://github.com/keweihan/SimpleECS/tree/main (A basic sequential 2D particle simulator implemented in C++).
- Quadtree-related Resources:
  - https://edwardsjohnmartin.github.io/publications/papers/morrical2017parallel.pdf
  - https://github.com/pvigier/Quadtree

## Code Overview
### Data Structures

**General entity-component system overview:**
- An `Entity` can have multiple `Component`. 
- The type of `Component` an entity has dictates behavior of the entity. 
- The behavior a Component adds to an entity is implemented in `Component::update()` and `Component::onCollide()` functions. 

**Key components for collision:**

`Collider` is the component for **bounds** information for a given entity (i.e. how wide is it, where can it collide) and gives an object the ability to collide.
- `include/Collider.h`
- `include/BoxCollider.h`


`PhysicsBody` defines the container of **physics** information for a given entity as well as defines logic for velocity updates in `onCollide()`. Requires Collider for collision logic.
- `include/PhysicsBody.h`

#### Collision Algorithm
Collision data structures have the following hierarchy
```
ColliderSystem --has one-->  ColliderGrid
ColliderGrid -has many-> ColliderCell
ColliderCell --has many-> Entities (Balls)
Ball (Entity) --has one -->  Components (BoxCollider, PhysicsBody)
```

Relevant files:
- `src/Collision/ColliderSystem.h`
- `src/Collision/ColliderCell.h`
- `src/Collision/ColliderGrid.h`

