---
layout: post
title: Overtone in a Ubuntu VM
---

This is a guide to getting Overtone working in a virtual machine using Ubuntu 14.10 as the guest.
The host system I am using is Windows 8.1, but the guide should hopefully work for other host systems as well.  
I tried VMware Player first but had problems getting JACK audio working.
This lead me to use VirtualBox instead.

Getting Ubuntu up and running in VirtualBox
===========================================
Download and install VirtualBox for your host system from [the VirtualBox download page](https://www.virtualbox.org/wiki/Downloads).  
Create a new VM (virtual machine).  
By naming it Ubuntu, VirtualBox will select the type and version as Linux Ubuntu for you.  
After the VM is created select it from the list and press <kbd>Settings</kbd>.  
Enable shared clipboard by going to <kbd>General-></kbd><kbd>Advanced-></kbd><kbd>Shared Clipboard</kbd> and change it to Bidirectional.  
It's a good idea to go to <kbd>Display-></kbd><kbd>Video</kbd> and increase the dedicated Video memory and enabling 3D acceleration.  
The VM is now ready for performing installation by selecting your Ubuntu ISO.  
I had some screen artifacts during the installation. [StackOverflow helped me](http://askubuntu.com/questions/541006/ubuntu-14-10-does-not-install-in-virtualbox) and I fixed the issue by hitting <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F1</kbd> and then <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F7</kbd>.

Fixing screen resolution and copy-paste
---------------------------------------
The VM in it's current state will not detect if you resize the VM window and the previously enabled copy-paste between host and guest will not work either.  
[StackOverflow helped me again](http://askubuntu.com/questions/452108/cannot-change-screen-size-from-640x480-after-14-04-installation-on-virtualbox-os) and I found at that to fix this we need to install the VirtualBox Guest Additions.  
Run the following command in a terminal:

    sudo apt-get install virtualbox-guest-dkms
Restart the VM.

Setting up dependencies
=======================
With a working VM running Ubuntu it's time to set up the dependencies for Overtone.  
Let's start by installing all the packages we will need from apt-get.  
Run the following command in a terminal:

    sudo apt-get install supercollider openjdk-8-jdk qjackctl pulseaudio-module-jack
Integrating PulseAudio with JACK
--------------------------------
If you are fine with having to start JACK on every boot you don't need to perform the following steps.  
The default configuration of PulseAudio yields control of the audio equipment to JACK when the JACK server starts.  
PulseAudio will not be able to receive input or send output of any audio signals on the audio interface used by JACK.  
I find it a bit tedious to start JACK using QjackCtl and found a solution on [the Fedora Project website](http://docs.fedoraproject.org/en-US/Fedora/15/html/Musicians_Guide/sect-Musicians_Guide-Integrating_PulseAudio_with_JACK.html).  
### PulseAudio configuration
You'll need to edit the PulseAudio configuration file to use the JACK module.  
Be careful! You will be editing an important system file as the root user!  
Run the following command in a terminal:

    sudo gedit /etc/pulse/default.pa
Add the following lines, underneath the line that says #load-module module-alsa-sink:

    load-module module-jack-sink
    load-module module-jack-source
Restart PulseAudio by running the following command in a terminal:

    killall pulseaudio
PulseAudio restarts automatically.  
Confirm that this has worked by opening QjackCtl. The display should confirm that JACK is "Active".  
Uncheck Setup...->Misc->Stop JACK audio server on application exit. Otherwise QjackCtl will kill JACK when you close it and all sound will be gone.
Leiningen
---------
Leiningen is the project manager used by clojure.  
Install Leiningen by running the following commands in a terminal:

    mkdir ~/bin
    cd ~/bin
    wget https://raw.github.com/technomancy/leiningen/stable/bin/lein
    chmod +x lein
The bin directory will be added to `$PATH` automatically if you restart the machine but you can also run:

    export PATH=$PATH:~/bin
to add it directly without restarting.

New clojure project
-------------------
With Leingen installed we are ready to start programming if we want.
Create a new clojure project named tutorial by running:

    mkdir ~/src
    cd ~/src
    lein new tutorial
You can start a new REPL using

    lein repl
This is all you minmally need, but I find it nice to have a good IDE when I program.

Installing IntelliJ IDEA and the Cursive plugin
-----------------------------------------------
I have tried several IDEs for clojure and I find IntelliJ IDEA the easiest to set up and use.  
Download the IntelliJ tarball from [the IntelliJ IDEA download page](https://www.jetbrains.com/idea/download/).  
Extract the contents to where you want it installed.  
I choose to extract the contents of the first folder (for me named idea-IU-139.659.2) to ~/programs/intellij/  
Follow the install-Linux-tar.txt.  
I have written out the steps I used below:  
Navigate to inside the IntelliJ installation directory with the terminal and then into the bin folder.  
Start IntelliJ by running:

    ./idea.sh
The installtion wizard will help you with creating a desktop entry.
### Cursive plugin installation
Now it's time to install the Cursive plugin.  
Follow the installation guide from [the cursive plugin userguide](https://cursiveclojure.com/userguide/).  
I have written out the steps I used below (for IntelliJ 14):  
To install the plugin, open the IntelliJ Settings, then select <kbd>Plugins-></kbd><kbd>Browse Repositories-></kbd><kbd>Manage Repositories</kbd>.  
Add a repository with the url:

    http://cursiveclojure.com/plugins-14.xml
Return to Browse Repositories, then in the <kbd>Repository:</kbd> dropdown select the new repo.  
You should see a plugin called cursive-0.1.xx (where xx is some version number).  
Install it using the install button, then close the Browse Repositories window and the Settings window.  
Restart IntelliJ when it prompts you to.

Hack away!
==========
The setup is now done and it's time to start making some music.  
In IntelliJ select <kbd>Import project</kbd> and select the project we created earlier.  
Select <kbd>Import project from external model</kbd>, and then <kbd>Leiningen</kbd>.  
Press Next a few times until it's time to select a project SDK.  
Since this is our first run we need to add a new JDK.
Press the plus and then JDK.
IntelliJ will find the jvm folder and you select the java-8-openjdk folder.
Press Next and then Finish.  
<kbd>Alt</kbd>+<kbd>1</kbd> will open the Project tool window.
Open the `project.clj` and add

    [overtone "0.9.1"]
to the dependencies.  
My `project.clj` after the modification:

    (defproject tutorial "0.1.0-SNAPSHOT"
      :description "FIXME: write description"
      :url "http://example.com/FIXME"
      :license {:name "Eclipse Public License"
                :url "http://www.eclipse.org/legal/epl-v10.html"}
      :dependencies [[org.clojure/clojure "1.6.0"]
                     [overtone "0.9.1"]])

Open the `src/tutorial/core.clj` file and modify it to:

    (ns tutorial.core
        (:require [overtone.live :refer :all]))

    (definst foo [] (saw 220))

We need to start a REPL and load our code.
Right click the project and press Run 'REPL for tutorial'.
Wait for the REPL to start.
To load our code have the tutorial.core file opened and focused and click <kbd>Tools-></kbd><kbd>REPL-></kbd><kbd>Load file in REPL</kbd>.

Wait for the Overtone welcome message.  
Then to change the REPL namespace to the tutorial.core file we need have it opened and focused and click <kbd>Tools-></kbd><kbd>REPL-></kbd><kbd>Switch REPL NS to current file</kbd>.  
If it all works, running:

    (foo)
in the REPL will start a saw tone instrument.  
You can kill it by running

    (kill foo)
or

    (stop)
