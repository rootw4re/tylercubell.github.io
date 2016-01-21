---
title: 'Vagrant Resize Disk on Windows'
author: Tyler
layout: post
permalink: /vagrant-resize-disk-on-windows
categories:
  - Posts
---

1. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) for Windows.
2. Add VirtualBox directory to PATH environment variable.
        Windows Key + R->sysdm.cpl->Advanced Tab->Environment Variables->Select "path"->Edit->Add this:
        ;C:\Program Files\Oracle\VirtualBox 

3. Open command prompt with elevated privileges. For Windows 10 right click on start button and select "Command Prompt (Admin)".
4. Execute these commands:


        VBoxManage clonehd "C:\Users\YourName\VirtualBox VMs\Vagrant_YourProject\box-disk1.vmdk" "C:\Users\YourName\VirtualBox VMs\Vagrant_YourProject\cloned.vdi" --format vdi
        VBoxManage modifyhd "C:\Users\YourName\VirtualBox VMs\Vagrant_YourProject\cloned.vdi" --resize size_in_megabytes
        VBoxManage clonehd "C:\Users\YourName\VirtualBox VMs\Vagrant_YourProject\cloned.vdi" "C:\Users\YourName\VirtualBox VMs\Vagrant_YourProject\resized.vmdk" --format vmdk


5. Open VirtualBox GUI.
6. Select your project and click the settings button.
7. Navigate to storage tab on the left.
8. In the storage tree remove "box-disk1.vmdk" and add "resized.vmdk".
