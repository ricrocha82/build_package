##########################################################
Complete Docker Container Creation and Publishing Tutorial
##########################################################

.. contents:: Table of Contents
   :depth: 3
   :local:

Overview
========

This tutorial shows how to create and publish a bioinformatics Docker container (CRESSENT) to Docker Hub using Podman on Linux.

Prerequisites
=============

- Podman installed (``sudo apt install podman-docker``)
- Docker Hub account
- Your bioinformatics tool/conda package ready

Part 1: Preparation
===================

1.1 Check Your Environment
--------------------------

.. code-block:: bash

   # Check if podman is installed
   podman --version

   # Check your current directory and files
   pwd
   ls -la

   # Look for your conda package or source files
   ls dist/

1.2 Create Project Directory Structure
--------------------------------------

.. code-block:: bash

   # Navigate to your project directory
   cd /path/to/your/project

Example structure should look like::

   cressent/
   ├── Dockerfile
   ├── dist/ # or image
   │   └── cressent-1.0.0-pyhdfd78af_0.tar.bz2
   ├── README.md
   └── other-files...

Part 2: Building the Docker Image
==================================

2.1 Create a Dockerfile
------------------------

.. code-block:: bash

   # Create Dockerfile in your project root
   cat > Dockerfile << 'EOF'
   FROM continuumio/miniconda3:latest

   LABEL maintainer="Your Name (your.email@example.com)"
   LABEL version="1.0.0"
   LABEL description="CRESSENT: A comprehensive toolkit for CRESS DNA virus analysis"
   LABEL org.opencontainers.image.source="https://github.com/yourusername/yourproject"

   # Install mamba for faster package management
   RUN conda install -c conda-forge mamba -y

   # Method 1: Install from bioconda (recommended)
   RUN mamba install -y -c conda-forge -c bioconda cressent=1.0.0

   # Method 2: Use local conda package (alternative)
   # COPY dist/cressent-1.0.0-pyhdfd78af_0.tar.bz2 /tmp/
   # RUN mamba install -y /tmp/cressent-1.0.0-pyhdfd78af_0.tar.bz2

   # Clean up to reduce image size
   RUN mamba clean --all --yes && \
       rm -rf /var/lib/apt/lists/*

   # Set working directory where user data will be mounted
   WORKDIR /app

   # Set entrypoint to your tool
   ENTRYPOINT ["cressent"]

   # Default command shows help
   CMD ["--help"]
   EOF

2.2 Build the Docker Image
---------------------------

.. code-block:: bash

   # Build the image with your Docker Hub username
   podman build -t docker.io/yourusername/yourproject:1.0.0 .
   podman build -t docker.io/yourusername/yourproject:latest .

   # Example for CRESSENT:
   podman build -t docker.io/ricrocha82/cressent:1.0.0 .
   podman build -t docker.io/ricrocha82/cressent:latest .

2.3 Verify the Build
---------------------

.. code-block:: bash

   # Check that images were created
   podman images

   # Test the container works
   podman run --rm docker.io/yourusername/yourproject:1.0.0 --help

Part 3: Publishing to Docker Hub
=================================

3.1 Login to Docker Hub
------------------------

.. code-block:: bash

   # Login to Docker Hub
   podman login docker.io

   # Enter your Docker Hub username and password when prompted

3.2 Tag Existing Images (if needed)
------------------------------------

.. code-block:: bash

   # If you have an existing image that needs proper tagging
   podman images  # Check current images

   # Tag existing image with proper Docker Hub format
   podman tag localhost/existing-image:tag docker.io/yourusername/yourproject:1.0.0
   podman tag localhost/existing-image:tag docker.io/yourusername/yourproject:latest

3.3 Test Before Pushing
------------------------

.. code-block:: bash

   # Test the properly tagged image
   podman run --rm docker.io/yourusername/yourproject:1.0.0 --help

   # Test with volume mounting (how users will actually use it)
   mkdir -p test_data
   podman run --rm -ti -v "$(pwd):/app" docker.io/yourusername/yourproject:1.0.0 --help

3.4 Push to Docker Hub
-----------------------

.. code-block:: bash

   # Push specific version
   podman push docker.io/yourusername/yourproject:1.0.0

   # Push latest tag
   podman push docker.io/yourusername/yourproject:latest

3.5 Logout (Optional)
----------------------

.. code-block:: bash

   # Logout when done
   podman logout docker.io

Part 4: Documentation for Users
================================

4.1 Create Usage Documentation
-------------------------------

Add this to your README.md:

.. code-block:: rst

   Docker Usage
   ============

   Quick Start
   -----------

   .. code-block:: bash

      # Pull the Docker image
      docker pull yourusername/yourproject

      # Run with your data directory mounted
      docker run --rm -ti -v "$(pwd):/app" yourusername/yourproject [command] [options]

      # Show help
      docker run --rm yourusername/yourproject --help

   Examples
   --------

   .. code-block:: bash

      # Basic analysis
      docker run --rm -ti -v "$(pwd):/app" yourusername/yourproject analyze -i data.fasta -o results/

      # Interactive mode
      docker run --rm -ti -v "$(pwd):/app" yourusername/yourproject bash

      # Specific analysis with parameters
      docker run --rm -ti -v "$(pwd):/app" yourusername/yourproject build_tree -i sequences.fasta --method ml

   Notes
   -----

   - The ``-v "$(pwd):/app"`` mounts your current directory to ``/app`` in the container
   - The ``-ti`` flags provide an interactive terminal
   - The ``--rm`` flag automatically removes the container after use

Part 5: Complete Example Workflow
==================================

Example: CRESSENT Container
----------------------------

.. code-block:: bash

   # 1. Navigate to project directory
   cd /mnt/c/Users/ricro/Documents/cressent

   # 2. Check what you have
   ls -la
   ls dist/

   # 3. Create Dockerfile (see Part 2.1)

   # 4. Login to Docker Hub
   podman login docker.io

   # 5. Build images
   podman build -t docker.io/ricrocha82/cressent:1.0.0 .
   podman build -t docker.io/ricrocha82/cressent:latest .

   # 6. Verify build
   podman images
   podman run --rm docker.io/ricrocha82/cressent:1.0.0 --help

   # 7. Push to Docker Hub
   podman push docker.io/ricrocha82/cressent:1.0.0
   podman push docker.io/ricrocha82/cressent:latest

   # 8. Test user experience
   docker pull ricrocha82/cressent
   docker run --rm -ti -v "$(pwd):/app" ricrocha82/cressent --help

Part 6: Troubleshooting
========================

Common Issues and Solutions
----------------------------

6.1 Permission Errors
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # If you get Docker daemon permission errors, use podman instead
   # Replace 'docker' with 'podman' in all commands

6.2 CGgroup Warnings in WSL
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::
   These warnings are normal and don't affect functionality::

      WARN[0000] The cgroupv2 manager is set to systemd but there is no systemd user session available

   Just ignore them - your container will work fine.

6.3 Entrypoint Issues
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # If container doesn't run your tool directly, users can specify it:
   docker run --rm -ti -v "$(pwd):/app" yourusername/yourproject yourtool --help

6.4 Image Size Too Large
~~~~~~~~~~~~~~~~~~~~~~~~~

Add these to Dockerfile to reduce size:

.. code-block:: dockerfile

   RUN mamba clean --all --yes && \
       rm -rf /var/lib/apt/lists/* && \
       conda clean -a

Part 7: Automation (Advanced)
==============================

7.1 GitHub Actions for Automatic Building
-------------------------------------------

Create ``.github/workflows/docker-publish.yml``:

.. code-block:: yaml

   name: Build and Push Docker Image

   on:
     push:
       tags: ['v*']
     release:
       types: [published]

   jobs:
     build-and-push:
       runs-on: ubuntu-latest
       
       steps:
       - name: Checkout
         uses: actions/checkout@v4
         
       - name: Login to Docker Hub
         uses: docker/login-action@v3
         with:
           username: ${{ secrets.DOCKER_HUB_USERNAME }}
           password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
           
       - name: Build and push
         uses: docker/build-push-action@v5
         with:
           context: .
           push: true
           tags: |
             yourusername/yourproject:latest
             yourusername/yourproject:${{ github.ref_name }}

Summary Checklist
=================

.. list-table:: Container Publishing Checklist
   :widths: 10 90
   :header-rows: 1

   * - Status
     - Task
   * - ☐
     - Project directory with Dockerfile ready
   * - ☐
     - Docker Hub account created
   * - ☐
     - Podman installed and working
   * - ☐
     - Login to Docker Hub (``podman login docker.io``)
   * - ☐
     - Build image (``podman build -t docker.io/username/project:tag .``)
   * - ☐
     - Test image (``podman run --rm image:tag --help``)
   * - ☐
     - Push image (``podman push docker.io/username/project:tag``)
   * - ☐
     - Update documentation with usage instructions
   * - ☐
     - Test user experience (``docker pull username/project``)
   * - ☐
     - Logout if desired (``podman logout docker.io``)

Final Result
============

**Your container is now available for the community to use!**

.. code-block:: bash

   docker pull yourusername/yourproject
   docker run --rm -ti -v "$(pwd):/app" yourusername/yourproject --help

.. see also::
   - `Docker Hub Documentation <https://docs.docker.com/docker-hub/>`_
   - `Podman Documentation <https://podman.io/docs>`_
   - `Bioconda Guidelines <https://bioconda.github.io/contributor/guidelines.html>`_
