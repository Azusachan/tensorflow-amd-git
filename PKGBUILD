# Maintainer: anakano <azusanakan0 at outlook dot com>
# Contributor: wuxxin <wuxxin@gmail.com>
# Contributor: acxz <akashpatel2008 at yahoo dot com>
# Contributor: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Contributor: Konstantin Gizdov (kgizdov) <arch@kge.pw>
# Contributor: Adria Arrufat (archdria) <adria.arrufat+AUR@protonmail.ch>
# Contributor: Thibault Lorrain (fredszaq) <fredszaq@gmail.com>

pkgbase=tensorflow-amd-git

# Flags for building without/with cpu optimizations
_build_no_opt=1
_build_opt=1

pkgname=()
[ "$_build_no_opt" -eq 1 ] && pkgname+=(tensorflow-amd-git python-tensorflow-amd-git)
[ "$_build_opt" -eq 1 ] && pkgname+=(tensorflow-opt-amd-git python-tensorflow-opt-amd-git)

pkgver=2.11.0
_srcname="tensorflow"
_pkgver=2.11.0
pkgrel=1
pkgdesc="Library for computation using data flow graphs for scalable machine learning (AMD upstream)"
url="https://www.tensorflow.org/"
license=('APACHE')
arch=('x86_64')
depends=('c-ares' 'pybind11' 'openssl' 'lmdb' 'libpng' 'curl' 'giflib' 'icu' 'libjpeg-turbo' 'openmp')
makedepends=('bazel' 'python-numpy' 'rocm-hip-sdk' 'rccl' 'git' 'miopen' 'python-pip' 'python-wheel'
             'python-setuptools' 'python-h5py' 'python-keras-applications' 'python-keras-preprocessing'
             'cython' 'patchelf' 'python-requests')
optdepends=('tensorboard: Tensorflow visualization toolkit')
source=("tensorflow::git+https://github.com/ROCmSoftwarePlatform/tensorflow-upstream.git#branch=r2.11-rocm-enhanced"
        fix-c++17-compat.patch
        fix-cusolver-version.patch
        remove-log-spam.patch)

sha512sums=('SKIP'
            'e919b77f17c8508c21607a988e4451542900bb2a1453b55bff865e3b2602faebf4543bd640f60d84ac9b8d7b8599986f115077f7d411732fba3866cb5d6ad2e2'
            'fc263a0d0d318899914b26e8e06b3beb7c8e55118f5eb54c94fd97d8749151decfe6bf399130db43481d12908f4fdb58ff388acbcd90507d84f1d06d96f6019f'
            'fde73feeb2bbb814ba229c2b879e5e5944fd658e9810937753a25f2650f57c49f8a435924b47a1a54eb2852f9713b19a15d42b307593e26a74ad65aeee22c36a')

# consolidate common dependencies to prevent mishaps
_common_py_depends=(python-termcolor python-astor python-gast03 python-numpy python-protobuf
                    absl-py python-h5py python-keras python-keras-applications python-keras-preprocessing
                    python-tensorflow-estimator python-opt_einsum python-astunparse python-pasta
                    python-flatbuffers python-typing_extensions)

get_pyver () {
  python -c 'import sys; print(str(sys.version_info[0]) + "." + str(sys.version_info[1]))'
}

check_dir() {
  # first make sure we do not break parsepkgbuild
  if ! command -v cp &> /dev/null; then
    >&2 echo "'cp' command not found. PKGBUILD is probably being checked by parsepkgbuild."
    if ! command -v install &> /dev/null; then
      >&2 echo "'install' command also not found. PKGBUILD must be getting checked by parsepkgbuild."
      >&2 echo "Cannot check if directory '${1}' exists. Ignoring."
      >&2 echo "If you are not running nacmap or parsepkgbuild, please make sure the PATH is correct and try again."
      >&2 echo "PATH should not be '/dummy': PATH=$PATH"
      return 0
    fi
  fi
  # if we are running normally, check the given path
  if [ -d "${1}" ]; then
    return 0
  else
    >&2 echo Directory "${1}" does not exist or is a file! Exiting...
    exit 1
  fi
}

