+++
title = "Building RetDec from Source ( Windows 11 )"
date = "2025-08-16"

[taxonomies]
tags=["detdec"]

[extra]
comment = true
+++

# Introduction

[RetDec](https://github.com/avast/retdec) is an open-source machine-code decompiler originally developed by Avast.  

Although the project provides **prebuilt binaries for both Linux and Windows**, I ran into issues when trying the Windows release: it complained about a missing `libcrypto-1_1-x64.dll`, and even after manually adding the library, it still didn’t run properly.  

Because of that, I decided to **build RetDec from source on Windows 11 (10.0.26100.0)**.  
This post documents the process — including the prerequisites, Visual Studio setup, and a couple of tweaks I had to make along the way.

First things first:

```sh
git clone https://github.com/avast/retdec
```

# Prerequisites

I built it successfully on Windows 11 with the following installed.

```sh
python --version
Python 3.12.6

cmake --version
cmake version 3.29.2

openssl -v
OpenSSL 3.3.0 9 Apr 2024 (Library: OpenSSL 3.3.0 9 Apr 2024)
```

For Visual Studio 2022, I used the Desktop Development with C++ workload plus the following components.

```json
{
  "version": "1.0",
  "components": [
    "Microsoft.VisualStudio.Component.CoreEditor",
    "Microsoft.Component.MSBuild"
    "Microsoft.VisualStudio.Component.TextTemplating",
    "Microsoft.VisualStudio.Component.NuGet",
    "Component.Microsoft.Web.LibraryManager",
    "Microsoft.VisualStudio.Component.DiagnosticTools",
    "Microsoft.VisualStudio.Component.Debugger.JustInTime",
    "Component.Microsoft.VisualStudio.LiveShare.2022",
    "Microsoft.VisualStudio.Component.WslDebugging",
    "Microsoft.VisualStudio.Component.IntelliCode",
    "Microsoft.VisualStudio.Component.VC.CoreIde",
    "Microsoft.VisualStudio.Component.VC.Tools.x86.x64",
    "Microsoft.VisualStudio.Component.VC.DiagnosticTools",
    "Microsoft.VisualStudio.Component.Windows11SDK.26100",
    "Microsoft.VisualStudio.Component.VC.ATL",
    "Microsoft.VisualStudio.Component.VC.ATLMFC",
    "Microsoft.VisualStudio.Component.VC.Modules.x86.x64",
    "Microsoft.VisualStudio.Component.VC.Redist.14.Latest",
    "Microsoft.VisualStudio.ComponentGroup.NativeDesktop.Core",
    "Microsoft.VisualStudio.Component.Windows11Sdk.WindowsPerformanceToolkit",
    "Microsoft.VisualStudio.Component.CppBuildInsights",
    "Microsoft.VisualStudio.ComponentGroup.WebToolsExtensions.CMake",
    "Microsoft.VisualStudio.Component.VC.CMake.Project",
    "Microsoft.VisualStudio.Component.VC.TestAdapterForBoostTest",
    "Microsoft.VisualStudio.Component.VC.TestAdapterForGoogleTest",
    "Microsoft.VisualStudio.Component.VC.ASAN",
    "Microsoft.VisualStudio.Component.Vcpkg",
    "Microsoft.VisualStudio.Component.VC.CLI.Support",
    "Microsoft.VisualStudio.Component.VC.Llvm.ClangToolset",
    "Microsoft.VisualStudio.Component.VC.Llvm.Clang",
    "Microsoft.VisualStudio.ComponentGroup.NativeDesktop.Llvm.Clang",
    "Microsoft.VisualStudio.Component.Windows11SDK.22621",
    "Microsoft.VisualStudio.Component.Windows10SDK.19041",
    "Microsoft.VisualStudio.ComponentGroup.VC.Tools.142.x86.x64",
    "Microsoft.VisualStudio.Workload.NativeDesktop"
  ],
  "extensions": []
}
```

# Build

With the prerequisites and Visual Studio components ready, we can finally configure the build.  
RetDec uses **CMake** to generate a Visual Studio solution, so the first step is creating a dedicated `build` folder and running CMake against the source tree:

```powershell
Set-Location retdec
New-Item -ItemType Directory -Name build Out-Null; Set-Location build
cmake .. -G"Visual Studio 17 2022"
```

If everything is set up correctly, you should see CMake detect your compiler toolchain, SDKs, and dependencies. At this stage, a Visual Studio solution is generated under the build folder.

# Tweaks

Unfortunately, the build didn’t succeed on the first try.
Running MSBuild directly failed during the utils target because it couldn’t resolve stdc++fs (the C++17 filesystem library).

A simple fix was to modify `src\utils\CMakeLists.txt` by commenting out the section that references `stdc++fs`

```sh
find_library(STD_CPP_FS stdc++fs)
# Library found -> link against it.
if (STD_CPP_FS)
	message(STATUS "-- Library stdc++fs found -> Linking with ${STD_CPP_FS}")
	target_link_libraries(utils
		PRIVATE
			stdc++fs
	)
```

With that tweak in place, build again:

```sh
cmake --build . --config Release -- -m
```

This will take a while — it pulls in a lot of code, so don’t be surprised if your fans spin up.

# Installation

Once the build finishes, install RetDec to the default location (C:\Program Files\retdec) with:

```
cmake --build . --config Release --target install
```

Finally, update your `PATH` environment variable so the bin directory is available system-wide
`C:\Program Files\retdec\bin`

# First Run

To verify the installation, I ran the decompiler against a test binary

```sh
PS C:\repos\app\target\debug> retdec-decompiler app.exe
Running phase: Unpacking ( 0.01s )
No matching plugins found for 'MSVC'
No matching plugins found for 'Rust (64-bit) x86_64-pc-windows-msvc'
No matching plugins found for 'Microsoft'
Running phase: Initialization ( 0.24s )
Running phase: Providers initialization ( 0.24s )
Running phase: Input binary to LLVM IR decoding ( 0.83s )
Running phase: LLVM ( 7.93s )
Running phase: x86 address spaces optimization ( 8.67s )
Running phase: x87 fpu register analysis ( 8.96s )
Running phase: Main function identification optimization ( 8.96s )
Running phase: Libgcc idioms optimization ( 8.97s )
Running phase: LLVM instruction optimization ( 8.97s )
Running phase: Conditional branch optimization ( 9.38s )
Running phase: Syscalls optimization ( 12.10s )
Running phase: Stack optimization ( 12.10s )
Running phase: Constants optimization ( 18.52s )
Running phase: signed/unsigned types fixing ( 86.30s )
Running phase: converting LLVM intrinsic functions to standard functions ( 90.91s )
Running phase: obtaining debug information ( 91.39s )
Running phase: alias analysis [simple] ( 91.50s )
Running phase: optimizations ( 91.92s )
 -> running GotoStmtOptimizer ( 91.92s )
 -> running RemoveUselessCastsOptimizer ( 92.14s )
 -> running UnusedGlobalVarOptimizer ( 92.41s )
 -> running DeadLocalAssignOptimizer ( 92.94s )
Warning: out of memory; trying to recover
 -> running SimpleCopyPropagationOptimizer ( 147.23s )
Warning: out of memory; trying to recover
 -> running CopyPropagationOptimizer ( 228.30s )
Warning: out of memory; trying to recover
 -> running SimplifyArithmExprOptimizer ( 313.33s )
 -> running IfStructureOptimizer ( 314.05s )
 -> running LoopLastContinueOptimizer ( 314.33s )
 -> running PreWhileTrueLoopConvOptimizer ( 314.57s )
 -> running WhileTrueToForLoopOptimizer ( 323.02s )
 -> running WhileTrueToWhileCondOptimizer ( 336.03s )
 -> running IfBeforeLoopOptimizer ( 337.01s )
 -> running LLVMIntrinsicsOptimizer ( 337.31s )
 -> running VoidReturnOptimizer ( 347.28s )
 -> running BreakContinueReturnOptimizer ( 347.44s )
 -> running BitShiftOptimizer ( 347.68s )
 -> running DerefAddressOptimizer ( 348.12s )
 -> running EmptyArrayToStringOptimizer ( 348.38s )
 -> running BitOpToLogOpOptimizer ( 348.38s )
 -> running SimplifyArithmExprOptimizer ( 348.63s )
 -> running UnusedGlobalVarOptimizer ( 349.12s )
 -> running DeadLocalAssignOptimizer ( 349.59s )
 -> running DeadLocalAssignOptimizer ( 349.59s )
 -> running NoGlobalVarDefValidator ( 864.01s )
 -> running ReturnValidator ( 889.74s )
Warning: out of memory; trying to recover
 -> running SimpleCopyPropagationOptimizer ( 424.09s )
Warning: out of memory; trying to recover
 -> running CopyPropagationOptimizer ( 522.97s )
Warning: out of memory; trying to recover
 -> running SelfAssignOptimizer ( 628.08s )
 -> running VarDefForLoopOptimizer ( 628.35s )
 -> running VarDefStmtOptimizer ( 628.62s )
 -> running EmptyStmtOptimizer ( 651.34s )
 -> running GotoStmtOptimizer ( 651.57s )
 -> running SimplifyArithmExprOptimizer ( 651.73s )
 -> running DeadCodeOptimizer ( 652.17s )
 -> running DerefToArrayIndexOptimizer ( 652.38s )
 -> running IfToSwitchOptimizer ( 652.72s )
 -> running CCastOptimizer ( 652.77s )
 -> running CArrayArgOptimizer ( 660.14s )
Running phase: variable renaming [readable] ( 661.09s )
Running phase: converting constants to symbolic names ( 668.12s )
Running phase: module validation ( 675.61s )
 -> running BreakOutsideLoopValidator ( 675.61s )
Running phase: emission of the target code [c] ( 890.70s )
Running phase: finalization ( 1105.87s )
Running phase: cleanup ( 1106.07s )
```