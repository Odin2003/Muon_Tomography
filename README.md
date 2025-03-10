# Muon-Tomography
Code to run muon tomography Geant4 simulations on the HPCC.

## Environment

Follow same setup for DREAM simulation:

**On HPCC**, everything (ROOT and GEANT4) is compiled inside the singularity environment. Log into the interactive node ([more information](https://www.depts.ttu.edu/hpcc/userguides/Job_User_Guide.pdf)) with e.g.

```
interactive -p nocona
```
From there run the singularity container with the following command:
```
singularity run --cleanenv --bind /lustre:/lustre /lustre/work/yofeng/SimulationEnv/alma9forgeant4_sbox/
```
The corresponding docker image can be found [here](https://hub.docker.com/repository/docker/yongbinfeng/alma9geant/general), with the build file [here](https://github.com/TTU-HEP/SimulationEnv).

**If running with Docker** on your local machine, firstly pull the docker image
```
docker pull yongbinfeng/alma9geant:latest
```
Then run the container with the following command:
```
docker run -it --rm -h dreamsim -v /path/to/DREAMSIM:/DREAMSIM yongbinfeng/alma9geant:latest
```
**Note** if you have conda installed, exit the conda environment before running the singularity container, otherwise it might cause conflicts with different ROOT versions etc.

## Running the Simulation

### Structure of software:

- `sim/exampleB4a.cc`: main program
- `sim/src/B4DetectorConstruction.cc`:setup of the cairn, hill, and camera
- `sim/include/TessellatedVertices.hh`: Definition of vertices for cairn and hill
- `sim/include/TessellatedGeometry.hh`: Arranging vertices to form cairn and hill
- `sim/src/TessellatedGeometry.cc`: Used in B4DetectorConstruction.cc to implement cairn and hill
- `sim/src/B4aSteppingAction.cc`:access hits at each step
- `sim/src/MuonTree.cc`:analysis and hit handling

### Compiling:

Inside the singularity environment, build program in "build" area,
```
cd /path/to/sim/directory
cd sim
mkdir build
cd build
cmake ..
make -j 4
```

### Running the code:

Move a macro file from `sim/scripts' to the build area. C0 corresponds to a detector position on the surface just outside of the cairn. C3 is buried 10m underground. RAA means the cairn is made of Rock2, the chamber of air and the passage (currently not implemented) also of air. RRR means all are made of Rock2 for the comparison runs. Here is the positioning:

![Image](https://github.com/user-attachments/assets/0442c105-2bed-488e-9482-267b1133ecee)

Then run the code using (in the build directory)

``` 
./exampleB4a  -b batch_run_C0RAA_01.mac -numberOfEvents 1000 -runNumber 987 -runSeq 99
```

Edit the number of events and other parameters as needed. The macro file can be used to change parameters such as at what angles muons are generated, materials of the chamber/hill, camera positioning and so on. This works well to debug with low number of events (<50 000). To run many events, see job submission.


### Job submission:
To submit large amounts of events, use jobSumission.py in the "jobs" directory. Change the run_cmd for different macro files. Total number of events is njobs*nevents_per_job. When the file has been edited to your preferences, run

```
python jobSubmission.py
```

(Make sure to change directory locations in jobSubmission.py to your build directory) and 

```
source submit_all.sh
```

to submit the jobs. Submission, error and output files are kept in the log directory that will be created when running jobSubmission.py.

Output of root files for analysis is in the build directory. 


## Visualization

If you want to visualize the simulation, this will have to be done locally. Set up Geant4 with QT, OpenGL and GDML. In the cmake step of Geant4 configuring use

```
cmake -DCMAKE_INSTALL_PREFIX=../geant4-install \
      -DGEANT4_INSTALL_DATA=ON \
      -DGEANT4_USE_QT=ON \
      -DGEANT4_USE_GDML=ON \
      -DGEANT4_USE_OPENGL_X11=ON \
      -DGEANT4_USE_RAYTRACER=ON \
      -DQt5_DIR=$(brew --prefix qt5)/lib/cmake/Qt5 \
      ..
```

Then 
```
make
```

Now you can run in the same way as before, locally, creating again a build directory within sim and compiling the code with cmake .. and make. Then you can run it using 

```
./exampleB4a -i batch_run_C0RAA_01.mac
```

This will launch a QT window where you can run one event or multiple muons using 

```
/run/beamOn 100
```

Visualized in Geant4, the geometry looks like

<img width="896" alt="image" src="https://github.com/user-attachments/assets/ca1b8884-3566-4aa6-8e87-99e540688110" />

Here is a muon getting "stuck" in the cairn:

![image](https://github.com/user-attachments/assets/1a316f86-49ca-47e3-931e-d60e3dd14da2)


## Analysis

For an analysis code, see os_analysis.ipynb. This plots basic histograms and the ratio of air to rock runs.