prepare() {
  # Allow any bazel version
  echo "*" > $_srcname/.bazelversion

  # Get rid of hardcoded versions. Not like we ever cared about what upstream
  # thinks about which versions should be used anyway. ;) (FS#68772)
  sed -i -E "s/'([0-9a-z_-]+) .= [0-9].+[0-9]'/'\1'/" $_srcname/tensorflow/tools/pip_package/setup.py

  cd "${srcdir}/$_srcname"
  # change rocblas.h to rocblas/rocblas.h
  sed -i 's/rocblas.h"/rocblas\/rocblas.h"/g' tensorflow/stream_executor/rocm/rocm_blas.h
  sed -i 's/rocblas.h"/rocblas\/rocblas.h"/g' tensorflow/core/util/gpu_solvers.h
  sed -i 's/rocm\/include\/rocblas.h"/rocblas\/rocblas.h"/g' tensorflow/stream_executor/rocm/rocblas_wrapper.h
  sed -i 's/rocblas.h"/rocblas\/rocblas.h"/g' tensorflow/compiler/xla/stream_executor/rocm/rocm_blas.cc
  sed -i 's/rocblas.h"/rocblas\/rocblas.h"/g' tensorflow/compiler/xla/stream_executor/rocm/rocm_blas.h
  sed -i 's/rocblas.h"/rocblas\/rocblas.h"/g' tensorflow/compiler/xla/stream_executor/rocm/rocblas_wrapper.h

  patch -Np1 -i "${srcdir}/fix-cusolver-version.patch"
  patch -Np1 -i "${srcdir}/remove-log-spam.patch"
  cd "${srcdir}"

  cp -r $_srcname tensorflow-amd-git
  cp -r $_srcname tensorflow-opt-amd-git

  # These environment variables influence the behavior of the configure call below.
  export PYTHON_BIN_PATH=/usr/bin/python
  export USE_DEFAULT_PYTHON_LIB_PATH=1
  export TF_NEED_JEMALLOC=1
  export TF_NEED_KAFKA=1
  export TF_NEED_OPENCL_SYCL=0
  export TF_NEED_AWS=1
  export TF_NEED_GCP=1
  export TF_NEED_HDFS=1
  export TF_NEED_S3=1
  export TF_ENABLE_XLA=1
  export TF_NEED_GDR=0
  export TF_NEED_VERBS=0
  export TF_NEED_OPENCL=0
  export TF_NEED_MPI=0
  export TF_NEED_TENSORRT=0
  export TF_NEED_NGRAPH=0
  export TF_NEED_IGNITE=0
  export TF_NEED_ROCM=1
  # Uncomment this when you want to specify specific ROCM_ARCH(s)
  # Otherwise tensorflow will automatically detect your architecture
  # See: https://github.com/tensorflow/tensorflow/commit/c04822a49d669f2d74a566063852243d993e18b1
  # export TF_ROCM_AMDGPU_TARGETS=gfx803,gfx900,gfx904,gfx906,gfx908
  # See https://github.com/tensorflow/tensorflow/blob/master/third_party/systemlibs/syslibs_configure.bzl
  export TF_SYSTEM_LIBS="boringssl,curl,cython,gif,icu,libjpeg_turbo,lmdb,nasm,png,pybind11,zlib"
  export TF_SET_ANDROID_WORKSPACE=0
  export TF_DOWNLOAD_CLANG=0
  export TF_NCCL_VERSION=$(pkg-config nccl --modversion | grep -Po '\d+\.\d+')
  export TF_IGNORE_MAX_BAZEL_VERSION=1
  export NCCL_INSTALL_PATH=/usr
  # Does tensorflow really need the compiler overridden in 5 places? Yes.
  export CC=gcc-11
  export CXX=g++-11
  export GCC_HOST_COMPILER_PATH=/usr/bin/${CC}
  export HOST_C_COMPILER=/usr/bin/${CC}
  export HOST_CXX_COMPILER=/usr/bin/${CXX}
  export TF_CUDA_CLANG=0  # Clang currently disabled because it's not compatible at the moment.
  export CLANG_CUDA_COMPILER_PATH=/usr/bin/clang
  export TF_CUDA_PATHS=/opt/cuda,/usr/lib,/usr
  export TF_CUDA_VERSION=$(/opt/cuda/bin/nvcc --version | sed -n 's/^.*release \(.*\),.*/\1/p')
  export TF_CUDNN_VERSION=$(sed -n 's/^#define CUDNN_MAJOR\s*\(.*\).*/\1/p' /usr/include/cudnn_version.h)
  # https://github.com/tensorflow/tensorflow/blob/1ba2eb7b313c0c5001ee1683a3ec4fbae01105fd/third_party/gpus/cuda_configure.bzl#L411-L446
  # according to the above, we should be specifying CUDA compute capabilities as 'sm_XX' or 'compute_XX' from now on
  # add latest PTX for future compatibility
  export
  TF_CUDA_COMPUTE_CAPABILITIES=sm_52,sm_53,sm_60,sm_61,sm_62,sm_70,sm_72,sm_75,sm_80,sm_86,sm_87,compute_87

  export BAZEL_ARGS="--config=mkl -c opt"
}

build() {
  if [ "$_build_no_opt" -eq 1 ]; then
    echo "Building with rocm and without non-x86-64 optimizations"
    cd "${srcdir}"/tensorflow-amd-git
    export CC_OPT_FLAGS="-march=x86-64"
    export TF_NEED_CUDA=0
    export TF_NEED_ROCM=1
    ./configure
    bazel \
      build \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:build_pip_package
    bazel-bin/tensorflow/tools/pip_package/build_pip_package --gpu "${srcdir}"/tmprocm
  fi


  if [ "$_build_opt" -eq 1 ]; then
    echo "Building with rocm and with non-x86-64 optimizations"
    cd "${srcdir}"/tensorflow-opt-amd-git
    export CC_OPT_FLAGS="-march=haswell -O3"
    export TF_NEED_CUDA=0
    export TF_NEED_ROCM=1
    ./configure
    bazel \
      build --config=avx2_linux \
        ${BAZEL_ARGS[@]} \
        //tensorflow:libtensorflow.so \
        //tensorflow:libtensorflow_cc.so \
        //tensorflow:install_headers \
        //tensorflow/tools/pip_package:build_pip_package
    bazel-bin/tensorflow/tools/pip_package/build_pip_package --gpu "${srcdir}"/tmpoptrocm
  fi
}

