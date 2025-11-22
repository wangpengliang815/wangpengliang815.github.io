# 背景介绍

> NuGet 是用于微软.NET（包括 .NET Core）开发平台的软件包管理器。NuGet 能够使得在项目中添加、移除和更新引用的工作变得更加快捷方便

通常使用 NuGet 都是官方的服务，也可以搭建针对自己公司或部门的私有 NuGet 托管一些自己的类库，公司内部的类库等。搭建私有 NuGet 的方法有很多，比如：`NuGet.Server`、`ProGet` 等。这里使用 `BaGet`。

 GitHub地址：https://loic-sharma.github.io/BaGet/， 安装方式选择 Docker。

# Configure BaGet

在服务器上自定义创建目录，目录结构如下：

```bash
projects
├─baget 
├───baget.env
├───baget-data
```

`baget.env` 文件用于存储BaGet的配置，内容如下：

```bash
# The following config is the API Key used to publish packages.
# You should change this to a secret value to secure your server.
ApiKey=ApiKey # 推送包是需要提供这个ApiKey

Storage__Type=FileSystem
Storage__Path=/var/baget/packages
Database__Type=Sqlite
Database__ConnectionString=Data Source=/var/baget/baget.db
Search__Type=Database
```

[配置指南介绍](https://loic-sharma.github.io/BaGet/configuration/)

# Run BaGet

1. Pull BaGet's latest docker image

```bash
docker pull loicsharma/baget
```

2. run BaGet

```bash
docker run -d --restart always --name nuget-server -p 80:80 --env-file /projects/baget/baget.env -v /projects/baget/baget-data:/var/baget loicsharma/baget:latest
```

# Browse packages

You can browse packages by opening the URL [`http://localhost:80`](http://localhost:80/) in your browser.

# Publish packages

项目中右键打包生成 `you project.1.0.0.nupkg` 文件，包的具体描述和版本号可在项目属性中更改，在控制台中通过命令将包推送到 Baget，命令如下：

```bash
dotnet nuget push -s http://localhost:80/v3/index.json -k NUGET-SERVER-API-KEY you project.1.0.0.nupkg
```

- `-k NUGET-SERVER-API-KEY` ：`baget.env` 中配置的ApiKey
- `you project.1.0.0.nupkg`：要上传的包路径
- `-s http://localhost:80/v3/index.json`：baget 地址

# Restore packages

现在可以在 Visual Studio 使用以下包源还原包：

```bash
http://localhost:80/v3/index.json
```

# Import packages

将本地包源推送到 Baget，需要确保已安装 [nuget.exe](https://www.nuget.org/downloads)。在 PowerShell 中，运行：

```powershell
$source = "C:\Users\Administrator\.nuget\packages"
$destination = "http://localhost:80/v3/index.json"

&D:\Nuget\nuget\nuget.exe setapikey "NUGET-SERVER-API-KEY" -Source $destination

$packages = D:\Nuget\nuget\nuget.exe list -AllVersions -Source $source

$packages | % {
  $id, $version = $_ -Split " "
  $nupkg = $id + "." + $version + ".nupkg"
  $path = [IO.Path]::Combine($source, $id, $version, $nupkg)

  Write-Host "D:\Nuget\nuget\nuget.exe push -Source $destination ""$path"""
  & D:\Nuget\nuget\nuget.exe push -Source $destination $path
}
```