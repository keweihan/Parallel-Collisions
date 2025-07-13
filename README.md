# Parallel Collisions
An exercise in parallelizing collision detection and resolution in [SimpleECS](https://github.com/keweihan/SimpleECS) using CUDA. Achieves 3.5x speedup over a sequential implementation at high entity counts. 



https://github.com/user-attachments/assets/c09d6ef4-db78-4ed6-9f24-0912e2562397




## Getting Started
### Prerequsites
- [Python](https://www.python.org/) (>=3.8)
- [Conan](https://conan.io/downloads) 
- [CMake](https://cmake.org/download/)
- [CUDA](https://developer.nvidia.com/cuda-downloads) 

### Setup
1. Run installer helper `python3 scripts/build.py install`

### Build and run
1. Configure scene in `demos/collisionStress.cpp`
2. Build executable `python src/build.py build --type release`
3. Execute runnable in `./build/bin/collisionStress`

*Adapted from CMU Parallel Computer Architechture and Programming capstone project with Jiya Zhang*
