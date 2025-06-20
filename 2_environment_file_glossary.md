# Environment file glossary

## requirements.in
This is a text file maintained by you (the developer - rather than being generated).
List all the runtime dependencies of your project. Just the names. No version
numbers needed. Only list things you actually import, rather than sub dependencies.

```
# requirements.in
pandas
simpy
vidigi
```

## requirements-dev.in
This is a text file maintained by you (the developer - rather than being generated).
List all the development dependencies of your project. These are things that are
useful for working on the project but not actually needed to run the project.
For example `pip-tools` or `ipykernel`. Do not list all your run time dependencies
again - just reference the requirements.in file.

```
# requirements-dev.in

# Include all the runtime dependencies
-r requirements.in

# Add the development-specific tools
pip-tools
pytest
black
ruff
```

## requirements.txt
This is the file actually used by venv to create the environment. It is generated
from the requirements.in file (not manually updated). It contains all the dependencies
and sub-dependencies with version numbers. You will use this when the project
is deployed on a server.

## requirements-dev.txt
This is the same as the above but generated from the requirements-dev.in file.
You will use it (via the .yml) when you create an environment on your development
machine.

## environment.yml
This is the file you use to re-create the conda development environment for the
project. It contains the python version, sets the conda channels, installs pip
and then references the requirements-dev.txt file.

```yml
name: my-project-dev

channels:
  - conda-forge
  - nodefaults #keyword to stop it mixing channels

dependencies:
    # conda dependencies first - then pip dependencies
  - python=3.11
  - pip
  - pip:
    - -r requirements-dev.txt
```

## readme list of runtime system library dependencies
When creating environments locally conda installs any non-python dependencies as
well. Venv will not do this. So you need to know in advance if you have any system
library dependencies and list them in the README. These need to be installed in
advance of installing python packages on the server. You can do this manually
or via docker.

**Example `README.md`:**
```markdown
# Project Installation

## Server Prerequisites

This project requires the following system-level libraries. On a Debian/Ubuntu server, install them with:

```bash
sudo apt-get update && sudo apt-get install -y libgeos-dev
```