_package() {
  # install headers first
  install -d "${pkgdir}"/usr/include/tensorflow
  cp -r bazel-bin/tensorflow/include/* "${pkgdir}"/usr/include/tensorflow/
  # install python-version to get all extra headers
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  pip install --ignore-installed --upgrade --root "${pkgdir}"/ $WHEEL_PACKAGE --no-dependencies
  # move extra headers to correct location
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  find "${_srch_path}" -maxdepth 1 -mindepth 1 -type d -print0 | while read -rd $'\0' _folder; do
    cp -nr "${_folder}" "${pkgdir}"/usr/include/tensorflow/
  done
  # clean up unneeded files
  rm -rf "${pkgdir}"/usr/bin
  rm -rf "${pkgdir}"/usr/lib
  rm -rf "${pkgdir}"/usr/share
  # make sure no lib objects are outside valid paths
  local _so_srch_path="${pkgdir}/usr/include"
  check_dir "${_so_srch_path}"  # we need to quit on broken search paths
  find "${_so_srch_path}" -type f,l \( -iname "*.so" -or -iname "*.so.*" \) -print0 | while read -rd $'\0' _so_file; do
    # check if file is a dynamic executable
    ldd "${_so_file}" &>/dev/null && rm -rf "${_so_file}"
  done

  # install the rest of tensorflow
  tensorflow/c/generate-pc.sh --prefix=/usr --version=${pkgver}
  sed -e 's@/include$@/include/tensorflow@' -i tensorflow.pc -i tensorflow_cc.pc
  install -Dm644 tensorflow.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow.pc
  install -Dm644 tensorflow_cc.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow_cc.pc
  install -Dm755 bazel-bin/tensorflow/libtensorflow.so "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver}
  ln -s libtensorflow.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver:0:1}
  ln -s libtensorflow.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_cc.so "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver}
  ln -s libtensorflow_cc.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver:0:1}
  ln -s libtensorflow_cc.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_cc.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_framework.so "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver}
  ln -s libtensorflow_framework.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver:0:1}
  ln -s libtensorflow_framework.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_framework.so
  install -Dm644 tensorflow/c/c_api.h "${pkgdir}"/usr/include/tensorflow/tensorflow/c/c_api.h
  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE

  # Fix interoperability of C++14 and C++17. See https://bugs.archlinux.org/task/65953
  patch -Np0 -i "${srcdir}"/fix-c++17-compat.patch -d "${pkgdir}"/usr/include/tensorflow/absl/base

  # Fix FS#75571
  find "${pkgdir}"/usr/lib -type f -exec patchelf --replace-needed libiomp5.so libomp.so '{}' \; -print
}

_python_package() {
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  pip install --ignore-installed --upgrade --root "${pkgdir}"/ $WHEEL_PACKAGE --no-dependencies

  # create symlinks to headers
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include/
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  find "${_srch_path}" -maxdepth 1 -mindepth 1 -type d -print0 | while read -rd $'\0' _folder; do
    rm -rf "${_folder}"
    _smlink="$(basename "${_folder}")"
    ln -s /usr/include/tensorflow/"${_smlink}" "${_srch_path}"
  done

  # tensorboard has been separated from upstream but they still install it with
  # tensorflow. I don't know what kind of sense that makes but we have to clean
  # it out from this pacakge.
  rm -rf "${pkgdir}"/usr/bin/tensorboard

  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE
}

package_tensorflow-amd-git() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (AMD upstream with ROCM)"
  depends+=(rocm-hip-sdk miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow)

  cd "${srcdir}"/tensorflow-amd-git
  _package tmprocm
}

package_tensorflow-opt-amd-git() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (AMD upstream with ROCM and AVX2 CPU optimizations)"
  depends+=(rocm-hip-sdk miopen rccl)
  conflicts=(tensorflow)
  provides=(tensorflow tensorflow-amd-git)

  cd "${srcdir}"/tensorflow-opt-amd-git
  _package tmpoptrocm
}

package_python-tensorflow-amd-git() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (AMD upstream with ROCM)"
  depends+=(tensorflow-amd-git rocm-hip-sdk miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow)

  cd "${srcdir}"/tensorflow-amd-git
  _python_package tmprocm
}

package_python-tensorflow-opt-amd-git() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (AMD upstream with ROCM and AVX2 CPU optimizations)"
  depends+=(tensorflow-opt-amd-git rocm-hip-sdk miopen rccl "${_common_py_depends[@]}")
  conflicts=(python-tensorflow)
  provides=(python-tensorflow python-tensorflow-amd-git)

  cd "${srcdir}"/tensorflow-opt-amd-git
  _python_package tmpoptrocm
}

# vim:set ts=2 sw=2 et:
