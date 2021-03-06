Using Installed version
=======================


\section intro Introcution
You may have installed Feel++ via a packet manager or the `cmake` process.
You may also only have compiled Feel++ but not installed it in the system.

We considere here you have various files:
* sources files
* config files
* geo files
* ...

\section cmakelists CMakeLists.txt

In order to take care of that various situation, here is provided a default `CMakeLists.txt` to be put at the top of your project directory:
\code{cmake}
cmake_minimum_required(VERSION 2.8)
if ( ${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR} )
	FIND_PATH(FEELPP_CMAKE_MODULES FindFeel++.cmake
						PATH  /usr/share/feel/cmake/modules/
									/usr/local/share/feel/cmake/modules/
									/where/I/have/installed/feel++ )
	if ( FEELPP_CMAKE_MODULES )
		set(CMAKE_MODULE_PATH ${FEELPP_CMAKE_MODULES})
	else()
		message(FATAL_ERROR "Feel++ does not seem to have been installed on this platform")
	endif()
	Find_Package(Feel++)
endif()
feelpp_add_application(
 applicationName
  SRCS file.cpp file1.hpp file2.hpp
  GEO geoFile1 geoFile2
  DEFS A_DEF=2
  CFG cfgFile1.cfg cfgFile2.cfg )
\endcode{cmake}

As you can see, (line 2) we check if your project is not added as a subproject to the Feel++ one (basically, located in the `research` directory.

The one important line is the last. That macro will generate all the process to actually compile your application.
You can fine the whole definition of that macro [here](https://github.com/feelpp/feelpp/blob/develop/cmake/modules/feelpp.macros.cmake).
As you can see here, my project is composed of 3 files, with two different geometry.
The `DEFS` entry is used if you have defined something like
`
	auto mesh = loadMesh(_mesh = new Mesh<Simplex<DIM>> );
`
Here, your DIM will be set to 2 at compile time, you can off course ask for DIM to be equal to 3.

Sections GEO, DEfS and CFG are optional.
\section compil Compiling
You have to use the cmake process.

\subsection system Is Feel++ installed on the system ?
Is that case, the root directory for the cmake process is the top one of the Feel++ projet.
\subsection local Are you defining a sub project ?
Is that case, the root directory for the cmake process is the top one of your project.

\subsection whatever To compile
```cpp
	cmake -DCMAKE_CXX_COMPILER=/usr/bin/clang++ /root/directory
	make -n 2 feelpp_applicationName
	cd where/is/my/app
	./feelpp_applicationName
```

\subsection comp_opt Option at cmake
You can customize your compilation to use various compiler or way of compiling.
It is highly recommended to use clang instead of gcc.
```cpp
	cmake /root/directory \
		-DCMAKE_CXX_COMPILER=/usr/bin/clang++ \
		-DCMAKE_C_COMPILER=/usr/bin/clang \
		-DCMAKE_BUILD_TYPE=RelWithDebInfo
```




*/
}
