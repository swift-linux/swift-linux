Navigating the Ecosystem
========================

Now that Swift is installed, there's still a major problem: the ecosystem. Many Swift
libraries are designed with macOS and/or iOS in mind, meaning they rely (either
intentionally or accidentally) on Cocoa APIs and Darwin C library bindings.

Pure Swift
**********

The best indicator that a library will work on Linux is if it mentions *pure Swift*
somewhere in the name. This term basically means that it likely won't depend on any
Cococa APIs, at minimum.

In addition, the presence of a ``Package.swift`` file, used by the Swift package
manager, means that it doesn't require something like CocoaPods to build.

Note that when I say *pure Swift*, I'm not referring to `this GitHub organization
<https://github.com/PureSwift>`_. Although they *create* pure Swift libraries, that's
about it.

Libraries
*********

Here are some awesome libraries that work on Linux:

Web
---

- `Vapor <https://vapor.codes/>`_: A popular backend web framework. The developers also
  maintain the PPA mentioned in :ref:`Installing Swift`.
- `Kitura <http://www.kitura.io/>`_: A web framework by IBM. Note that, although they
  have Linux support, the documentation is very macOS-centric. It tends to assume you
  *develop* on macOS and *deploy* to Linux.

CLI
---

- `Console <https://github.com/vapor/console>`_: A fantastic CLI library that's part
  of the Vapor project.
- `Progress.swift <https://github.com/jkandzi/Progress.swift>`_: Progress bars.

GUI
---

- `SwiftGtk <https://github.com/rhx/SwiftGtk>`_: GTK+ bindings. These seem to be rather
  complete and are auto-generated.
- `Qlift <https://github.com/Longhanks/qlift>`_: Qt bindings. Not sure how complete these
  are.
- `Cacao <https://github.com/PureSwift/Cacao>`_: A UIKit implementation that works on
  macOS *and* Linux. No commits since late 2017, and `there are open bugs related to
  building it <https://github.com/PureSwift/Cacao/issues/35>`_.

Parsing
-------

- `Kanna <https://github.com/tid-kijyun/Kanna>`_: HTML parsing.
- `SwiftSoup <https://github.com/scinfu/SwiftSoup>`_: HTML parsing.
- `Yams <https://github.com/jpsim/Yams>`_: YAML parsing.

Miscellaneous
-------------

- `Regex <https://github.com/crossroadlabs/Regex>`_: Regular expressions.

Submitting
----------

Feel free to `submit more libraries <https://github.com/swift-linux/swift-linux>`_!
