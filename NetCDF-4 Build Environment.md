## NETCDF environmental variables

These environmental variables are primarily used when building or installing software that depends on the NetCDF-4 C library. They tell the build system (like setup.py or a Makefile) exactly where to find the library files and headers

1) Standard NetCDF-4 Variables

- NETCDF4_DIR: The primary installation path (prefix) for the NetCDF-4 library. The build system uses this to look for subdirectories like bin, lib, and include.
- NETCDF4_INCLUDE: The specific path to the header files (e.g., netcdf.h).
- NETCDF4_LIB: The specific path to the compiled library files (e.g., libnetcdf.so or .a).
- NETCDF4_VERSION: Specifies or identifies the specific version of the NetCDF library being used.

2) Parallel NetCDF-4 Variables
These variables are used when NetCDF-4 is built with Parallel I/O support, typically requiring HDF5 and MPI. 

- NETCDF4_PARALLEL_DIR: The base directory for the parallel-enabled version of NetCDF.
- NETCDF4_PARALLEL_INCLUDE: The include directory for the parallel version.
- NETCDF4_PARALLEL_LIB: The library directory for the parallel version.
- NETCDF4_PARALLEL_VERSION: The version number for the parallel implementation.

## Environmental variables

- Display environmental variables: ```echo $NETCDF_CONFIG```
- Display environmental variables: ```printenv $NETCDF_CONFIG```
- Delete an environmental variable: ```unset $MY_VAR```

## On nc-config and nf-config files

Since version 4.2, the NetCDF libraries for C and Fortran have been split into separate packages. While ```nc-config``` is the utility for the C library, ```nf-config``` is the dedicated tool for the Fortran library. ```nf-config --fc``` specifically reports the Fortran compiler used to build your NetCDF-Fortran library, ensuring compatibility, while ```nc-config``` often returns a blank for ```--fc```, because it may not be able to find the path to the Fortran utility or was not configured to "talk" to it during installation. 

Depending on your environment variables, you may need to use these direct versions:

- ```nf-config --fc```: The standard utility for querying NetCDF-Fortran build options.
- ```nc-config --all```: Used to view all settings for the C-based NetCDF library.
- ```nf-config --fflags```: Shows the Fortran compiler flags required for building.

Configuration files (config files) are plain-text documents used by software, operating systems, and servers to store user preferences, startup settings, and parameters. They allow users to customize behavior—such as defining theme, file paths, or network connections—without changing the core application code. They are typically written in human-readable text formats like YAML, JSON, INI, or XML. On Linux/Unix, these are often found in /etc (system-wide) or ~/.config (user-specific). They often use key-value pairs (e.g., theme = dark) to define settings. 

Instead of having settings hardcoded into the program, a configuration file lets the program behave differently based on the environment or user. This means, for example, a web application can have different database connection settings for its development phase and its final live phase.
