---
title: Data Management on CARC Systems
author: Derek Strong <br> dstrong[at]usc.edu <br> Research Computing Associate <br> CARC at USC <br>
date: <code>2021-04-02</code>
---


## Outline

- Storage file systems
- Managing files
- Transferring files


## Storage file systems

| File system | Recommended storage | Recommended activities |
|---|---|---|
| /home1 | personal files, configuration files | creating and editing scripts/programs |
| /project | shared files, software installations, job scripts and files, I/O files | creating and editing scripts/programs, compiling, I/O, transferring data |
| /scratch & /scratch2 | temporary and I/O files | I/O, testing |

<br>

- Mounted on both the Discovery and Endeavour clusters
- Run BeeGFS/ZFS, and files are automatically compressed (on the fly)


## myquota

```
ttrojan@discovery1:~$ myquota
--------------------------
/home1/ttrojan
      user/group     ||           size          ||    chunk files    
     name     |  id  ||    used    |    hard    ||  used   |  hard   
--------------|------||------------|------------||---------|---------
       ttrojan|555555||  127.23 MiB|  100.00 GiB||     4530|  2000000

--------------------------
/scratch/ttrojan
      user/group     ||           size          ||    chunk files    
     name     |  id  ||    used    |    hard    ||  used   |  hard   
--------------|------||------------|------------||---------|---------
       ttrojan|555555||  446.78 MiB|   10.00 TiB||     5797|unlimited

--------------------------
/scratch2/ttrojan
      user/group     ||           size          ||    chunk files    
     name     |  id  ||    used    |    hard    ||  used   |  hard   
--------------|------||------------|------------||---------|---------
       ttrojan|555555||  200.34 MiB|   30.00 TiB||     4002|unlimited

--------------------------
/project/ttrojan_123
      user/group     ||           size          ||    chunk files
     name     |  id  ||    used    |    hard    ||  used   |  hard
--------------|------||------------|------------||---------|---------
   ttrojan_123| 55555||   16.92 GiB|    5.00 TiB||     1134| 30000000
```


## Home file system

- `/home1/<username>`
- Personal directories
  - Default directory when logging in
  - Personal and configuration files
- Quota: 100 GB / 2 M files
- File recovery with daily snapshots and a two-week window
- Use `cd` to quickly change to your home directory


## Project file system

- `/project/<PI_username>_<id>`
- High-performance, parallel I/O file system
- Group directories
  - Project files
  - Share data, software, and other files among group members
- Quota: Variable TB / 30 M files
  - 5 TB default
  - 10 TB free per PI across projects
  - After that, $40/TB/year
- File recovery with daily snapshots and a two-week window


## Project file system (continued)

- Create alias to quickly change to project directories
  - `alias cdp=“cd /project/ttrojan_123”`
  - Add to `~/.bashrc` to automatically set when logging in
- By default, all files have read/write permissions for owner and group members
  - If needed, permissions can be changed to restrict access
  - Use `chmod` or `setfacl`


## Scratch file systems

- `/scratch/<username>`
- `/scratch2/<username>`
- High-performance, parallel I/O file system
- Quota: 10 TB for /scratch and 30 TB for /scratch2
- No file recovery
- Use `cds` and `cds2` commands to quickly change to those directories


## Using /tmp space

- Local `/tmp` directories on compute nodes are restricted to 1 GB of RAM space, shared among jobs running on the same node
- These can become full and are cleared
- Define temporary directories in your /scratch or /project directories
- Redirect temporary files created in the background by programs
- Set the `TMPDIR` environment variable:

```
mkdir /scratch/<username>/tmp
export TMPDIR=/scratch/<username>/tmp
```

- Add the export line to your `~/.bashrc` to automatically set this variable when logging in


## Managing files using the command line

- Organizing files
- Sharing files
- Archiving and compressing files
- Project data management plans
- Could also use GUI SFTP clients (more later)


## Organizing files

- `ls` to list files
- `mkdir` to make new diretories
- `mv` to move or rename files
- `cp` to copy files
- `rm` to delete files


## Checking file disk usage

- Use `du` to check disk usage
- To list the ten largest files or subdirectories in the current directory:

```
du -s * | sort -nr | head -n 10
```

- Remember ZFS compresses files, so for uncompressed file sizes:
  - Use `ls -lh` to list files and view file sizes in current directory
  - Alternatively, use `du -sh --apparent-size *`


## Sharing files

- Project directories are meant for sharing files
- Use `chmod`, `chgrp`, or `setfacl` commands to set file permissions
- For example, to provide read-only access to a subdirectory:

```
chmod 750 /project/ttrojan_123/dir
```

- For fine-grained access control, use `setfacl`
- To check permissions, use `ls -l` or `getfacl`


## Archiving files

- Use `tar` to create an archive file from a set of files or directories:

```
tar -cvf project.tar <dir>
```

- Add the `--remove-files` option to also delete the files
- To list contents of an archive file, use the `-t` option

```
tar -tvf project.tar
```

- To extract an archive file, use the `-x` option:

```
tar -xvf project.tar
```


## Compressing files

- Compression/uncompression tools
  - `gzip` or `pigz` or `xz`
- For large data, can be useful to compress before transferring files (more later)
- To compress with `xz` using multiple cores:

```
xz -v -T4 dataset.csv
```

- To uncompress with `xz` using multiple cores:

```
xz -dv -T4 dataset.csv.xz
```


## Archiving and compressing files

- Can also archive and compress directly with `tar`
- To archive and compress (using `gzip`):

```
tar -czvf project.tar.gz <dir>
```

