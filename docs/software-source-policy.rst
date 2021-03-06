.. _debops_policy__software_source_policy:

DebOps Software Source Policy
=============================

.. include:: includes/all.rst

:Date drafted: 2016-07-23
:Date effective: 2016-09-01
:Last changed: 2016-07-25
:Version: 0.1.0
:Authors: - drybjed_

.. This version may not correspond directly to the debops-policy version.

Terminology
-----------

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14, [`RFC2119`_].

Goals of the Policy
-------------------

One of the goals of the DebOps Project is to provide a framework for
a reliable, secure and reproducible management of servers and workstations "in
production", which means hosts that are used in production environment, on live
Internet-facing networks. For that to be possible, the project needs to have
a clear set of rules about how and from where to get the software required to
manage these hosts.

Host Operating System(s)
------------------------

For the server machines, base Linux distribution selected to be used with
the DebOps Project is the current Debian GNU/Linux Stable release.

Due to how Debian development occurs, some systems managed by DebOps might
switch from Debian Stable to Debian Oldstable release during their life cycle.
Therefore, DebOps SHOULD support current Debian Oldstable release at least as
long, as its officially supported by Debian security updates. However, some
time after next Debian Stable release, due to incompatibilities between
releases various parts of the DebOps Project are expected to only support
current Debian Stable.

Support for Debian LTS release is optional, depending on the status of the
release and interested user base.

For other systems (workstations, laptops, etc.) preferred OS distributions are
Debian GNU/Linux and Debian derivatives like Ubuntu Linux, or others, depending
on the choice of the administrator. Support for these distributions may vary,
any required changes to make the roles and playbooks work on other Debian-based
systems are welcome.

Support for different Linux variants in different roles might be considered in
the future, depending on compatibility or amount of the differences between
these distributions and Debian and its derivatives.

Software repositories and sources
---------------------------------

Software used on hosts managed using the DebOps Project comes from various
sources. This is a list of these sources, ordered from the most to least
preferred and supported by the project maintainers:

Debian Software Repository - Stable release
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This repository is considered the most reliable source of software. Packages
here are guaranteed to be stable (without any changes apart from security
updates), signed and trusted by Debian Developers. The Debian Software Repository
is considered to be reliable due to the requirement of software sources
included in the Repository, as well as availability of multiple mirrors
worldwide. The availability of binary packages ensures that no unnecessary
dependencies are needed on production systems which increases reliability and
security.

Debian Software Repository - Stable Backports
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Packages backported from the next Debian release (Testing) are considered less
stable than the current Debian Stable releases. However, due to build process
based on Debian Stable packages and presence in the Debian Software Repository,
they are considered as compatible with Debian Stable and considered less risky
than packages from external repositories. It is preferred that software
packages from outside of Debian SHOULD be made accessible via Debian Backports
after getting through usual Debian package management process (Unstable,
Testing).

Local APT repositories
~~~~~~~~~~~~~~~~~~~~~~

APT packages built by the local system administrator or developer and
distributed using a local APT repository are an acceptable alternative to the
Debian Software Repository. This software source can be used to check if Debian
Testing packages can be backported to Debian Stable before using it in
production.

Upstream APT repositories
~~~~~~~~~~~~~~~~~~~~~~~~~

APT repositories provided by a software vendor with binary ``.deb`` packages
that are signed by a valid OpenPGP key should be available if necessary in the
Ansible roles or playbooks. However unless the software is not available in
Debian Software Repository, upstream repositories MUST require a condition
that enables them.  This should give the local system administrators an option
to install either Debian or upstream version of the package as they see fit.
Configuration required to enable these repositories (installation of an OpenPGP
key and configuration of APT sources) should be performed using tools available
in Ansible, and not using external scripts provided by the vendor. The OpenPGP
key used to sign the packages MUST be imported by its key fingerprint. Those
measures should ensure idempotency and reliability.

Open Source software distributed as binary archives
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some software vendors release binary archives of Open Source software built
from the published sources. This method of installation is acceptable only in a
case where the software vendor provides a means of authentication of said
binary archives, for example through hashes of the files authenticated by known
OpenPGP keys that belong to the software vendor. The Ansible roles which use
this method of installation MUST validate downloaded archives against the
provided hashes, authenticity of which is checked using the OpenPGP keys.

Software installed from git repositories
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some software packages (for example, web-based applications) are distributed
using ``git`` repositories. Programs installed in this fashion are acceptable,
provided that the repository itself is downloaded first to ensure that the
source code is available. If a repository contains a "stable" branch or its
equivalent, it should be preferable to a latest stable tag which should be
considered before "development" branch. If software distributed this way
requires compilation, and vendor provides releases of binaries or tarballs
through the ``git`` repository or an alternative method, they might be
considered for use as well, when software is managed directly on production
servers. If ``git`` repository contains instructions that make building a
``.deb`` package easier, this option should be considered as a preferable with
instructions how to do it in the Ansible role documentation, in which case
installation from ``git`` sources should be provided as an option.
If the repository provides signed releases, they MUST be verified and used.
It can be considered to require a certain commit hash corresponding to a signed
release. A certain (ideally audited) commit hash MUST be specified in case no
code signing is used to provide at least some sort of authenticity in
conjunction with the :ref:`debops_policy__code_signing_policy`.

Software installed by other package managers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Package managers other than APT, like ``pip``, ``gem``, ``phar``, etc. are
considered less reliable than software installed from source repositories.
Different package managers might have different security practices,
less stable software and reliability issues, and potentially might induce
dependency issues in the managed systems, therefore software from Debian
Software Repository should have priority if possible. If not possible, it
should be tried to install most of the dependencies of the program from Debian
Software Repository.
