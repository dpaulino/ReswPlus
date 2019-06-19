<img src="https://user-images.githubusercontent.com/1226538/56482508-6fbd2d00-6479-11e9-8fc0-b20d5f3171ad.png" height="80" />

# ReswPlus - Advanced File Code Generator for Resw files.


ReswPlus is a Visual Studio extension enriching your existing .resw files with many high valuable features:
- Access to strings via strongly typed static properties.
- Generates methods to format your strings with named and strongly typed parameters.
- Adds pluralization support (support plural rules for 196 languages!).
- In addition to pluralization, also supports empty states when the number of items is zero.
- Generate a Markup extension to access to your strings with compile-time verification.

Currently supported: 
- Visual Studio 2017 and 2019 (all versions).
- C#, VB.Net and C++/CX apps (C++/winrt coming soon).

![reswplus](https://user-images.githubusercontent.com/1226538/56525314-a76eb800-64ff-11e9-9e39-1bb4cd2dd012.gif)



|                                                 | Resw | Resw + ReswPlus | Resx | Android XML (for reference) |
|-------------------------------------------------|------|-----------------|------|-------------|
| Modify UI properties via resource files (x:uid) | ✅    | ✅               |      |             |
| Strongly typed accessors                        |      | ✅               | ✅    | ✅           |
| Plural forms                                    |      | ✅               |      | ✅           |
| 'None' state                                    |      | ✅               |      |             |
| Strongly typed string formatting                |      | ✅               |      |             |
| Support Resources in libraries                  |      | ✅               | ✅    |             |

## How to install

ReswPlus supports Visual Studio 2017 and 2019.

In Visual Studio, select `Tools > Extensions and Updates...`, then select `Online` on the left, search `ReswPlus` and click on Install. 

Alternatively, you can directly download the extension here: https://marketplace.visualstudio.com/items?itemName=rudyhuyn.ReswPlus

## How to activate ReswPlus in my project

In your project, right-click on the resw file of the default language of your application (commonly `/Strings/en/Resources.resw`) and select the menu `ReswPlus`.

In the submenu, select:
- `Generate strongly typed class`: stronged typed generation, string formatting, custom markup
- `Generate strongly typed class with pluralization`: all the above + pluralization and empty state support (the nuget package ReswPlusLib will be automatically added to your project)

<img src="https://user-images.githubusercontent.com/1226538/59745769-57278400-922a-11e9-8395-f87f8faeb4bd.png" height="120" />

It will automatically generate a file xxx.generated.cs associated to your Resource file.

<img src="https://user-images.githubusercontent.com/1226538/56481455-f111c100-6473-11e9-8c04-f512a6136fd2.png" height="120" />

The generated code file will be automatically updated when the .resw file is modified and saved.

## Features
### Strongly typed static properties
ReswPlus generates a class exposing all strings from your .resw files as strongly typed static properties, providing a compile-time-safe way to access those strings XAML-side or code-side. 

Contrary to `ResourceLoader.GetString("IdString"),` the compiler will verify how your XAML and C# access your strings and will fail the compilation if a resource doesn't exist.

This feature will allow you to localize your applications using bindings (including native bindings) and code-behind (similar to .resx files in WPF/Silverlight applications) and will allow you to use converter, functions, etc...

The privilegied way to access strings XAML-side is using native bindings. An alternative is normal binding, but these don't provide verification at compilation time. To fix this issue, ReswPlus also generates a custom MarkupExtension (verified at compilation time), also supporting Converters.

**Code generated:**
```csharp
public class Resources {
    private static ResourceLoader _resourceLoader;
    static Resources()
    {
        _resourceLoader = ResourceLoader.GetForViewIndependentUse();
    }
    public static string WelcomeTitle => _resourceLoader.GetString("WelcomeTitle");
}
```

**How to use it:**

XAML - native binding:
```xaml
<TextBlock Text="{x:Bind strings:Resources.WelcomeTitle}" />
```
XAML - special markup (generated by Resw):
```xaml
<TextBlock Text="{strings:Resources Key=WelcomeTitle}" />
```
Code behind:
```csharp
titlebar.Title = Resources.WelcomeTitle;
```
These 3 ways are compile-time verified.

### String formatting

ReswPlus can generate strongly typed methods to format your strings. Simply add the tag `#ReswPlusTyped[...]` in the comment column and ReswPlus will automatically generate a method YourResourceName_Format(..) with strongly typed parameters.

Types currently supported for parameters: _string, int, uint, object, byte, long, double, char, ulong, decimal_

Resw also allows you to name the parameters to make the code easy to read.

**Example:**

The resource:

| Key                  | Value                                           | Comment                                                    |
|----------------------|-------------------------------------------------|------------------------------------------------------------|
| ForecastAnnouncement | The temperature in {2} is {0}°F ({1}°C)         | #ReswPlusTyped[d(fahrenheit), d(celsius), s(city)]         |

will generate the following code, with strong type and named parameters based on the hashtag in the comment section):

