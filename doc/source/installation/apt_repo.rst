.. _apt_repo:

===================================
 Percona :program:`apt` Repository
===================================

*Debian* and *Ubuntu* packages from *Percona* are signed with a key. Before using the repository, you should add the key to :program:`apt`. To do that, run the following commands: ::

  $ apt-key adv --keyserver keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A

Add this to :file:`/etc/apt/sources.list`, replacing ``VERSION`` with the name of your distribution: ::

  deb http://repo.percona.com/apt VERSION main
  deb-src http://repo.percona.com/apt VERSION main

Remember to update the local cache: ::

  $ apt-get update

Supported Platforms
===================

 * x86
 * x86_64 (also known as ``amd64``)

Supported Releases
==================

Debian
------

 * 6.0 (squeeze)
 * 7.0 (wheezy)

Ubuntu
------

 * 10.04LTS (lucid)
 * 12.04LTS (precise)
 * 12.10 (quantal)
 * 13.04 (raring)
 * 13.10 (saucy)


Release Candidate Repository
============================

To subscribe to the release candidate repository, add two lines to the :file:`/etc/apt/sources.list` file, again replacing ``VERSION`` with your server's release version: ::

  deb http://repo.percona.com/apt-rc VERSION main
  deb-src http://repo.percona.com/apt-rc VERSION main


Apt-Pinning the packages
========================

In some cases you might need to "pin" the selected packages to avoid the upgrades from the distribution repositories. You'll need to make a new file :file:`/etc/apt/preferences.d/00percona.pref` and add the following lines in it: :: 

  Package: *
  Pin: release o=Percona Development Team
  Pin-Priority: 1001

For more information about the pinning you can check the official `debian wiki <http://wiki.debian.org/AptPreferences>`_.
