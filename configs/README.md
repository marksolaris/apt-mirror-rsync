apt-mirror-rsync Mirror Configuration
=====================================

### Mirror config files

The default directory I use to store my mirror configs on the local mirror host is <CODE>/etc/apt/sources.rsync.d</CODE>.<BR>

In this github area are some sample mirror config files, built from the list at <A HREF="https://launchpad.net/ubuntu/+archive">https://launchpad.net/ubuntu/+archivemirrors</A>. Create the <CODE>/etc/apt/sources.rsync.d</CODE> directory and copy across an example file. The name of the file must match the remote hostname you are going to rsync from. When choosing a mirror look for the sites in your country and with the highest advertised speed. Some mirrors will refuse to serve rsync clients connecting from other countries.<BR>

There are over a dozen examples in the config/ directory (<A HREF="https://github.com/marksolaris/apt-mirror-rsync/tree/main/configs">https://github.com/marksolaris/apt-mirror-rsync/tree/main/configs</A>) with a variety of examples on using the <CODE>RELEASES_REMOTE_URL_COMPONENT</CODE> keyword to show you how the location translation works. Remember you only need one working upstream mirror per OS that you want to mirror, don't install all of the config files! We'll use <CODE>mirror.aarnet.edu.au</CODE> to demonstrate here.<BR>

    mkdir -p /etc/apt/sources.rsync.d
    cp configs/mirror.aarnet.edu.au /etc/apt/sources.rsync.d/mirror.aarnet.edu.au
    vi /etc/apt/sources.rsync.d/mirror.aarnet.edu.au

The mirror config file has two purposes, one is to tell <CODE>apt-mirror-rsync</CODE> to use or ignore the whole file, and then to tell the program what remote repositories you want to mirror to your local disks.<BR>

Any line starting with a <CODE>#</CODE> is a comment line and is ignored.<BR>

We use the OS codenames in this README and the mirror config files. It's how the software is shipped by the vendor. To understand which is which, look at <A HREF="https://en.wikipedia.org/wiki/Ubuntu_version_history">https://en.wikipedia.org/wiki/Ubuntu_version_history</A> and <A HREF="https://www.releases.ubuntu.com/">https://www.releases.ubuntu.com/</A><BR>

### Enabling The Mirror

#### <CODE>HOST_STATUS</CODE> keyword<BR>

If this field is set to <CODE>OK</CODE> then the file is used during the mirroring process. Any other value will stop <CODE>apt-mirror-rsync</CODE> from using any content in the file. This means you can put notes in this field like "Currently offline" etc to leave the file in-place in the <CODE>/etc/apt/sources.rsync.d</CODE> directory, but not use it.<BR>

### Defining A Release To Mirror

The rest of the file would contain release definitions for file mirroring, and how the program is meant to handle the copying. A full example to copy all of the <CODE>focal</CODE> Ubuntu release is shown here:<BR>

    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-archive
    RELEASES_LOCAL_DIST_DIR archive.ubuntu.com/ubuntu
    RELEASES focal focal-updates focal-backports focal-security
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH i386 amd64
    RELEASES_KEEP 2

Importantly all of these keywords need to be included in each release section for it to be imported. It's the simplest way to enable complex combinations in the same file to be run against the same remote host.<BR>

#### <CODE>RELEASES_STATUS</CODE> keyword<BR>

As above the <CODE>RELEASES_STATUS</CODE> needs to be set to <CODE>OK</CODE> for that section to be used. You can leave a section in the file and adjust this setting to skip it during processing.<BR>

The next two fields tell the program where the information is on the remote site, and where you want to store it locally.<BR>

#### <CODE>RELEASES_REMOTE_URL_COMPONENT</CODE> keyword<BR>

This field says where the repository files exist on the remote host. To have the filename be called <CODE>mirror.aarnet.edu.au</CODE> and this field set to <CODE>RELEASES_REMOTE_URL_COMPONENT ubuntu-archive</CODE> then <CODE>apt-mirror-rsync</CODE> will open a connection to <CODE>rsync://mirror.aarnet.edu.au/ubuntu-archive/</CODE> and look for the files to copy. Inside any (normal) APT distribution there will be a <CODE>pool/</CODE> directory in that location for the binaries, and a <CODE>dists/</CODE> directory for the metadata files which describe the archive.

#### <CODE>RELEASES_LOCAL_DIST_DIR</CODE> keyword<BR>

This field tells the program where to store the files, (under the <CODE>APT_MIRROR_DIR</CODE> tree), for the release you are mirroring. Almost always you want to keep it the same as the vendors location in the <CODE>/etc/apt/sources.list</CODE> file. Doing so will let you easily tune your clients, or even change your DNS set up, to enable them to update their OS packages. For Ubuntu archive you would set this field to be <CODE>RELEASES_REMOTE_URL_COMPONENT archive.ubuntu.com/ubuntu</CODE> and for the ARM64 releases you'd set it to be <CODE>RELEASES_REMOTE_URL_COMPONENT ports.ubuntu.com/ubuntu-ports</CODE>.<BR>

#### <CODE>RELEASES</CODE> keyword<BR>

This setting controls which releases you want to mirror from the remote host. To have a full archive of <CODE>focal</CODE> you would put <CODE>RELEASES focal focal-updates focal-backports focal-security</CODE> on the line. If you're not sure, copy what's in the <CODE>/etc/apt/sources.list</CODE> file that the OS vendor shipped.<BR>

#### <CODE>RELEASES_REPO</CODE> keyword<BR>

Inside each OS release is a set of standard sections called repositories. The default list for <CODE>focal</CODE> and all Ubuntu releases generally is <CODE>main restricted universe multiverse</CODE> so if you want to have healthy local mirrors keep this set the same. <CODE>RELEASES_REPO main restricted universe multiverse</CODE>.<BR>

#### <CODE>RELEASES_ARCH</CODE> keyword<BR>

Depending on your machines CPU, this field will be one or more of <CODE>amd64</CODE>, <CODE>i386</CODE> <CODE>arm64</CODE>. Usually if you have <CODE>amd64</CODE> then you should also include <CODE>i386</CODE>. The <CODE>arm64</CODE> packages are issued in the <CODE>ubuntu-ports</CODE> bundle so wouldn't be mixed with the other two architechtures.<BR>

#### <CODE>RELEASES_KEEP</CODE> keyword<BR>

This setting controls how many versions of a package are retained on your local disks. I set mine to <CODE>RELEASES_KEEP 2</CODE> so that I have the latest file and a n+1 backup copy. Sometimes though the mirror automatically deletes the older versions so you will have to do maintenance to reclaim disk space from no longer referenced packages.<BR>

### Full Example

Having all of these fields in each section lets you configure very fine-grained actions such as mirroring one specific release/repo/arch combination to a specific directory, if you thought that was a good idea. In normal use one section is enough to mirror all of the metadata files and binary files for an entire OS release such as <CODE>noble</CODE> or <CODE>resolute</CODE>.<BR>

To demonstrate, here's how to mirror four <CODE>amd</CODE> OS release versions, and two <CODE>arm64</CODE> OS releases. Note the change of URL and local directory for the <CODE>arm64</CODE> sections. The one mirror config file can handle multiple scenarios, whatever the remote mirror does to present the archives.<BR>

    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-archive
    RELEASES_LOCAL_DIST_DIR archive.ubuntu.com/ubuntu
    RELEASES focal focal-updates focal-backports focal-security
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH i386 amd64
    RELEASES_KEEP 2

    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-archive
    RELEASES_LOCAL_DIST_DIR archive.ubuntu.com/ubuntu
    RELEASES jammy jammy-updates jammy-backports jammy-security
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH i386 amd64
    RELEASES_KEEP 2

    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-archive
    RELEASES_LOCAL_DIST_DIR archive.ubuntu.com/ubuntu
    RELEASES noble noble-updates noble-backports noble-security
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH i386 amd64
    RELEASES_KEEP 2

    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-archive
    RELEASES_LOCAL_DIST_DIR archive.ubuntu.com/ubuntu
    RELEASES resolute resolute-updates resolute-backports resolute-security
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH i386 amd64 arm64
    RELEASES_KEEP 2

    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-ports
    RELEASES_LOCAL_DIST_DIR ports.ubuntu.com/ubuntu-ports
    RELEASES noble noble-updates noble-backports noble-security
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH arm64
    RELEASES_KEEP 2

    RELEASES_STATUS OK
    RELEASES_REMOTE_URL_COMPONENT ubuntu-ports
    RELEASES_LOCAL_DIST_DIR ports.ubuntu.com/ubuntu-ports
    RELEASES resolute resolute-updates resolute-backports resolute-security
    RELEASES_REPO main restricted universe multiverse
    RELEASES_ARCH arm64
    RELEASES_KEEP 2

