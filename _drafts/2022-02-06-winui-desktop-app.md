---
layout: post
title:  "Windows App SDK Desktop App with native interop - Part 2"
date:   2022-01-30 12:00:00 +0700
categories: custommayd winui desktop xaml native interop
---

I've been meaning to put together a tutorial for WinUI and some native interop and I settled on the idea of building an app that displays the current desktop setup. This will include the size, the relative position and the wallpaper currently being displayed on each desktop. This app will evolve into something that will allow the user to browse online images and apply as the wallpaper for one or more desktops.

## Designing the App

I want to use this app as the foundation for an exploration of WinUI3 and Windows App SDK on the desktop. I've spent a number of years working with the earlier iterations of the desktop frameworks: UWP, Universal Windows Apps on Windows 8.x, Silverlight and WPF, so this is an opportunity to see the differences.

![Desktop layout with 3 monitors]({{ site.url }}/assets/2022-01-30 14_31_50-DeskDropper.png)


![Desktop layout with 1 monitor]({{ site.url }}/assets/2022-01-30 15_56_37-DeskDropper.png)


https://docs.microsoft.com/en-us/windows/win32/api/shobjidl_core/nn-shobjidl_core-idesktopwallpaper

https://docs.microsoft.com/en-us/dotnet/framework/interop/how-to-create-wrappers-manually

IDL -> TLB

midl "C:\Program Files (x86)\Windows Kits\10\Include\10.0.22000.0\um\ShObjIdl_core.idl"

tlbimp .\ShObjIdl_core.tlb

ShellCoreObjects.dll

C:\Program Files (x86)\Windows Kits\10\Include\10.0.22000.0\um\ShObjIdl_core.idl(1745) : error MIDL2455 : The feature cannot be used on the target system : disable_consistency_check [ Parameter 'apt' of Procedure 'SelectAndPositionItems' ( Interface 'IFolderView' ) ]