# Electron

!!! tip
    Learn how we packaged [GitHub Desktop with Conveyor](https://hydraulic.software/blog/8-packaging-electron-apps.html), a production-grade Electron app. Over 1,500 lines of code can be deleted, the resulting config is easy to read, and the app can be packaged and released from your laptop. We show how to combine it with GitHub Releases, Sites and Actions.

!!! tip
    The [tutorial](../tutorial/new.md) shows you how to generate and package a simple Electron app.

## Overview

No changes to your JS are necessary to get working software updates with Conveyor. [Curious why you'd use Conveyor over other tools?](../comparisons/electron-comparisons.md)

## Synopsis

A basic `conveyor.conf` looks like this:

```
// You must import the electron stdlib config.
include required("/stdlib/electron/electron.conf")

// You may also import your package.json file. Defaults will be set from it when possible.
package-json {
    include required("package.json") 
}

// Override the Electron version. This is otherwise taken from your package.json file. 
app.electron.version = 19.0.1

// Change where it's fetched from. The default is GitHub.
app.electron.download-base-url = github.com/electron/electron/releases/download/
```

## Keys

**`app.electron.version`** If set then this config is for an Electron app, and this is the version of Electron to bundle. 

**`app.electron.download-base-url`** Where to find Electron builds to download. The URLs are composed like this: 

```
${app.electron.download-base-url}/v${app.electron.version}/electron-v${app.electron.version}-$os-$cpu.zip
```

so they must follow the same layout as that used on GitHub. You can specify a `file:` URL here if necessary.

## App resources

The stdlib `electron.conf` file will import the following files to your `resources/app` directory:

- `*.json`
- `*.js`
- `*.css`
- `*.html`
- The `node_modules` directory.

You can add more [inputs](inputs.md) or replace the default set like this:

```
app {
    // Add more top level files.
    inputs += "*.ts"
    
    // Add a  directory preserving the placement. 
    // (otherwise, the contents are copied to the top level).
    inputs += node_modules -> node_modules
    
    // Replace the inputs entirely.
    inputs = [ "app.asar" ]
}
```

## Adapting a project that used `npx create-electron-app`

The easiest way to get started with Conveyor and Electron is to create a fresh project using `conveyor create electron com.example.my-app`, replacing the reverse DNS name with one that is unique to your project (e.g. `io.github.username.projectname`). [Learn more in the Electron tutorial](../tutorial/hare/electron.md).

If you have a project that was already created using `npx create-electron-app` you'll need to adapt it for Conveyor. We'll be removing references to Electron Forge and Squirrel.

1. Run `npx create-electron-app`.
2. `rm forge.vconfig.js`, this file isn't needed anymore.
3. In `package.json`:
   - Remove references to electron-forge and Squirrel. Conveyor doesn't use Squirrel for updates.
   - Replace script "start" with "electron ."
4. In `src/index.js`, remove the reference to Squirrel.
5. `rm -rf node_modules && npm install`. This will get rid of the packages you don't need any more.

Now add a new `conveyor.conf` file and adapt it for your needs:

```hocon
include required("/stdlib/electron/electron.conf")

// Import metadata from your package.json file, like your fsname, version and which version of Electron to use.
package-json {
  include required("package-lock.json")
}

app {
  display-name = "My project"
  rdns-name = org.example.my-project
  site.base-url = "localhost:3000"

  inputs = [
    package.json
    src -> src    

    {
      from = node_modules
      to = node_modules      
      remap = [ "-electron/dist/**" ]
    }
  ]
}
```

## Caveats

Be aware of the following caveats:

* There's no API to control or monitor updates yet. Note that such an API doesn't necessarily make sense on some platforms e.g. Linux where the user's package manager will apply updates, or on Windows where the app can be updated silently in the background when it's not running.
* Conveyor doesn't make ASAR files at the moment, so all files will be shipped unpacked. If your app consists of very large numbers of small files this may reduce performance. You could use a bundler, or [make an asar file yourself](https://github.com/electron/asar) and supply it as an input to work around this limitation.
* You should remove any code that invokes Squirrel.

## stdlib config

The standard Electron config you're asked to import looks like this:

```hocon
app {
  fsname = ${?package-json.name}
  display-name = ${?package-json.productName}
  version = ${?package-json.version}
  electron.version = ${?package-json.packages.node_modules/electron.version}
  contact-email = ${?package-json.author.email}

  // Electron doesn't place the binary in a bin directory, so we have to fix that up here.
  linux {
    desktop-file."Desktop Entry".Exec = ${app.linux.install-path}/${app.fsname}
    symlinks += ${app.linux.prefix}/bin/${app.fsname} -> ${app.linux.install-path}/${app.fsname}
  }

  mac {
    info-plist {
      NSSupportsAutomaticGraphicsSwitching = true
      NSRequiresAquaSystemAppearance = false
      NSHighResolutionCapable = true
      NSQuitAlwaysKeepsWindows = false

      LSEnvironment {
        MallocNanoZone = "0"
      }
      NSAppTransportSecurity {
        NSAllowsArbitraryLoads = true
      }
      NSBluetoothAlwaysUsageDescription = This app needs access to Bluetooth
      NSBluetoothPeripheralUsageDescription = This app needs access to Bluetooth
      NSCameraUsageDescription = This app needs access to the camera
      NSMainNibFile = MainMenu
      NSPrincipalClass = AtomApplication
    }
  }

  // A simple default input.
  inputs = ${app.inputs} [
    "*.{json,js,ts,css,html}"

    {
      from = node_modules
      to = node_modules
      remap = [ "-electron/dist/**" ]
    }
  ]
}
```
