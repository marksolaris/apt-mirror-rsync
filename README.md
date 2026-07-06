apt-mirror-rsync
================
The APT mirror script I want to use<BR>

This script implements mirroring of remote APT mirrors via rsync. It's written entirely in Perl for agility. Inspiration was taken from <CODE>mirror-rsync</CODE> and <CODE>apt-mirror</CODE>. This mirror script is designed to be fast, concise, informative and efficient. Using <CODE>rsync</CODE> is vastly faster than using a <CODE>wget</CODE> based mirror script and only new data is fetched so there isn't wasted bandwidth.<BR>

TL;DR
-----

If you want to get going quickly, these steps will install the script, the script conf file and a mirror site definition file. You must tune those mirror configurations before starting. Then you sanity check the mirror configs and start mirroring. See the Installation section below for more details.<BR>

You will want to observe the progress the first time via <CODE>--progress</CODE>, and usually want to only see things that change on the system so use <CODE>--ignore-zero</CODE> (aka <CODE>-p -z</CODE>). Afterwards view how much data is being used by all the repos in your mirror tree.<BR>

    cp apt-mirror-rsync /usr/local/bin/.
    chmod 755 /usr/local/bin/apt-mirror-rsync

    cp apt-mirror-rsync.conf /etc/apt/.
    vi /etc/apt/apt-mirror-rsync.conf

    mkdir -p /etc/apt/sources.rsync.d
    cp configs/<sitename> /etc/apt/sources.rsync.d/.
    vi /etc/apt/sources.rsync.d/<sitename>

    apt-mirror-rsync --list-mirrors
    apt-mirror-rsync --config
    apt-mirror-rsync --progress --ignore-zero
    apt-mirror-rsync --dist-stat

Installation
------------

##### Script Install

As the <CODE>root</CODE> user copy the <CODE>apt-mirror-rsync</CODE> script to your system's <CODE>bin</CODE> directory, usually <CODE>/usr/local/bin</CODE>.<BR>

    cp apt-mirror-rsync /usr/local/bin/.
    chmod 755 /usr/local/bin/apt-mirror-rsync

##### Main config file

To use <CODE>apt-mirror-rsync</CODE> you will need to set up the script tuning file so it knows where to store the APT repos and how to behave on your system. Copy the <CODE>apt-mirror-rsync.conf</CODE> file to <CODE>/etc/apt</CODE> and edit it for your system. The required values to tune are <CODE>APT_MIRROR_DIR</CODE> and <CODE>DIST_WORK_DIR</CODE>. The other values are generic to most systems. See the file for explanations of each field.<BR>

    cp apt-mirror-rsync.conf /etc/apt/apt-mirror-rsync.conf
    vi /etc/apt/apt-mirror-rsync.conf

##### Pro-tip: Turn off atime on the partition

If you use a SSD for your mirror drive it will get a lot of read and write activity. Reads on a SSD are 'free' but writes incur a gradual wear penalty which over time will degrade your device. Even HDDs will benefit from less activity from not recording file accesses. To maximise the SSD device lifetime turn off <CODE>atime</CODE> on the mountpoint so each access doesn't update the file and directory inodes and thus create unnecessary write activity each time you read something. Configure similar to this in your <CODE>/etc/fstab</CODE> and remount it or reboot:<BR>

    /dev/disk/by-label/SSD /mirror xfs defaults,noatime 0 1

##### Mirror config files

The default remote rsync mirror config directory I use is <CODE>/etc/apt/sources.rsync.d</CODE>. Some sample mirror config files, built from the list at <A HREF="https://launchpad.net/ubuntu/+archive">https://launchpad.net/ubuntu/+archivemirrors</A>, are available in the config/ directory of this github repo. Create the <CODE>/etc/apt/sources.rsync.d</CODE> directory and copy across an example file. The name of the file must match the remote hostname you are going to rsync from. When choosing a mirror look for the sites in your country and with the highest advertised speed. Some mirrors will refuse to serve rsync clients connecting from other countries.<BR>

There are over a dozen examples in the config/ directory with a variety of <CODE>REMOTE_URL_COMPONENT</CODE> examples to show you how the location translation works. Remember you only need one working upstream mirror per OS that you want to mirror, don't install all of the config files! We'll use <CODE>mirror.aarnet.edu.au</CODE> to demonstrate here.<BR>

    mkdir -p /etc/apt/sources.rsync.d
    cp configs/mirror.aarnet.edu.au /etc/apt/sources.rsync.d/mirror.aarnet.edu.au
    vi /etc/apt/sources.rsync.d/mirror.aarnet.edu.au

