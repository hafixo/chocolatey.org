Title: Create Your Own Chocolatey Packages
Published: 20160113
Author: Rob Reynolds
Tags: HowTo, Packaging
Keywords: howto, chocolatey, packages
Summary: Creating Chocolatey packages can save you a lot of time. And Chocolatey has ways to make that process much smoother.
---
**Note:** Originally posted on [Puppet's blog](https://puppet.com/blog/chocolatey-creating-your-own-chocolatey-packages). *Reposted with permission.*

Chocolatey is a package manager. It works by managing packages that contain software or know how to manage the underlying software themselves. For folks unfamiliar with packaging concepts, a Chocolatey package is really just a fancy version of a zip file that knows about metadata, versioning and dependencies related to underlying software, plus optional automation scripts that are run during installation, upgrade and uninstallation of the package.

The packaging framework Chocolatey uses is known as [NuGet](https://www.nuget.org/) (and also part of where Chocolatey gets its[name](https://chocolatey.org/docs/history)). Chocolatey extends NuGet to add more metadata to packaging (the nuspec file) and known automation scripts for execution during installation, upgrade, and uninstallation. These automation scripts are simply PowerShell scripts so it's easy to start getting familiar with the additional functions that Chocolatey provides for managing software!

This is useful because it helps define dependencies related to other software and encapsulates everything about installing a particular piece of software in a single place.

Creating your own Chocolatey packages is surprisingly simple for system administrators who have experience with other packaging frameworks. Chocolatey has extensive [documentation](https://github.com/chocolatey/choco/wiki/CreatePackages) for creating packages, however some of the information applies only to the community repository ([https://chocolatey.org](https://chocolatey.org/)). When you [host your own internal packages](https://puppetlabs.com/blog/chocolatey-hosting-your-own-server), different rules apply.

The most important thing to note about creating packages is that Chocolatey already has a built-in package template that does most of the work for you - choco new. If you use the Chocolatey CLI (choco.exe aka choco), make sure you are on the latest version of choco available to take advantage of fixes in the templates. You can also create your [own templates](https://github.com/chocolatey/choco/wiki/How-To-Create-Custom-Package-Templates)!

### Creating a Package for Notepad++

We can start by creating a package for Notepad++. Typing `choco new notepadplusplus` on the command line would yield output similar to the following:

~~~
Creating a new package specification at C:\temppackages\notepadplusplus
Generating template to a file
at 'C:\temppackages\notepadplusplus\notepadplusplus.nuspec'
Generating template to a file
at 'C:\temppackages\notepadplusplus\tools\chocolateyinstall.ps1'
Generating template to a file
at 'C:\temppackages\notepadplusplus\tools\chocolateyuninstall.ps1'
Generating template to a file
at 'C:\temppackages\notepadplusplus\tools\ReadMe.md'
Successfully generated notepadplusplus package specification files
at 'C:\temppackages\notepadplusplus
~~~

Notice that it created the package files plus a ReadMe with more information on what is available. Open notepadplusplus.nuspec, and edit it to look like this:

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">
  <metadata>
    <id>notepadplusplus</id>
    <title>Notepad++ (Install)</title>
    <version>6.8.8</version>
    <authors>Don Ho</authors>
    <owners>my company</owners> <!-- also known as package mantainers -->
    <description>Notepad++ is a free (as in "free speech" and also as in "free beer") source code editor and Notepad replacement that supports several languages.</description>
 </metadata>
<files>
  <file src="tools\**" target="tools" />
</files>
</package>
~~~

Unless you are sharing with the world, you don't need most of what is in the nuspec template file, so only required items are included above. When creating a package, be sure to match the version of the package in the nuspec file to the version of the underlying software as closely as possible. We are packaging Notepad++ 6.8.8 in this case, so the version of the package in the nuspec file should also be 6.8.8. [More on versioning](https://github.com/chocolatey/choco/wiki/CreatePackages#versioning-recommendations).

Open chocolateyInstall.ps1 and edit it to look like the following:

~~~powershell
$ErrorActionPreference = 'Stop';

$packageName= 'notepadplusplus'
$toolsDir   = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)"
$fileLocation = Join-Path $toolsDir 'npp.6.8.8.Installer.exe'

$packageArgs = @{
  packageName   = $packageName
  fileType      = 'exe'
  file         = $fileLocation

  silentArgs    = "/S"
  validExitCodes= @(0)
}

Install-ChocolateyInstallPackage @packageArgs
~~~

Note: The above is [Install-ChocolateyINSTALLPackage](https://github.com/chocolatey/choco/wiki/HelpersInstallChocolateyInstallPackage), not to be confused with [Install-ChocolateyPackage](https://github.com/chocolatey/choco/wiki/HelpersInstallChocolateyPackage). The names are very close to each other, however the latter will also download/checksum software from a URI (URL, ftp, file) which is not necessary for this example.

Delete the ReadMe.md and chocolateyUninstall.ps1 files. We delete the uninstall as it isn ot necessary due to the autoUninstaller feature of Chocolatey. Download the latest version of [Notepad++](https://notepad-plus-plus.org/download/) and move it to the tools folder of the package (update the version in both the nuspec and the chocolateyInstall.ps1 if necessary). Note: Normally if the underlying software makes the size of a package over 100MB, it is recommended you move the software installer/archive to a share drive and point to it instead. Because this is less than 5MB, we will just embed the Notepad++ installer right into the package.

Now pack it up by using choco pack from the command line. Ensure that notepadplusplus.nuspec is in the current directory of the shell. This should create a notepadplusplus.6.8.8.nupkg file.

Now that you’ve created your Notepad++ package, you can go ahead and install it across your systems.