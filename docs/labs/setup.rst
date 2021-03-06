Development Environment
=======================

Before hacking on the programming assignments, you will need to setup the development environment as described below.

Our labs are designed to run on `Ubuntu 18.04 <https://en.wikipedia.org/wiki/Ubuntu>`__  (Linux Distribution). We recommend that you install this specific version of the operating system (**18.04**). Otherwise, you may face issues with running the shell scripts in the labs.

This page contains information on how you could download and install the Ubuntu development environment. We will assume familiarity with Unix commands throughout.

Ubuntu 18.04 LTS
----------------

If you are already using Ubuntu 18.04 on your laptop, then you can skip these steps and directly proceed to `Package Installation <#package-installation>`__. If that is not the case, then you should install a virtual machine.

Virtual Machine Installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Our recommended way of setting up your virtual machine is to use `Vagrant <https://www.vagrantup.com/intro>`__  along with  `VirtualBox <https://www.virtualbox.org/manual/ch01.html#virt-why-useful>`__. 

- Vagrant enables users to create and configure lightweight, reproducible, and  portable development environments. 
- VirtualBox is a general-purpose full virtualizer for x86 hardware.

#. Download and install Virtualbox
    - (Windows) Turn off Hyper-V in "Turn Windows features on or off"
    - Download and install `VirtualBox <https://www.virtualbox.org/wiki/Download_Old_Builds_6_0>`__ 
        - Install **6.0.24** version.
        - Vagrant currently does not work with version *6.1* of VirtualBox.

#. Download and install `Vagrant <http://www.vagrantup.com/downloads.html>`_
	- Install **2.2.10** version.

#. Install the Ubuntu Virtual Machine (VM)

.. code-block:: sh

        # Open Terminal (e.g., `Terminal` in Mac, `Powershell` in Windows)
	
        # Download the VM
        [host] $ vagrant box add ubuntu/bionic64
	
	# Install scp plugin for copying files between host and VM
	[host] $ vagrant plugin install vagrant-scp

        # Setup buzzdb folder
	[host] $ cd Desktop
        [host] $ mkdir buzzdb
	[host] $ cd buzzdb

        # Initialize the VM -- generates a Vagrantfile within the buzzdb folder
        [host] $ vagrant init ubuntu/bionic64
	
	# Start the VM
        [host] $ vagrant up
	
	# Enter into the VM
        [host] $ vagrant ssh
	[vm] $ pwd
	[vm] $ ls
			
	# To temporarily exit the VM and return to the host (at any point in time)
	[vm] $ exit
	
	# To shutdown the VM and return to the host
	[vm] $ sudo shutdown now
	
	# Check hostname of VM (e.g., "default")
	[host] $ vagrant status
    
        # Copy the Vagrant SSH configuration to the end of the local SSH configuration file
        # Skip this, if using windows. To enable ssh in windows, you can setup putty (https://www.putty.org/)
        [host] $ vagrant ssh-config >> vi ~/.ssh/config
    
        # You should now be able to ssh into the VM (user@hostname) 
        # Skip this if using windows
        [host] $ ssh vagrant@default

.. admonition:: Warning: Remote connection disconnect. Retrying...
    :class: warning

        If you get this error while running ``vagrant up``, you might want to run ``vagrant destroy`` to stop the running machine and destroy all the created resources during the machine creation process. **Note it will delete all your work**. In case you have already started working, the following reference might help `<https://github.com/hashicorp/vagrant/issues/9834>`__.

#. Bump up the resources assigned to the VM
    - Search for `VirtualBox` in the `Vagrantfile` generated within the `buzzdb` folder
    - Increase memory size (to 2 GB or higher if possible)
    - Increase number of CPUs (to 4 or higher if possible)

.. code-block:: sh

        # Edit the Vagrantfile
        [host] $ cd buzzdb
	[host] $ vi Vagrantfile or [windows] $ notepad Vagrantfile

.. code-block:: ruby

    config.vm.provider "virtualbox" do |vb|
        # Customize the amount of memory assigned to the VM
        vb.memory = "2048"
	
	# Customize the number of CPUs
	vb.cpus = 4
    end 

Package Installation 
--------------------

Once you have Ubuntu OS up and running, install all the required packages for the programming assignments:

.. code-block:: sh

    # Start the vm
    [host] $ cd buzzdb
    [host] $ vagrant ssh
    
    # Install packages
    [vm] $ sudo apt-get -y update
    [vm] $ sudo apt-get -y install build-essential 
    [vm] $ sudo apt-get -y install zip unzip git cmake llvm valgrind clang clang-tidy clang-format googletest zlib1g-dev libgflags-dev libbenchmark-dev
    [vm] $ cd /usr/src/googletest; sudo mkdir build; cd build; sudo cmake ..; sudo make; sudo cp googlemock/*.a googlemock/gtest/*.a /usr/lib; cd /vagrant/;

    # Install zsh + oh-my-zsh | for command completion and searching through command history
    # Reference: https://hackernoon.com/oh-my-zsh-made-for-cli-lovers-bea538d42ec1
    [vm] $ sudo apt-get -y install zsh
    [vm] $ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"


Editor Installation
-------------------

We recommend using `VSCode <https://code.visualstudio.com/>`_ for the programming assignments.

#. Here's a guide for `Getting started with VSCode <https://code.visualstudio.com/docs>`_. VSCode comes with a built-in terminal. 

#. Install these two extensions in VSCode: 
    - `C++ <https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools>`_
    - `Remote SSH <https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh>`_ (only need if you are using a VM)
    
#. You can now connect to the remote host (i.e., the VM) using the `Remote SSH extension <https://code.visualstudio.com/docs/remote/ssh#_connect-to-a-remote-host>`__
#. If you are using windows, you might find `Visual Studio Code with Vagrant <https://medium.com/@lopezgand/connect-visual-studio-code-with-vagrant-in-your-local-machine-24903fb4a9de>`__ helpful.
    
--------------

Questions or comments regarding the course?
Send an e-mail to `arulraj@gatech.edu <mailto:arulraj@gatech.edu>`__.
