# MiniDisc on Mac 💽✨

The year is 2021, and — somewhat incredibly! — if you'd like to use your modern Macintosh to transfer music to a MiniDisc over USB, you're spoilt for choice!

This page is a quick guide to the different options that are out there, to help you get up and running with your MiniDisc-on-Mac method of choice.

## The options

Back in the day, transferring music to MiniDisc was only possible with proprietary Sony software, and that software was generally — with [a few exceptions!](http://minidisc.org/part_Sony_MZ-M100+RH10.html) — only available for Windows. Thanks to the efforts of the contributors to the [linux-minidisc](https://wiki.physik.fu-berlin.de/linux-minidisc/doku.php) project, however, we now have an open-source way to connect to NetMD devices over USB and transfer music, edit disc and track titles, and more.

Though they vary in interface and implementation, the `linux-minidisc` project is the basis for the three main options for sending music to NetMD players from a modern Mac: **Web MiniDisc**, **Platinum-MD**, and **netmdcli**.

* ☁️ **[Web MiniDisc](https://stefano.brilli.me/webminidisc/)**

This incredible piece of engineering is by far the easiest option to use. It just works™! As long as you have Google Chrome installed, you can use this webapp to transfer music straight to your NetMD device over USB, in SP (standard full quality), LP2, and LP4 modes, and easily edit track and disc titles, all without installing any software on your Mac!

* 💎 **[Platinum-MD](https://github.com/gavinbenda/platinum-md)**

Platinum-MD is a local application rather than a webapp, but otherwise offers a similar experience and feature-set to Web MiniDisc. It takes a bit more work to set up, requiring manual installation of some necessary components through the terminal, but once that's done it provides a nicely intuitive graphical interface for easily moving music to your NetMD player.

* \>_   **[netmdcli](https://github.com/morgansleeper/MiniDisc-on-Mac/blob/main/README.md#installing-netmdcli-on-a-mac)**

A key component of the `linux-minidisc` package, `netmdcli` is the command line tool that powers the core features of both Web MiniDisc and Platinum-MD. While it's more complicated to set up than either of the GUI options, it's also speedy, lightweight, and fun to use — and besides, it's hard to imagine a more aesthetically-appropriate way to send your music to MiniDisc than from the soft glow of a semi-transparent terminal!

Web MiniDisc is plug-and-play, and Platinum-MD offers installation instructions on its download page, but if you're interested in using `netmdcli`, there's comparatively little in the way of Mac-specific help out there. The rest of this guide will show you how to install and use `netmdcli` on a Mac!

## Installing `netmdcli` on a Mac

To install and use `netmdcli` on a Mac, you'll need to do the following:

* Download the `linux-minidisc` source code, of which `netmdcli` is one part
* Install command line utilities, the Homebrew package manager, and various libraries that `linux-minidisc` relies on
* Build the software from the source code.

We'll walk through these step-by-step!

**1. Download the source code**

There are a lot of different forks (versions) of the `linux-minidisc` project! Currently, the one I find most stable/up-to-date/friendly for NetMD transfers is the [vuori 'NetMD-fixes' fork](https://github.com/vuori/linux-minidisc) at https://github.com/vuori/linux-minidisc.

Head to that page, click the green 'Code' button near the top right, and select 'Download ZIP' to save the source code to your computer. It should unzip as a folder named 'linux-minidisc-master'; we'll come back to that in a bit to build the software.

**2. Install Command Line Tools for Xcode:**

This is a suite of utilities from Apple that includes a compiler, which you'll need to build `linux-minidisc` from the source code.

Open the Terminal app, and then type and run:

```zsh
xcode-select --install
```

**3. Install [Homebrew](https://brew.sh/)**

This is a package manager for Mac OS X that lets you easily install all kinds of useful software & libraries.

In the terminal, run:

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

If you're using an Apple Silicon/M1 Mac, there's one more step to get Homebrew fully set up; at the end of its installation process, it will give you a few commands to copy and paste into the terminal so that you can easily use the `brew` command from any terminal window, that will look something like this, with your username in place of the '_______':

```zsh
==> Next steps:
- Add Homebrew to your PATH in /Users/_______/.zprofile:
    echo 'eval $(/opt/homebrew/bin/brew shellenv)' >> /Users/_______/.zprofile
    eval $(/opt/homebrew/bin/brew shellenv)
```

Run those lines of code, and you should be set!

**4. Install the dependencies for `linux-minidisc`**

`linux-minidisc` relies on a suite of libraries that can be installed using Homebrew.

  * `qt` (for building the software)
  * `pkg-config` (for building the software)
  * `glib` (for core functions)
  * `libgcrypt` (for transferring PCM/WAV audio)
  * `libusb` and `libusb-compat` (for USB support)
  * `mad` and `libid3tag` (for MP3 support)

**4.1 Installing `qt`**

To use the `qmake` command we need from the `qt` package, you'll need to both install `qt` and then 'link' `qmake` so that you can invoke it from the terminal.

First, install `qt` by typing and running the following in the terminal:

```zsh
brew install qt
```

In the summary output that Homebrew provides at the end of the installation, take note of which version of `qt` was installed, since you'll need that to make the link; in the following output, for instance, the version is `5.15.2`:

```zsh
==> Summary
🍺  /usr/local/Cellar/qt/5.15.2: 10,688 files, 365.0MB
```

(If for any reason you miss this, you can always re-run `brew install qt`, and Homebrew will just tell you which version you already have installed.)

Now, you can create a link for `qmake` so that you can run it directly from the terminal. Type in and run the following, replacing `5.15.2` in the path with the exact version number of `qt` you have installed:

— For Apple Silicon/M1 Macs:

```zsh
ln -s /opt/homebrew/Cellar/qt/5.15.2/bin/qmake /opt/homebrew/bin/qmake
```

— For Intel Macs:

```zsh
ln -s /usr/local/Cellar/qt/5.15.2/bin/qmake /usr/local/bin/qmake
```

**4.2 Installing the other dependencies**

To install the rest of the dependencies all at once, in the terminal, type and run:

```zsh
brew install pkg-config glib libgcrypt libusb libusb-compat mad libid3tag
```

**5. Building `linux-minidisc` and `netmdcli`**

Now that you have the dependencies installed, you can build the software itself from the source code.

In the terminal, navigate to the root folder of the source code you downloaded, 'linux-mindisc-master'. (You can do this by typing in `cd` and then the path to the folder, or by typing `cd` and dragging the folder onto the terminal, then running the command). To check that you're in the right place, if you then type in `ls` and hit return, you should see the contents of the 'linux-minidisc-master' folder:

```zsh
COPYING		VERSION		himdcli		netmd
COPYING.LIB	basictools	libhimd		netmdcli
README		build		libnetmd	qhimdtransfer
README.md	docs		md.pro		testdata
```

Next, type and run:

```zsh
qmake
```
And then:

```zsh
make
```

This should build the software, and if it was successful, you should see a command-line program named `netmdcli` in the subfolder of the same name!

---

## Using `netmdcli`

*Coming soon!*
