# tensorflow-aarch64-crossbuild
cross build tensorflow for aarch64 ubuntu 16.04 in centos 7.4 host

1. Install ubuntu in centos7.4 for chroot

   refer to https://help.ubuntu.com/community/DebootstrapChroot,
   download debootstrap-1.0.78+nmu1ubuntu1
   and install ubuntu 16.04

2. In chroot env, refer to https://www.tensorflow.org/install/install_sources, 
   install bazel(https://docs.bazel.build/versions/master/install-ubuntu.html),
   aarch64 cross tool, and clone tensorflow.
```
   cym@allegro:~/project$ sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```
   
3. In tensorflow code dir, add following at the bottom of file WORKSPACE
```
    new_local_repository(
      name = "aarch64_compiler",
      path = "/",
      build_file = "aarch64_compiler.BUILD",
    )
```

4. In tensorflow code dir, create file aarch64_compiler.BUILD with following content.
```
[cym@allegro test]$ cat aarch64_compiler.BUILD 

package(default_visibility = ['//visibility:public'])

filegroup(
  name = 'gcc',
  srcs = [
    'usr/bin/aarch64-linux-gnu-gcc',
  ],
)

filegroup(
  name = 'ar',
  srcs = [
    'usr/bin/aarch64-linux-gnu-ar',
  ],
)

filegroup(
  name = 'ld',
  srcs = [
    'usr/bin/aarch64-linux-gnu-ld',
  ],
)

filegroup(
  name = 'nm',
  srcs = [
    'usr/bin/aarch64-linux-gnu-nm',
  ],
)

filegroup(
  name = 'objcopy',
  srcs = [
    'usr/bin/aarch64-linux-gnu-objcopy',
  ],
)

filegroup(
  name = 'objdump',
  srcs = [
    'usr/bin/aarch64-linux-gnu-objdump',
  ],
)

filegroup(
  name = 'strip',
  srcs = [
    'usr/bin/aarch64-linux-gnu-strip',
  ],
)

filegroup(
  name = 'as',
  srcs = [
    'usr/bin/aarch64-linux-gnu-as',
  ],
)

filegroup(
  name = 'compiler_pieces',
  srcs = glob([
    'usr/lib/gcc-cross/aarch64-linux-gnu/5/**',
    'usr/aarch64-linux-gnu/**',
  ]),
)

filegroup(
  name = 'compiler_components',
  srcs = [
    ':gcc',
    ':ar',
    ':ld',
    ':nm',
    ':objcopy',
    ':objdump',
    ':strip',
    ':as',
  ],
)
```
5. In tensorflow code dir, run "mkdir -p tools/aarch64_compiler/" and create files CROSSTOOL, BUILD.
```
[cym@allegro test]$ cat tools/aarch64_compiler/BUILD 

package(default_visibility = ["//visibility:public"])

cc_toolchain_suite(
  name = 'toolchain',
  toolchains = {
  'aarch64|compiler':':gcc-linux-aarch64',
  },
)

filegroup(
    name = "empty",
    srcs = [],
)

cc_toolchain(
  name = 'gcc-linux-aarch64',
  all_files = ':empty',
  compiler_files = ':empty',
  cpu = 'aarch64',
  dwp_files = ':empty',
  dynamic_runtime_libs = [':empty'],
  linker_files = ':empty',
  objcopy_files = 'empty',
  static_runtime_libs = [':empty'],
  strip_files = 'empty',
  supports_param_files = 1,
)

[cym@allegro test]$ cat tools/aarch64_compiler/CROSSTOOL
major_version: "local"
minor_version: ""
default_target_cpu: "aarch64"

default_toolchain {
  cpu: "aarch64"
  toolchain_identifier: "aarch64-linux-gnu"
}

toolchain {
  abi_version: "aarch64"
  abi_libc_version: "aarch64"
  builtin_sysroot: ""
  compiler: "compiler"
  host_system_name: "aarch64"
  needsPic: true
  supports_gold_linker: true
  supports_incremental_linker: false
  supports_fission: false
  supports_interface_shared_objects: false
  supports_normalizing_ar: false
  supports_start_end_lib: true
  target_libc: "aarch64"
  target_cpu: "aarch64"
  target_system_name: "aarch64"
  toolchain_identifier: "aarch64-linux-gnu"

  cxx_flag: "-std=c++11"
  linker_flag: "-lstdc++"
  linker_flag: "-lm"
  linker_flag: "-fuse-ld=gold"
  linker_flag: "-Wl,-no-as-needed"
  linker_flag: "-Wl,-z,relro,-z,now"
  linker_flag: "-pass-exit-codes"

  cxx_builtin_include_directory: "/usr/aarch64-linux-gnu/include/c++/5/"
  cxx_builtin_include_directory: "/usr/aarch64-linux-gnu/include/c++/5/backward"
  cxx_builtin_include_directory: "/usr/aarch64-linux-gnu/include/"
  cxx_builtin_include_directory: "/usr/lib/gcc-cross/aarch64-linux-gnu/5/include"
  cxx_builtin_include_directory: "/usr/lib/gcc-cross/aarch64-linux-gnu/5/include-fixed"

  objcopy_embed_flag: "-I"
  objcopy_embed_flag: "binary"

  unfiltered_cxx_flag: "-fno-canonical-system-headers"
  unfiltered_cxx_flag: "-Wno-builtin-macro-redefined"
  unfiltered_cxx_flag: "-D__DATE__=\"redacted\""
  unfiltered_cxx_flag: "-D__TIMESTAMP__=\"redacted\""
  unfiltered_cxx_flag: "-D__TIME__=\"redacted\""
  compiler_flag: "-U_FORTIFY_SOURCE"
  compiler_flag: "-fstack-protector"
  compiler_flag: "-Wall"
  compiler_flag: "-Wunused-but-set-parameter"
  compiler_flag: "-Wno-free-nonheap-object"
  compiler_flag: "-fno-omit-frame-pointer"

  tool_path { name: "ld" path: "/usr/bin/aarch64-linux-gnu-ld" }
  tool_path { name: "cpp" path: "/usr/bin/aarch64-linux-gnu-cpp" }
  tool_path { name: "dwp" path: "/usr/bin/aarch64-linux-gnu-dwp" }
  tool_path { name: "gcov" path: "/usr/bin/aarch64-linux-gnu-gcov" }
  tool_path { name: "nm" path: "/usr/bin/aarch64-linux-gnu-nm" }
  tool_path { name: "objcopy" path: "/usr/bin/aarch64-linux-gnu-objcopy" }
  tool_path { name: "objdump" path: "/usr/bin/aarch64-linux-gnu-objdump" }
  tool_path { name: "strip" path: "/usr/bin/aarch64-linux-gnu-strip" }
  tool_path { name: "gcc" path: "/usr/bin/aarch64-linux-gnu-gcc" }
  tool_path { name: "ar" path: "/usr/bin/aarch64-linux-gnu-ar" }

  compilation_mode_flags {
    mode: DBG
    # Enable debug symbols.
    compiler_flag: "-g"
  }
  compilation_mode_flags {
    mode: OPT

    # No debug symbols.
    # Maybe we should enable https://gcc.gnu.org/wiki/DebugFission for opt or
    # even generally? However, that can't happen here, as it requires special
    # handling in Bazel.
    compiler_flag: "-g0"

    # Conservative choice for -O
    # -O3 can increase binary size and even slow down the resulting binaries.
    # Profile first and / or use FDO if you need better performance than this.
    compiler_flag: "-O2"
    compiler_flag: "-D_FORTIFY_SOURCE=1"

    # Disable assertions
    compiler_flag: "-DNDEBUG"

    # Removal of unused code and data at link time (can this increase binary size in some cases?).
    compiler_flag: "-ffunction-sections"
    compiler_flag: "-fdata-sections"
    linker_flag: "-Wl,--gc-sections"
  }
  linking_mode_flags { mode: DYNAMIC }

    feature {
      name: 'coverage'
      provides: 'profile'
      flag_set {
        action: 'preprocess-assemble'
        action: 'c-compile'
        action: 'c++-compile'
        action: 'c++-header-parsing'
        action: 'c++-header-preprocessing'
        action: 'c++-module-compile'
        flag_group {
        flag: '-fprofile-arcs'
        flag: '-ftest-coverage'
        }
      }

      flag_set {
        action: 'c++-link-interface-dynamic-library'
        action: 'c++-link-dynamic-library'
        action: 'c++-link-executable'
        flag_group {
        flag: '-lgcov'
        }
      }
    }

}
```
6. In chroot env, configure tensorflow
```
cym@allegro:~/project/tensorflow/test$ ./configure 
You have bazel 0.7.0 installed.
Please specify the location of python. [Default is /usr/bin/python]: 


Found possible Python library paths:
  /usr/local/lib/python2.7/dist-packages
  /usr/lib/python2.7/dist-packages
Please input the desired Python library path to use.  Default is [/usr/local/lib/python2.7/dist-packages]

Do you wish to build TensorFlow with jemalloc as malloc support? [Y/n]: 
jemalloc as malloc support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Google Cloud Platform support? [Y/n]: n
No Google Cloud Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Hadoop File System support? [Y/n]: n
No Hadoop File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Amazon S3 File System support? [Y/n]: n
No Amazon S3 File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with XLA JIT support? [y/N]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with GDR support? [y/N]: n
No GDR support will be enabled for TensorFlow.

Do you wish to build TensorFlow with VERBS support? [y/N]: n
No VERBS support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL support? [y/N]: n
No OpenCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: n
No CUDA support will be enabled for TensorFlow.

Do you wish to build TensorFlow with MPI support? [y/N]: n
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:  


Add "--config=mkl" to your bazel command to build with MKL support.
Please note that MKL on MacOS or windows is still not supported.
If you would like to use a local MKL instead of downloading, please set the environment variable "TF_MKL_ROOT" every time before build.
Configuration finished
```

8. In chroot env, run build
```
cym@allegro:~/project/tensorflow/test$ bazel build -c opt //tensorflow/examples/label_image --cpu=aarch64 --crosstool_top=//tools/aarch64_compiler:toolchain --host_crosstool_top=@bazel_tools//tools/cpp:toolchain --verbose_failures
```
when find  "/external/nsync/BUILD:402:13: Configurable attribute "copts" doesn't match this configuration (would a default condition help?).", refer to  https://lengerrong.blogspot.nl/2017/09/fix-up-configurable-attribute-copts.html


when prompt "can't find aarch64-linux-gnu/python2.7/pyconfig.h"
download libpython2.7-dev:arm64 and extract it to get /usr/include/aarch64-linux-gnu/python2.7/pyconfig.h
   in Ubuntu.

9. copy bazel-bin/tensorflow/examples/label_image/label_image and 
/home/cym/.cache/bazel/_bazel_cym/2ca964ca343a882262a4dc7d61ff9eb3/execroot/org_tensorflow/bazel-out/aarch64-linux-gnu-opt/bin/tensorflow/libtensorflow_framework.so to aarch64 device 

Reference:  
 1. https://github.com/snipsco/tensorflow-build.git
