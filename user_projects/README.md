# README

## Description
The individual folders herein represent the simulations used in the paper.
The three `well-mixed-#` folders correspond to the three stochastic realizations of the well-mixed initialization.
Similarly, for the three `structured-#` folders.
The one `spatial-informed` folder corresponds to the unique spatial-informed initialization from the spatial transcriptomics data.

## PhysiCell Setup
To set up a PhysiCell directory and otherwise be ready to run PhysiCell models, see the OS-specific [PhysiCell installation instructions](https://github.com/physicell-training/ws2023/blob/main/agenda.md).

## Manual running
You may copy any of these folders into the `user_projects` folder of a PhysiCell v1.14.2 directory.
Run the projects with the following commands from a shell within the PhysiCell directory:
```bash
make load PROJ=well-mixed-1 # replace with desired project
make # add -j 4 flag to parallelize the compilation
```
If on Windows, execute the project with:
```bash
project.exe
```
If using macOS or Linux, run the project with:
```bash
./project
```

## Automated running with Julia and pcvct
To run all 9 simulations in an automated manner, you can use the [pcvct.jl](https://github.com/drbergman/pcvct) package.
Follow the steps in the [documentation](https://drbergman.github.io/pcvct/stable/man/getting_started/) to get set up with Julia and pcvct.
In particular, once you have a pcvct project set up and have run `initializeModelManager()` on the project, you can call `importProject("./path/to/user_projects/well-mixed-1")`, for example, to import the project.
Note the terminal output that will show the location of the imported folders.
Then, using the `VCT/GenerateData.jl` script, you can change the folders to point to the newly-imported folders:
```julia
using pcvct

@assert isdir("data") && isdir("PhysiCell") && isdir("VCT") "Make sure you are in the right directory to intialize the model manager"
initializeModelManager()

# importProject(joinpath("path", "to", "user_projects", "well-mixed-1")) # only needed once

config_folder = custom_code_folder = rules_folder = ic_cell_folder = "well-mixed-1" # they should all go in folders with this name for the well-mixed-1 project

inputs = InputFolders(config_folder, custom_code_folder;
    rulesets_collection=rules_folder,
    ic_cell=ic_cell_folder)

out = run(inputs) # will run one replicate
```

You can also easily run all 9 simulations at once:
```julia
using pcvct

@assert isdir("data") && isdir("PhysiCell") && isdir("VCT") "Make sure you are in the right directory to intialize the model manager"
initializeModelManager()

projects = ["well-mixed-1", "well-mixed-2", "well-mixed-3", "structured-1", "structured-2", "structured-3", "spatial-informed"] # to best match the results of the paper, it may be better to only use one each of the mixed and structured projects
# path_to_projects = [joinpath("path", "to", "user_projects", project) for project in projects]
# importProject.(path_to_projects) # import them all in one line (only needed once; will create duplicate folders to safely import)

inputss = [InputFolders(project, project;
    rulesets_collection=project,
    ic_cell=project) for project in projects]

monads = [createTrial(inputs; n_replicates=3) for inputs in inputss] # will prepare a "monad" for each, which just means a collection of replicates (3 in this case)
trial = Trial(monads) # bundle them all into a single Trial object
out = run(trial) # will run all replicates (21 in this case since each of the 6 random initializations has 3 replicates)
```