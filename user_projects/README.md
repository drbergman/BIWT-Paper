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

View results using [PhysiCell-Studio](https://github.com/PhysiCell-Tools/PhysiCell-Studio).

## Automated running with Julia and pcvct
To run all 9 simulations in an automated manner, you can use the [pcvct.jl](https://github.com/drbergman/pcvct) package.

### Install Julia and pcvct
Follow the (three!) steps in the [documentation](https://drbergman.github.io/pcvct/stable/man/getting_started/) to get install Julia and pcvct.
1. [Install Julia](https://julialang.org/install/)
2. Add the PCVCTRegistry: `] registry add https://github.com/drbergman/PCVCTRegistry`
3. Add the pcvct package: `] add pcvct`

### Create a pcvct project
After installing pcvct, create a pcvct project (if you didn't get there in the setup guide).
First, launch Julia from the terminal:
```sh
julia
```
Then, in the Julia REPL, run:
```julia-repl
julia> using pcvct
julia> createProject() # this will use the current directory as the project directory
julia> # createProject("path/to/pcvct_project") # this will create a new project in the specified directory
```

### Import the PhysiCell projects
Change directory into the pcvct project you just created.
If still in the Julia REPL, you can enter `;` to enter shell mode and then `cd` to change directories as you would in a terminal.
Be sure to return to the Julian mode by entering `delete` or `backspace`.

Now that you are in the pcvct project directory, you can import the PhysiCell projects.
Within Julia, you can run the following commands to import the projects.
Replace `joinpath("path", "to", "user_projects", project)` with the path to the project you want to import either relative to the current directory or the absolute path.
```julia-repl
julia> using pcvct
julia> @assert isdir("data") && isdir("PhysiCell") && isdir("VCT") "Make sure you are in the right directory to intialize the model manager"
julia> initializeModelManager()
julia> projects = ["well-mixed-1", "well-mixed-2", "well-mixed-3", "structured-1", "structured-2", "structured-3", "spatial-informed"]
julia> path_to_projects = [joinpath("path", "to", "user_projects", project) for project in projects]
julia> importProject.(path_to_projects) # import them all in one line (only needed once; will create duplicate folders to safely import)
```
If you want to import a single project, you can run:
```julia-repl
julia> importProject(joinpath("path", "to", "user_projects", "well-mixed-1"))
```

### Run the simulations
You can now run the simulations with the following script either running in the REPL or launching with a script as `julia script_name.jl`.
```julia
using pcvct
@assert isdir("data") && isdir("PhysiCell") && isdir("VCT") "Make sure you are in the right directory to intialize the model manager"
initializeModelManager()

projects = ["well-mixed-1", "well-mixed-2", "well-mixed-3", "structured-1", "structured-2", "structured-3", "spatial-informed"] # to best match the results of the paper, it may be better to only use one each of the mixed and structured projects
inputss = [InputFolders(project, project;
    rulesets_collection=project,
    ic_cell=project) for project in projects]
monads = [createTrial(inputs; n_replicates=3) for inputs in inputss] # will prepare a "monad" for each, which just means a collection of replicates (3 in this case)
trial = Trial(monads) # bundle them all into a single Trial object
out = run(trial) # will run all replicates (21 in this case since each of the 6 random initializations has 3 replicates)
```

If [PhysiCell-Studio](https://github.com/PhysiCell-Tools/PhysiCell-Studio) is installed, it can be set up to view these results.
See the [pcvct documentation](https://drbergman.github.io/pcvct/stable/man/physicell_studio/).