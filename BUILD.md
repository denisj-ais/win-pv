# Building OpenXT Tools
---
The drivers can be built on any Windows 10+ machine with a copy of the EWDK.
Best practice is to NOT install any version of Visual Studio, or any SDKs or WDKs as these will create path settings that conflict
 with the EWDK leading to random and spurious errors during builds as the build occasionally attempts to use those binaries instead.

## Download the EWDK

Download the EWDK ISO from [EWDK download page](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk#download-icon-enterprise-wdk-ewdk)

As of May 2025, only the latest EWDK is available and all others have been deprecated and removed from public download by Microsoft.
The version used by the vs2022 build scripts is Windows 11 25H2 (September 2025) with Build Tools 17.14.0 -- aka 26100.6584.
As of this writing the preview for Windows 11 26H1 (aka 28000) is available but uses the same build tools and only should be used if it addresses a particular issue in the drivers.
The version used by the vs2019 build scripts is 22621 or 22000 and finding the relevant EWDK ISO is left as an exercise for the reader.

## Install EWDK

Mount the ISO, and copy all files to a local directory (this is required because the ISO has errors that may need to be addressed)

## Fetch the submodules

`git submodule init`  
`git submodule update`

## Fetch externals (Wix toolset)

`powershell ./fetch-externals.ps1`

## Start build environment

In the root of the EWDK:  
`LaunchBuildEnv.cmd amd64`

`amd64` is REQUIRED. Any other option (including no option) will cause failures.

## Build the drivers

In the root of the win-pv repo:  
`buildall.bat checked`

Or, for a free build:  
`buildall.bat free`

This will build all drivers in the win-pv folder and (if successful) package them into an .MSI installer

## Other build options

To build only a particular driver and not all, change to the directory of the driver of interest:  
`powershell ./build.ps1 -Arch:x64 -Type:checked`

- `-Arch:x64` is required and the only working option using the latest EWDK as x86 targets have been deprecated.
- `-Type:free` can be used for a free build.

glasswddm and pv-display-helper require that ivc has been built previously

## If you encounter build errors

### DrvCat fails with a message about not finding Microsoft.Kits.Logger.dll
1. edit the file {ewdk root}/program files/microsoft visual studio/2022/buildtools/msbuild/current/bin/amd64/MsBuild.exe.config
2. in the section `<runtime>/<assemblyBinding>`: add `<probing privatePath="..\..\..\..\Windows Kits\10\bin\10.0.26100.0\x64">`

### CodeAnalysis fails with a 'File not Found' exception with LoadRuleSet(..) on the top of the stack
If this happens to you, there is no known workaround to get CodeAnalysis working on that dev machine
- edit the project file and change `<RunCodeAnalysis>true</RunCodeAnalysis>` to false

### New compile warnings appear and fail due to WarningsAsErrors being enabled
Currently 28719 (Banned API usage) and 28735 (Banned Crimson API usage) are known issues and will be fixed in a future version
- edit the project file and ignore those warnings for now
