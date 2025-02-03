# Libs

this folder should contain most of the actual code running on the pcb along with the python tests

## Build requirements

To build the projects, you must have make and clang installed on your system. Currently windows is not supported because the copyobj script requires bash. If someone would like to make a copyobj.ps1 script or set it manually in the makefile, that would be great

## Building

each project can be built with make, simply pass the name of the project folder into the makefile and any other dependencies the project has. the makefile will also detect if a python test exists (in the folder (PROJECT)\_py) and build the module for that too:

```sh
make PROJECT=cd
```

_or_

```sh
make PROJECT=apogee "DEPS := cd"
```

_or_

```sh
make PROJECT=apogee "DEPS := cd pid"
```

As many dependencies as you want can be specified by saying `DEPS := ...`

If a project has a dep.mk file at the top level, dependencies do not need to be specified (for example, see `lib/apogee/dep.mk`).

All project build files will either be stored in `./build/win64` or `./build/unix64` depending on what type of system you have.

Make will also build an executable for the project at `build/PROJECT(.exe)` so make sure that all projects at least have a stub `int main(){}`.

## Running python tests

To run the python tests you need to have rocketpy installed via pip on your system or on the virtual environment

To install globally, just run

```sh
pip install rocketpy
```

To use the virtual environemnt

```sh
source bin/activate

# for windows
. ./bin/Activate.ps1
```

then you can run

```sh
pip install rocketpy
```

then you can just cd to a \_py directory and run the python file there:

```sh
cd apogee_py

python test.py
```
