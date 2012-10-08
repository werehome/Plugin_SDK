Plugin SDK for CryEngine SDK
=====================================
Purpose is to create a tailored way for automatically loading plugins.
For redistribution please see license.txt.

Available Plugins
-----------------
- [CryMono](http://crymono.inkdev.net/) - Brings the power of .NET into the world of CryEngine
- [Plugin_Flite](https://github.com/hendrikp/Plugin_Flite) - Provides Text to Speech
- [Plugin_Crash](https://github.com/hendrikp/Plugin_Crash) - Crashes the process and serves as sample plugin
- **WIP** [Oohh](https://github.com/CapsAdmin/oohh) - Advanced Lua scripting, featuring various builtin libraries (like Garry's Mod)
- **WIP** Plugin_Videoplayer - Videoplayer for 2D screen and 3D objects using WebM format
- **WIP** Plugin_D3D - Exposes Direct3D functionality
- **WIP** Plugin_OSC - [Open Sound Control](http://opensoundcontrol.org/) protocol support for integrating external applications
- **WIP** Plugin_Camera - Advanced configurable camera system (Third Person, Stategy/Top, Side-Scroller, Static, whatever)

Feature requests/latest version on github.
https://github.com/hendrikp/Plugin_SDK

Installation / Integration
==========================
Extract the files to your Cryengine SDK Folder so that the Code and BinXX/Plugins directory match up.

If you have a custom GameDll that doesn't contain the PluginManager yet then you will need to recompile it see C++ Integration.

Installing the Visual Studio 2010 Plugin Wizard
-----------------------------------------------
An installer is planned, until then:

* Copy the following 3 files to "Documents\Visual Studio 2010\Wizards\PluginWizard"
  * PluginWizard.vsz
  * PluginWizard.vsdir
  * PluginWizard.ico

* Open the file "PluginWizard.vsz" with a text editor
  * Fix the path so it points to your Plugin SDK Wizard installation directory 
    ```Param="ABSOLUTE_PATH = C:\cryengine3_3.4.0\Code\Plugin_SDK\wizard\vc10"```

* The Wizard should now work, please note the following things (otherwise the wizard won't work)
  * the project name shouldn't contain "Plugin" as this will be added automatically
  * untick the checkbox to create solution directory
  * only create projects in the Code directory e.g. "C:\cryengine3_3.4.0\Code"

Creating a new plugin
=====================
https://github.com/hendrikp/Plugin_SDK/wiki/Creating-a-new-Plugin

You can also look into this sample:
https://github.com/hendrikp/Plugin_Crash

CVars
=====
* ```pm_list```
  List one info row for all plugins
* ```pm_dump```
  Dump info on a specific plugins
* ```pm_dumpall```
  Dump info on all plugins
* ```pm_unload```
  Unload a specific plugin using its name. Works only limited for plugins containing resources that are in use (the CryEngine usage of those objects needs to be fully cleaned up beforehand).
* ```pm_unloadall```
  Unload all plugins with the exception of the plugin manager. Works only limited for plugins containing resources that are in use (the CryEngine usage of those objects needs to be fully cleaned up beforehand).
* ```pm_reload```
  Reload a specific plugin using its path
* ```pm_reloadall```
  Reload all plugins

C++ Integration
===============
Follow these steps to integrate the Plugin SDK into your GameDLL.

Repository Setup
----------------
* Clone the repository to your local machine to your Code directory e.g. C:\cryengine3_3.4.0\Code

```
    git clone https://github.com/hendrikp/Plugin_SDK.git
```

Compiler Settings
-----------------
* Open the Solution CryEngine_GameCodeOnly.sln
* Open CryGame Properties
* Set the Dropdowns to All Configurations, All Platforms
* Add to C/C++ -> General -> Additional Include Directories:

```
    ;$(SolutionDir)..\Plugin_SDK\inc
```

* Apply those properties and close the dialog

Source Files
------------
* Inside the CryGame Project (Solution Explorer)
  Open the following File: Startup Files / GameStartup.cpp
* Add behind the existing includes the following:

```C++
    #include <IPluginManager_impl.h>
```

* Find the function CGameStartup::Init
* Add in the middle of the function (before ```REGISTER_COMMAND("g_loadMod", RequestLoadMod,VF_NULL,"");```) the following:

```C++
	PluginManager::InitPluginManager(startupParams);
	PluginManager::InitPluginsBeforeFramework();
    REGISTER_COMMAND("g_loadMod", RequestLoadMod,VF_NULL,""); // <--
```

* Add at the end of the function (before ```return pOut;```) the following:

```C++
    PluginManager::InitPluginsLast();
    return pOut; // <--
```

* Replace the line **(its important you don't add semicolons or braces)**

```C++
    LRESULT CALLBACK CGameStartup::WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
```

* with the following:

```C++
    PLUGIN_SDK_WINPROC_INJECTOR(LRESULT CALLBACK CGameStartup::WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam))
```

Compiling
---------
Either add the project file to your CryGame solution and the plugin manager as dependency for each plugin.

Or use the supplied seperate solution to compile the plugin manager.

Code Style
----------
In general look how its done in the CryEngine FreeSDK. Exceptions prove the rule.

* For variables (e.g. sTestMe or m_sTestMe)
  * lower camel casing
  * hungarian notation
  * m_ prefix for member variables
  * g prefix for global variables (in general dont use globals and when still needed then static)
* For functions (e.g. DoSomething)
  * upper camel casing
* Naming conventions
  * class declarations begin with C
  * structur declarations begin with S
  * interface declarations begin with I
  * char*/all string definitions with s
  * pointer defintions begin with p
  * integers defintions begin with n/i
  * floats begin with f
  * doubles begin with d
  * rest can be looked up e.g. here http://www.yolinux.com/TUTORIALS/LinuxTutorialC++CodingStyle.html
* Macros in capitals and words seperated by _
* Dont use RTTI
* Everything should be in a namespace
* Use tools/stylecode.bat before committing changes
* The rest will be fixed when your pull gets merged

Contribution Guideline
----------------------
* Change as little as possible to have the effect you want
* Discuss major/version changes beforehand
* Update the doxygen documentation

Contributing
------------
* Fork it
* Create a branch (`git checkout -b my_PluginSDK`)
* Put your name into authors.txt
* Commit your changes (`git commit -am "Added ...."`)
* Push to the branch (`git push origin my_PluginSDK`)
* Open a Pull Request


