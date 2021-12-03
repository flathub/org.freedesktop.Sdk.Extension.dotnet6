# Dotnet 6 SDK extension

## How to use
You need to add following lines to flatpak manifest:
```json
"sdk-extensions": [
    "org.freedesktop.Sdk.Extension.dotnet6"
],
"build-options": {
    "append-path": "/usr/lib/sdk/dotnet6/bin",
    "append-ld-library-path": "/usr/lib/sdk/dotnet6/lib",
    "env": {
        "PKG_CONFIG_PATH": "/app/lib/pkgconfig:/app/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib/sdk/dotnet6/lib/pkgconfig"
    }
},
```

###  Scripts
* `install.sh` - copies dotnet runtime to package.
* `install-sdk.sh` - copies dotnet SDK to package.

### Publishing project

```json
"build-commands": [
    "install.sh",
    "dotnet publish -c Release YourProject.csproj",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net6.0/publish/ /app/bin/",
]
```

### Using nuget packages
If you want to use nuget packages it is recommended to use the [Flatpak .NET Generator](https://github.com/flatpak/flatpak-builder-tools/tree/master/dotnet) tool. It generates sources file that can be included in manifest.

```json
"build-commands": [
    "install.sh",
    "dotnet publish -c Release --source ./nuget-sources YourProject.csproj",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net6.0/publish/ /app/bin/"
],
"sources": [
    "sources.json",
    "..."
]
```

### Publishing self contained app and trimmed binaries
Dotnet 6 gives option to include runtime in published application and trim their binaries. This allows you to significantly reduce the size of the package and get rid of `/usr/lib/sdk/dotnet6/bin/install.sh`. 

First you need to have following lines in `sources.json`. These packages are needed to build a project for specific runtime. 

```json
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm/6.0.0/microsoft.aspnetcore.app.runtime.linux-arm.6.0.0.nupkg",
    "sha512": "b7146bf669f636ec63d46496aa2ee356b1e6deb4644a3b1f4c44073e0cdb2ad87977cd57b477d3dcfe969ef2e9810ed7e4a0338698668e1f115afbb312de52d7",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm.6.0.0.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/6.0.0/microsoft.aspnetcore.app.runtime.linux-arm64.6.0.0.nupkg",
    "sha512": "656ad0c88677c31f2d269140b308437b534b935cfd1ae4207167580bd61bd0e0cac75139aba0e20954e9b12ef4df6a0c25f2c0c7e5d73d8dedcc315dbf9f165c",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.6.0.0.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/6.0.0/microsoft.aspnetcore.app.runtime.linux-x64.6.0.0.nupkg",
    "sha512": "4129d888119de2007856e85c26c7711219be95cf9ea2825a97c52b290263b1c8d8b9b539d73140eb1fcfbfaf5de84dbca548fe3d99dd241d5e135000ecf03992",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.6.0.0.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm/6.0.0/microsoft.netcore.app.runtime.linux-arm.6.0.0.nupkg",
    "sha512": "24f8d5865cb21da0b0230f3d825e83b6ca5d0a74a0896181fe79efa2da510e28252927d4c846764c78c7ab233eeb12721c605325e8f4ffee62a1343297e1a7a9",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm.6.0.0.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/6.0.0/microsoft.netcore.app.runtime.linux-arm64.6.0.0.nupkg",
    "sha512": "b3d345672eb2ae07afadacb735743c8de0e50b2e9795b2ffbeb06c0da9cd9199fbbc45a2cd0b14c840c015b22b760bb8802a72a4a3845395713a0e9a2f7a633a",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.6.0.0.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/6.0.0/microsoft.netcore.app.runtime.linux-x64.6.0.0.nupkg",
    "sha512": "1e6777d48d8bacccf9d80b670c2a9f230f252783a8a6d7aac845bd8f2dda6550225973ba87076898ee297354a9defcbe3ccee9444ef7ed6ded66da6dee4ca3f2",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.6.0.0.nupkg"
},
```

Then add build options:

```json
"build-options": {
    "arch": {
        "arm": {
            "env" : {
                "RUNTIME": "linux-arm"
            }
        },
        "aarch64": {
            "env" : {
                "RUNTIME": "linux-arm64"
            }
        },
        "x86_64": {
            "env" : {
                "RUNTIME": "linux-x64"
            }
        }
    }
},
"build-commands": [
    "mkdir -p /app/bin",
    "dotnet publish -c Release --source ./nuget-sources YourProject.csproj --runtime $RUNTIME --self-contained true",
    "cp -r --remove-destination /run/build/YourProject/bin/Release/net6.0/$RUNTIME/publish/* /app/bin/",
],
```
