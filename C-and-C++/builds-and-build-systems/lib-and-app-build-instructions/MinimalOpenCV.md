# OpenCV Minimizing Build

## for more info see config reference

https://docs.opencv.org/4.x/db/d05/tutorial_config_reference.html

## Install OpenCV

```bash
curl -L -o 4.11.0.zip https://github.com/opencv/opencv/archive/refs/tags/4.11.0.zip

unzip 4.11.0.zip -d ./

# use the same version (tag) for opencv_contrib as the main opencv source, mixing versions often fails to build
git clone --branch 4.11.0 --depth 1 https://github.com/opencv/opencv_contrib.git

cd opencv-4.11.0

cmake -S . -B build \
	-DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ \
	-DBUILD_SHARED_LIBS=FALSE -DCMAKE_BUILD_TYPE=Release  \
	-DOPENCV_EXTRA_MODULES_PATH="../opencv_contrib/modules" -DCMAKE_INSTALL_PREFIX="/usr/local" \
	-DBUILD_PERF_TESTS=OFF -DBUILD_TESTS=OFF -DBUILD_DOCS=OFF  -DWITH_CUDA=OFF -DBUILD_EXAMPLES=OFF -DBUILD_opencv_apps=OFF -DINSTALL_CREATE_DISTRIB=ON \
	\
	-DOPENCV_ENABLE_ALLOCATOR_STATS=OFF \
	-DBUILD_OPENEXR=OFF -DWITH_OPENEXR=OFF \
	-DBUILD_JASPER=OFF -DWITH_JASPER=OFF \
	-DBUILD_WEBP=OFF -DWITH_WEBP=OFF \
	-DBUILD_TIFF=OFF -DWITH_TIFF=OFF \
	-DBUILD_JPEG=ON -DBUILD_JPEG_TURBO_DISABLE=OFF -DWITH_JPEG=ON \
	-DBUILD_OPENJPEG=OFF -DWITH_OPENJPEG=OFF \
	-DBUILD_SPNG=OFF -DWITH_SPNG=OFF \
	-DWITH_JPEGXL=OFF \
	\
	-DWITH_IMGCODEC_HDR=OFF -DWITH_IMGCODEC_SUNRASTER=OFF -DWITH_IMGCODEC_PXM=OFF -DWITH_IMGCODEC_PFM=OFF \
	\
	-DWITH_ANDROID_MEDIANDK=OFF \
	\
	-DWITH_AVIF=OFF \
	-DBUILD_JAVA=OFF -DBUILD_opencv_java_bindings_generator=OFF -DBUILD_FAT_JAVA_LIB=OFF \
	-DBUILD_opencv_js=OFF -DBUILD_opencv_js_bindings_generator=OFF \
	-DWITH_TESSERACT=OFF \
	\
    -DBUILD_gen_opencv_python_source=OFF -DBUILD_opencv_python2=OFF -DBUILD_opencv_python3=OFF  \
	-DCPU_BASELINE=SSE3 -DCPU_DISPATCH=SSE4_1,SSE4_2,AVX,AVX2,FP16,AVX512_SKX -DENABLE_LTO=ON \
	\
	-DBUILD_opencv_calib3d=OFF \
	-DBUILD_opencv_bgsegm=OFF \
	-DBUILD_opencv_features2d=OFF \
	-DBUILD_opencv_objdetect=OFF \
	-DBUILD_opencv_ml=OFF \
	-DBUILD_opencv_mcc=OFF \
	-DBUILD_opencv_shape=OFF \
	-DBUILD_opencv_superres=OFF \
	-DBUILD_opencv_videostab=OFF \
	-DBUILD_opencv_viz=OFF
	
#========= install eigen from source (optional) : start ==============
# if you have eigen3 already installed yet that eigen library of yours don't have
# 'unsupported' you should install eigen 3.4 from source
# 1. download : https://gitlab.com/libeigen/eigen/-/archive/3.4.0/eigen-3.4.0.zip
# 2. extract
cd eigen-3.4.0
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
sudo make install
#========= install eigen from source (optional) : end ================

#========= create symlink (optional) : start ================
# if the previous opencv cmake config command didn't work and it's saying it was unable to find `/usr/local/include/eigen3`
# yet you already have an Eigen library installed in your computer with a different folder name, you can
# create a symlink like the command below so that CMake can find your eigen3 installation then you
# can just re-run the previous config command above.

sudo ln -s /usr/local/include/Eigen /usr/local/include/eigen3
#========= create symlink (optional) : end ================

cmake --build build --config Release

cmake --build build --target install --config Release
```

