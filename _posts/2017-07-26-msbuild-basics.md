---
layout: post
title: MSBuild Basics
---

# MSBuild Basics

MSBuild is the music conductor of build tooling. It is extremely powerful for understanding and customizing your build process.

## Quick Installation

You should have `MSBuild` installed already on your computer. Ensure you are in the correct path or add the `msbuild` path to your `Environment Variables`.

Basic MSBuild command:

```
msbuild [INPUT].csproj /t:[TARGET]
```

## Project File (.csproj is the MSBuild file)

```
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
</Project>
```

**Xamarin.Android Example:**

```
<Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

All content will be placed inside of the `<Project>` tag. This includes, properties, items, targets, etc.

Think of this as the files being build, parameters for build, and much more.

## Properties

MSBuild properties are key-value pairs. The `key` is the name that you use to refer the property. The `value` is the value of the property. 

When declaring a property, they must be contained inside a `<PropertyGroup>` element. This element must be inside of the `<Project>` element.

**Note:** You can have separate properties as long as they are in their own element.

**Xamarin.Android Example:**

```
<PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    <ProductVersion>8.0.30703</ProductVersion>
    <SchemaVersion>2.0</SchemaVersion>
    <ProjectGuid>{221B57B2-7897-411E-992F-D8046688CFA3}</ProjectGuid>
    <ProjectTypeGuids>{EFBA0AD7-5A72-4C68-AF49-83D382785DCF};{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}</ProjectTypeGuids>
    <OutputType>Library</OutputType>
    <AppDesignerFolder>Properties</AppDesignerFolder>
    <RootNamespace>App8</RootNamespace>
    <AssemblyName>App8</AssemblyName>
    <FileAlignment>512</FileAlignment>
    <AndroidApplication>True</AndroidApplication>
    <AndroidResgenFile>Resources\Resource.Designer.cs</AndroidResgenFile>
    <GenerateSerializationAssemblies>Off</GenerateSerializationAssemblies>
    <AndroidUseLatestPlatformSdk>True</AndroidUseLatestPlatformSdk>
    <TargetFrameworkVersion>v7.1</TargetFrameworkVersion>
    <AndroidManifest>Properties\AndroidManifest.xml</AndroidManifest>
  </PropertyGroup>
```

## Tasks and Targets

### Task

A `Task` is the smallest unit of work. Typically known as an action or routine.

Example of a `Task`:

```
<Message Text="Hello Support Team Members!" />
```

### Target

A `Target` is a series of executable steps/tasks.

Example of an empty `Target`:

```
<Target Name="HelloSupportTeam">
</Target>
```

### Default Tasks

`Copy`, `Move`, `Exec`, `ResGen`, `Csc` - Copy Files, Move Files, Execute a program, Generate Resources, C# Compiler

`Message` - Sends a message to the loggers that are listening in the build process.

```
<Target Name="HelloSupportTeam">
    <Message Text="Hello Support Team Members!" />
</Target>
```

### Running a Target

A `Target` can be run in MSBuild simply by the following syntax 

```
msbuild MyProject.csproj /t:<TargetName> or msbuild MyProject.csproj /target:<TargetName>
```

Let's try running the `HelloSupportTeam` target now.

```
msbuild MyProject.csproj /t:HelloSupportTeam
```

You should see the following output:

```
HelloSupportTeam:
  Hello Support Team Members!
```

## Properties inside a Task

Let's now define a `<PropertyGroup>` with our message instead.

```
<PropertyGroup>
    <HelloSupportTeamMessage>Hello again brownbaggers!</HelloSupportTeamMessage>
</PropertyGroup>
```

Now we need to use the property variable instead. Let's edit our `<Message>` Task. The syntax for a property in MSBuild is `$(PropertyName)`:

```
<Target Name="HelloSupportTeam">
    <Message Text="$(HelloSupportTeamMessage)" />
</Target>
```

## Items (Aka Files)

Items are file-based references. Similar to `<PropertyGroup>`, you define them by using an `<ItemGroup>` instead.

**Xamarin.Android Example:**

```
  <ItemGroup>
    <Compile Include="MainActivity.cs" />
    <Compile Include="Resources\Resource.Designer.cs" />
    <Compile Include="Properties\AssemblyInfo.cs" />
  </ItemGroup>
```

To evaulate an item, you would use the `@(ItemType)` syntax. Let's now spit out a `Message` to see what is all going to be compiled here.

```
<Target Name="PrintCompileInfo">
    <Message Text="Compile: @(Compile)" />
</Target>
```

## Item Metadata

Items have metadata to them as well. There are many well-known metadata items that we can query, however I will not be going into detail on them all. Here is a sample of getting the directory of an item:

```
<Target Name="PrintMetadata">
    <Message Text="%40(Compile-> '%25(Directory)'): @(Compile->'%(Directory)')" />
</Target>
```

## Conditions (<https://youtu.be/yhOKhJaM1QE?t=8>)

`Condition` are used to evaluate a true or false condition.

**Xamarin.Android Example:**

```
  <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
```

You can check multiple types of conditions such as:

- `==` - Equality
- `!=` - Inequality
- `Exists` - If something exists
- `!Exists` - If something does not exist

Conditions are especially useful for adding an `Item` based on a condition such as a `$(Configuration)` being `Debug` or `Release`

## Initial Targets

You may be asking yourself...How the hell do we know what Target get executed first? Well the truth is, I slipped a detail past you really quickly. Let's go back to our original definition of our `<Project>`:

```
<Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

This will default any `Target` named `Build` will be the default target if no other `Target` is defined when invoking.

## Extending MSBuild

Okay now for the actual fun part. Extending the build process.

There's a few ways to extend the MSBuild process in the sense of a pre or post build action.

1. Pre and Post build events
2. Override `BeforeBuild` / `AfterBuild` target
3. Extend the `BuildDependsOn` list

```
<Target Name="BeforeBuild">
</Target>

<Target Name="AfterBuild">
</Target>
```

## Common files names

### .proj or .csproj

.proj is the generic. .csproj is a C# specific. .vbproj is the Visual Basic specific

### .targets

.targets is a file that contains shared targets which are imported into other files

### .props 

.props is a file that contains settings for the build process

### .tasks

.tasks is a file that contains `UsingTask` definitions

## Intellisense of items

There's a few `xsd` files that are used in VS for general intellisense. You can also look for an item directly by opening the schema:

```
C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Xml\Schemas\1033\MSBuild
```

## Debugging Tasks

Three main ways:

1. Use logs of logging statements and examine the logger
2. Use `Debugger.Launch()` to prompt for a debugger attachment
3. Start `MSBuild` as an external program, and then debug normally

## Other Resources

The most invaluable resource you'll have on MSBuild is Sayed's book:

<http://msbuildbook.com/>

Video Resources:

<https://channel9.msdn.com/Shows/Code-Conversations/Introduction-to-MSBuild-in-NET-Core-with-Nate-McMaster>

<https://channel9.msdn.com/Shows/Code-Conversations/Advanced-MSBuild-Extensibility-with-Nate-McMaster>

<https://channel9.msdn.com/Shows/Code-Conversations/Sharing-MSBuild-Tasks-as-NuGet-Packages-with-Nate-McMaster>