The mirror config file contains a full description of each field, and in the case of the example <CODE>mirror.aarnet.net.au</CODE> file it's set to mirror all of the focal, jammy, noble and resolute Ubuntu releases. You likely don't want to have all of those OS releases so comment out the ones you don't want. FYI they take nearly 3TB to mirror all of them.<BR>

Inside the file the <CODE>HOST_STATUS</CODE> field lets you leave a mirror definition file in-place in the <CODE>sources.rsync.d</CODE> directory and if the value isn't set to <CODE>OK</CODE> then the file contents will be ignored. It's simpler than having to move the file out of the directory like you need to do with the normal APT repo source files when you don't want them parsed.<BR>

A lot of the remote mirrors on the <A HREF="https://launchpad.net/ubuntu/+archive">https://launchpad.net/ubuntu/+archivemirrors</A> list will keep their files in a different path on the mirror site than where Canonical does. You can use the <CODE>REMOTE_URL_COMPONENT</CODE> and the <CODE>LOCAL_DIST_DIR</CODE> to create the mapping you want, fixing the problem. Keeping the destination for your local files the same as what a normal <CODE>apt-get</CODE> would want (i.e. <CODE>archive.ubuntu.com/ubuntu</CODE> makes it simpler for all of your local clients.

Once you have tuned your mirror config file you should verify that your settings are sane and what you intended.  You can list your configured mirror names with <CODE>apt-mirror-rsync --list-mirrors</CODE>. If that's working then you can list the contents of the mirrors with <CODE>apt-mirror-rsync --config</CODE>. That will show you what the program will attempt to mirror down to your local disk.<BR>

    host:/root root# apt-mirror-rsync -l
    mirror.aarnet.edu.au

    host:/root root# apt-mirror-rsync -c
    mirror.aarnet.edu.au
      status OK
      remote location rsync://mirror.aarnet.edu.au/ubuntu-archive
      local directory /mirror/repos/archive.ubuntu.com/ubuntu
        focal main                     amd64 keep: 2  i386 keep: 2
        focal multiverse               amd64 keep: 2  i386 keep: 2
        focal restricted               amd64 keep: 2  i386 keep: 2
        focal universe                 amd64 keep: 2  i386 keep: 2
        focal-backports main           amd64 keep: 2  i386 keep: 2
        focal-backports multiverse     amd64 keep: 2  i386 keep: 2
        [...]

Running The Script
------------------

The full list of available command options are shown with <CODE>-h</CODE> or <CODE>--help</CODE>:<BR>

    # apt-mirror-rsync --help
    usage: /usr/local/bin/apt-mirror-rsync -h | -c | -d | -f | -l | -p | -r | -s | -t | -v | -z
      -h | --help               : this help
      -c | --config [<mirror>]  : show one/all mirror configs
      -d | --debug              : debug activity
           --debug=files        : debug file activity
           --debug=rsync        : debug rsync status
           --dry-run            : print out what would occur
      -f | --force              : force destructive action
      -l | --list-mirrors       : list configured mirrors
      -p | --progress           : show syncing progress
      -r | --dist-remove <repo> : remove files for a repo
      -s | --dist-stat [<repo>] : show one/all repo stats
      -t | --times              : show date and time stamps
      -v | --verbose            : display more information
      -z | --ignore-zero        : only show when binaries update

If all the setup steps described above are done, your storage disk is ready and the remote mirrors are responding then you can start mirroring. With my preferred interactive args of <CODE>-p -z</CODE> it will show the host you are fetching from, each new Packages file fetched for every binary release you are copying, and then the rsync transfer output is shown like this:<BR>

    host:/root root# apt-mirror-rsync -p -z
    Syncing mirror.aarnet.edu.au
    Fetched new jammy-updates/main/binary-amd64 Packages.
    19963 amd64 pkgs using 216.07G. Need 209 pkgs. Downloading 1007.72MB.
      232.93M  23%    3.10MB/s    0:04:15  xfr#29, to-chk=180/239)

