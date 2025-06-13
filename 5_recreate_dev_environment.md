# Recreating a development environment from a .yml

This requires a .yml file being set up, as specified in 8 new project

Navigate Anaconda Powershell Prompt to the location of the .yml file
```bash
# This single command creates the environment with the correct name,
# installs the correct Python version, and installs all conda and pip packages.
conda env create -f environment.yml
```

After its finished it should be available for you to select and activate through VSCode.