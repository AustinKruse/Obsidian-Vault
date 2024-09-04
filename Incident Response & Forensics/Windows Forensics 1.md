## Hands-on Challenge

**The Setup:**

If preferred, use the following credentials to log into the machine:

**Username:** THM-4n6

**Password:** 123

Once we log in, we will see two folders on the Desktop named `triage` and `EZtools`. The `triage` folder contains a triage collection collected through KAPE, which has the same directory structure as the parent. This is where our artifacts will be located. The `EZtools` folder contains Eric Zimmerman's tools, which we will be using to perform our analysis. You will also find RegistryExplorer, EZViewer, and AppCompatCacheParser.exe in the same folder.

**The Challenge:**

﻿Now that we know where the required toolset is, we can start our investigation. We will have to use our knowledge to identify where the different files for the relevant registry hives are located and load them into the tools of our choice. Let's answer the questions below using our knowledge of registry forensics.

**Scenario:**

One of the Desktops in the research lab at Organization X is suspected to have been accessed by someone unauthorized. Although they generally have only one user account per Desktop, there were multiple user accounts observed on this system. It is also suspected that the system was connected to some network drive, and a USB device was connected to the system. The triage data from the system was collected and placed on the attached VM. Can you help Organization X with finding answers to the below questions?

**Note:** When loading registry hives in RegistryExplorer, it will caution us that the hives are dirty. This is nothing to be afraid of. We just need to remember the little lesson about transaction logs and point RegistryExplorer to the .LOG1 and .LOG2 files with the same filename as the registry hive. It will automatically integrate the transaction logs and create a 'clean' hive. Once we tell RegistryExplorer where to save the clean hive, we can use that for our analysis and we won't need to load the dirty hives anymore. RegistryExplorer will guide you through this process.

# Questions
---------------------
1. **How many user created accounts are present on the system?**

   ```plaintext
   3
   ```
	I Opened up the 5 hives, went to the SAM hive & clicked the users folder.  This showed `THM-4n6`, `thm-user`, & `thm-user2`, they are the only user accounts with User ID's over 1000.
	![](c1f0095f0c99b94ef7fda44f97961730.png)

2. **What is the username of the account that has never been logged in?**

   ```plaintext
   thm-user2
   ```

3. **What's the password hint for the user THM-4n6?**

   ```plaintext
   count
   ```

4. **When was the file 'Changelog.txt' accessed?**

```plaintext
2021-11-24 18:18:48
```

	![[assets/Pasted image 20240829085020.png]]
	Searched for keyword `Recent` & there are a few spots in the different hives to find recent information, like Apps, Docs, Items, and more.  The information was found in the NTUSER.DAT hive.

5. **What is the complete path from where the python 3.8.2 installer was run?**

   ```plaintext
   Z:\setups\python-3.8.2.exe
   ```
	In the RecentApps folder, of the NTUSER.DAT hive (since this the hive where all `THM-4n6` files will be),

6. **When was the USB device with the friendly name 'USB' last connected?**

   ```plaintext
   2021-11-24 18:40:06
   ```
	This USB information can be found in the SOFTWARE hive, `SOFTWARE\Microsoft\Windows Portable Devices` and in the `Devices`
	![](43ac820c78937470fc6d4ef3fccf3aa6.png)
	![](ea203a61c31a64373bac818a9104c398.png)
	This shows the GUID `{E251921F-4DA2-11EC-A783-001A7DDA7110}` & the friendly name `USB`.  Now to determine when it was last connected, we check `SYSTEM\USBSTOR` & cross-reference the GUID.
	![](e48343a0d1f4b2e321ae97d97105c0bd.png)

# Cheat Sheet
-------------------
 
![](WindowsForensicsCheatsheet-TryHackMe-1642092762578.pdf)

What is the path for the five main registry hives, DEFAULT, SAM, SECURITY, SOFTWARE, and SYSTEM?

`C:\Windows\System32\Config` & NTUSER.DAT (hidden file) hive is in the users home directory.

# Sources
- [https://tryhackme.com/r/room/windowsforensics1](https://tryhackme.com/r/room/windowsforensics1)
- [Eric Zimmerman's tools](https://ericzimmerman.github.io/#!index.md)
