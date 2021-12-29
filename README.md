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
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm/6.0.1/microsoft.aspnetcore.app.runtime.linux-arm.6.0.1.nupkg",
    "sha512": "b72cd9ca4ce670267afe79762ff71d23898dd2e8fb6a4bc671c1d0e0e0c06be57c5670014b4a83875343f09e0977a68a3b8da1ccf56239d76fad22115300e150",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm.6.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-arm64/6.0.1/microsoft.aspnetcore.app.runtime.linux-arm64.6.0.1.nupkg",
    "sha512": "e4cf63e86a96e09b6873146dc974c76b096d1c2306eacd531efc3f1cee9f2be13a4b897b326341b3d081598e93f9d34fa67fe9e8a54e3ea2949dc4568c7e550e",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-arm64.6.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.aspnetcore.app.runtime.linux-x64/6.0.1/microsoft.aspnetcore.app.runtime.linux-x64.6.0.1.nupkg",
    "sha512": "865af2ac328070403202eb5f0c436abbf8edb3fd484dec92b4c0cd6d42d36c8c7e396bc9bfd13cd0b6f877baf72b3fbfb0b425a794711dbab4b0297b20143ce5",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.aspnetcore.app.runtime.linux-x64.6.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm/6.0.1/microsoft.netcore.app.runtime.linux-arm.6.0.1.nupkg",
    "sha512": "3999195f9d22da631152073d47b221db2432a1ac5efa4bbb08f1e6716e95365150a69aeb754c9e1da277c447d29af0deb626226ae63171f27b47d777028217d5",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm.6.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-arm64/6.0.1/microsoft.netcore.app.runtime.linux-arm64.6.0.1.nupkg",
    "sha512": "81e9e34b6e8e7e5632b00f9ab87011c4ad66cf914560dfa95e3596c7a79dc9b310f7f592d4affe4f06dc80fd43214b13023d5c6f6f2caa893dadc7814afa90c5",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-arm64.6.0.1.nupkg"
},
{
    "type": "file",
    "url": "https://api.nuget.org/v3-flatcontainer/microsoft.netcore.app.runtime.linux-x64/6.0.1/microsoft.netcore.app.runtime.linux-x64.6.0.1.nupkg",
    "sha512": "01d0e6c885a357270dabb65878deed2c147841297d8688a543b7605f4704d2507c70f86f825a59a975ff7789a4b74677262947fc97a7a25f23556292266b50ef",
    "dest": "nuget-sources",
    "dest-filename": "microsoft.netcore.app.runtime.linux-x64.6.0.1.nupkg"
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
