---
layout: post
title:  "Using Windows APIs with .NET 5 Preview 8 and Console Apps"
date:   2020-09-10 12:00:00 +0700
categories: dotnet5 preview8 .net5 UWP C#
---

Unless you have been living under a rock, or don't care about .NET (then why are you here...?), you will know that Microsoft is gearing up for the next version of .NET - .NET 5.0. which should ship sometime in Nov 2020:

![.NET Schedule](https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2019/05/dotnet_schedule.png?WT.mc_id=WD-MVP-5000646)

You can learn more about about what .NET 5 is from [Richard Lander's blog post](https://devblogs.microsoft.com/dotnet/introducing-net-5?WT.mc_id=WD-MVP-5000646) but this image sums everything up quite nicely:

![dotnet unified](https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2019/05/dotnet5_platform.png?WT.mc_id=WD-MVP-5000646)

Essentially, .NET 5.0 unifies everything - there will be just one .NET going forward, and you will be able to use it to target Windows, Linux, macOS, iOS, Android, tvOS, watchOS and WebAssembly and more.

## Previews

The development of .NET 5.0 has seen a number of preview releases along the way, but [Preview 6](https://devblogs.microsoft.com/dotnet/announcing-net-5-0-preview-6/?WT.mc_id=WD-MVP-5000646) really stuck out to me - it removed the built-in support for C#/WinRT. This change meant that .NET 5.0 no longer directly consumed [WinMD files](https://docs.microsoft.com/en-us/uwp/winrt-cref/winmd-files?WT.mc_id=WD-MVP-5000646) to create C# language projections of the WinRT APIs - instead a new toolset has been created called [C#/WinRT](https://docs.microsoft.com/en-us/windows/uwp/csharp-winrt/?WT.mc_id=WD-MVP-5000646). This tool can be used to create interop assemblies from WinMD files. Of course, having to create the interop assemblies required for your app, while providing fine-grained control, also adds complexity to projects. To mitigate this, the [Microsoft.Windows.SDK.Contracts nuget package](https://www.nuget.org/packages/Microsoft.Windows.SDK.Contracts/) could be referenced instead.

However, [.NET 5.0 Preview 8](https://devblogs.microsoft.com/dotnet/announcing-net-5-0-preview-8/?WT.mc_id=WD-MVP-5000646) has recently been announced and this introduced a streamlined way to access WinRT APIs from .NET 5.0. The [Windows Developer Team blog](https://blogs.windows.com/windowsdeveloper/author/windows-developer-team/?WT.mc_id=WD-MVP-5000646) recently posted [Calling Windows APIs in .NET5](https://blogs.windows.com/windowsdeveloper/2020/09/03/calling-windows-apis-in-net5/?WT.mc_id=WD-MVP-5000646) which discusses how Windows .NET5 applications can now access Windows APIs through a new set of [Target Framework Monikers (TFMs)](https://docs.microsoft.com/en-us/dotnet/standard/frameworks?WT.mc_id=WD-MVP-5000646). TFMs have been around for a while, but they have now been extended to support platform specific APIs. This is the recommended mechanism for .NET going forward.

## Using WinRT API in a .NET 5.0 Console App

The blog post [Calling Windows APIs in .NET5](https://blogs.windows.com/windowsdeveloper/2020/09/03/calling-windows-apis-in-net5/?WT.mc_id=WD-MVP-5000646) walks through the process of adding a call to the WinRT location API from a Windows Forms application. Given the fact that it seems unlikely that the Location API has a requirement on a parent window handle (whereas the `FileOpenPicker` does require a parent window), I thought I would try it in a Console application.

> **Note**: Ensure you have installed .NET 5.0 Preview 8 SDK from [here](https://dotnet.microsoft.com/download/dotnet/5.0)</br>
> Also the steps assume you are using Visual Studio Code.

1. Open a Terminal, change directory to the where you wish to create the **LocationConsole** app.

1. To create the application folder, enter the following commands:

    ```powershell
    mkdir LocationConsole
    cd LocationConsole
    ```

1. To create the console application, enter the following command:

    ```powershell
    dotnet new console
    ```

1. To open the folder in Visual Studio code, enter the following command:

    ```powershell
    code .
    ```

1. Open the **LocationConsole.csproj** and update the `<TargetFramework>` to use the new TFM net5.0-windows10.0.17763.0:

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net5.0-windows10.0.17763.0</TargetFramework>
    </PropertyGroup>

    </Project>
    ```

    > **Note**: 