```csharp
#region ForecastAnnouncement
/// <summary>
///   Looks up a localized string similar to: The current temperature in {2} is {0}°F ({1}°C)
/// </summary>
public static string ForecastAnnouncement => _resourceLoader.GetString("ForecastAnnouncement");

/// <summary>
///   Format the string similar to: The current temperature in {2} is {0}°F ({1}°C)
/// </summary>
public static string ForecastAnnouncement_Format(int tempFahrenheit, int tempCelsius, string city)
{
	return string.Format(ForecastAnnouncement, tempFahrenheit, tempCelsius, city);
}
#endregion
```

### Pluralization

ReswPlus can generate methods to easily access your pluralized strings. Simply right-click on your resw file, select `ReswPlus` > `Generate strongly typed class with pluralization`, the nuget package `ReswPlusLib` will be automatically added to your project and generate all the functions necessary to manage your localization.

**Example:**

The resources:

| Key               | Value            | Comment           |
|-------------------|------------------|-------------------|
| MinutesLeft_One   | {0} minute left  | #ReswPlusTyped[Q] |
| MinutesLeft_Other | {0} minutes left |                   |

Will automatically generate the following code:

```csharp
#region MinutesLeft
/// <summary>
///   Get the pluralized version of the string similar to: {0} minute left
/// </summary>
public static string MinutesLeft(double number)
{
	return Huyn.PluralNet.ResourceLoaderExtension.GetPlural(_resourceLoader, "MinutesLeft", (decimal)number);
}
/// <summary>
///   Format the string similar to: {0} minute left
/// </summary>
public static string MinutesLeft_Format(double pluralCount)
{
	return string.Format(MinutesLeft(pluralCount), pluralCount);
}
#endregion
```

PluralNet will then automatically select one of the string based on the number passed as a parameter. While English has only 2 plural forms, some languages have up to 5 different forms, 196 different languages are supported by this library.

Pluralization can be used in combination with string formatting.

**Example:**

| Key              | Value                          | Comment                                 |
|------------------|--------------------------------|-----------------------------------------|
| FileShared_One   | {0} shared {1} photo from {2}  | #ReswPlusTyped[s(username), Q, s(city)] |
| FileShared_Other | {0} shared {1} photos from {2} |                                         |

Will generate:

```csharp
#region FileShared
/// <summary>
///   Get the pluralized version of the string similar to: {0} shared {1} photos from {2}
/// </summary>
public static string FileShared(double number)
{
	return Huyn.PluralNet.ResourceLoaderExtension.GetPlural(_resourceLoader, "FileShared", (decimal)number);
}
/// <summary>
///   Format the string similar to: {0} shared {1} photos from {2}
/// </summary>
public static string FileShared_Format(string username, double paramDouble2, string city)
{
	return string.Format(FileShared(paramDouble2), username, paramDouble2, city);
}
#endregion
```

### Empty States

In addition to plural forms, one common task normally delegated to the ViewModel is to display a special message when a quantity is zero, some examples: *'no search results', 'history empty', 'no new messages'* etc... 

ReswPlus provides a way to automate this task and automatically provide the empty state string when necessary, simply add a `_None` state to your pluralized strings:

Example:

Resources:

| Key                    | Value                | Comment                           |
|------------------------|----------------------|-----------------------------------|
| ReceivedMessages_None  | No new messages      |                                   |
| ReceivedMessages_One   | You got {0} message  | #ReswPlusTyped[Q(numberMessages)] |
| ReceivedMessages_Other | You got {0} messages |                                   |

The following code:

```xaml
<TextBlock Text="{x:Bind strings:Resources.ReceivedMessages_Format(ViewModel.NumberMessage), Mode=OneWay}" />
```

will automatically display `No new messages`, `You got 1 message`, `You got 1 messages` based on the value of `ViewModel.NumberMessage`.


