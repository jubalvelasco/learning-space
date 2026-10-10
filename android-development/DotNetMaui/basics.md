# .NET (MAUI) Basics

## Introduction to Project Files

- `MainPage.xaml` -> contains:

    - `MainPage.xaml.cs` -> defines `MainPage` class and inherits `ContentPage` class:

        - ```csharp
            public partial class MainPage : ContentPage
            {
                int count = 0;

                public MainPage()
                {
                    InitializeComponent();
                }

                // ... event functions in UI
            }
            ```

- `AppShell.xaml` -> contains:

    - ```xml
        <ShellContent
            Title="Home"
            ContentTemplate="{DataTemplate local:MainPage}"
            Route="MainPage" />
        ```

        The xaml file renders the `MainPage` as shown above.

    - `AppShell.xaml.cs` -> defines `AppShell` class and inherits `Shell` class:

        - ```csharp
            public partial class AppShell : Shell
            {
                public AppShell()
                {
                    InitializeComponent();
                }
            }
            ```

    - Pages that are part of the Shell visual hierarchy (declared with `ShellContent` in `AppShell.xaml`) are registered automatically through their `Route`. Every other page you want to navigate to must be registered with `Routing.RegisterRoute` (see below), because `AppShell` provides the url based navigation.

- `App.xaml` -> contains:

    - `App.xaml.cs` -> defines `App` class and inherits `Application` class:
        
        - Creates an instance of the `AppShell` registered pages

        - ```csharp
            public partial class App : Application
            {
                public App()
                {
                    InitializeComponent();
                    MainPage = new AppShell();
                }
            }
            ```

        - NOTE: in .NET 9+ setting `MainPage` is deprecated, override `CreateWindow` instead:
            ```csharp
            protected override Window CreateWindow(IActivationState? activationState)
                => new Window(new AppShell());
            ```

- `MauiProgram.cs` - Starting point for cross-platform development, uses the `App` class to build the app.

    - ```csharp
        public static class MauiProgram
        {
            public static MauiApp CreateMauiApp()
            {
                var builder = MauiApp.CreateBuilder();
                builder
                    .UseMauiApp<App>()
                    .ConfigureFonts(fonts => {
                        // ...
                    });

                return builder.Build();
            }
        }
        ```

- **Platforms** - folder containing the starting point of the program for each specific platform.
    - Each platform runs `MauiProgram.cs`'s `CreateMauiApp()`

        ```csharp
        MauiProgram.CreateMauiApp();
        ```

## Visual Structure

```mermaid
graph RL
    
  ACTOR["USER/ACTOR"]

  subgraph "Page Layers"
    MAP["MainPage.xaml"]
    MAPCS["MainPage.xaml.cs"]
  end

  subgraph "AppShell"
    AS["AppShell.xaml"]
    ASCS["AppShell.xaml.cs"]
  end

  subgraph "App"
    AP["App.xaml"]
    APCS["App.xaml.cs"]
  end

  MP["MauiProgram.cs"]

  subgraph "Platforms Layer"
    P1["iOS"]
    P2["Android"]
    P3["windows"]
  end

  ACTOR <--> MAP
  MAP <--> MAPCS
  AS <--> MAP
  AS <--> ASCS
  AP <--> APCS
  APCS <--> ASCS
  MP <--> APCS
  P1 <--> MP
  P2 <--> MP
  P3 <--> MP
```

## Architecture - Event Driven Development

.NET (MAUI) is a stateful application

```mermaid
graph LR
ACTOR["User/Actor"]
VIEW["View (UI)"]
STATE["State"]
EVENTS["Events (e.g. On Button Click)"]

VIEW --> EVENTS
EVENTS --> STATE
STATE --> VIEW
ACTOR --> VIEW
```

- **State** is the data in a page class
- **View** is an element or layout of a page, or a page itself.
    - `.xaml` files represent the **UI**
    - `.xaml.cs` files define the **UI Logic** (code-behind)
    - There are many types of Views like View UI Elements and View Layouts which can be nested together.
    - In a **Page** (`<ContentPage>`) there should only be one **Parent View Layout** (the `Content` property holds a single view, so put the other views inside a layout)

## Architecture - MVVM Development

N/A

## URL Navigation - Creating New Pages

In Visual Studio you can do the following to create new pages with the right namespaces.

1. _Project\File View Panel_ > _Right Click_ > _Project Name_ > `Add` > `New Folder` > _Input Folder Name_

2. _Right Click New Folder_ > `Add` > `New Item` > `.NET MAUI Content Page (XAML)` > _Input Page Name_

In VS Code in ubuntu development, you might need to fix the namespace of the `.xaml.cs` files (and the `x:Class` attribute in the `.xaml` file) of the newly created page inside new folders, because by default it can only create pages in the local namespace.

## URL Navigation - Register New Pages

To start with url based navigation, we first need to register the newly created pages in the `AppShell`.

```csharp
public partial class AppShell : Shell
{
	public AppShell()
	{
		InitializeComponent();
        // ...

        // you might need to include the namespace of the page you want to register.
        Routing.RegisterRoute(
            nameof(PageName), // string: this could be any value but using the page name is cleaner.

            typeof(PageName) // type of the page class
        );
	}
}
```

## URL Navigation - Setting Up New Main Page

In `AppShell.xaml` we can setup and choose a new main page

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Shell
    ...
    xmlns:local="clr-namespace:ProjectName"
    xmlns:mynamespace="clr-namespace:ProjectName.NewFolder"
    ...
    Title="PageTitle">

    <ShellContent
        ...
        ContentTemplate="{DataTemplate mynamespace:NewMainPage}"
        Route="NewMainPage" />

</Shell>
```

The folder and page `NewFolder/NewMainPage.xaml.cs` should have a namespace of:

```cs
namespace ProjectName.NewFolder;
```

Or you can just create a page anywhere in the project structure that has a namespace in the local scope:

```cs
namespace ProjectName;
```

## URL Navigation - Go To Other Pages

When clicking a button go to another page.

From **NewMainPage.xaml**

```xml
<Button
    Text="btnTxt"
    x:Name="btnName"
    Clicked="btnEHandlerFuncName" />
```

From **NewMainPage.xaml.cs**

```cs
private async void btnEHandlerFuncName(object sender, EventArgs e)
{
    // in order for this to work the page should be registered in the AppShell
    // (the string must match the route name used in Routing.RegisterRoute)
    await Shell.Current.GoToAsync(nameof(OtherPage));
}
```

## URL Navigation - Go Back To Parent Page

```cs
// go back one page (pop the navigation stack)
await Shell.Current.GoToAsync("..");

// or, go to an absolute route (`//` + the Route of a ShellContent in AppShell.xaml),
// this resets the navigation stack
await Shell.Current.GoToAsync($"//{nameof(NewMainPage)}");
```
