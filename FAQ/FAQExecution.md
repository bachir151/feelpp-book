Runtime FAQ
===========



# Parallel computing

## How to bind Feel++ mpi processes to cores

For performance reasons, it can be very efficient to bind mpi processes to cores.
You should read the mpirun manual and the binding section.

Here is the command line to bind the processes:
```cpp
mpirun -np 4 -bind-to-core feelpp_qs_laplacian
```

#  Feel++ and memory

## How to monitor the memory usage in Feel++

It is not easy to have a precise account for memory usage in a code with
standard unix tools. `top` , `ps`  or `htop`  can tell you more or less what memory has
been allocated with the OS to your application but it is not, and by far, a
precise account.

You can compile and link your application with PETSc in Debug mode and use the
following function `logMemoryUsage(` <string>) to monitor in LOG files
accurately the amount of memory spent by PETSc. The string allows you to write a
message in the LOG file to better identify the place where you monitored the
memory. Also this function displays system information using the `ps`  command.
Your code can be instrumented as follows:
```cpp
Environment::logMemoryUsage("memory usage before:");

// some code here which require memory allocation
// that we wish to monitor
// ...

Environment::logMemoryUsage("memory usage after:");
```

Otherwise there are tools like `valgrind`  (OpenSource) or `ddt/map`
(Commercial) that allow to better better the memory usage for data structures
other than PETSc ones.


# Error or Exception messages

## locale::facet::_S_create_c_locale name not valid

 On Ubuntu systems and possibly others, locale (language, money, ...) information
might be set and we don't handle that very well at the moment. You might get the
following message at the execution of any Feel++ applications
```cpp
terminate called after throwing an instance of 'std::runtime_error'
  what():  locale::facet::_S_create_c_locale name not valid
```

To fix this, we suggest that you `unset` all environment variables related to
locale information. It includes `LANG`,  `LC_CTYPE` and `LC_ALL`.

Just type

```cpp
unset LANG LC_CTYPE LC_ALL
```

then the Feel++ application should execute without problems.
