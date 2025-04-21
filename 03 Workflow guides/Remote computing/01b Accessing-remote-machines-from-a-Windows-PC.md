# Accessing specific machines from a Windows PC

# Common vocabulary

* **_SSH:_** Secure shell. A protocol that allows user to remotely access, manipulate files, submit jobs etc. on a remote HPC system’s servers
* **_SSH keys:_** Two files (one public key file and one private key file) used for security purposes when logging into some HPCs. Both are kept on your computer. The public key file you will send to the HPC administrator (e.g. Herbert Fruchtl for hypatia). The private key is used by your computer to identify itself when logging into the remote HPC.
* **_SFTP:_** Secure file transfer protocol. A protocol that allows users to transfer files between their local computer and a remote server.

## Hypatia

### Logging on using PuTTY

1. Email Hypatia admin and ask them to make an account for you (see the [St Andrews HPC page](https://www.st-andrews.ac.uk/high-performance-computing/getting-started/) for details). Your St Andrews username will be your hypatia username. 

2. Download [PuTTY](https://www.putty.org/), a free SSH client for Windows.

3. Upon launching PuTTY, the “PuTTY Configuration” window will open. Here you can set several settings and save them as a dedicated session for easy access later. The default settings should be mostly fine, with a few changes:  
   1. In the main “Session” page:  
      1. Host Name (or IP address): 138.251.14.231 (note, the hostname “hypatia” is sadly preliminary and may be replaced, so we currently are using the IP address to log in until further notice.)  
      2. Default settings that should be fine: Port: 22; Connection type: SSH  
   2. Note, if ssh keys are implemented later (currently not required for logging on to hypatia), your private key file should be added under “Connection > SSH > Auth > Credentials”.  
   3. In “Session”, name your session under “Saved Sessions” and click “Save”. From now on, you can select this session and click “Open” to launch it (or click “Load” to load its settings should you wish to make any changes. Don’t forget to click “Save” after making edits!)  

4. Upon your first login, you will use a password given to you by the Hypatia admin. You will be prompted to change it to whatever you like (10 character minimum).

## ARCHER2

### Logging on using PuTTY

1. Download [PuTTY](https://www.putty.org/), a free SSH client for Windows.

2. Generate an SSH key pair following the steps under “Access credentials” on [this page](https://docs.archer2.ac.uk/user-guide/connecting/#access-credentials). You will also need to set up MFA (multi-factor authentication). The “Access credentials” instructions linked above will point you to [this page](https://epcced.github.io/safe-docs/safe-for-users/#how-to-turn-on-mfa-on-your-machine-account) to do that. There are various MFA apps you can use, which will generate a 6-digit authentication code (TOTP code) for you to use each time you log in.

3. Upon launching PuTTY, the “PuTTY Configuration” window will open. Here you can set several settings and save them as a dedicated session for easy access later. The default settings should be mostly fine, with a few changes:
   1. In the main “Session” page:  
      1. Host Name (or IP address): login.archer2.ac.uk   
      2. Default settings that should be fine: Port: 22; Connection type: SSH  
   2. In “Connection > SSH > Auth > Credentials”:
      1. Click “Browse” to upload the path on your local computer to your “Private key file for authentication” (generated as part of your SSH key pair, Step 2 above)
   3. Back on “Session”, name and save these settings under “Saved Sessions”. From now on, you can select this session and click “Open” to launch it (or click “Load” to load its setting should you wish to make any changes. Don’t forget to click “Save” if you do!)
4. Double-click or select and click “Open” on your saved ARCHER2 session to launch the log in page to archer. Enter your ARCHER2 username, your passphrase for your private key if you made one, and the TOTP code from your MFA app.


### Browsing/Transferring files using WinSCP

1. Download [WinSCP](https://winscp.net/eng/download.php), a free SHTP client for Windows
2. Launch WinSCP and at the top left click “New Tab”
3. On the “Login” window that opens, click “New Site” and set the following settings:
   1. File protocol: SFTP
   2. Host name: login.archer2.ac.uk
   3. Port number: 22
   4. User name: your ARCHER username 
4. Now click “Advanced…”, and under “SSH > Authentication”, click the “…” under “Private key file” to browse to the path on your local computer to your ARCHER2 private key file.
5. Click “Ok” to close the advanced settings and click “Save” back on the main “Login” window to save your settings. 
6. Now, you can double-click the “site” you’ve saved to launch into ARCHER. You may need to enter the passphrase for your private key if you made one, and you may need to enter a TOTP code from your MFA app if you haven’t logged on to ARCHER in the last few hours (see Step 2 of “Logging on to ARCHER2 from a Windows machine using PuTTY” for a link on how to set up MFA).


### Transferring files from ARCHER2 to JASMIN  

Transferring data to JASMIN can be done via the online [Globus Connect Server](https://www.globus.org/globus-connect-server). More information on Globus can be found on the [Globus](https://github.com/ODyn-StAndrews/ODyn-StAndrews.github.io/wiki/Globus) page on this Wiki. See documentation from [JASMIN](https://help.jasmin.ac.uk/docs/data-transfer/globus-transfers-with-jasmin/) and [ARCHER](https://docs.archer2.ac.uk/data-tools/globus/) on how to access their respective collections and initiate a data transfer (the same instructions are detailed on both pages).  

Note you can also transfer data between ARCHER/JASMIN and your local machine using the Globus Connect Personal app (more info [here](https://help.jasmin.ac.uk/docs/data-transfer/globus-connect-personal/)).


## JASMIN

### Logging on to JASMIN from a Windows machine using MobaXterm

1. First, complete the first three steps on [this page](https://help.jasmin.ac.uk/docs/getting-started/get-started-with-jasmin/) (and links within) which are: (1) Generate an SSH key, (2) Register for a JASMIN portal account, and (3) Request “jasmin-login” access.  
2. Download [MobaXTERM](https://mobaxterm.mobatek.net/), an application for Windows that is both an SSH and SFTP client (allowing users to connect to remote servers and easily access and edit remote files from their local computer).
3. Launch MobaXTERM and click on “Settings” on the top bar. Go to the “SSH” tab.
   1. Under “SSH agents” ensure “Use internal SSH agent MobAgent” and “Forward SSH agents” are both ticked.
   2. Also click the (+) button under “Load following keys at MobAgent startup” and add the path on your local computer to the private key file you generated as part of Step 1.
   3. Click “OK” and close the “Settings” window.
4. Click on the little tab with a “+” on it near the top (under the top bar where the “Settings” button was) to open a new terminal window within MobaXterm.
5. SSH onto any of the JASMIN login servers (numbered 01-04) with the command: `ssh -A username@login-01.jasmin.ac.uk` 
   1. There are four login servers (numbered login-01 to login-04). If you ever fail to login and get a “timed out” message, try using a different login server.
6. If you have been granted access to the [scientific analysis servers](https://help.jasmin.ac.uk/docs/interactive-computing/sci-servers/) (needed to access the group workspace), you can log on to any of those with: `ssh username@sci-vm-01@jasmin.ac.uk`  
   1. There are six virtual machines (numbered 01-06) and two physical machines (sci-ph-01 and sci-ph-02).
   2. You can then access the group workspace with: `cd /gws/nopw/j04/co2clim/`


### Browsing files using WinSCP

1. Download [WinSCP](https://winscp.net/eng/download.php), a free SHTP client for Windows
2. Launch WinSCP and at the top left click “New Tab”
3. On the “Login” window that opens, click “New Site” and set the following settings:
   1. File protocol: SFTP
   2. Host name: xfer-vm-01.jasmin.ac.uk (or -02 or -03)
   3. Port number: 22
   4. User name: your JASMIN username 
4. Now click “Advanced…”, and under “SSH > Authentication”, click the “…” under “Private key file” to browse to the path on your local computer to your JASMIN private key file.
5. Click “Ok” to close the advanced settings and click “Save” back on the main “Login” window to save your settings. 
6. Now, you can double-click the “site” you’ve saved to launch into JASMIN. You may need to enter the passphrase for your private key if you made one.


### Transferring files to/from JASMIN
Transferring data to JASMIN can be done via the online [Globus Connect Server](https://www.globus.org/globus-connect-server). More information on Globus can be found on the [Globus](https://github.com/ODyn-StAndrews/ODyn-StAndrews.github.io/wiki/Globus) page on this Wiki. See documentation from [JASMIN](https://help.jasmin.ac.uk/docs/data-transfer/globus-transfers-with-jasmin/) and [ARCHER](https://docs.archer2.ac.uk/data-tools/globus/) on how to access their respective collections and initiate a data transfer (the same instructions are detailed on both pages).  

Note you can also transfer data between ARCHER/JASMIN and your local machine using the Globus Connect Personal app (more info [here](https://help.jasmin.ac.uk/docs/data-transfer/globus-connect-personal/)).

