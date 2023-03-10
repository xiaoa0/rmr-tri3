---
toc: true
layout: post
title: GUI with WSL
description: How to set up an X11 server to have a GUI run from WSL
author: Rohan Juneja
image: /images/gui_setup.png
type: pbl
week: 0
---

# Guide on how to get GUI on WSL
1. Download [VcXsrv](https://sourceforge.net/projects/vcxsrv/) on your windows computer and run the installer (I recommend to add a desktop icon during installation to easily access the application)
2. In a file explorer open C:\Program Files\VcXsrv and search for "xlaunch.exe" and open (Alternatively if you set up a desktop icon during installation you can double click on that instead)
create a "multiple window display" with "start no client", and make sure in extra settings to check "Disable access control", then press next/finish
![vcxrsv_image_1]({{ site.baseurl }}/images/vcxsrv-image-1.png)
![vcxrsv_image_2]({{ site.baseurl }}/images/vcxsrv-image-2.png)
3. Open a WSL window and open the ~/.bashrc file in vscode (code ~/.bashrc) or with nano (nano ~/.bashrc), in this file add the following lines
```
export DISPLAY=$(ip route list default | awk '{print $3}'):0
export LIBGL_ALWAYS_INDIRECT=1
```
5. REOPEN your WSL window, and THEN Run ``code ~/vscode/[name of apcsa repo]`` to open a vscode window with your apcsa repo
6. Run your code, you may need to click the following icon on your taskbar to see the gui
![java_icon]({{ site.baseurl }}/images/java-icon.png)

## Note: If you are getting errors about "java.lang.NoClassDefFoundError: Could not initialize class java.awt.GraphicsEnvironment$LocalGE"
- Run ``xhost +local:`` in WSL & restart VSCode 
- [See more details](https://github.com/rycus86/docker-intellij-idea/issues/11)
