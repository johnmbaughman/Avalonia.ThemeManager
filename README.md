# Avalonia Theme Manager

[![Build status](https://dev.azure.com/wieslawsoltes/GitHub/_apis/build/status/Sources/Avalonia.ThemeManager)](https://dev.azure.com/wieslawsoltes/GitHub/_build/latest?definitionId=72)

[![NuGet](https://img.shields.io/nuget/v/Avalonia.ThemeManager.svg)](https://www.nuget.org/packages/Avalonia.ThemeManager)

[![Github All Releases](https://img.shields.io/github/downloads/wieslawsoltes/avalonia.thememanager/total.svg)](https://github.com/wieslawsoltes/avalonia.thememanager)
[![GitHub release](https://img.shields.io/github/release/wieslawsoltes/avalonia.thememanager.svg)](https://github.com/wieslawsoltes/avalonia.thememanager)
[![Github Releases](https://img.shields.io/github/downloads/wieslawsoltes/avalonia.thememanager/latest/total.svg)](https://github.com/wieslawsoltes/avalonia.thememanager)

## About

Theme manager for [Avalonia](https://github.com/AvaloniaUI/Avalonia) applications.

## Usage

Theme manager searches user provided themes directory for `*.xaml` theme files otherwise built-in `Light` and `Dark` theme are used.

The `ThemeSelector` is created and initalized by calling static `Create` method.

The `ThemeSelector` uses `Styles[0]` property of `Window` to insert selected theme `Style`.

`App.xaml`
```XAML
<Application x:Class="AvaloniaApp.App"
             xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Application.Styles>
        <StyleInclude Source="avares://Avalonia.Themes.Default/DefaultTheme.xaml"/>
        <StyleInclude Source="avares://Avalonia.Themes.Default/Accents/BaseLight.xaml"/>
    </Application.Styles>
</Application>
```

`App.xaml.cs`
```C#
using System;
using Avalonia;
using Avalonia.Logging.Serilog;
using Avalonia.Markup.Xaml;
using Avalonia.ThemeManager;

namespace AvaloniaApp
{
    public class App : Application
    {
        public static IThemeSelector Selector;

        [STAThread]
        static void Main(string[] args)
        {
            BuildAvaloniaApp().Start(AppMain, args);
        }

        static void AppMain(Application app, string[] args)
        {
            Selector = ThemeSelector.Create("Themes");
            Selector.LoadSelectedTheme("AvaloniaApp.theme");

            app.Run(new MainWindow());

            Selector.SaveSelectedTheme("AvaloniaApp.theme");
        }

        public static AppBuilder BuildAvaloniaApp()
            => AppBuilder.Configure<App>()
                         .UsePlatformDetect()
                         .UseReactiveUI()
                         .LogToDebug();

        public override void Initialize()
        {
            AvaloniaXamlLoader.Load(this);
        }
    }
}
```

`MainWindow.xaml`
```XAML
<Window x:Class="AvaloniaApp.MainWindow"
        xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:app="clr-namespace:AvaloniaApp;assembly=AvaloniaApp"
        xmlns:manager="clr-namespace:Avalonia.ThemeManager;assembly=Avalonia.ThemeManager"
        Title="AvaloniaApp" Width="800" Height="600"
        Foreground="{DynamicResource ThemeForegroundBrush}">
    <Window.Resources>
        <manager:ObjectEqualityMultiConverter x:Key="ObjectEqualityMultiConverter"/>
    </Window.Resources>
    <Grid RowDefinitions="Auto,*">
        <Menu Grid.Row="0">
            <MenuItem Header="_View">
                <MenuItem Header="_Theme" DataContext="{x:Static app:App.Selector}" Items="{Binding Themes}">
                    <MenuItem.Styles>
                        <Style Selector="MenuItem">
                            <Setter Property="Header" Value="{Binding Name}"/>
                            <Setter Property="Command" Value="{Binding Selector.ApplyTheme}"/>
                            <Setter Property="CommandParameter" Value="{Binding}"/>
                            <Setter Property="Icon">
                                <Template>
                                    <CheckBox>
                                        <CheckBox.IsChecked>
                                            <MultiBinding Mode="OneWay" Converter="{StaticResource ObjectEqualityMultiConverter}">
                                                <Binding Path="DataContext" RelativeSource="{RelativeSource Self}"/>
                                                <Binding Path="Selector.SelectedTheme"/>
                                            </MultiBinding>
                                        </CheckBox.IsChecked>
                                    </CheckBox>
                                </Template>
                            </Setter>
                        </Style>
                    </MenuItem.Styles>
                </MenuItem>
            </MenuItem>
        </Menu>
    </Grid>
</Window>
```

`MainWindow.xaml.xs`
```C#
using Avalonia;
using Avalonia.Controls;
using Avalonia.Markup.Xaml;
using Avalonia.ThemeManager;

namespace AvaloniaApp
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            this.InitializeComponent();
            this.AttachDevTools();
            App.Selector.EnableThemes(this);
        }

        private void InitializeComponent()
        {
            AvaloniaXamlLoader.Load(this);
        }
    }
}
```

The `EnableThemes(...);` can be used to enable themes for multiple windows.

## License

Avalonia.ThemeManager is licensed under the [MIT license](LICENSE.TXT).
