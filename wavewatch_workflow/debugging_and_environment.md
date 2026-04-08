# Environment & Debugging Notes

## 1) Resetting the module environment
Purpose: Clear all loaded software modules and avoid conflicts.

module purge

## 2) Handling library version mismatches (HDF5)
Purpose: Suppress errors caused by incompatible HDF5 versions.

export HDF5_DISABLE_VERSION_CHECK=1

## 3) Monitoring model output

tail log.ww3

Useful while ww3_shel is running.
