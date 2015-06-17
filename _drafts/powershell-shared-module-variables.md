---
layout: post
title: 'Powershell Modules - Shared Variables'
tags:
- Powershell
- Powershell Modules
- Modules
- Powershell Variables
- Powershell Variable Scope
---


# Preface
I'm working on a large powershell module that does a lot of really neat things. I'm relatively new to Powershell so I have to go back to the basics when I think of things like shared resources and variable scoping. This post is an exploration on how to use module-wide variables to store stuff needed across several subsections of the module.

## Setup
Create a new folder in your powershell module path called "Testing". Inside this new folder, we're going to create a "Testing.psm1" file.

To get your powershell module directory:

```powershell
function Get-UserModulePath {

 $Path = $env:PSModulePath -split ";" -match $env:USERNAME

 if (-not (Test-Path -Path $Path))
 {
     New-Item -Path $Path -ItemType Container | Out-Null
 }
     $Path
}

Invoke-Item (Get-UserModulePath)

```

For me, my module directory is: C:\Users\nhudacin\My Documents\WindowsPowerShell\Modules\Testing

## Example 1
Declaring a shared variable before any functions are declared and without using the $script scope tag.

```powershell
# shared variable exploration

$regularVariable_DefinedInModule = 'Regular Variable Defined In Module'

function topLevelFunction{
    Write-Host "The value of the variable: $regularVariable_DefinedInModule"
}

Export-ModuleMember topLevelFunction
```

Save the above as "Testing.psm1" in the "Testing" module folder you created (see the setup section)

Import the module:

```powershell
PS Z:\> Import-Module -Name Testing -Force
```

Run the script:

```powershell
PS Z:\> topLevelFunction
The value of the variable: Regular Variable Defined In Module
```
