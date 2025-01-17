---
id: guide-installation
title: Installing SerialPort
---

## Installation Instructions

For most "standard" use cases (a supported Node.js on Mac, Linux, or Windows on a x64 processor), Node SerialPort will install nice and easy with:

```bash
npm install serialport
```

## Compilation Problems

We use [prebuild](https://github.com/prebuild/prebuild) to compile and post binaries of the library for most common use cases (Linux, Mac, Windows on standard processor platforms). If you have a special case, Node SerialPort will work, but it will compile the binary during the install. Compiling with nodejs is done via [node-gyp](https://github.com/nodejs/node-gyp) which requires Python 3.x, so please ensure you have it installed and in your path for all operating systems.

This assumes you have everything on your system necessary to compile ANY native module for Node.js. If you don't, then please ensure the following are true for your system before filing a "Does not install" issue.

## Installation Special Cases

### Alpine Linux

[Alpine](http://www.alpinelinux.org/) is a (very) small distro, but it uses the [musl](https://www.musl-libc.org/) standard library instead of [glibc](https://www.gnu.org/software/libc/) (used by most other Linux distros) so it requires compilation. It's commonly used with Docker. A user has confirmed that Node-Serialport works with [alpine-node](https://github.com/mhart/alpine-node).

```bash
# If you don't have node/npm already, add that first
sudo apk add --no-cache nodejs

# Add the necessary build and runtime dependencies
sudo apk add --no-cache make gcc g++ python3 linux-headers udev

# Then we can install serialport, forcing it to compile
npm install serialport --build-from-source

# If you're installing as root, you'll also need to use the --unsafe-perm flag
```

### Electron

[Electron](https://electron.atom.io/) is a framework for creating cross-platform desktop applications. It comes with its own version of the Node.js runtime.

Electron has a different [application binary interface (ABI)](https://en.wikipedia.org/wiki/Application_binary_interface) from Node.js, so it is necessary to make sure the correct ABI version is installed to match the electron version of your project, rather than the node.js version installed on your machine.  The easiest way to achieve this is to use electron-rebuild:

1. Run `npm install --save-dev electron-rebuild`
2. Add `electron-rebuild` to your project's package.json's install hook
3. Run `npm install`

If you have trouble on Windows, try: `.\node_modules\.bin\electron-rebuild.cmd`

Each release of SerialPort is published with prebuilt support for a large number of environments (and ABI combinations), you can see the supported environments in the [`assets for our latest release`](https://github.com/serialport/node-serialport/releases/latest).  If you are using an environment which doesn't have a prebuild available then you will need to recompile it.

To recompile `serialport` (or any native Node.js module) for Electron, you can use `electron-rebuild`.  You may need to install additional build tools in order to use electron-rebuild to recompile for your environment; more info can be found at [Electron's README](https://github.com/electron/electron-rebuild/blob/master/README.md).

Once electron-rebuild, and the required build tools are installed you can configure the recompile process:

1. Add `"buildDependenciesFromSource": true,"npmRebuild": false,` to your project.json's build configuration; more info at [Electron-builder](https://www.electron.build/configuration/configuration).

Additional troubleshooting info for electron can be found within the [electron documentation](https://www.electronjs.org/docs/tutorial/using-native-node-modules#troubleshooting).

#### Invoking SerialPort within the renderer processes
If you wish to invoke serialport within your renderer processes then you will need to override some of the Electron default settings.

1. Add 'app.allowRendererProcessReuse = false' within your main process - [Required since Electron V9](https://github.com/electron/electron/issues/18397)
2. Add 'contextIsolation: false' to your BrowserWindow webPreferences within your main process - [Required since Electron V12](https://www.electronjs.org/docs/tutorial/context-isolation)

For example:
```
app.allowRendererProcessReuse = false

import { app, BrowserWindow } from "electron"

async function createMainWindow() {
    const window = new BrowserWindow({
      webPreferences: {
        nodeIntegration: true,
        contextIsolation: false
    }
}
```

Over time we should migrate away from invocation within the renderer process, but many existing projects still rely on these workarounds.

For an example Electron project, check out [`electron-serialport`](https://github.com/serialport/electron-serialport).

### NW.js

[NW.js](https://nwjs.io/) is an app runtime based on Chromium and node.js.

Like Electron, NW.js also requires compilation against its own specific headers.

To instruct `prebuild` to build against the correct headers, place a file named `.prebuildrc` on your package root with the following content:

```
build_from_source=true
runtime=node-webkit
target=<target_version>
```

Where `<target_version>` is the NW.js version you are building against (for example, `0.26.6`).

### Illegal Instruction

The pre-compiled binaries assume a fully capable chip. Intel's [Galileo 2](https://software.intel.com/en-us/iot/hardware/galileo), for example, lacks a few instruction sets from the `ia32` architecture. A few other platforms have similar issues. If you get `Illegal Instruction` when trying to run Node-Serialport, you'll need to ask npm to rebuild the Serialport binary.

```bash
# Will ask npm to build serialport during install time
npm install serialport --build-from-source

# If you have a package that depends on serialport, you can ask npm to rebuild it specifically...
npm rebuild serialport --build-from-source
```

### Mac OS X

Ensure that you have at a minimum the xCode Command Line Tools installed appropriate for your system configuration. If you recently upgraded the OS, it probably removed your installation of Command Line Tools, please verify before submitting a ticket. To compile `node-serialport` with Node.js 4.x+, you will need to use g++ v4.8 or higher.

### Raspberry Pi Linux

Follow the instructions for [setting up a Raspberry pi for use with Johnny-Five and Raspi IO](https://github.com/nebrius/raspi-io/wiki/Getting-a-Raspberry-Pi-ready-for-NodeBots). These projects use Node Serialport under the hood.

| Revision       |      CPU              | Arm Version |
|   ----         |      ---              |     ---     |
| A, A+, B, B+   | 32-bit ARM1176JZF-S   |    ARMv6    |
| Compute Module | 32-bit ARM1176JZF-S   |    ARMv6    |
| Zero           | 32-bit ARM1176JZF-S   |    ARMv6    |
| B2             | 32-bit ARM Cortex-A7  |    ARMv7    |
| B3             | 32-bit ARM Cortex-A53 |    ARMv8    |

To enable the serial port on *Raspbian*, you launch `raspi-config`, then select `Interfacing Options`, then `Serial`.  You will then be asked two questions:

1. Would you like a login shell to be accessible over serial?
2. Would you like the serial port hardware to be enabled?

You must answer *No* to question 1 and *Yes* to question 2.  If the login shell is left active, you will experience hangs and or disconnects.

*DietPi* also has the ability to enable the serial port in `dietpi-config`; however, it doens't have a way to disable the login shell that we know of.

### sudo / root
If you're going to use `sudo` or root to install Node-Serialport, `npm` will require you to use the unsafe parameters flag.

```bash
sudo npm install serialport --unsafe-perm --build-from-source
```

Failure to use the flag results in an error like this:

```bash
root@rpi3:~# npm install -g serialport
/usr/bin/serialport-list -> /usr/lib/node_modules/serialport/bin/serialport-list.js
/usr/bin/serialport-term -> /usr/lib/node_modules/serialport/bin/serialport-terminal.js


> serialport@6.0.0-beta1 install /Users/wizard/src/node-serialport
> prebuild-install || node-gyp rebuild

prebuild-install info begin Prebuild-install version 2.2.1
prebuild-install info install installing standalone, skipping download.

gyp WARN EACCES user "root" does not have permission to access the dev dir "/root/.node-gyp/6.9.1"
gyp WARN EACCES attempting to reinstall using temporary dev dir "/usr/lib/node_modules/serialport/.node-gyp"
make: Entering directory '/usr/lib/node_modules/serialport/build'
make: *** No rule to make target '../.node-gyp/6.9.1/include/node/common.gypi', needed by 'Makefile'.  Stop.
make: Leaving directory '/usr/lib/node_modules/serialport/build'
gyp ERR! build error
gyp ERR! stack Error: `make` failed with exit code: 2

```

### Ubuntu/Debian Linux

The best way to install any version of Node.js is to use the [NodeSource Node.js binary distributions](https://github.com/nodesource/distributions#installation-instructions). Older versions of Ubuntu install Node.js with the wrong version and binary name. If your Node binary is `nodejs` instead of `node`, or if your Node version is [`v0.10.29`](https://github.com/fivdi/onoff/wiki/Node.js-v0.10.29-and-native-addons-on-the-Raspberry-Pi), then you should follow these instructions.

You'll need the package `build-essential` to compile `serialport`. If there's a binary for your platform, you won't need it. Keep rocking!

```
# Using Ubuntu and Node 6
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian and Node 6 as root
curl -sL https://deb.nodesource.com/setup_7.x | bash -
apt-get install -y nodejs
```

### Windows
Node-Serialport supports Windows 7, 8.1, 10, and 10 IoT. Precompiled binaries are available, but if you want to build it from source you'll need to follow the [node-gyp installation](https://github.com/nodejs/node-gyp#installation) instructions. Once you've got things working, you can install Node-Serialport from source with:

```powershell
npm install serialport --build-from-source
```

Node-gyp's documentation doesn't mention it, but it sometimes helps to create a C++ project in [Visual Studio](https://www.visualstudio.com/) so that it will install any necessary components not already installed during the past two hours of setup. This will solve some instances of `Failed to locate: "CL.exe"`.

An old issue that you may still run into: when working with multiple Serial Ports you can set the `UV_THREADPOOL_SIZE` environment variable to be set to 1 + the number of ports you wish to open at a time. Defaults to `4` which supports 3 open ports.
