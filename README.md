# How to build a package for your tool 
Notes and codes about how to build a conda package

# Install build

## 1. Submit to PyPi

```bash
cd path/to/
conda create -n conda-build
conda activate conda-build
python3 -m pip install --upgrade build # to build the tar file - source code
python3 -m pip install --upgrade twine # to sento to PyPi
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

python -m build

# after building, get the sha256 by running
sha256sum .../[package-name]-0.1.0.tar.gz
# it will give you the YOUR_CALCULATED_SHA256_HASH

# Upload the archives
twine upload dist/*
```

If the code ran successfully, it will create a file: [project-name]-[version].tar.gz

# Upload to GitHub

If you want to distribute your build artifacts on GitHub, first create a Git tag

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

git clone https://github.com/bioconda/bioconda-utils.git
cd bioconda-utils
conda install --file bioconda_utils/bioconda_utils-requirements.txt -c conda-forge -c bioconda 
python setup.py install
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
  run_exports:
    - 

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
conda mambabuild recipe -c bioconda -c conda-forge -c defaults # faster using boa
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

Fork the `bioconda-recipes` repo on GitHub first,

Go to bioconda/bioconda-recipes and click “Fork” to create a copy under your GitHub account.

then:
1. Clone Bioconda-recipes repository
```bash
git clone https://github.com/ricrocha82/bioconda-recipes.git
cd bioconda-recipes
```
2. Add Your Recipe
Inside `bioconda-recipes/recipes`:
```bash
mkdir cressent
cp /path/to/your/recipe/meta.yaml cressent/
# If you have a build.sh (not needed for noarch: python), copy it too:
# cp /path/to/your/recipe/build.sh cressent/
```
3. Test the Recipe Locally
From the root of `bioconda-recipes`:
```bash
bioconda-utils build recipes config.yml --packages cressent --docker --mulled-test
```
4. Lint (check formatting & requirements)
```bash
bioconda-utils lint recipes config.yml --packages cressent
```
5. Commit and Push to Your Fork
```bash
git checkout -b add-cressent
git add recipes/cressent
git commit -m "Add cressent version 1.0.0"
git push origin add-cressent
```

6. Open a Pull Request
Go to your fork on GitHub → click Compare & pull request (PR) → submit PR to bioconda/bioconda-recipes.

Follow the guidelines:
- Use an appropriate title starting with “Add” (e.g., “Add cressent version 1.1.0”).
- Include a description summarizing your changes.
- Ensure you have followed the Bioconda Recipe Guidelines.

Request reviews:

Once your PR is passing all CI tests, issue the Bot command:
```bash
@BiocondaBot please add label
```

This helps the maintainers know your PR is ready for review.

7. Address CI feedback and mergePermalink

a) Monitor CI:

Your PR will trigger automated tests and lint checks. If there are any errors (e.g., missing run_exports, long summary warnings), address them by updating your recipe and pushing new commits.

b) Merge:

Once all checks pass and the maintainers have reviewed your PR, you can merge it. Note that Bioconda now prefers squash & merge, so make sure to combine your commits appropriately.

8. After Merge
Once merged, within hours, your package will be available on Bioconda.
Users can install with:
```bash
Your package will be built and uploaded automatically to the Bioconda channel.
Users can install your package using:
conda config --add channels conda-forge
conda config --add channels bioconda
mamba create -n cressent -c conda-forge -c bioconda cressent
```
