Installing Swift
================

Unfortunately, the official Swift binaries only work on Ubuntu, and they're standalone
binaries independent of any package manager. Here are steps on getting it to work on
various distros/package tools.

Flatpak
*******

The easiest way to get started using Swift is via the
`Flatpak repo <https://flatpak.org/>`_. Make sure you've followed the
`official Flatpak setup instructions <https://flatpak.org/setup/>`_.

First, make sure you have the Freedesktop SDK installed::

  $ flatpak --user install flathub org.freedesktop.Sdk

Add the Flatpak remote::

  $ flatpak --user remote-add swift https://swift-flatpak.refi64.cmo/swift.flatpakrepo

Then, install the SDK extension and live SDK::

  $ flatpak --user install swift org.freedesktop.Sdk.Extension.swift4 org.freedesktop.Sdk.Extension.swift4.live

In order to run the Swift compiler via the Flatpak, you'll need to use this command::

  $ flatpak run -d org.freedesktop.Sdk.Extension.swift4.live swift ...

To shorten this, define an alias::

  $ alias swiftpak='flatpak run -d org.freedesktop.Sdk.Extension.swift4.live swift'

Now, you can just use *swiftpak*::

  $ swiftpak run myapp

Ubuntu
******

You can use the `official Swift binaries <https://swift.org/download/>`_. If you want
to be able to upgrade Swift via apt, then `try out the Vapor
PPA <https://docs.vapor.codes/3.0/install/ubuntu/>`_::

  $ eval "$(curl -sL https://apt.vapor.sh)"

or::

  curl -L https://repo.vapor.codes/apt/keyring.gpg | sudo apt-key add -
  echo "deb https://repo.vapor.codes/apt $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/vapor.list
  sudo apt update
  sudo apt install swift

Fedora
******

An RPM is available `here <https://github.com/corinnekrych/swift-rpm>`, though it
requires a build from source. The Ubuntu binaries should more-or-less work if you patch
them as mentioned in :ref:`Using the Ubuntu Binaries`.

Arch Linux
**********

You can install Swift on Arch Linux using either the AUR
`swift <https://aur.archlinux.org/packages/swift/>`_ package or the
`swift-bin <https://aur.archlinux.org/packages/swift-bin/>`_ package. (The former builds
from source, whereas the latter uses patched versions of the Ubuntu binaries.)

Others
******

For other distros, see :ref:`Extra Credit: Swift on Other Distros`.

If you've gotten Swift working on your favorite distros, feel free to
`create an issue to mention yours <https://github.com/swift-linux/swift-linux>`_!
