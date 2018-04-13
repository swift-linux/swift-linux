Extra Credit: Swift on Other Distros
====================================

Both Swift's sources and official binaries tend to assume a Ubuntu-like system. This is
a guide on how to make Swift run well on other distros, too.

Using the Ubuntu Binaries
*************************

Shared Libraries
----------------

Swift's binaries depend on the following shared libraries that may not be easily
available:

- `ICU <http://site.icu-project.org/>`_ 55. Many distros come with newer versions. If
  that is the case for you, you'll have to either find a package providing ICU 55
  (AUR has `icu55 <https://aur.archlinux.org/packages/icu55/>`_, which is what I use
  in swift-bin), use precompiled binaries (`ICU provides some for RHEL6
  <http://site.icu-project.org/download/55#TOC-ICU4C-Download>`_), or build ICU 55
  yourself.

- libcurl.so.3. **The 3 here is important!** When libcurl underwent an API changes,
  many distros updated the version from libcurl 3 to libcurl 4, even though the ABI
  remained unchanged (`source <https://askubuntu.com/questions/469360>`_). As Swift was
  built on Ubuntu distros that still run libcurl 3, running it on other distros may
  result  is issues regarding libcurl.so.3 being missing. *In most cases*, you can just
  run ``sed -i 's/libcurl.so.3/libcurl.so.4'`` on all the binaries in the Swift
  distribution. (You may still encounter version information warnings, described below
  in `ref`:Version Information:.)

- A similar issue exists with Swift requiring libedit.so.2, which in most other distros
  is (properly) named libedit.so.0. Again, a ``sed`` call should do the trick.

- Swift requires ncurses5, but many distros now carry version 6. However, they also
  usually will carry a package provided ncurses5 binaries; for instance, Fedora carries
  ``ncurses-compat-libs``, and Arch has ``ncurses5-compat-libs``.

Version Information
-------------------

After making any necessary changes as detailed above, you may also encounter these
dreaded warnings::

  usr/bin/swift: /usr/lib/libtinfo.so.5: no version information available (required by usr/bin/swift)

The reason for these is that some Linux distros encode *version information* into their
shared libraries (Ubuntu and Fedora do this), whereas others do not (rolling release
distros such as Arch fall into this category). As the Swift binaries where built on
Ubuntu, they expect your shared libraries to have version information, and on a distro
where they don't, you'll get the above warnings.

Unfortunately, this is compounded by two facts:

- These warnings are emitted by your *dynamic linker*, which is the program that runs
  ELF binaries. They are toggled by a verbosity option which, oddly enough, is hardcoded
  into the glibc source code. There is no way to turn them off.
- Swift tends to assume that *any* output to standard error by helper tools means that
  the tool failed. This makes it virtually impossible to build Swift code, since the
  compiler will see that the version warnings were printed to standard error and abort
  the compilation process.

The only solution is to created a patched copy of the dynamic linker that does *not*
emit these warnings. I created `qldv <https://github.com/kirbyfan64/qldv>`_ to accomplish
this task. (I also described the process a bit
`here <https://refi64.com/posts/qldv.html>`_, though qldv now uses a different algorithm
that is compatible with more distros.)

Of course, there's still the task of making the Swift binaries use the patched linker.
Luckily, `patchelf <https://nixos.org/patchelf.html>`_ handles this job quite nicely.

The TL;DR of this whole thing is that **you'll need to use qldv, combined with patchelf,
in order to silence the version information warnings**. Here is an example::

  # Create a patched copy of the dynamic linker, and save it to /usr/lib/swift/ld.so.
  # (qldv -find is to locate the dynamic linker; it's usually somewhere like
  /lib/ld-linux-x86-64.so.2.)
  $ qldv `qldv -find` /usr/lib/swift/ld.so
  # Patch all the Swift binaries to use this new linker.
  # (Except for liblldb-intel-mpxtable.so, which doesn't need to be patched and
  # doesn't work with patchelf.)
  $ find usr/bin -type f -not -name liblldb-intel-mpxtable.so -exec patchelf --set-interpreter /usr/lib/swift/ld.so {} \;

However, there's still one more issue: any binaries you compile will still emit version
warnings. The solution to this is to trick Swift into running patchelf on any binaries
it builds. The easiest way I've found to do this is to replace clang++ (which Swift
uses to link binaries) with a shell script that calls the *real* clang++ and then
runs patchelf on the result.

I've toyed with various methods to do this (using
`bubblewrap <https://github.com/projectatomic/bubblewrap>`_) was an interesting one,
but the best option I've found is to do this:

- Install Swift to a prefix *other* than /usr. This is because Swift first tries to
  locate clang++ in the same directory as the other Swift binaries, and only *then* does
  it refer to your PATH. From here on out, I'll assume you picked /usr/lib/swift, and
  that you had saved the patched ld.so to the same directory.

- Symlink all the Swift binaries from /usr/lib/swift/bin to /usr/bin.

- Create a shell script /usr/lib/swift/bin/clang++ containing the following:

  .. code-block:: bash

    #!/bin/bash
    /usr/bin/clang++ "$@" && patchelf --set-interpreter /usr/lib/swift/ld.so "${@: -1}"

  This just calls the *real* clang++ and then calls patchelf on the result (the last
  argument passed by Swift is always the output file).

Now, Swift will call the fake, shell script clang++, which calls the real clang++ and
patches the output binaries.

Include Directories
-------------------

You may still exhibit some issues regarding missing headers. This is simply because
Swift looks for system headers in /usr/include/x86_64-linux-gnu, but on some distros,
they're all in /usr/include. To fix this, again refer to trusty sed::

  $ sed -i 's|x86_64-linux-gnu/||' usr/lib/swift/linux/x86_64/glibc.modulemap
  $ sed -i 's|x86_64-linux-gnu/||' usr/lib/swift_static/linux/static-stdlib-args.lnk

Building from Source
********************

Building from source is far easier, surprisingly enough, but it also makes upgrades take
far longer. However, you'll still probably have to patch the sources a bit.

One thing you may have to change are references of /usr/bin/python to /usr/bin/python2,
since many distros now set /usr/bin/python to point to Python 3, which isn't
backwards-compatible with Python 2. You can see this being done in the
`AUR swift package
<https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=swift-language#n45>`_.

Outside of that change, other ones are far more distro-specific. For a great starting
point, check out `the changes needed to build Swift on Fedora
<https://github.com/corinnekrych/swift-rpm/blob/master/swift.spec#L61>`_.
