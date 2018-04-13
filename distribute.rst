Distributing Swift Applications
===============================

You've created your Swift application. Now, how do you get it to your users?

The easiest way is by using the Swift :ref:`Flatpak`. If you haven't installed it yet,
now's the time to do so.

Creating a Flatpak
******************

First off, make sure you already know the basics of creating Flatpaks; see the
`official developer guide <http://docs.flatpak.org/en/latest/first-build.html>`_ for
more information.

Let's use a simple *Hello, world!* project as our example. Create a ``Package.swift``:

.. code-block:: swift

  // swift-tools-version:4.0

  import PackageDescription

  let package = Package(
      name: "example",
      dependencies: [
      ],
      targets: [
          .target(
              name: "example",
              dependencies: []),
      ]
  )

and ``Sources/example/main.swift``:

.. code-block:: swift

  print("Hello, world!")

Now, let's assume the full application ID will be *org.mysite.Hello* (Flatpak uses
`reverse domain name notation
<https://en.wikipedia.org/wiki/Reverse_domain_name_notation>`_ for application IDs).
Create ``org.mysite.Hello.json`` containing the following:

.. code-block:: json

  {
    "app-id": "org.mysite.Hello",
    "runtime": "org.freedesktop.Platform",
    "runtime-version": "1.6",
    "sdk": "org.freedesktop.Sdk",
    "sdk-extensions": [
      "org.freedesktop.Sdk.Extension.swift4"
    ],
    "command": "/app/bin/example",
    "modules": [
      {
        "name": "sdk",
        "buildsystem": "simple",
        "sources": [
          {
            "type": "git",
            "path": "https://github.com/myuser/myrepo.git",
            "tag": "HEAD"
          },
          {
            "type": "script",
            "commands": [
              ". /usr/lib/sdk/swift4/enable.sh",
              "swift build -c release",
              "install -Dm 755 .build/release/example /app/bin/example",
              "/usr/lib/sdk/swift4/set-runtime.sh /app/lib /app/bin example"
            ],
            "dest-filename": "build-example.sh"
          }
        ],
        "build-commands": [
          "./build-example.sh"
        ]
      }
    ]
  }

Here's a breakdown of the interesting parts of this file:

- ``sdk-extensions``: **This is where the Swift SDK extension is used.** The SDK extension
  "extends" the previously chosen SDK with the Swift compiler and libraries.
- ``sources``: This is where the sources are chosen. The first is just the Git
  repository of our application, but the second is far more interesting and will be
  explained below.
- ``build-commands`` calls into ``./build-example.sh`` to build our code.

``build-example.sh`` does the following:

- Sources ``enable.sh`` to enable use of the Swift SDK extension.
- Builds the application.
- Installs it to ``/app/bin/example`` via the ``install`` command. Note that Flatpak
  requires your application to be installed to the ``/app`` prefix.
- Calls a script called ``set-runtime.sh``. This script will copy the Swift runtime
  libraries to the application directory, and it will set the dynamic linker of
  your application binaries to the library directory. The arguments are as follows:

  - The first is the directory to store the Swift runtime libraries and patched
    dynamic linker (see :ref:`Version Warnings` for information on why that is
    necessary).
  - The second is the directory where your application binaries are stored.
  - Any other arguments passed are assumed to be paths to binaries, relative to the
    second argument (the application directory). These binaries will all have their
    dynamic linker set to the patched one that doesn't emit version warnings.
