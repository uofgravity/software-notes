Connecting to IGR Linux Resources
=================================

The IGR has a number of linux-based computing resources which you can access either locally from desktop machines within the IGR,
or remotely from, for example, a laptop.
Connecting to these machines requires you to use a system called :abbr:`SSH (Secure Shell Protocol)`, which will allow you to open a terminal on them over the internet.

Prerequisites
-------------

In order to work through the instructions on this page you'll need a few things already set up on your machine.

1. The University of Glasgow's VPN client.
   You can find details about installing this on various platforms `on the IT website <https://www.gla.ac.uk/myglasgow/it/vpn/>`_.

2. A terminal on the machine you're using which is able to run the ``ssh`` command.
   You can check this by running

   .. code-block:: console

		   $ ssh -V

		   OpenSSH_8.2p1 Ubuntu-4ubuntu0.4, OpenSSL 1.1.1f  31 Mar 2020

   .. note::
      In these instructions where you see a line starting with a ``$`` character it's an indication that you need to paste this line into a terminal **omitting** the ``$`` symbol.
      The text below this line is the expected output from the command.

   If you get something along the lines of what you see above then everything should be ready for you to get started.
   However, if you see something like this:

   .. code-block:: console

		   $ ssh -V

		   ssh: command not found

   Then you'll need to install OpenSSH.
   If you're using Ubuntu as your flavour of linux you can do this by running

   .. code-block:: console

		   $ sudo apt update
		   $ sudo apt install openssh-client

   .. note::
      These instructions will assume that you're using some flavour of Linux or another.
      It is also possible to use a machine running MacOS or Windows.
      MacOS should be fairly straight-forward, and SSH should be available through the standard terminal.

      Windows is a little more complicated, but it's possible to use :ref:`Windows Subsystem for Linux` (WSL) to run Linux in a highly-integrated virtual machine, or to use SSH via PowerShell.

Connecting via SSH with a password
----------------------------------

We'll first try the most basic (but also most inconvenient) method for connecting to a cluster machine.
The machine we'll connect to is called ``wiay``, and can be accessed by running

.. code-block:: console

		$ ssh wiay.astro.gla.ac.uk

However, if you're not connected via an internal university network, such as ``eduroam`` you'll first need to connect to the University VPN.

There's a good chance that the command above won't work for you first time, because your username on the astronomy system is different from the one on your machine.
You may have an account which is either attached to your GUID or another username on the astronomy system.

To connect using it we add the username (in this example, ``macfarlane``) to the command like so:

.. code-block:: console

		$ ssh macfarlane@wiay.astro.gla.ac.uk

