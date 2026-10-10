# **Using dotnet in CLI**

[Microsoft's Official Documentation](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet)

These are my notes for dotnet 6.

## **My release build**

```
dotnet publish -p:PublishSingleFile=true -p:PublishTrimmed=true -r win-x64 --self-contained true -c Release
```

## **Create dotnet `gitignore`**

```
dotnet new gitignore
```

## **Create a solution file**

This will create a new solution file in the current directory.

```
dotnet new sln
```

## **Create a project app with the `console` template**

This will create a basic console C# application in a new `sampleApp` folder inside the current directory (`-o` sets the output folder; without it, the project is created directly in the current directory).

```
dotnet new console -o sampleApp
```

## **Create a project library with the `classlib` template**

This will create a library/module in a new `sampleLib` folder inside the current directory.

```
dotnet new classlib -o sampleLib
```

## **Add projects to the solution file**

This will add the projects to the solution file in the current directory. The project folders `sampleApp` and `sampleLib` should also exist in the same directory as the solution file.

```
dotnet sln add sampleApp sampleLib
```

## **Add a package to the library project**

```
cd sampleLib
dotnet add package <Package Name>
```

## **Reference the library/module to the application**

```
cd sampleApp
dotnet add reference ../sampleLib/sampleLib.csproj
```

## **Build the application**

```
cd sampleApp
dotnet build
```

## **Run the application**

```
cd sampleApp
dotnet run
```

## **Clean the `build` outputs**

```
cd sampleApp
dotnet clean
```

## **Publish, Bundle or Compile the application into release or debug builds**

This will create a single-file executable that can run on Windows x64 machines that have the .NET 6 runtime installed (it will not run on machines that don't have the runtime). In .NET 6/7 `dotnet publish` builds the **Debug** configuration by default (since .NET 8 the default is **Release**); add `-c Release` or `-c Debug` to be explicit.

```
dotnet publish -p:PublishSingleFile=true -r win-x64 --self-contained false
```