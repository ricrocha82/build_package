# How to build a package of your tool 
notes and codes about how to build a conda package

# Install build

```bash
conda create -n conda-build
conda activate conda-build
pip install build # to build the tar file - source code
```

This code will use the `pyproject.toml` file, which needs to be something like that.
This file should be in the root directory

```bash
[build-system]
requires = ["setuptools>=64.0.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "[package-name]"
version = "0.1.0"
description = "My tool is awesome !!!"
authors = [
    {name = "Pavan R.", email = "pavan@my_company"}
]
readme = "README.md"
requires-python = ">=3.10"
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]
dependencies = [
    "pandas",
    "matplotlib",
    "seaborn",
    "numpy",
    "gffutils",
    "click"
]

[project.urls]
Homepage = "https://github.com/ricrocha82/[package-name]"
Repository = "https://github.com/ricrocha82/[package-name]"

[project.scripts]
my_tool = "my_tool_core.cli:cli"

[tool.setuptools]
packages = ["my_tool_core"]
```

To execute the command, go to the  directory and run:

```bash
cd path/to/package

python -m build --sdist

# after building get the sha256 by runinng
sha256sum .../[package-name]-0.1.0.tar.gz
# it will give you the YOUR_CALCULATED_SHA256_HASH
```

If the code ran successfully, it will create a file: [project-name]-[version].tar.gz

# Upload to the GitHub

If you want to distribute your build artifacts on the GitHub, first create a Git tag

```bash
git tag v0.1.0
git push origin v0.1.0
```

Create a GitHub Release

1. Go to your repo on GitHub
2. Click "Releases" → "Draft a new release"
3. Set:
    - Tag: v0.1.0 (match the tag above)
    - Title: v0.1.0
    - Description: Add notes or changelog
4. Upload the .tar.gz file from dist/ as a binary
5. Click "Publish release"

# Build a Conda/Bioconda package

Install `conda-build`

```bash
conda install -c conda-forge -c defaults conda-build conda-verify
mamba install boa -c conda-forge
```

Create a `meta.yaml file` in a new folder called `recipe`
```bash
# simple examples
{% set name = "my_tool" %}
{% set version = "0.1.0" %}

package:
  name: {{ name|lower }}
  version: {{ version }}

source:
  path: ../..  # Path to the root of your project
  # OR
#   url: https://github.com/username/project/archive/v{{ version }}.tar.gz
#   sha256: YOUR_CALCULATED_SHA256_HASH
#   git_url: https://github.com/username/project.git
#   git_rev: v{{ version }}  # Git tag or commit hash
#   url: https://github.com/username/project/archive/v{{ version }}.tar.gz
#   sha256: YOUR_CALCULATED_SHA256_HASH
#   patches:
#     - fix-bug.patch
#     - add-feature.patch

build:
  number: 0
  script: {{ PYTHON }} -m pip install . --no-deps -vv
  entry_points:
    - my_tool = my_tool_core.cli:cli
  noarch: python

requirements:
  host:
    - python >=3.10
    - pip
    - setuptools >=64.0.0
    - wheel
  run:
    - python >=3.10
    - pandas
    - numpy
    - click
    # must have all the dependencies

test:
  imports:
    - my_tool_core
  commands:
    - my_tool --help

about:
  home: https://github.com/ricrocha82/[package-name]
  license: MIT
  license_family: MIT
  summary: This tool is awesome
  doc_url: https://github.com/ricrocha82/[package-name]
  dev_url: https://github.com/ricrocha82/[package-name]

extra:
  recipe-maintainers:
    - yourGitHubUsername  # Replace with your GitHub username
```

Run
```bash
# test the source code
conda build recipe --source
# build the recipe using the meta.yaml file
cd path/to/project_root
# conda build recipe -c bioconda -c conda-forge -c defaults 
# or
conda mambabuild recipe -c bioconda -c conda-forge -c defaults # fastar using boa
```
If all goes well, it creates a conda package (`.tar.bz2` or `.conda`)

It will be located at `conda-bld/noarch/[packa-name]-[version]-py_0.tar.bz2/.conda`

```bash
# Test installation
conda create -n test-[package-name]
conda activate test-[package-name]
mamba install -c local -c bioconda -c conda-forge [package-name] --yes # MAMBA is faster and deals better with dependencies
[package-name] --help
```

### adding to bioconda

1. Fork the Bioconda repository: go the [Bioconda Github page](https://github.com/bioconda/bioconda-recipes) and fork the repository. 
2. Create a new branch in the main branch. 
3. Add the `meta.yaml` file 
4. Submit a pull request to Bioconda
5. Address review comments

