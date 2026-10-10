### Common cmake build command format

##### Format :

```bash
cmake -G "Specify Generator (Optional)" -D_FLAGS_YOU_WANT_TO_ENABLE_OPTIONAL -DCMAKE_INSTALL_PREFIX="install/path" "CMakeLists.txt path"
```

ex (OpenCV for windows):

```bash
cd OpenCV
mkdir lib
mkdir build
cd build
cmake -G "MinGW Makefiles" -D CMAKE_BUILD_TYPE=Release -D OPENCV_ENABLE_ALLOCATOR_STATS=OFF -D BUILD_SHARED_LIBS=OFF -D CMAKE_INSTALL_PREFIX=../lib ..
mingw32-make            # MinGW's make is usually named mingw32-make on Windows
mingw32-make install

# or, generator independent:
# cmake --build . --config Release
# cmake --install .
```

- Generator -> ```-G "MinGW Makefiles"```
  - The generator decides which **build system files** cmake generates (Unix Makefiles, MinGW Makefiles, Ninja, Visual Studio solutions, ...). By default cmake picks a generator for your platform and searches your environment for an available C/C++ compiler. The "MinGW Makefiles" generator expects the MinGW (GCC for Windows) toolchain; to choose a specific compiler use `-D CMAKE_C_COMPILER=... -D CMAKE_CXX_COMPILER=...` (or the `CC`/`CXX` environment variables)

- D build flags -> ```-D CMAKE_BUILD_TYPE=Release -D OPENCV_ENABLE_ALLOCATOR_STATS=OFF -D BUILD_SHARED_LIBS=OFF```
  - these D flags set cache variables that can modify the type of build you want for your application, or library, either release or debug, static or shared, enabled/disabled features, etc.

- Installation path -> ```-D CMAKE_INSTALL_PREFIX=../lib```
  - this is where the final binaries and header files will go if you run the install step (```make install```)

- CMakeLists.txt path -> ```..```
  - this is where the **CMakeLists.txt** file is located, in the example's case, it is in the parent directory of the current directory. The current directory is OpenCV/build, while OpenCV is the directory where CMakeLists.txt is located, hence we go one directory up using ```..``` 
