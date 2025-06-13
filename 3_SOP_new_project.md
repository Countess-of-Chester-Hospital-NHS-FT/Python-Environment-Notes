# New development project workflow

When you start a new project and you don't know what packages you might need
use the following workflow.

### Create a new conda environment for the project and activate

```bash
    # Create an environment 
    conda create -n my-data-project python=3.10 

    # Activate it
    conda activate my-data-project
```

### Conda install pip

```bash
    # (my-data-project) >
    conda install pip
```

### Pip install any packages you need as you go, including pip-tools
Activate the environment before you install anything into it. 

Note - it is better to
either install packages this way (as you go through anaconda powershell prompt), 
or install everything from a.yml file. If you do a bit of both you might end up
with a conda channel clash (powershell uses defaults, .yml may use conda-forge)
and some packages may not work.

```bash
    # (my-data-project) >
    pip install pip-tools
```

### List any package you import in the relevant requirements.in file as you go. 
If it is a runtime dependency it goes in requirements.in, if it is a development
environment dependency it goes in requirements-dev.in.

```
# requirements.in
pandas
simpy
vidigi
```

```
# requirements-dev.in

# Include all the runtime dependencies
-r requirements.in

# Add the development-specific tools
pip-tools
pytest
ipykernel
```

### Create the requirements.txt files using pip-tools
When you have finished your development (or at any point at which you want to
be able to share the code), create the requirements.txt and requirements-dev.txt
files using pip-tools. Run these commands in anaconda powershell prompt. Make
sure the environment is activated and move the location of the prompt to the
root folder where your requirements.in file lives using `cd`. 

```bash
    # This reads requirements.in and generates requirements.txt
    pip-compile requirements.in

    # This reads requirements-dev.in and generates requirements-dev.txt
    pip-compile requirements-dev.in
```

### Edit the .yml file
Specify the version of python, the conda channel priority, list the pip dependency
then reference the requirements-dev.txt file.

**`environment.yml`**
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
### Notes on where these files should live
In simpler projects (where you have one 'product' per repo) these files should
live in the root repository as follows:

```
my_project/
├── .git/
├── .gitignore
├── venv/                 # This directory is in .gitignore
├── src/                  # Or a folder named after your project
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_main.py
├── README.md
├── requirements.in       # Runtime dependencies source
├── requirements-dev.in   # Development dependencies source
├── requirements.txt      # Locked runtime dependencies (for deployment)
├── requirements-dev.txt  # Locked development dependencies (for developers)
└── environment.yml       # (Optional) For easy Conda developer setup
```

In more complex projects where there might be subdirectories with products (app, model) requiring
different dependencies the requirements.in/.txt files should live in the subdirectories
as follows:

```
company_monorepo/
├── .git/
├── .gitignore
├── backend_api/
│   ├── src/
│   │   └── app.py
│   ├── requirements.in
│   └── requirements.txt  # Defines dependencies for the API
├── data_worker/
│   ├── src/
│   │   └── worker.py
│   ├── requirements.in
│   └── requirements.txt  # Defines dependencies for the worker
└── README.md
```

### Edit the README to add system library dependencies
Your `conda` environment transparently manages and installs system-level libraries (like C/C++ libraries for scientific computing, database connectors, or image processing) that `pip` and `venv` are completely unaware of. When you move to a production server and only use `pip install -r requirements.txt`, your application will almost certainly crash if it relies on any of these hidden dependencies.

Here is a comprehensive guide on how to discover, document, and manage them.

---

### Step 1: Discovering Your Non-Python Dependencies

You need to become a detective and find out what `conda` has been doing for you behind the scenes.

#### Method 1: The Conda Environment Export (Most Reliable)

The most direct way is to inspect what `conda` actually installed.

1.  Activate your development `conda` environment.
    ```bash
    conda activate my_project_env
    ```

2.  Export the *entire* environment to a YAML file.
    ```bash
    conda env export > full_conda_environment.yml
    ```

3.  **Analyze the file.** Open `full_conda_environment.yml` and look for packages that are clearly not Python packages. These often live under the main `dependencies:` list.

**Example Analysis of a `full_conda_environment.yml`:**

```yml
name: my-geo-project
channels:
  - conda-forge
dependencies:
  # --- Python Packages (also listed) ---
  - python=3.10
  - pandas=2.2.1
  - shapely=2.0.3
  - pip
  
  # --- NON-PYTHON DEPENDENCIES (The ones to watch out for!) ---
  - geos=3.12.1         # C++ library for geometric operations, needed by Shapely
  - gdal=3.8.4          # Geospatial Data Abstraction Library
  - proj=9.3.1          # Library for cartographic projections
  - libpq=16.2          # PostgreSQL client C library (if you used psycopg2 from conda)
  - openssl=3.2.1       # Cryptography library
  - libjpeg-turbo=3.0.0 # C library for handling JPEG images
  
  - pip:
    # --- Pip-installed packages ---
    - some-other-package==1.2.3
```

From this list, you've discovered that your application needs **GEOS, GDAL, PROJ, libpq, OpenSSL, and libjpeg**. These will **not** be installed by `pip`.

#### Method 2: Check Package Documentation

For your key libraries (the ones in your `requirements.in`), go to their official installation pages or PyPI pages. They often list system-level prerequisites.

*   **`psycopg2` (PostgreSQL driver):** The docs state it needs `libpq-dev` (on Debian/Ubuntu) or `postgresql-devel` (on RedHat/Fedora).
*   **`Pillow` (Image library):** The docs list optional but often-needed libraries like `libjpeg-dev` and `zlib1g-dev`.
*   **`lxml` (XML parser):** Needs `libxml2-dev` and `libxslt1-dev`.

This method is a good cross-reference and helps you find the correct names for the system's package manager (`apt`, `yum`, etc.).

Document the system level libraries and instructions for installing them in the Requirements section
of the README.md

