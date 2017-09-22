---
layout: post
title: Linker Analyzer
---

# Preface

Ah, the mono linker. One of the greatest enemies of the Xamarin developer. Have no fear fellow Xamarin developers, the linker analyzer is here!

## What is the Linker Analyzer?

The Linker Analyzer is a command line tool that analyzes dependencies which are recorded during the `LinkAssemblies` step. It will show you what items were marked and resulted in the linked assembly.

## Getting Started

You can get started with the following command against your Xamarin.Android application:

`msbuild /p:LinkerDumpDependencies=true /p:Configuration=Release YourAppProject.csproj`

This will generate a `linker-dependencies.xml.gz` file which you can extract to view the `linker-dependencies.xml` file.

## The `linker-dependencies.xml` file

The `linker-dependencies.xml` file is a xml that includes every item that was marked to keep in your application. You can open this file in an editor like `Notepad++` for an overview.

## Comparing `linker-dependencies.xml` files

Now one of the best tools in our toolkit is the ability to generate a `linker-dependencies.xml` file with each of the linker options enabled:

- Don't Link (Small file)
- Link SDK Assemblies (Medium file)
- Link All Assemblies (Large file)

We can then use a comparison tool such as:

<https://www.scootersoftware.com/download.php> (Windows)

<https://www.kaleidoscopeapp.com/> (Mac)

To compare between each linker option to see the behavior of what is being linked in our assembly. This is especially useful for optimizing our applications.

## Analyzing types

You can also analyze types using the `linkeranalyzer.exe` tool that is shipped with Mono.

**Note:** You may want to put the following on your `PATH`: 

- `C:\Program Files\Mono\lib\mono\4.5` (Windows)
- `/Library/Frameworks/Mono.framework/Versions/{Version}/lib/mono/4.5` (Mac)

You can then use this tool to determine why a type was marked by the linker. For example if we wanted to see why our custom application was marked by the linker, we might first start with the parent type to see dependencies:

```
linkeranalyzer.exe -t Android.App.Application linker-dependencies.xml.gz
```

**Output:**
```
Loading dependency tree from: linker-dependencies.xml.gz

--- Type dependencies: 'Android.App.Application' --------------------

--- TypeDef:Android.App.Application dependencies --------------------
Dependency #1
        TypeDef:Android.App.Application
        | TypeDef:mayday.Droid.MaydayApplication [2 deps]
        | Assembly:mayday.Droid, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null [1 deps]
        | Other:Mono.Linker.Steps.ResolveFromAssemblyStep
```

We can then see that `mayday.Droid.MaydayApplication` is our dependency as it is based on the `Application` type. Let's analyze that type now:

```
linkeranalyzer.exe -t mayday.Droid.MaydayApplication linker-dependencies.xml.gz
```

**Output:**
```
Loading dependency tree from: linker-dependencies.xml.gz

--- Type dependencies: 'mayday.Droid.MaydayApplication' -------------

--- TypeDef:mayday.Droid.MaydayApplication dependencies -------------
Dependency #1
        TypeDef:mayday.Droid.MaydayApplication
        | Assembly:mayday.Droid, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null [1 deps]
        | Other:Mono.Linker.Steps.ResolveFromAssemblyStep
Dependency #2
        TypeDef:mayday.Droid.MaydayApplication
        | TypeDef:mayday.Droid.MaydayApplication/<>c [2 deps]
        | TypeDef:mayday.Droid.MaydayApplication [2 deps]
        | Assembly:mayday.Droid, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null [1 deps]
        | Other:Mono.Linker.Steps.ResolveFromAssemblyStep
```

## Linker Statistics

You can also get the statistics of what's linked in your assemblies.

```
linkeranalyzer.exe --stat --verbose linker-dependencies.xml.gz
```

**Output:**
```
Loading dependency tree from: linker-dependencies.xml.gz

--- Statistics ------------------------------------------------------
Vertex type:    Other           count:18
Vertex type:    Assembly        count:3
Vertex type:    TypeDef         count:4606
Vertex type:    Method          count:40101
Vertex type:    Field           count:25680
Vertex type:    ExportedType    count:1251
Vertex type:    MemberRef       count:7672
Vertex type:    Property        count:27
Vertex type:    Module          count:45

Total vertices: 79403

--- Root vertices ---------------------------------------------------
Other:Mono.Linker.Steps.ResolveFromAssemblyStep
Other:Mono.Linker.Steps.ResolveFromXmlStep
Other:Mono.Tuner.SubStepDispatcher
Other:MonoDroid.Tuner.MonoDroidMarkStep

Total root vertices: 4
```

## Summary

This is only a surface level of how to use this tool to help diagnose linker issues in your application. This tool is extremely useful for seeing what is ultimately making your linked assemblies.

Huge shoutout to Radek Doulik(<https://github.com/radekdoulik>) for a wonderful tool and documentation!

**Further documentation:**

<https://github.com/mono/mono/tree/master/mcs/tools/linker-analyzer>

## Xamarin.Android Book

If you enjoyed this post, please consider subscribing to my upcoming book's email list:

You can signup at the following link: [Programming Xamarin.Android Email List](https://eepurl.com/cz_fj1)