Introduction to Flatpak
=======================

Flatpak is a technology for building, distributing, installing and running applications. It is primarily targeted at the Linux desktop, although it can also be used as the basis for application distribution in other contexts, such as embedded systems.

Flatpak has been designed and implemented with a number of goals:

* Allow applications to be installed on any Linux distribution.
* Provide consistent environments for applications, to facilitate testing and reduce bugs.
* Decouple applications from the operating system, so that applications don't depend on specific versions of each distribution.
* Allow applications to bundle their own dependencies, so that they can use libraries that aren't provided by a Linux distribution, and so they can depend on specific versions or even patched versions of a library.
* Increase the security of Linux desktops, by isolating applications in sandboxes.

More information about Flatpak can be found on `flatpak.org <http://flatpak.org/>`_.

How it works
------------

Flatpak can be understood through a small number of key concepts. These also help to explain how it differs from traditional software packages.

.. image:: ../images/diagram.svg

Runtimes
^^^^^^^^

Runtimes provide the basic dependencies that are used by applications. Various runtimes are available, from more minimal (but more stable) Freedesktop runtimes, to larger runtimes produced by desktops like GNOME or KDE. (See :doc:`available-runtimes` for an overview of the runtimes that are currently available.)

Each application must be built against a runtime, and this runtime must be installed on a host system in order for the application to run. Users can install multiple different runtimes at the same time, including different versions of the same runtime.

.. tip::
  Each runtime can be thought of as a ``/usr`` filesystem. Indeed, when an application is run, its runtime is mounted at ``/usr``.

Bundled libraries
^^^^^^^^^^^^^^^^^

If an application requires any dependencies that aren't in its runtime, they can be bundled along with the application itself. This allows applications to use dependencies that aren't available in a distribution, or to use a different version of a dependency from the one that's installed on the host.

Sandboxes
^^^^^^^^^

With Flatpak, each app is built and run in an isolated environment. By default, the application can only 'see' itself and its runtime. Access to user files, network, graphics sockets, subsystems on the bus and devices have to be explicitly granted. (As will be described later, Flatpak provides several ways to do this.) Access to other things, such as other processes, is deliberately not possible.

By necessity, some resources that are inside the sandbox need to be exposed outside, to be used by the host system. These are known as "exports", since they are files that are exported out of the sandbox, and include things like the application's ``.desktop`` file and icon.

Portals
^^^^^^^

Portals are a mechanism through which applications can interact with the host environment from within a sandbox. In this way, they give additional abilities to interact with data, files and services without the need to add sandbox permissions.

Interface toolkits can implement transparent support for portals. If an application uses one of these toolkits, there is no additional work required to access them.

Examples of capabilities that can be accessed through portals include:

* Inhibit the user session from ending, suspending, idling or getting switched away
* Network status information
* Notifications
* Open a URI
* Open files with a native file chooser dialog
* Printing
* Screenshots

Applications that aren't using a toolkit with support for portals can refer to the `xdg-desktop-portal API documentation <http://flatpak.org/xdg-desktop-portal/portal-docs.html>`_ for information on how to access them.

The flatpak command
--------------------

``flatpak`` is the command that is used to find, install and remove applications. ``flatpak --help`` provides a full list of available commands.

Most flatpak commands are performed system-wide by default. To perform a command for the current user only, use the ``--user`` option. This allows runtimes and application bundles to be installed per-user, for instance.

For more information on flatpak commands, see the :doc:`flatpak-command-reference`

Identifiers
-----------

Flatpak identifies each application, runtime and SDK using a unique name, which is sometimes used as part of a name/arch/branch triple.

Naming
^^^^^^

Flatpak names take the form of an inverse DNS address, such as ``com.company.App``. The final segment of this address is the object's name, and the preceding part is the domain that it belongs to. In order to prevent name conflicts, this domain should correspond to a DNS registered address. This means using a domain from a website, either for an application or an organization. For instance, if application ``App`` has its own website at ``app.com``, its Flatpak name would be ``com.app.App``. Multiple applications can belong to the same domain, such as ``org.organization.App1`` and ``org.organization.App2``.

If you do not have a registered domain for your application, it is easy to use a third party website to get one. For example, Github allows the creation of personal pages that can be used for this purpose. Here, a personal namespace of ``name.github.io`` could be used as the basis of application identifier ``io.github.name.App``.

If an application provides a D-Bus service, the D-Bus service name is expected to be the same as the application name.

Identifier triples
^^^^^^^^^^^^^^^^^^

Many flatpak commands only require the name of an application, runtime or SDK. However, in some circumstances it is also necessary to specify the architecture and branch (branches allow a particular version to be specified). This is done using a name/arch/branch triple. For example: ``org.gnome.Sdk/x86_64/3.14`` or ``org.gnome.Builder/i386/master``.

Under the hood
--------------

Flatpak uses a number of pre-existing technologies. It generally isn't necessary to be familiar with these in order to use Flatpak, although in some cases it might be useful. They include:

* The `bubblewrap <https://github.com/projectatomic/bubblewrap>`_ utility from `Project Atomic <http://www.projectatomic.io/>`_, which lets unprivileged users set up and run containers, using kernel features such as:

  * Cgroups
  * Namespaces
  * Bind mounts
  * Seccomp rules

* `systemd <https://www.freedesktop.org/wiki/Software/systemd/>`_ to set up cgroups for sandboxes
* `D-Bus <https://www.freedesktop.org/wiki/Software/dbus/>`_, a well-established way to provide high-level APIs to applications
* The OCI format from the `Open Container Initiative <https://www.opencontainers.org/>`_, as a convenient transport format for single-file bundles
* The `OSTree <https://ostree.readthedocs.io/en/latest/>`_ system for versioning and distributing filesystem trees
* `Appstream <https://www.freedesktop.org/software/appstream/docs/>`_ metadata, to allow Flatpak applications to show up nicely in software-center applications

