Build from source
==================

.. image:: https://github.com/kovidgoyal/kitty/workflows/CI/badge.svg
  :alt: Build status
  :target: https://github.com/kovidgoyal/kitty/actions?query=workflow%3ACI

.. highlight:: sh

|kitty| is designed to run from source, for easy hack-ability. Make sure
the following dependencies are installed first.


.. note::
   If you are making small changes only to the python parts of kitty, there is no need to
   build kitty at all, instead, assuming you have installed the official kitty
   binaries, you can simply set the KITTY_DEVELOP_FROM enviroment variable to
   point to the directory into which you have checked out the kitty source
   code. kitty will then load its python code from there. You should use a
   version of the source that matches the binary version as closely as
   possible, since the two are tightly coupled.


Dependencies
----------------

Run-time dependencies:

* ``python`` >= 3.6
* ``harfbuzz`` >= 2.2.0
* ``zlib``
* ``libpng``
* ``liblcms2``
* ``freetype`` (not needed on macOS)
* ``fontconfig`` (not needed on macOS)
* ``libcanberra`` (not needed on macOS)
* ``ImageMagick`` (optional, needed to use the ``kitty +kitten icat`` tool to display images in the terminal)
* ``pygments`` (optional, needed for syntax highlighting in ``kitty +kitten diff``)


Build-time dependencies:

* ``gcc`` or ``clang``
* ``pkg-config``
* For building on Linux in addition to the above dependencies you might also need to install the ``-dev`` packages for:
  ``libdbus-1-dev``, ``libxcursor-dev``, ``libxrandr-dev``, ``libxi-dev``, ``libxinerama-dev``,
  ``libgl1-mesa-dev``, ``libxkbcommon-x11-dev``, ``libfontconfig-dev``, ``libx11-xcb-dev``,
  ``liblcms2-dev``, and ``libpython3-dev``, if they are not already installed by your distro.


Install and run from source
------------------------------

.. code-block:: sh

    git clone https://github.com/kovidgoyal/kitty && cd kitty

Now build the native code parts of |kitty| with the following command::

    make

You can run |kitty|, as::

    ./kitty/launcher/kitty

If that works, you can create a symlink to the launcher in :file:`~/bin` or
some other directory on your PATH so that you can run |kitty| using
just ``kitty``.


Building kitty.app on macOS from source
-------------------------------------------

Run::

    make app

This :file:`kitty.app` unlike the released one does not include its own copy of
python and the other dependencies. So if you ever un-install/upgrade those dependencies
you might have to rebuild the app.

.. note::
   The released :file:`kitty.dmg` includes all dependencies, unlike the
   :file:`kitty.app` built above and is built automatically by using the
   `bypy framework <https://github.com/kovidgoyal/bypy>`_ however, that is
   designed to run on Linux and is not for the faint of heart.

.. note::
   Apple disallows certain functionality, such as notifications for unsigned applications.
   If you need this functionality, you can try signing the built kitty.app with
   a self signed certificate, see for example, `here
   <https://stackoverflow.com/questions/27474751/how-can-i-codesign-an-app-without-being-in-the-mac-developer-program/27474942>`_.

.. note::
   If you are facing issues with ``linker`` while building,
   try with a ``brew`` installed python instead, see :iss:`289`
   for more discussion.


Build and run from source with Nix
-------------------------------------------

On NixOS or any other Linux or macOS system with the Nix package manager
installed, execute `nix-shell
<https://nixos.org/guides/nix-pills/developing-with-nix-shell.html>`_ to create
the correct environment to build kitty or use ``nix-shell --pure`` instead to
eliminate most of the influence of the outside system, e.g. globally installed
packages. ``nix-shell`` will automatically fetch all required dependencies and
make them available in the newly spawned shell.

Then proceed with ``make`` or ``make app`` according to the platform specific instructions above.


Notes for Linux/macOS packagers
----------------------------------

The released |kitty| source code is available as a `tarball`_ from
`the GitHub releases page <https://github.com/kovidgoyal/kitty/releases>`_.

While |kitty| does use python, it is not a traditional python package, so please
do not install it in site-packages.
Instead run::

    python3 setup.py linux-package

This will install |kitty| into the directory :file:`linux-package`. You can run |kitty|
with :file:`linux-package/bin/kitty`.  All the files needed to run kitty will be in
:file:`linux-package/lib/kitty`. The terminfo file will be installed into
:file:`linux-package/share/terminfo`. Simply copy these files into :file:`/usr` to install
|kitty|. In other words, :file:`linux-package` is the staging area into which |kitty| is
installed. You can choose a different staging area, by passing the ``--prefix``
argument to :file:`setup.py`.

You should probably split |kitty| into two packages, :file:`kitty-terminfo` that
installs the terminfo file and :file:`kitty` that installs the main program.
This allows users to install the terminfo file on servers into which they ssh,
without needing to install all of |kitty|.

.. note::
   You need a couple of extra dependencies to build linux-package.
   :file:`tic` to compile terminfo files, usually found in the
   development package of :file:`ncurses`. Also, if you are building from
   a git checkout instead of the released source code tarball, you will
   need to install the dependencies from ``docs/requirements.txt`` to
   build the kitty documentation. They can be installed most easily with
   ``python -m pip -r docs/requirements.txt``.

This applies to creating packages for |kitty| for macOS package managers such as
brew or MacPorts as well.


Changing defaults for packages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

|kitty| has its defaults chosen to be suitable for a standalone distributable.
If you are packaging it a few of these might need to be changed.

update-checking
   |kitty| has its own update check mechanism, if you would like to turn
   it off for your package, use::

       ./setup.py linux-package --update-check-interval=0

shell-integration
   |kitty| by default installs its :ref:`shell_integration` files into the user's
   rc files. For a package, it might make more sense to distribute the shell
   integration scripts into the system-wide shell vendor locations. The
   shell integration files are found in the :file:`shell-integration`
   directory. Copy them to the system wide shell vendor locations for each
   shell, and use::

       ./setup.py linux-package --shell-integration=enabled\ no-rc

   This will prevent kitty from modifying the user's shell rc files to load
   the integration scripts.