- To uncompress and extract:

```
tar -xvf project.tar.gz
```


## Long-term archival storage

- [USC Digital Repository](https://repository.usc.edu/)
- [AWS Glacier](https://aws.amazon.com/glacier/)
- Research data repositories (e.g., OSF, Zenodo, Harvard Dataverse, and Dryad)


## Transferring files

- Data transfer nodes
- Many secure file transfer tools
- Choice depends on preference and type of transfer
- Local &rlarr; CARC systems
  - GUI: Cyberduck, FileZilla, WinSCP, MobaXterm, Globus
  - CLI: `sftp`, `scp`, `rsync`, `globus-cli`
- CARC systems &rlarr; Internet
  - CLI: `sftp`, `lftp`, `rclone`, `wget`, `curl`, `aria2c`, `git`


## Data transfer nodes

- Two dedicated, high-speed data transfer nodes
- hpc-transfer1.usc.edu
- hpc-transfer2.usc.edu
- 100 Gbps connectivity
- Note: Compute nodes do not have internet access


## Use cases

| System 1 | System 2 | Example Scenarios | Method |
|---|---|---|---|
| Personal computer | CARC file system for small-medium transfers | When transferring files from a personal computer to your CARC project folder that takes a moderate amount of time  | GUI, CLI |
| Personal computer | CARC file system for large or secure transfers | When transferring files from a personal computer to your CARC project directory that takes a large amount of time or needs to be encrypted | Globus |
| Amazon Web Services (AWS) | Any CARC file system | When transferring files from an AWS server to your CARC project directory | CLI |
| Other HPC center | Any CARC file system | When transferring files from another university or research institution to your CARC project directory | Globus |


## General recommendations

- Only transfer data that is necessary
- Compress large files using `xz` to reduce size of transfer (depending on network speed)
- Archive files using `tar` when transferring large numbers of files
- For large transfers to/from your local computer or other endpoint, use Globus


## Graphical transfer tools

- Drag and drop to transfer between local computer and CARC systems
- A few options for SFTP GUI clients:
  - Cyberduck
  - FileZilla
  - WinSCP
  - MobaXterm
- Can also use Globus web GUI
- [CARC User Guide for Transferring Files using a GUI](https://carc.usc.edu/user-information/user-guides/data-management/transferring-files-gui)


## Command-line transfer tools

- Many options depending on type of transfer

| Scenario | Options |
|---|---|
| Local &rlarr; CARC systems | `sftp`, `scp`, `rsync`, `globus-cli` |
| CARC systems &rlarr; Internet | *File servers*: `sftp`, `lftp` <br> *Downloads*: `wget`, `curl`, `aria2c` <br> *Cloud storage*: `rclone` <br> *Code*: `git` |

- For small-to-medium transfers to/from your local computer, use `sftp` or `rsync`
- For large transfers to/from your local computer or other endpoint, use `globus-cli`
- For backing up and syncing directories, use `rsync`
- For transfers to/from an FTP server, use `lftp`
- For faster internet downloads, use `aria2c`
- For transfers to/from cloud storage, use `rclone`
- [CARC User Guide for Transferring Files using a CLI](https://carc.usc.edu/user-information/user-guides/data-management/transferring-files-command-line)


## sftp example

```
$ sftp ttrojan@hpc-transfer1.usc.edu
Duo two-factor login for ttrojan

Enter a passcode or select one of the following options:

 1. Duo Push to XXX-XXX-5555
 2. Phone call to XXX-XXX-5555
 3. SMS passcodes to XXX-XXX-5555

Passcode or option (1-3): 1
Connected to hpc-transfer1.usc.edu.
sftp> lpwd
Local working directory: /home/tommy
sftp> lcd myimages
sftp> lls
myplot1.jpg myplot2.jpg
sftp> pwd
Remote working directory: /home1/ttrojan
sftp> cd /scratch/ttrojan/images
sftp> put myplot1.jpg myplot.jpg
Uploading myplot1.jpg to /scratch/ttrojan/myplot.jpg
myplot1.jpg                                 100%   10KB   2.4MB/s   00:01    
sftp> get myplot3.jpg myplot3.jpg
Fetching /scratch/ttrojan/myplot3.jpg to myplot3.jpg
/scratch/ttrojan/myplot3.jpg                100%   10KB   2.4MB/s   00:01  
```


## aria2c example

- `aria2c` allows multi-connection and concurrent downloads
- To download with multiple connections:

```
aria2c -x4 <url>
```

- To download multiple files concurrently:

```
aria2c -j4 -i urls.txt
```


## rclone

- `module load rclone`
- Similar to `rsync`, but for transferring files to/from cloud storage services
- Requires configuration
- [CARC User Guide for Rclone](https://carc.usc.edu/user-information/user-guides/data-management/transferring-files-rclone)


## Globus service

- For large, long-running transfers
- Ability to pause and restart transfers and sync directories
- Can be used to share data between external collaborators or data providers
- Web GUI or CLI
- [CARC User Guide for Globus](https://carc.usc.edu/user-information/user-guides/data-management/transferring-files-globus)


## Backing up files

- Local storage (e.g., external drive)
  - Use `rsync` or Globus
- Cloud storage
  - Use `rclone`
- Research data repositories (e.g., OSF, Zenodo, Harvard Dataverse, and Dryad)
- Create aliases or shell scripts to help automate backups


## Additional resources

- [CARC User Guides for Data Management](https://carc.usc.edu/user-information/user-guides/data-management)
- [Submit a support ticket](https://carc.usc.edu/user-information/ticket-submission)
