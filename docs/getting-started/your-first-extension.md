---
title: Your first extension
description: Shows you the few simple steps to get your first Visual Studio extension up and running.
date: 2021-10-12
---

A Visual Studio extension is written using the .NET Framework and C#.
If you're already a .NET developer, you will find that writing extensions is similar to writing most other .NET programs and libraries.

The extension we'll be writing today adds a command that inserts a new guid into the text editor when executed. It's simple, useful, and provides a good introduction to the various aspects of extension development.

If you're a visual learner, check out this short video of someone following the tutorial.

<div class="video-container">
<iframe src="https://www.youtube-nocookie.com/embed/4JL8EawZMvY?list=PLReL099Y5nRdz9jvxuy_LgHFKowkx8tS4&color=white" title="YouTube video player" allowfullscreen></iframe>
</div>

Before we start writing our first Visual Studio extension (it's easy, I promise!), make sure you've got the [tools needed](get-the-tools.md).

## [Create the project](#create-the-project)
There are several project templates to choose from, so we want to make sure we make the right choice. The templates we use in this cookbook, all have the moniker **(Community)** in the name.

The **VSIX Project w/Command (Community)** template comes with a command hooked up, making it easy to start from there. This is a great starting point for most extensions. If you know you want a tool window, use the **VSIX Project w/Tool Window (Community)** template. It also has a command to open the tool window.

Use the **Empty VSIX Project (Community)** or **VSIX Project (Community)** templates for MEF-only extensions or other advanced scenarios.

This time, we'll select the **VSIX Project w/Command (Community)** template, as shown in the screenshot below.

![New Project Dialog showing VSIX project templates](../assets/img/new-project-dialog.png)

After selecting the project template, we need to give our project a name. Call it **InsertGuid**.

![Configure your new project](../assets/img/configure-new-project.png)

After hitting the *Create* button, you should end up with a basic VSIX Project looking like this:

![New project files and folders](../assets/img/new-project-files.png)

## [Overview of the files](#overview-of-the-files)
Let's go over the most important files.

**InsertGuidPackage.cs** is what we refer to as the Package class. Its `InitializeAsync(...)` method is called by Visual Studio to initialize your extension. It's from here you add event listeners and register commands, tool windows, settings and other things.

**source.extension.vsixmanifest** is the manifest file for your extension. It contains meta data such as title and description, but also information about what the extension contains.

**VSCommandTable.vsct** is an XML file where commands and keybindings are declaratively defined, so they can be registered with Visual Studio.

**Commands/MyCommand.cs** is the command handler for the command defined in the *VSCommandTable.vsct* file. It controls what happens when the command is executed - aka. the button is clicked.

## [Modifying the command](#modify-command)
First, we want to make sure our command has the right name, icon, and position within the Visual Studio menu system.

Open the *VSCommandTable.vsct* file and find a `<Group>` and a `<Button>`. Notice how the button specifies the group as being its parent and the group's parent is the built-in *VSMainMenu/Tools* menu.

For our extension, we want the *Insert GUID* command button to be under the *Edit* main menu, so we are going to re-parent the group to the Edit menu. Replace *Tools* with *Edit* just like in the following snippet:

```xml
<Group guid="InsertGuid" id="MyMenuGroup" priority="0x0600">
  <Parent guid="VSMainMenu" id="Edit"/>
</Group>
```

You get full IntelliSense for the placements to make it easy to find the right spot.

![VSCT parent IntelliSense](../assets/img/vsct-parent-intellisense.png)

The `<Button>` needs updating as well. We'll give it a new icon by updating the `id` attribute of the `<Icon>` element to *PasteAppend*. Update the `<ButtonText>` text with a good, descriptive name, and update the `<LocCanonicalName>` with the technical name of your command - this is the name shown users when they assign custom keyboard shortcuts to your command in the **Tools -> Options -> Environment -> Keyboard** dialog.

```xml
<Button guid="InsertGuid" id="MyCommand" priority="0x0100" type="Button">
  <Parent guid="InsertGuid" id="MyMenuGroup" />
  <Icon guid="ImageCatalogGuid" id="PasteAppend" />
  <CommandFlag>IconIsMoniker</CommandFlag>
  <Strings>
    <ButtonText>Insert GUID</ButtonText>
    <LocCanonicalName>.Edit.InsertGuid</LocCanonicalName>
  </Strings>
</Button>
```

> Always start the `<LocCanonicalName>` with a dot character. It ensures no other text is automatically prepended, and the dot will not be shown.  

We can use the thousands of icons available within Visual Studio's image library and even get a preview shown in IntelliSense:

![VSCT icon IntelliSense](../assets/img/vsct-icon-intellisense.png)

Now, we've updated the name, icon, and location of our command and it's time to write some code to insert the guid into the text editor.

Open the */Commands/MyCommand.cs* file and modify it to insert a new guid when executed:

```csharp
using System;
using Community.VisualStudio.Toolkit;
using EnvDTE;
using Microsoft.VisualStudio.Shell;
using Task = System.Threading.Tasks.Task;

namespace InsertGuid
{
    [Command(PackageIds.MyCommand)]
    internal sealed class MyCommand : BaseCommand<MyCommand>
    {
        protected override async Task ExecuteAsync(OleMenuCmdEventArgs e)
        {
            await Package.JoinableTaskFactory.SwitchToMainThreadAsync();
            var view = await VS.Documents.GetCurrentTextViewAsync();

            view?.TextBuffer.Insert(view.Caret.Position.BufferPosition, Guid.NewGuid().ToString());
        }
    }
}
```

We're using the `VS` object to get the active editor text view, and then insert the guid at its text buffer's caret position.

> You'll see `await JoinableTaskFactory.SwitchToMainThreadAsync()` and `ThreadHelper.ThrowIfNotOnUIThread()` in many places in this cookbook. They handle thread switching best practices and you don't need to know when and how to use them at this point - compiler warnings with Code Fixes (light bulbs) makes that super easy.

The first draft of our extension is now complete and it is time to test it.

## [Running and debugging](#running-and-debugging)
Running your extension is as easy as running any other .NET project. Simply hit *F5* to run with the debugger attached or *Ctrl+F5* for running without.

Doing so will start the Experimental Instance of Visual Studio with your extension installed. The Experimental Instance is your regular version of Visual Studio, but with separate settings and extensions installed. It helps keep things separate.

When the Experimental Instance starts up, you should see the *Insert GUID* command in the *Edit* main menu.

![Insert GUID command located in the Edit main menu](../assets/img/insert-guid-command.png)

Open any text based file and execute the command to insert a new guid. That's it!

## [Summary](#summary)
You've now created your first extension that adds a command button to the main menu and interacts with the text editor when executed.

Congratulations!!

You can find the code for this extension in the [samples repository](https://github.com/VsixCommunity/Samples).

## [Additional resources](#additional-resources)

* [Anatomy of extensions](extension-anatomy.md)
* [Menus & Commands](../recipes/menus-buttons-commands.html)
* [Best practices checklist](../publish/checklist.html)
