# Python packaging using `conda-build` and `flit`
This tutorial demonstrates how to create an __installable python package__ from regular python source code.

### Useful link
- [Conda-build](https://docs.conda.io/projects/conda-build/en/stable/resources/define-metadata.html)
- [Flit](https://flit.pypa.io/en/latest/)
- [Configuring setuptools using pyproject.toml files](https://setuptools.pypa.io/en/latest/userguide/pyproject_config.html)
- PEP 621: [Storing project metadata in pyproject.toml](https://peps.python.org/pep-0621/) 
- Anaconda: [Uploading and installing conda packages](https://docs.anaconda.com/free/anacondaorg/user-guide/packages/conda-packages/)
- Demo video packaging with pip: [Packaging python code with pyproject.toml](https://youtu.be/v6tALyc4C10?si=D60Lz81g5KG_ZXNY)

# 1. Package build process
## Packaging environemt
Firstly, create packaging environemt. This environment is used to build and deploy the package - not to test or use it. Thus, it doesn't require all the dependencies that the package to be build has. 
```
conda create -n build_env conda-build>=3.28 flit>=3.9 anaconda-client
```
- `flit` is be backend, specified in `pyproject.toml`
- `anaconda-client` is only required if the package will be uploaded to `anaconda.org`

Activate the environment:
```
conda activate build_env 
```
## Build
Execute `conda build .` from the root directory of the project (where the `pyproject.toml` is located) and specify on output folder als a "local package channel":
```
conda build . --output-folder "C:\Users\holge\Desktop\github\local_package_channel"
```

Afterwards that package can be found in the respective channel using `conda search -c CHANNEL_NAME`. With `--override-channels` only that channel is returned and noch all default channels, too.

```
(build_env) C:\Users\holge\Desktop\github\python_package_conda-build_flit>conda search -c "C:\Users\holge\Desktop\github\local_package_channel" --override-channels              
Loading channels: done
# Name                       Version           Build  Channel
trial_p                       2023.1            py_0  local_package_channel
```
## Upload to `anaconda.org`
Upload the created package to anaconda using `anaconda login` and `anaconda upload`: 
```
(build_env) C:\Users\holge\Desktop\github\python_package_conda-build_flit>anaconda login
Using Anaconda API: https://api.anaconda.org
Username: munich-ml
Password: 
munich-ml's login successful

(build_env) C:\Users\holge\Desktop\github\python_package_conda-build_flit>anaconda upload C:\Users\holge\Desktop\github\local_package_channel\noarch\trial_p-2023.1-py_0.tar.bz2 
Using Anaconda API: https://api.anaconda.org
Using "munich-ml" as upload username
Processing "C:\Users\holge\Desktop\github\local_package_channel\noarch\trial_p-2023.1-py_0.tar.bz2"
Detecting file type...
File type is "Conda"
Extracting conda attributes for upload
Creating package "trial_p"
Creating release "2023.1"
Uploading file "munich-ml/trial_p/2023.1/noarch/trial_p-2023.1-py_0.tar.bz2"
32.1kB [00:01, 27.9kB/s]
Upload complete

conda located at:
  https://anaconda.org/munich-ml/trial_p
```

# 2. Package installation and usage
To install and use the newly created `trial_p` package, let's create a fresh conda environement:
```
(base) C:\Users\holge\Desktop\github\python_package_conda-build_flit>conda create -n user_env
```
Do the `install`:
```
(base) C:\Users\holge\Desktop\github\python_package_conda-build_flit>conda activate user_env

(user_env) C:\Users\holge\Desktop\github\python_package_conda-build_flit>conda install trial_p -c munich-ml
```

## Usage 
```
(user_env) C:\Users\holge\Desktop\github\python_package_conda-build_flit>python
>>> import trial_p
>>> dir(trial_p)
[... , 'main', 'useless_function']

>>> trial_p.useless_function()
'nonesense'

>>> trial_p.main()            
Executing main in __init__.py

>>> from trial_p import useless_module
>>> dir(useless_module) 
['USELESS_CONSTANT', 'UseLessClass', ... , 'another_useless_function']

>>> useless_module.USELESS_CONSTANT
88
```
All works as expected!

### Executable scripts 
`trial_p` is registerd as an executable script, here's a demostration:
```
(user_env) C:\Users\holge\Desktop\github\python_package_conda-build_flit> cd C:\

(user_env) C:\>trial_p
Executing main in __init__.py
```
The executable scripts are configured within the `[project.scripts]` section of the `pyproject.toml`:
```toml
[project.scripts] 
trial_p = "trial_p:main"
# This makes an executable script. 
# Typing "trial_p" in cmd will execute the main function in the trial_p package.
# That requires that a main() function is defined in __init__.py
```
