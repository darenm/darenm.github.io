---
layout: post
title:  "Exploring IoT on Raspberry PI 3 V2 with .NET/C#"
date:   2019-10-15 12:00:00 +0700
categories: iot .netcore c# raspberrypi
---
While working on various courses for Azure IoT, I have had a bunch of experience working with C/C++ and Python on Linux and C#/UWP on Windows 10 IoT Core. However I have noticed that the Raspberry Pi isn't necessarily an ideal platform for Windows 10 Core - in fact the later models, including the Raspberry Pi 4, are not yet supported. So, if I want to explore IoT in C# on the Raspberry Pi, I need to look at .NET Core.

## Install .NET Core 3.0

Complete the following steps to install .NET Core 3.0 on a Raspberry PI Model 3 V2 running Raspbian Buster. Raspbian Buster is a variant of Debian 10.

### Prerequisites

The [.NET Core github repo][1] contains an [article][2] that lists the following prerequisites for .NET Core 3.0 on Debian 10:

* libicu63
* libssl1.1

1. In a terminal session on your Raspberry PI, to install .NET Core runtime dependencies, run the following commands:

    ```bash
    sudo apt-get -y update
    sudo apt-get -y install wget libicu63 libssl1.1
    ```

We will use `wget` to download the distro on the Raspberry Pi.

### Download the Linux .NET Core Distro

At the moment, there is no simple way to install .Net Core via the `apt-get` command - instead we will need to download the `tar` for the distribution and install it ourselves.

Microsoft maintains a download page for .NET Core 3.0 here

1. In a browser, open <https://dotnet.microsoft.com/download/dotnet-core/3.0>.

1. Find the latest version of the Linux ARM32 SDK and click the URL.

    This will open the download page as well as automatically start the download. Useful, if you are using a browser on the PI, but not so useful if you are running on the desktop. Fortunately, the page also provides a click here to download manually link as well as the ability to copy a direct link to the download. Copy that URL.

    This is the URL I see: <https://download.visualstudio.microsoft.com/download/pr/8ddb8193-f88c-4c4b-82a3-39fcced27e91/b8e0b9bf4cf77dff09ff86cc1a73960b/dotnet-sdk-3.0.100-linux-arm.tar.gz>

1. In a terminal session on your Raspberry PI, to download the distribution, run the following command:

    ```bash
    wget <url-from-above>
    ```

    For example:

    ```bash
    wget https://download.visualstudio.microsoft.com/download/pr/8ddb8193-f88c-4c4b-82a3-39fcced27e91/b8e0b9bf4cf77dff09ff86cc1a73960b/dotnet-sdk-3.0.100-linux-arm.tar.gz
    ```

1. To create a directory to extract the tar archive, enter the following command:

    ```bash
    sudo mkdir -p /usr/share/dotnet
    ```

    The -p argument creates parent directories as necessary.

    > **Note**: The dotnet runtime expects to be located in **/usr/share/dotnet**. Alternatively, you can set the **DOTNET_ROOT** environment variable to specify the runtime location or register the runtime location in **/etc/dotnet/install_location**.

1. To extract the archive, enter the following command:

    ```bash
    sudo tar -zxf dotnet-sdk-3.0.100-linux-arm.tar.gz -C /usr/share/dotnet
    ```

    > **Note**: This will run silently and take a few minutes.

    The arguments mean:

    * z - filter the archive through gzip (as the file name ends in .gz we know gzip was used)
    * x - extract the archive
    * f - means the archive is a file, not from tape

1. To create a symbolic link to the dotnet command so it is accessible by the existing path settings, enter the following command:

    ```bash
    sudo ln -s /usr/share/dotnet/dotnet /usr/bin
    ```

1. To verify the dotnet command is available, enter the following command:

    ```bash
    dotnet --version
    ```

    For the version I have installed, you should see the following:

    ```bash
    dotnet --version
    3.0.100
    ```

## Create a Sample Project

In these steps we will create a simple "Hello, World" app and run it. We will then create a single file deployment, copy the output and demonstrate that it runs "standalone".

1. Change directory to the location you wish to create the project.

1. To create a directory for your project and then navigate into it, enter the following commands:

    ```bash
    mkdir helloworld
    cd helloworld
    ```
    
1. To create a new console application, enter the following command:

    ```
    dotnet new console
    ```
    
    This command will create the source file and project file for a simple console application. It will also automatically run dotnet restore to download any dependencies.

    This is a very simple app:

    ```csharp
    using System;

    namespace helloworld
    {
        class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Hello World!");
            }
        }
    }
    ```
    
1. To run the app, enter the following command:

    ```bash
    dotnet run
    ```

    After a moment, the application will compile, run and then write the following to the console:

    ```bash
    dotnet run
    Hello World!
    ```

    > **Note**: If you see the error below, ensure the SDK was installed in the correct location.
    _A fatal error occurred. The required library libhostfxr.so could not be found._

1. To create a self-contained release build, enter the following command:

    ```bash
    dotnet publish -r linux-arm -c Release --self-contained
    ```

    You will see something similar tp the following:

    ```bash
    dotnet publish -r linux-arm -c Release --self-contained
    Microsoft (R) Build Engine version 16.3.0-preview-19455-02+4a2d77107 for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

    Restore completed in 26.79 sec for /home/pi/helloworld/helloworld.csproj.
    You are using a preview version of .NET Core. See: https://aka.ms/dotnet-core-preview
    helloworld -> /home/pi/helloworld/bin/Release/netcoreapp3.0/linux-arm/helloworld.dll
    helloworld -> /home/pi/helloworld/bin/Release/netcoreapp3.0/linux-arm/publish/
    ```

    The publish command will build the app in release mode and will publish the self contained app. It will create a new folder with the name of publish which will have a standalone copy of the .NET Core runtime with the .exe file. To run the app on another machine (even one with out .NET Core installed) just requires copying this folder and running the executable file.

1. To view the files in the publish folder, run the following command:

    ```bash
    ls -l bin/Release/netcoreapp3.0/linux-arm/publish
    ```

    You will see there are **a lot** of files.

    To just view the application executable, enter the following command:

    ```bash
    ls -l bin/Release/netcoreapp3.0/linux-arm/publish/helloworld
    ```

    You will see something similar to:

    ```bash
    ls -l bin/Release/netcoreapp3.0/linux-arm/publish/helloworld
    -rwxr--r-- 1 pi pi 73400 Sep 18 17:35 bin/Release/netcoreapp3.0/linux-arm/publish/helloworld
    ```

    The size of the executable is pretty small, as you would expect for our small app - of course, the folder does also contain the entire set of libraries for the .NET Core runtime as well. However, with .NET Core 3.0 we can go one better and produce a single executable file.

1. To delete the Publish folder, enter the following command:

    ```bash
    rm -r bin/Release
    ```

1. To create a single file, run the following command:

    ```bash
    dotnet publish -r linux-arm -c Release /p:PublishSingleFile=true
    ```

1. To view the files in the publish folder, run the following command:

    ```bash
    ls -l bin/Release/netcoreapp3.0/linux-arm/publish
    ```

    You will see something similar to:

    ```bash
    ls -l
    total 76476
    -rwxr--r-- 1 pi pi 78289466 Sep 18 18:30 helloworld
    -rw-r--r-- 1 pi pi      416 Sep 18 17:35 helloworld.pdb
    ```

    You can see we now have a single executable helloworld. However, check out the size of the file - it is massive! That is because it now contains all of the .NET Core runtime as well as our tiny hello world app.

    Thankfully, .NET Core 3.0 now allows us to trim unnecesary files before we create the single file.

1. To delete the Publish folder, enter the following command:

    ```bash
    rm -r bin/Release
    ```

1. To create a single, trimmed file, run the following command:

    ```bash
    dotnet publish -r linux-arm -c Release /p:PublishSingleFile=true  /p:PublishTrimmed=true
    ```

1. To view the files in the publish folder, run the following command:

    ```bash
    ls -l bin/Release/netcoreapp3.0/linux-arm/publish
    ```

    You will see something similar to:

    ```bash
    ls -l bin/Release/netcoreapp3.0/linux-arm/publish
    total 30076
    -rwxr--r-- 1 pi pi 30791588 Sep 18 18:41 helloworld
    -rw-r--r-- 1 pi pi      416 Sep 18 17:35 helloworld.pdb
    ```

    You can see we still have a single executable helloworld. However, check out the size of the file - it is now half the size of the untrimmed file. It's not perfect - but certainly better!

So, to sum up - in this article we installed .NET Core 3.0, created a simple console app, and then explored various ways to compile and publish that app.

In the next article, I am going to explore interacting with some simple electronics via GPIO.
