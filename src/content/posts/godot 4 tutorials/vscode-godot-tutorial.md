---
title: VSCode with Godot 4 and Common Bugs Tutorial
published: 2025-02-07
description: 'Quick tutorial on how to connect VSCode with Godot and common bugs'
image: ''
tags: [Godot, Tutorial]
category: 'Godot 4 Tutorials'
draft: false 
lang: ''
---

# Requirements
1. VSCode
2. Godot
3. [godot-tools plugin for VSCode](https://marketplace.visualstudio.com/items?itemName=geequlim.godot-tools)

*Other Optional Plugins (not related to the setup):*
1. [C# Tools for Godot](https://marketplace.visualstudio.com/items?itemName=neikeq.godot-csharp-vscode) -> C# Support
2. [Godot Files](https://marketplace.visualstudio.com/items?itemName=alfish.godot-files) -> Shader support
3. [Godot Hover Docs](https://marketplace.visualstudio.com/items?itemName=RedMser.godot-hover-docs) -> Get documentation information for C++ (GDExtension)
4. [Godot Snippets for C#](https://marketplace.visualstudio.com/items?itemName=altamkp.godot-snippets-vscode-csharp) -> Snippest for C#
5. [Godot Docs for C#](https://marketplace.visualstudio.com/items?itemName=altamkp.godot-docs-vscode-csharp) -> Hover over to get documentation for C#
6. [Godot Editor Theme](https://marketplace.visualstudio.com/items?itemName=mireille-arseneault.godot-editor-theme) -> Godots theme for VSCode

---

# Tutorial

1. Close all the scripts open in the editor
:::tip
You can do this quickly via right clicking where the list of scripts are on the left and say "close all"
:::
2. Go to `Editor Settings`
3. Go to `Text Editor -> External`
4. In `Exec Path` get the location of the executable for `VS Code`
5. In `Exec Flags` input `{project} --goto {file}:{line}:{col}`
6. Enable `Use External Editor`
7. Double click a script to open it will open `VSCode`

Now whenever you open a project in Godot, VSCode will load automatically!

---

# Common Bugs

Godot can be a bit finnicky with it's *language server* (how external editors are able to do things like autocomplete). To my knowledge, this is not a `VSCode` issue but a `Godot` issue as I've had mixed results with a multitude of editors include `nvim`, `Zed` and `Rider`. `VSCode` however for the most part works fine, but you may run into a bug here and there. Here are the solutions I've found to fix it.

## Changing code in VSCode makes Godot bring up a warning that files were changed outside the project and whether to load the new fields or keep the ones in the project

Sometimes, Godot and VSCode may disconnect. There are two things you can do to try to fix it
1. Close VSCode and reopen by opening a script from Godot
    - If VSCode is open before the Godot project, it may cause connection issues
2. Restart the Godot Project
3. Make sure in your VSCode `godot-tools` setup, you have the correct version of the Godot application selected
4. Disable the external editor settings in Godot and re-enable

## VSCode gives a warning that it can't connect to Godot

If when loading up VSCode it gives this error pop-up, then select it to view the version of Godot you have set in your `godot-tools` plugin and make sure it matches the Godot version you're using