Try this for max compatibility : `-DCPU_BASELINE=SSE3 -DCPU_DISPATCH=`

disable module, example: cmake -DBUILD_opencv_calib3d=OFF ../opencv

some modules: calib3d core features2d flann highgui imgcodecs imgproc ts videoio

eigen can also be enabled or disabled : `-DWITH_EIGEN=ON` / `-DWITH_EIGEN=OFF`


## Super Minimal?

```
cmake -S . -B build \
	-DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ \
	-DBUILD_SHARED_LIBS=FALSE -DCMAKE_BUILD_TYPE=Release  \
	-DOPENCV_EXTRA_MODULES_PATH="../opencv_contrib/modules" -DCMAKE_INSTALL_PREFIX="/usr/local" \
	-DBUILD_PERF_TESTS=OFF -DBUILD_TESTS=OFF -DBUILD_DOCS=OFF  -DWITH_CUDA=OFF -DBUILD_EXAMPLES=OFF -DBUILD_opencv_apps=OFF -DINSTALL_CREATE_DISTRIB=ON \
	\
	-DOPENCV_ENABLE_ALLOCATOR_STATS=OFF \
	-DBUILD_OPENEXR=OFF -DWITH_OPENEXR=OFF \
	-DBUILD_JASPER=OFF -DWITH_JASPER=OFF \
	-DBUILD_WEBP=OFF -DWITH_WEBP=OFF \
	-DBUILD_TIFF=OFF -DWITH_TIFF=OFF \
	-DBUILD_JPEG=ON -DBUILD_JPEG_TURBO_DISABLE=OFF -DWITH_JPEG=ON \
	-DBUILD_OPENJPEG=OFF -DWITH_OPENJPEG=OFF \
	-DBUILD_SPNG=OFF -DWITH_SPNG=OFF \
	-DWITH_JPEGXL=OFF \
	\
	-DWITH_IMGCODEC_HDR=OFF -DWITH_IMGCODEC_SUNRASTER=OFF -DWITH_IMGCODEC_PXM=OFF -DWITH_IMGCODEC_PFM=OFF \
	\
	-DWITH_ANDROID_MEDIANDK=OFF \
	\
	-DWITH_AVIF=OFF \
	-DBUILD_JAVA=OFF -DBUILD_opencv_java_bindings_generator=OFF -DBUILD_FAT_JAVA_LIB=OFF \
	-DBUILD_opencv_js=OFF -DBUILD_opencv_js_bindings_generator=OFF \
	-DWITH_TESSERACT=OFF \
	\
    -DBUILD_gen_opencv_python_source=OFF -DBUILD_opencv_python2=OFF -DBUILD_opencv_python3=OFF -DBUILD_opencv_python_tests=OFF  \
	-DCPU_BASELINE=SSE3 -DCPU_DISPATCH=SSE4_1,SSE4_2 -DENABLE_LTO=ON \
	\
	-DBUILD_opencv_viz=OFF \
    -DBUILD_opencv_bgsegm=ON \
    \
    -DWITH_PROTOBUF=ON -DBUILD_PROTOBUF=ON \
    \
    -DBUILD_opencv_dnn=ON \
    -DBUILD_opencv_dnn_plugins=OFF \
    -DBUILD_opencv_dnn_superres=OFF \
    -DBUILD_opencv_dnn_objdetect=OFF \
    \
    -DBUILD_opencv_flann=OFF \
    -DBUILD_opencv_intensity_transform=OFF \
    -DBUILD_opencv_ml=OFF \
    -DBUILD_opencv_phase_unwrapping=OFF \
    -DBUILD_opencv_photo=OFF \
    -DBUILD_opencv_plot=OFF \
    -DBUILD_opencv_quality=OFF \
    -DBUILD_opencv_reg=OFF \
    -DBUILD_opencv_signal=OFF \
    -DBUILD_opencv_surface_matching=OFF \
    -DBUILD_opencv_xphoto=OFF \
    -DBUILD_opencv_alphamat=OFF \
    -DBUILD_opencv_features2d=OFF \
    -DBUILD_opencv_freetype=OFF \
    -DBUILD_opencv_fuzzy=OFF \
    -DBUILD_opencv_hfs=OFF \
    -DBUILD_opencv_img_hash=OFF \
    -DBUILD_IlmImf=OFF \
    -DBUILD_opencv_line_descriptor=OFF \
    -DBUILD_opencv_saliency=OFF \
    -DBUILD_opencv_text=OFF \
    -DBUILD_opencv_calib3d=OFF \
    -DBUILD_opencv_datasets=OFF \
    -DBUILD_opencv_mcc=OFF \
    -DBUILD_opencv_objdetect=OFF \
    -DBUILD_opencv_rapid=OFF \
    -DBUILD_opencv_rgbd=OFF \
    -DBUILD_opencv_shape=OFF \
    -DBUILD_opencv_structured_light=OFF \
    -DBUILD_opencv_videostab=OFF \
    -DBUILD_opencv_wechat_qrcode=OFF \
    -DBUILD_opencv_xfeatures2d=OFF \
    -DBUILD_opencv_ximgproc=OFF \
    -DBUILD_opencv_xobjdetect=OFF \
    -DBUILD_opencv_waldboost_detector=OFF \
    -DBUILD_opencv_aruco=OFF \
    -DBUILD_opencv_bioinspired=OFF \
    -DBUILD_opencv_ccalib=OFF \
    -DBUILD_opencv_dpm=OFF \
    -DBUILD_opencv_face=OFF \
    -DWITH_ADE=OFF \
    -DBUILD_opencv_superres=OFF \
    -DBUILD_opencv_tracking=OFF \
    -DBUILD_opencv_stereo=OFF \
    -DBUILD_opencv_annotation=OFF \
    -DBUILD_opencv_visualisation=OFF \
    -DBUILD_opencv_interactive-calibration=OFF \
    -DBUILD_opencv_model_diagnostics=OFF
```