You'll now be asked to enter your password; do this and then hit enter.
You should now be presented with a terminal (which will be a ``bash`` terminal unless you've explicitly set things up differently).

.. _ssh-keys:
Using an SSH key
----------------

An alternative approach to always using your password to authenticate is to use an *SSH Key-pair*.
These use a public and a private key in order to complete authentication between two machines.
The way that this works isn't important for setting things up, but it relies on having a piece of data on each machine which act in a way analogous to a lock and a key.

In order to use this authentication method we need to generate the key pair.
You do this using the ``ssh-keygen`` command.

.. code-block:: console

		$ ssh-keygen

		Generating public/private rsa key pair.
		Enter passphrase (empty for no passphrase):

The command will first ask you to enter a passphrase.
This is simply a password which protects the keypair.
While you can choose to omit it, it's generally not a good idea, and we can set things up on your machine later so that the keypair is kept "unlocked" to prevent you needing to enter the password anytime you use the keypair.
Once you've entered the passphrase you'll be asked to enter it again.

.. code-block:: console

		Enter same passphrase again:

Once you've re-entered the same passphrase and hit :kbd:`Enter` you'll get a message with some additional information about the keypair.
		
.. code-block:: console

		Your identification has been saved in ~/.ssh/id_rsa
		Your public key has been saved in ~/.ssh/id_rsa.pub
		The key fingerprint is:
		SHA256:Vhs5U3y5IwSy2y31N0wysPS5eAc7hapr/VJ0wD9dIKo daniel@laptop
		The key's randomart image is:
		+---[RSA 3072]----+
		|        . .=+ .o |
		|         o.+*+= .|
		|        . Bo.Oo+o|
		|         = B+o%+.|
		|        E +oo*o=o|
		|       .  ....o..|
		|         .. .    |
		|         ..o     |
		|        ..  o.   |
		+----[SHA256]-----+

You don't need to worry a great deal about what this means, but there are pointers to the location of two files.
One of these is your *private* key, and the other is the *public* key.
It's very important that you keep the *private* key secret, but the *public* key can be used safely on other machines.

The *public* part of the key is just a plaintext file which will look something like this:

.. code-block::

   ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCt/N1gcLv90aBW1TTV/sFa2PYR1fitlbuBK3FTluDmLfPu11PF96h0zUDCP/cXjdB53KRraL2ArtWd++wo6MzgC1A3T67PzB2mjzwOB5GlUgdiJ5n23EBfg4QRnN+kaX4IDYIDf7bdBVut1GAoHi9iVPXZOkCBFa8xMUKZtG/GkwF2ZeimK8L0a/BLy1OJTTM2Ni7VdGopoF6dRKF+t0E2m/n2EsdFVfNULpRGKyIUO0QY7LhcyQW5WDUPu9DgL81nTFMPQAAawvefg3DjVh4WcE0ia3a396dDA4ugWuJqMZvcWAlTHFcJZTSJwylX4TxvzmDq0vtYDJCNKtAyYO9xIl9O/Lkju7vIia3Fpl1zl9P6tHakXRRAk6moGZFKBLyoctKFFhoR9z0sEQjuM/eQmrBAoDB0X0aTNc5JVYuh/v69wQ+v+45vK3BYLEn0yph1DvvdIQ+AWH6jI7WWn9gE8NDj/WoyukSnIRwaSOpJLGB52JNnU5+B2P+AtDRMSq0= daniel@laptop


Normally, to set up keypair authentication we can run the command ``ssh-copy-id`` and your key will be copied to the remote machine and things will be set up.
For example

.. code-block:: console

		$ ssh-copy-id macfarlane@wiay.astro.gla.ac.uk

		/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/daniel/.ssh/id_rsa.pub"
		/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
		/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
		macfarlane@wiay.astro.gla.ac.uk's password:

You'll need to enter your password (hopefully for the last time!), and your key will be copied to the remote machine.

You can double check that everything is working by running

.. code-block:: console

		$ ssh macfarlane@wiay.astro.gla.ac.uk

If you aren't asked for a password then you're all set!

However, on some astro accounts and some machines the ssh credentials are stored a different way, and we'll need to take an additional step.

You'll need to go to the `P&A Identity Management page <https://it.physics.gla.ac.uk/identity>`_ and enter your username there.
This will bring up a page with some basic information, but you'll then need to enter your password to be able to access more detailed settings.

Among other settings, one option on this page is a place to paste an SSH key.
You can copy the contents of your *public* key (which lives in ``~/.ssh/id_rsa.pub``) and paste it here and then press ``Change!``.


Some helpful SSH Settings
-------------------------

We can reduce the number of keystrokes even more by setting up some shortcuts in the ``~/.ssh/config`` file.
For example, if I wanted to be able to log in to ``wiay.astro.gla.ac.uk`` using the much shorter command

.. code-block:: console

		$ ssh wiay

Then I could add these lines to my config file:

.. code-block::

   Host wiay
     Hostname wiay.astro.gla.ac.uk
     User macfarlane

.. note::
   Pay attention to the indentation of the lines after the ``Host name``.
     
If you have lots of machines where you use different usernames this can be especially helpful, since you won't need to remember them in addition to the name of every machine.
