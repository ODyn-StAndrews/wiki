# Accessing remote machines (General)
It is common to do computations and analysis on remote machines, particularly High Performance Computers, where the computational power exceeds that of a standard desktop or laptop. Here, we include some general information about accessing remote machines from different operating systems (Linux/Unix and Windows), and links to information on accessing the main machines that we work on in the group (ARCHER2, JASMIN, Hypatia).

# Accessing machines through different operating systems
## Linux
Almost all remote HPC machines use a linux OS, so accessing them is most straightforward from a machine that uses the same OS.

## Unix (Mac)
Unix is the OS configuration of Apple Mac machines and its structure is based on that of linux. As such, it is similarly straightforward to remotely access linux machines.
### Terminal
The Terminal application on your Mac machine allows you to interact with the OS through a command line interface. This is sufficient for most purposes, e.g. to `ssh` into those machines, or to copy files to/from those machines.

## Windows
Windows is an entirely distinct OS from linux, so requires some specific programs and approaches to easily interact with remote machines. Once these are set up, however, this can be done just as easily as with other OSs. Detailed guidelines on how to access specific machines from a Windows OS can be found on the [[Accessing remote machines from a Windows PC]] page.
### PuTTY
PuTTY is an "SSH client" that allows you to login to a remote machine from your Windows machine. Download [PuTTY](https://www.putty.org/)
### WinSCP
WinSCP is an "SFTP client" (Secure File Transfer Protocol) that allows you to transfer data between remote machines. Download [WinSCP](https://winscp.net/eng/index.php)
### MobaXterm
MobaXterm is an SSH and SFTP client, that is particular useful in accessing and transferring data to/from [[JASMIN]]. Download [MobaXterm](https://mobaxterm.mobatek.net/).