## When compiling your program with the installed openCV you might need to specify some paths

`OpenCV_DIR` must point to the directory that contains `OpenCVConfig.cmake` (not just the install prefix).

```
-DOpenCV_DIR=path-to-dir-containing-OpenCVConfig.cmake -DEigen3_DIR=path-to/share/eigen3/cmake
```

ex:

```bash
-DOpenCV_DIR=/usr/local/lib/cmake/opencv4 -DEigen3_DIR=/usr/local/share/eigen3/cmake
# alternatively, pass the install prefix: -DCMAKE_PREFIX_PATH=/usr/local
```


```
# minimal one line
cmake -S . -B build -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DBUILD_SHARED_LIBS=FALSE -DCMAKE_BUILD_TYPE=Release  -DOPENCV_EXTRA_MODULES_PATH="../opencv_contrib/modules" -DCMAKE_INSTALL_PREFIX="/usr/local" -DBUILD_PERF_TESTS=OFF -DBUILD_TESTS=OFF -DBUILD_DOCS=OFF  -DWITH_CUDA=OFF -DBUILD_EXAMPLES=OFF -DBUILD_opencv_apps=OFF -DINSTALL_CREATE_DISTRIB=ON -DOPENCV_ENABLE_ALLOCATOR_STATS=OFF -DBUILD_OPENEXR=OFF -DWITH_OPENEXR=OFF -DBUILD_JASPER=OFF -DWITH_JASPER=OFF -DBUILD_WEBP=OFF -DWITH_WEBP=OFF -DBUILD_TIFF=OFF -DWITH_TIFF=OFF -DBUILD_JPEG=ON -DBUILD_JPEG_TURBO_DISABLE=OFF -DWITH_JPEG=ON -DBUILD_OPENJPEG=OFF -DWITH_OPENJPEG=OFF -DBUILD_SPNG=OFF -DWITH_SPNG=OFF -DWITH_JPEGXL=OFF -DWITH_IMGCODEC_HDR=OFF -DWITH_IMGCODEC_SUNRASTER=OFF -DWITH_IMGCODEC_PXM=OFF -DWITH_IMGCODEC_PFM=OFF -DWITH_ANDROID_MEDIANDK=OFF -DWITH_AVIF=OFF -DBUILD_JAVA=OFF -DBUILD_opencv_java_bindings_generator=OFF -DBUILD_FAT_JAVA_LIB=OFF -DBUILD_opencv_js=OFF -DBUILD_opencv_js_bindings_generator=OFF -DWITH_TESSERACT=OFF -DBUILD_gen_opencv_python_source=OFF -DBUILD_opencv_python2=OFF -DBUILD_opencv_python3=OFF -DBUILD_opencv_python_tests=OFF  -DCPU_BASELINE=SSE3 -DCPU_DISPATCH=SSE4_1,SSE4_2 -DENABLE_LTO=ON -DBUILD_opencv_viz=OFF -DBUILD_opencv_bgsegm=ON -DWITH_PROTOBUF=ON -DBUILD_PROTOBUF=ON -DBUILD_opencv_dnn=ON -DBUILD_opencv_dnn_plugins=OFF -DBUILD_opencv_dnn_superres=OFF -DBUILD_opencv_dnn_objdetect=OFF -DBUILD_opencv_flann=OFF -DBUILD_opencv_intensity_transform=OFF -DBUILD_opencv_ml=OFF -DBUILD_opencv_phase_unwrapping=OFF -DBUILD_opencv_photo=OFF -DBUILD_opencv_plot=OFF -DBUILD_opencv_quality=OFF -DBUILD_opencv_reg=OFF -DBUILD_opencv_signal=OFF -DBUILD_opencv_surface_matching=OFF -DBUILD_opencv_xphoto=OFF -DBUILD_opencv_alphamat=OFF -DBUILD_opencv_features2d=OFF -DBUILD_opencv_freetype=OFF -DBUILD_opencv_fuzzy=OFF -DBUILD_opencv_hfs=OFF -DBUILD_opencv_img_hash=OFF -DBUILD_IlmImf=OFF -DBUILD_opencv_line_descriptor=OFF -DBUILD_opencv_saliency=OFF -DBUILD_opencv_text=OFF -DBUILD_opencv_calib3d=OFF -DBUILD_opencv_datasets=OFF -DBUILD_opencv_mcc=OFF -DBUILD_opencv_objdetect=OFF -DBUILD_opencv_rapid=OFF -DBUILD_opencv_rgbd=OFF -DBUILD_opencv_shape=OFF -DBUILD_opencv_structured_light=OFF -DBUILD_opencv_videostab=OFF -DBUILD_opencv_wechat_qrcode=OFF -DBUILD_opencv_xfeatures2d=OFF -DBUILD_opencv_ximgproc=OFF -DBUILD_opencv_xobjdetect=OFF -DBUILD_opencv_waldboost_detector=OFF -DBUILD_opencv_aruco=OFF -DBUILD_opencv_bioinspired=OFF -DBUILD_opencv_ccalib=OFF -DBUILD_opencv_dpm=OFF -DBUILD_opencv_face=OFF -DWITH_ADE=OFF -DBUILD_opencv_superres=OFF -DBUILD_opencv_tracking=OFF -DBUILD_opencv_stereo=OFF -DBUILD_opencv_annotation=OFF -DBUILD_opencv_visualisation=OFF -DBUILD_opencv_interactive-calibration=OFF -DBUILD_opencv_model_diagnostics=OFF
```

Be careful when disabling some modules, if one of a module's dependencies is disabled, cmake will automatically disable that module too, for example if we say `-DBUILD_opencv_bgsegm=ON` but we disabled one of the following modules `opencv_core`, `opencv_imgproc`, `opencv_video`, `opencv_calib3d`, `WRAP`,  `python`, `java`, `objc`
then `bgsegm` will be disabled too. look for `ocv_define_module` and `ocv_add_module`

to know which modules a module depends on, look for the `CMakeLists.txt` of each module in the opencv_contrib https://github.com/opencv/opencv_contrib/tree/master/modules repository
 and the official opencv repository https://github.com/opencv/opencv/tree/4.x/modules