You can see in this example the jammy-updates/main/binary-amd64 repo has 19963 packages configured and is already using 216GB on my drive. The new Packages file lists 209 updated packages with 1007MB to fetch. The <CODE>rsync</CODE> output gives a running progress output of total bytes fetched, a changing percentage, the download speed and how many files out of the 209 have been transferred so far. The percentage changes up and down because the rsync protocol is doing the 19963 files in smaller batches. You'll get used to reading it. The program is using 1024 byte units for calculations because it's not a storage vendor wanting to make their numbers look better. <CODE>rsync</CODE> also does this so it's logical to match that.<BR>

    Fetched new jammy-updates/main/binary-i386 Packages.
    Mirror synced archive.ubuntu.com/ubuntu jammy-updates main
    Fetched new jammy-updates/multiverse/binary-amd64 Packages.
    Fetched new jammy-updates/multiverse/binary-i386 Packages.
    Mirror synced archive.ubuntu.com/ubuntu jammy-updates multiverse

Sometimes like above there will be new Packages files (the repo metadata files) but no new binaries for that particular release-repo-arch combination. The script fetches the Packages files to the temporary work directory and then when things are all fetched to make that repo complete (including all of the architechtures) it merges the information back into the main mirror area so that your APT clients can have the latest configs. The "Mirror synced" indicates that merging step. Some might argue all of the repos in a release need to be merged at the same time instead of gradually like this.<BR>

First Run Example
-----------------

Below is the full output of an initial run of the script where the only files existing on the system are the script, the script configuration file and the mirror config file. Shown is the contents of the mirror configuration file, a display of what it will try to copy to local disk and then apt-mirror-rsync begins building the directory trees and starting the mirroring from the remote site. This example shows the gradual transfer of the focal OS release and all of the repos inside it. At the end a total of all the bytes transferred is given.

