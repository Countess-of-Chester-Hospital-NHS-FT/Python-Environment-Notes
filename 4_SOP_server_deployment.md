# Deploying a project on a server using venv
This assumes python (the version required by the project) is already installed on the server.

## Install system level libraries first
The package manager and syntax will depend on the operating system of the server. 
There should be info on this in the requirements section of the README

**On Debian / Ubuntu:**
```bash
sudo apt-get update
sudo apt-get install -y \
    libgdal-dev \
    libgeos-dev \
    libproj-dev \
    libpq-dev \
    libjpeg-dev
```

## Python Environment Setup

After installing the system prerequisites, set up the Python environment:

```bash
# 1. Create a virtual environment
python3 -m venv venv

# 2. Activate it
source venv/bin/activate

# 3. Install Python packages
pip install -r requirements.txt

# 4. Run the application
```

## Notes on Docker

We don't currently use Docker, but its useful to understand what it is.

A `Dockerfile` is another comprehensive executable form of documentation. 
It codifies all the system and Python dependencies into a single, portable image.

This moves your deployment from the multi-step process above to a fully automated one.

**Example `Dockerfile`:**

```dockerfile
# 1. Start from an official Python base image
# Use a specific version for reproducibility
FROM python:3.11-slim

# Set a working directory in the container
WORKDIR /app

# 2. Install the system-level dependencies you discovered
# This is the equivalent of your README instructions
RUN apt-get update && apt-get install -y --no-install-recommends \
    gdal-bin \
    libgdal-dev \
    libgeos-dev \
    libproj-dev \
    libpq-dev \
    libjpeg-dev \
    # Clean up the apt cache to keep the image small
    && rm -rf /var/lib/apt/lists/*

# 3. Copy only the requirements file first to leverage Docker's layer caching
COPY requirements.txt .

# 4. Install the Python packages using pip
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy the rest of your application code into the container
COPY . .

# 6. Define the command to run your application
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "my_app:app"]
```

With this `Dockerfile`, deployment becomes trivial:
1.  `docker build -t my-project .`
2.  `docker run -p 8000:8000 my-project`

The Docker container **is** your server environment, perfectly configured every time.
