# week-00-Challenge

[LFX Mentorship Summer 2022 Challenge · Discussion #1442 · WasmEdge/WasmEdge (github.com)](https://github.com/WasmEdge/WasmEdge/discussions/1442)

Target： add brand new host functions

## Background knowledge

- [通过 Host Function，打开 WebAssembly 新世界\_serverless 技术社区的博客-CSDN 博客](https://blog.csdn.net/weixin_42376823/article/details/123261219)
- [别再抱怨 Wasm 能力有限了， Host Function 用起来呀 (qq.com)](https://mp.weixin.qq.com/s/cVadim-euH0nNnsqWk8TFw)
- [Build for Windows - WasmEdge Runtime](https://wasmedge.org/book/en/extend/build_on_windows.html)
- [Build WasmEdge from source - WasmEdge Runtime](https://wasmedge.org/book/en/extend/build.html) <- 不要折腾 windows 了，还是 docker 的开发流程最简单。
- 编译命令换一下 `cmake -DCMAKE_BUILD_TYPE=Release -DWASMEDGE_BUILD_AOT_RUNTIME=OFF -DWASMEDGE_BUILD_TESTS=OFF .. && make -j8`

## Prepare environment

Conveniently, I pulled the `wasmedge/wasmedge: latest` docker image, `docker run -d -t wasmedge/wasmedge:latest` and clone my fork into this image directly.

Check out the personal branch from the latest master, my branch name is `lfx-challenge-tieway59`.

```shell
# Instal tools
choco install cmake ninja vswhere

$vsPath = (vswhere -latest -property installationPath)
Import-Module (Join-Path $vsPath "Common7\Tools\Microsoft.VisualStudio.DevShell.dll")
Enter-VsDevShell -VsInstallPath $vsPath -SkipAutomaticLocation -DevCmdArguments "-arch=x64 -host_arch=x64 -winsdk=10.0.19041.0"

# Download our pre-built LLVM 13
$llvm = "LLVM-13.0.0-win64.zip"
curl -sLO https://github.com/WasmEdge/llvm-windows/releases/download/llvmorg-13.0.0/LLVM-13.0.0-win64.zip -o $llvm
# if you are using latest PowerShell, please don't add `-sLO`
Expand-Archive -Path $llvm

# Set LLVM environment
$llvm_dir = "$pwd\\LLVM-13.0.0-win64\\LLVM-13.0.0-win64\\lib\\cmake\\llvm"
$Env:CC = "clang-cl"
$Env:CXX = "clang-cl"
```

independent shell for me to compile:

```shell
cd D:\Twy59sGthb\WasmEdge
$vsPath = (vswhere -latest -property installationPath)
Import-Module (Join-Path $vsPath "Common7\Tools\Microsoft.VisualStudio.DevShell.dll")
Enter-VsDevShell -VsInstallPath $vsPath -SkipAutomaticLocation -DevCmdArguments "-arch=x64 -host_arch=x64 -winsdk=10.0.19041.0"

$llvm_dir = "D:\\PrgrmsFrDvlp\\LLVM-13.0.0-win64\\LLVM-13.0.0-win64\\lib\\cmake\\llvm"
$Env:CC = "clang-cl"
$Env:CXX = "clang-cl"

cmake -DCMAKE_BUILD_TYPE=Release -DWASMEDGE_BUILD_AOT_RUNTIME=OFF .
```

## Step 1. Create host function definitions

ref: [[Challenge Tutorial\] 1. Create host function definitions · WasmEdge/WasmEdge@b6c2d6c (github.com)](https://github.com/WasmEdge/WasmEdge/commit/b6c2d6c83cd49314014e127468fe1c8d142081e1)

create file `include/host/challenge_tieway59/tw59env.h`

`challenge_tieway59/tw59env.h` is my customed directory names.

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#pragma once

#include <cstdint>
#include <string>
#include <vector>

namespace WasmEdge {
namespace Host {

class HostFuncExampleEnvironment {
public:
  /// Define your environment variables and data structures

  /// Const

  /// Variables
  uint32_t ClassID;
  std::string ClassName;
  std::vector<std::string> Students;

  /// Initial Configurations
};

} // namespace Host
} // namespace WasmEdge
```

create file `include/host/challenge_tieway59/tw59base.h`

this file depends on `tw59env.h`

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#pragma once

#include "common/errcode.h"
#include "host/challenge_tieway59/tw59env.h"
#include "runtime/hostfunc.h"

namespace WasmEdge {
namespace Host {

template <typename T> class HostFuncExample : public Runtime::HostFunction<T> {
public:
  HostFuncExample(HostFuncExampleEnvironment &HostEnv)
      : Runtime::HostFunction<T>(0), Env(HostEnv) {}

protected:
  HostFuncExampleEnvironment &Env;
};

} // namespace Host
} // namespace WasmEdge
```

create file `include/host/challenge_tieway59/tw59func.h`

this file depends on `tw59base.h`

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#pragma once

#include "common/defines.h"
#include "host/challenge_tieway59/tw59base.h"

namespace WasmEdge {
namespace Host {

class HostFuncExampleSetClassID
    : public HostFuncExample<HostFuncExampleSetClassID> {
public:
  HostFuncExampleSetClassID(HostFuncExampleEnvironment &HostEnv)
      : HostFuncExample(HostEnv) {}
  Expect<void> body(Runtime::Instance::MemoryInstance *MemInst, uint32_t CID);
};

class HostFuncExampleAddStudent
    : public HostFuncExample<HostFuncExampleAddStudent> {
public:
  HostFuncExampleAddStudent(HostFuncExampleEnvironment &HostEnv)
      : HostFuncExample(HostEnv) {}
  Expect<uint32_t> body(Runtime::Instance::MemoryInstance *MemInst,
                        uint32_t StudentPtr, uint32_t StudentLen);
};

class HostFuncExampleSetClassName
    : public HostFuncExample<HostFuncExampleSetClassName> {
public:
  HostFuncExampleSetClassName(HostFuncExampleEnvironment &HostEnv)
      : HostFuncExample(HostEnv) {}
  Expect<void> body(Runtime::Instance::MemoryInstance *MemInst,
                    uint32_t ClassNamePtr, uint32_t ClassNameLen);
};

class HostFuncExamplePrint : public HostFuncExample<HostFuncExamplePrint> {
public:
  HostFuncExamplePrint(HostFuncExampleEnvironment &HostEnv)
      : HostFuncExample(HostEnv) {}
  Expect<void> body(Runtime::Instance::MemoryInstance *MemInst);
};

} // namespace Host
} // namespace WasmEdge
```

create file `include/host/challenge_tieway59/tw59module.h`

this file depends on `tw59env.h`

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#pragma once

#include "host/challenge_tieway59/tw59env.h"
// #include "runtime/importobj.h" <-- this file is gone in the latest head
// refer to Breaking changes 0.10.0-alpha.2 (2022-05-20)
#include "runtime/instance/module.h"

#include <cstdint>

namespace WasmEdge {
namespace Host {

class HostFuncExampleModule : public Runtime::Instance::ModuleInstance { // <-- use ModuleInstance instead of ImportObj
public:
  HostFuncExampleModule();

  HostFuncExampleEnvironment &getEnv() { return Env; }
  // error: member initializer 'ModuleInstance' does not name a non-static data member or base class
  const HostFuncExampleEnvironment &getEnv() const noexcept { return Env; }

private:
  HostFuncExampleEnvironment Env;
};

} // namespace Host
} // namespace WasmEdge
```

list of all the new files created in this step:

```text
include/host
            ├─challenge_tieway59
            │      tw59base.h
            │      tw59env.h
            │      tw59func.h
            │      tw59module.h
            └─wasi
```

## Step 2. Finish host function implementation

ref: [[Challenge Tutorial\] 2. Finish host function implementation · WasmEdge/WasmEdge@9d10176 (github.com)](https://github.com/WasmEdge/WasmEdge/commit/9d101761101cb2a4235da638767d78e1e75cac15)

create file `lib/host/challenge_tieway59/tw59func.cpp`

this file depends on `tw59func.h` but they are in different root directory.

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#include "host/challenge_tieway59/tw59func.h"

namespace WasmEdge {
namespace Host {

Expect<void> HostFuncExampleSetClassID::body(
    [[maybe_unused]] Runtime::Instance::MemoryInstance *MemInst, uint32_t CID) {
  Env.ClassID = CID;
  return {};
}

Expect<uint32_t>
HostFuncExampleAddStudent::body(Runtime::Instance::MemoryInstance *MemInst,
                                uint32_t StudentPtr, uint32_t StudentLen) {
  // Check memory instance from module.
  if (MemInst == nullptr) {
    return Unexpect(ErrCode::ExecutionFailed);
  }

  char *Student = MemInst->getPointer<char *>(StudentPtr);
  std::string NewStudent;
  std::copy_n(Student, StudentLen, std::back_inserter(NewStudent));
  Env.Students.push_back(std::move(NewStudent));
  return Env.Students.size();
}

Expect<void>
HostFuncExampleSetClassName::body(Runtime::Instance::MemoryInstance *MemInst,
                                  uint32_t ClassNamePtr,
                                  uint32_t ClassNameLen) {
  // Check memory instance from module.
  if (MemInst == nullptr) {
    return Unexpect(ErrCode::ExecutionFailed);
  }

  char *ClassName = MemInst->getPointer<char *>(ClassNamePtr);
  std::string NewClassName;
  std::copy_n(ClassName, ClassNameLen, std::back_inserter(NewClassName));
  Env.ClassName = std::move(NewClassName);
  return {};
}

Expect<void> HostFuncExamplePrint::body([
    [maybe_unused]] Runtime::Instance::MemoryInstance *MemInst) {
  std::cout << "Class ID: " << Env.ClassID << std::endl;
  std::cout << "Class Name: " << Env.ClassName << std::endl;
  for (auto &Student : Env.Students) {
    std::cout << "Student: " << Student << std::endl;
  }
  return {};
}

} // namespace Host
} // namespace WasmEdge
```

create file `lib/host/challenge_tieway59/tw59module.cpp`

this file depends on `tw59func.h` & `tw59module.h`

```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#include "host/challenge_tieway59/tw59module.h"

#include "host/challenge_tieway59/tw59func.h"

namespace WasmEdge {
namespace Host {

/// Register your functions in module.
HostFuncExampleModule::HostFuncExampleModule()
    : ImportObject("host_function_example") {
  addHostFunc("host_function_example_set_class_id",
              std::make_unique<HostFuncExampleSetClassID>(Env));
  addHostFunc("host_function_example_add_student",
              std::make_unique<HostFuncExampleAddStudent>(Env));
  addHostFunc("host_function_example_set_class_name",
              std::make_unique<HostFuncExampleSetClassName>(Env));
  addHostFunc("host_function_example_print",
              std::make_unique<HostFuncExamplePrint>(Env));
}

} // namespace Host
} // namespace WasmEdge
```

create file `lib/host/challenge_tieway59/CMakeLists.txt`

this file depends on `tw59func.cpp` & `tw59module.cpp`

```cpp
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2019-2022 Second State INC

wasmedge_add_library(wasmedgeHostModuleHostFuncExample
  tw59func.cpp
  tw59module.cpp
)

target_link_libraries(wasmedgeHostModuleHostFuncExample
  PUBLIC
  wasmedgeCommon
  wasmedgeSystem
)
```

add `wasmedgeHostModuleHostFuncExample` to `lib/vm/CMakeLists.txt`

this name is build target name of `lib/host/challenge_tieway59/CMakeLists.txt`

```cmake
  wasmedgeExecutor
  wasmedgeHostModuleWasi
  wasmedgeHostModuleWasmEdgeProcess  # in latest matser this entry is gone.
  wasmedgeHostModuleHostFuncExample # <-- here
)
```

list of all the new files created in this step:

```text
\lib\host
        │
        ├─challenge_tieway59
        │      CMakeLists.txt
        │      tw59func.cpp
        │      tw59module.cpp
        │
        └─wasi
```

## Step 3. Register the implementation in CMakeLists

[[Challenge Tutorial\] 3. Register the implementation in CMakeLists · WasmEdge/WasmEdge@e5df14e (github.com)](https://github.com/WasmEdge/WasmEdge/commit/e5df14ece51b1da5e2a86ecda60f86036f241122)

add `add_subdirectory(host_function_example)` into `lib/host/CMakeLists.txt`

```cmake
add_subdirectory(wasi)
add_subdirectory(wasmedge_process) # in latest matser this entry is gone.
add_subdirectory(challenge_tieway59) # <-- here use folder name!
```

## Step 4. Register example module in the CLI tool

[[Challenge Tutorial\] 4. Register example module in the CLI tool · WasmEdge/WasmEdge@9b2aad6 (github.com)](https://github.com/WasmEdge/WasmEdge/commit/9b2aad68be1ec6e74e664756bc7faa2d34903cd0)

in `tools/wasmedge/wasmedger.cpp`

```cpp
#include "common/filesystem.h"
#include "common/types.h"
#include "common/version.h"
#include "hhost/challenge_tieway59/tw59module.h" //+
#include "host/wasi/wasimodule.h"
#include "host/wasmedge_process/processmodule.h"
#include "po/argument_parser.h"

  const auto InputPath = std::filesystem::absolute(SoName.value());
  WasmEdge::VM::VM VM(Conf);

  // Register your module in VM.                      //+
  WasmEdge::Host::HostFuncExampleModule ExampleMod;   //+
  VM.registerModule(ExampleMod);                      //+

  WasmEdge::Host::WasiModule *WasiMod =
      dynamic_cast<WasmEdge::Host::WasiModule *>(
          VM.getImportModule(WasmEdge::HostRegistration::Wasi));
```

Till here, I finish setting the example code.

## Step 5. Test building

```shell
mkdir -p tw59build && cd tw59build
cmake -DCMAKE_BUILD_TYPE=Release -DWASMEDGE_BUILD_AOT_RUNTIME=OFF -DWASMEDGE_BUILD_TESTS=OFF  .. && make -j8
```

you can find the build result in the log.

## Step 6. rust examples

ref [Commits · second-state/wasmedge_hostfunctionexample_interface (github.com)](https://github.com/second-state/wasmedge_hostfunctionexample_interface/commits/master)

```shell
cd ~/WasmEdge/examples
cargo new tw59challenge
rustup target add wasm32-wasi
```

`tw59challenge/.cargo/config.toml`

```toml
[build]
target="wasm32-wasi"

[target.wasm32-wasi]
runner = "../../tw59build/tools/wasmedge/wasmedge"
```

`tw59challenge/Cargo.toml` (this file is generated from cargo and not modified)

```toml
[package]
name = "tw59challenge"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

`tw59challenge/src/main.rs` this file is mostly copied from [hydai's example lib](https://github.com/second-state/wasmedge_hostfunctionexample_interface/commit/a79e98ae2dfecdded5f959f63a1ef5bca5a19642#diff-b1a35a68f14e696205874893c07fd24fdb88882b47c23cc0e0c80a30c7d53759), I only add the `main()` entry point to test the dev setup.

````rust
//! How to use this crate
//! # Adding this as a dependency
//! ```rust, ignore
//! [dependencies]
//! wasmedge_hostfunctionexample_interface = "^1.0.0"
//! ```
//!
//! # Bringing this into scope
//! ```rust, ignore
//! use wasmedge_hostfunctionexample_interface::*;
//! ```

use std::ffi::CString;

pub mod wasmedge_hostfunctionexample {
    use std::os::raw::c_char;
    #[link(wasm_import_module = "host_function_example")]
    extern "C" {
        pub fn host_function_example_set_class_id(cid: u32);
        pub fn host_function_example_add_student(student: *const c_char, len: u32) -> u32;
        pub fn host_function_example_set_class_name(name: *const c_char, len: u32);
        pub fn host_function_example_print();
    }
}

pub fn set_class_id(cid: u32) {
    unsafe {
        wasmedge_hostfunctionexample::host_function_example_set_class_id(cid);
    }
}

pub fn set_class_name<S: AsRef<str>>(name: S) {
    let name = CString::new((name.as_ref()).as_bytes()).expect("");
    unsafe {
        wasmedge_hostfunctionexample::host_function_example_set_class_name(
            name.as_ptr(),
            name.as_bytes().len() as u32,
        );
    }
}

pub fn add_student<S: AsRef<str>>(name: S) -> u32 {
    let name = CString::new((name.as_ref()).as_bytes()).expect("");
    let student_size: u32;
    unsafe {
        student_size = wasmedge_hostfunctionexample::host_function_example_add_student(
            name.as_ptr(),
            name.as_bytes().len() as u32,
        );
    }
    student_size
}

pub fn print() {
    unsafe {
        wasmedge_hostfunctionexample::host_function_example_print();
    }
}

fn main() {
    // test example function
    set_class_id(101);
    set_class_name("WasmEdge");
    ["Peter", "Paul", "Marry"].iter().for_each(|s| {
        println!("add {}, student number = {}", s, add_student(*s));
    });
    print();
}
````

Finally, just write `cargo run` in terminal to run this. here is an example output.

```shell
~/WasmEdge/examples/tw59challenge# cargo run
   Compiling tw59challenge v0.1.0 (/root/WasmEdge/examples/tw59challenge)
    Finished dev [unoptimized + debuginfo] target(s) in 0.82s
     Running `/root/WasmEdge/examples/tw59challenge/../../tw59build/tools/wasmedge/wasmedge target/wasm32-wasi/debug/tw59challenge.wasm`
add Peter, student number = 1
add Paul, student number = 2
add Marry, student number = 3
Class ID: 101
Class Name: WasmEdge
Student: Peter
Student: Paul
Student: Marry
```

## Step 7. design my on host function

To add a new function to this structure, we need to modify three files:

- `include/host/challenge_tieway59/tw59func.h` add host function class.
- `lib/host/challenge_tieway59/tw59func.cpp` add host function body.
- `lib/host/challenge_tieway59/tw59module.cpp` register host function in module.
- then build it.

My host function is an implementation of Binary GCD ([Stein's Algorithm for finding GCD - GeeksforGeeks](https://www.geeksforgeeks.org/steins-algorithm-for-finding-gcd/)).

`include/host/challenge_tieway59/tw59func.h` add host function class. Add this class into the end of `Host` namespace.

```c++
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#pragma once

#include "common/defines.h"
#include "host/challenge_tieway59/tw59base.h"

namespace WasmEdge {
namespace Host {

// --snip--

class HostFuncExampleBinaryGCD                                      //+
    : public HostFuncExample<HostFuncExampleBinaryGCD> {            //+
public:                                                             //+
  HostFuncExampleBinaryGCD(HostFuncExampleEnvironment &HostEnv)     //+
      : HostFuncExample(HostEnv) {}                                 //+
  Expect<uint32_t> body(Runtime::Instance::MemoryInstance *MemInst, //+
                    uint32_t a, uint32_t b);                        //+
};

} // namespace Host
} // namespace WasmEdge
```

`lib/host/challenge_tieway59/tw59func.cpp` add host function body. Be careful about the parameters and return type.

```c++
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#include "host/challenge_tieway59/tw59func.h"

namespace WasmEdge {
namespace Host {

// --snip--

Expect<uint32_t> HostFuncExampleBinaryGCD::body(
    [[maybe_unused]] Runtime::Instance::MemoryInstance *MemInst, uint32_t a,
    uint32_t b) {

  /* GCD(0, b) == b; GCD(a, 0) == a,
     GCD(0, 0) == 0 */
  if (a == 0)
    return b;
  if (b == 0)
    return a;

  /*Finding K, where K is the
    greatest power of 2
    that divides both a and b. */
  int k;
  for (k = 0; ((a | b) & 1) == 0; ++k) {
    a >>= 1;
    b >>= 1;
  }

  /* Dividing a by 2 until a becomes odd */
  while ((a & 1) == 0)
    a >>= 1;

  /* From here on, 'a' is always odd. */
  do {
    /* If b is even, remove all factor of 2 in b */
    while ((b & 1) == 0)
      b >>= 1;

    /* Now a and b are both odd.
       Swap if necessary so a <= b,
       then set b = b - a (which is even).*/
    if (a > b) {
      // Swap u and v.
      auto t = a;
      a = b;
      b = t;
    }

    b = (b - a);
  } while (b != 0);

  /* restore common factors of 2 */
  return a << k;
}

} // namespace Host
} // namespace WasmEdge
```

`lib/host/challenge_tieway59/tw59module.cpp` register host function in module.

```C++
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: 2019-2022 Second State INC

#include "host/challenge_tieway59/tw59module.h"

#include "host/challenge_tieway59/tw59func.h"

namespace WasmEdge {
namespace Host {

/// Register your functions in module.
HostFuncExampleModule::HostFuncExampleModule()
    : ModuleInstance("host_function_example") {

// --snip--
  addHostFunc("host_function_example_binary_gcd",    //+
              std::make_unique<HostFuncExampleBinaryGCD>(Env)); //+
}

} // namespace Host
} // namespace WasmEdge
```

as I have built the whole project before, just go to the build directory and run:

```shell
~/WasmEdge/tw59build$ cmake -DCMAKE_BUILD_TYPE=Release -DWASMEDGE_BUILD_AOT_RUNTIME=OFF -DWASMEDGE_BUILD_TESTS=OFF  .. && make -j8
-- Build spdlog: 1.9.0
-- Build type: Release
-- Configuring done
-- Generating done
-- Build files have been written to: /root/WasmEdge/tw59build
[ 10%] Built target spdlog
[ 15%] Built target wasmedgeCommon
[ 17%] Built target wasmedgePO
[ 21%] Built target wasmedgeValidator
[ 27%] Built target wasmedgeSystem
Scanning dependencies of target wasmedgeHostModuleHostFuncExample # this is our updated module
[ 28%] Building CXX object lib/host/challenge_tieway59/CMakeFiles/wasmedgeHostModuleHostFuncExample.dir/tw59func.cpp.o
[ 33%] Built target wasmedgeLoaderFileMgr
[ 43%] Built target wasmedgeHostModuleWasi
[ 66%] Built target wasmedgeExecutor
[ 71%] Built target wasmedgePluginWasmEdgeProcess
[ 73%] Built target wasmedgePlugin
[ 85%] Built target wasmedgeLoader
[ 90%] Built target wasmedgeHostModuleWasmEdgeProcess
[ 91%] Linking CXX static library libwasmedgeHostModuleHostFuncExample.a
[ 92%] Built target wasmedgeHostModuleHostFuncExample
[ 95%] Built target wasmedgeVM
Scanning dependencies of target wasmedgeCAPI
Scanning dependencies of target wasmedge
[ 96%] Building CXX object tools/wasmedge/CMakeFiles/wasmedge.dir/wasmedger.cpp.o
[ 97%] Building CXX object lib/api/CMakeFiles/wasmedgeCAPI.dir/wasmedge.cpp.o
[ 98%] Linking CXX executable wasmedge
[ 98%] Built target wasmedgeCAPI
[100%] Linking CXX shared library libwasmedge_c.so
[100%] Built target wasmedge
[100%] Built target wasmedge_c_shared
```

## Step 8. Run my host function in rust

Open `main.rs` as I created before.

Add the host function definition to `extern "C"` section:

```rust
pub mod wasmedge_hostfunctionexample {
    use std::os::raw::c_char;
    #[link(wasm_import_module = "host_function_example")]
    extern "C" {
  // --snip--
        pub fn host_function_example_binary_gcd(a: u32, b: u32) -> u32;  //+
    }
}
```

Wrap the unsafe `ffi`:

```rust
pub fn binary_gcd(a: u32, b: u32) -> u32 {
    unsafe { wasmedge_hostfunctionexample::host_function_example_binary_gcd(a, b) }
}
```

Now use the function like native rust:

```rust
fn main() {
    // test example function
    set_class_id(101);
    set_class_name("WasmEdge");
    ["Peter", "Paul", "Marry"].iter().for_each(|s| {
        println!("add {}, student number = {}", s, add_student(*s));
    });
    print();

    dbg!(binary_gcd(2, 2));                 //+
    dbg!(binary_gcd(2, 4));                 //+
    dbg!(binary_gcd(12, 16));               //+
    dbg!(binary_gcd(111111111, 111111112)); //+
}
```