In normal use your directory tree will have been created the first time you mirrored an OS release, so you won't see that repeating again.

    host:/root root# grep -v ^# /etc/apt/sources.rsync.d/mirror.aarnet.edu.au
    HOST_STATUS OK
    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-archive
    RELEASES_LOCAL_DIST_DIR archive.ubuntu.com/ubuntu
    RELEASES focal
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH i386 amd64
    RELEASES_KEEP 2

    host:/root root# apt-mirror-rsync --config
    mirror.aarnet.edu.au ubuntu-archive
      remote location rsync://mirror.aarnet.edu.au/ubuntu-archive
      local directory /airgap/repos/mirror/archive.ubuntu.com/ubuntu
        focal main                     amd64 keep: 2 status: OK  i386 keep: 2 status: OK
        focal multiverse               amd64 keep: 2 status: OK  i386 keep: 2 status: OK
        focal restricted               amd64 keep: 2 status: OK  i386 keep: 2 status: OK
        focal universe                 amd64 keep: 2 status: OK  i386 keep: 2 status: OK

    host:/root root# apt-mirror-rsync -p -z
    Syncing mirror.aarnet.edu.au
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/main
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/main
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/main/binary-amd64
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/main/binary-amd64
    Fetched new focal/main/binary-amd64 Packages.
    6090 amd64 pkgs using 0B. Need 6090 pkgs. Downloading 6.27GB.
              6.27G 100%    3.05MB/s    0:35:02 (xfr#6090, to-chk=0/8432)   
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/main/binary-i386
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/main/binary-i386
    Fetched new focal/main/binary-i386 Packages.
    4495 i386 pkgs using 2.41G. Need 2036 pkgs. Downloading 1.91GB.
              1.91G 100%    3.05MB/s    0:10:41 (xfr#2036, to-chk=0/2678)   
    Mirror synced archive.ubuntu.com/ubuntu focal main
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/multiverse
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/multiverse
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/multiverse/binary-amd64
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/multiverse/binary-amd64
    Fetched new focal/multiverse/binary-amd64 Packages.
    813 amd64 pkgs using 0B. Need 813 pkgs. Downloading 5.31GB.
              5.31G  99%    3.05MB/s    0:29:42 (xfr#806, to-chk=0/1324)   
            925.35K 100%    3.07MB/s    0:00:00 (xfr#7, to-chk=0/20) 
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/multiverse/binary-i386
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/multiverse/binary-i386
    Fetched new focal/multiverse/binary-i386 Packages.
    430 i386 pkgs using 3.25G. Need 23 pkgs. Downloading 12.76MB.
             12.76M 100%    3.11MB/s    0:00:04 (xfr#23, to-chk=0/42) 
    Mirror synced archive.ubuntu.com/ubuntu focal multiverse
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/restricted
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/restricted
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/restricted/binary-amd64
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/restricted/binary-amd64
    Fetched new focal/restricted/binary-amd64 Packages.
    143 amd64 pkgs using 0B. Need 143 pkgs. Downloading 437.28MB.
            437.28M 100%    3.05MB/s    0:02:23 (xfr#143, to-chk=0/160) 
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/restricted/binary-i386
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/restricted/binary-i386
    Fetched new focal/restricted/binary-i386 Packages.
    50 i386 pkgs using 53.26K. Need 44 pkgs. Downloading 121.89MB.
            121.89M 100%    3.05MB/s    0:00:39 (xfr#44, to-chk=0/53) 
    Mirror synced archive.ubuntu.com/ubuntu focal restricted
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/universe
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/universe
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/universe/binary-amd64
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/universe/binary-amd64
    Fetched new focal/universe/binary-amd64 Packages.
    53206 amd64 pkgs using 0B. Need 53206 pkgs. Downloading 68.09GB.
             67.83G  99%    3.00MB/s    6:25:43 (xfr#52836, to-chk=0/81751)   
            269.04M 100%    3.06MB/s    0:01:27 (xfr#370, to-chk=0/680)  
    mkdir -p /mirror/dist_temp/archive.ubuntu.com/ubuntu/dists/focal/universe/binary-i386
    mkdir -p /mirror/repos/archive.ubuntu.com/ubuntu/dists/focal/universe/binary-i386
    Fetched new focal/universe/binary-i386 Packages.
    29245 i386 pkgs using 44.33G. Need 2713 pkgs. Downloading 5.14GB.
              5.01G  99%    3.05MB/s    0:28:00 (xfr#2668, to-chk=0/3397)   
            134.13M 100%    3.05MB/s    0:00:43 (xfr#45, to-chk=0/79) 
    Mirror synced archive.ubuntu.com/ubuntu focal universe
    Fetched 87.68G for focal.
    Fetched 87.68G from mirror.aarnet.edu.au.
    Finished all mirroring after 8h 15m 35s

Normal Use
----------

A subsequent run with the same parameters will finish much faster as all of the files have already been downloaded and the directories already exist. This example shows why rsync and smart file processing is so much more efficient. The original <CODE>apt-mirror</CODE>, (<CODE>wget</CODE> based), mirror script would fully download all of the release metadata, even if it already existed, before it would even start copying binaries. This could take 45 minutes or more to download over 1,000 metadata files if you mirror a lot of remote repositories.

    host:/root root# apt-mirror-rsync -p -z
    Syncing mirror.aarnet.edu.au
    Fetched new focal/main/binary-amd64 Packages.
    Fetched new focal/main/binary-i386 Packages.
    Mirror synced archive.ubuntu.com/ubuntu focal main
    Fetched new focal/multiverse/binary-amd64 Packages.
    Fetched new focal/multiverse/binary-i386 Packages.
    Mirror synced archive.ubuntu.com/ubuntu focal multiverse
    Fetched new focal/restricted/binary-amd64 Packages.
    Fetched new focal/restricted/binary-i386 Packages.
    Mirror synced archive.ubuntu.com/ubuntu focal restricted
    Fetched new focal/universe/binary-amd64 Packages.
    Fetched new focal/universe/binary-i386 Packages.
    Mirror synced archive.ubuntu.com/ubuntu focal universe
    Finished all mirroring after 4s 

A normal run of <CODE>apt-mirror-rsync</CODE> without any arguments will simply mirror the files without announcing the transfer activity. This is the command you would add to automated <CODE>cron</CODE> jobs so your archives are updated overnight etc.

    host:/root root# apt-mirror-rsync
    host:/root root#

A double-sync happens when symlinks occur inside the repo, some debs are shared between architechtures etc so they exist as symlinks in several places. The symlink file is copied in the first rsync and then another rsync is done to fetch the packages the symlinks are pointing to.

    Fetched new focal/universe/binary-i386 Packages.
    29245 i386 pkgs using 44.33G. Need 2713 pkgs. Downloading 5.14GB.
              5.01G  99%    3.05MB/s    0:28:00 (xfr#2668, to-chk=0/3397)   
            134.13M 100%    3.05MB/s    0:00:43 (xfr#45, to-chk=0/79) 

