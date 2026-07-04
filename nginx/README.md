Mirrors
-------


To utilise your local repository mirrors on all of your client machines you will need to set up a webserver (without HTTPS) and export the directories. Depending how you configure your environment you can leave all the APT and YUM clients the same, or manually adjust each <CODE>/etc/apt/sources.list</CODE> and <CODE>/etc/yum.repos.d</CODE> file.<BR>

This is a client which is configured to use a local mirror site. You just insert the new hostname in front of the original one:<BR>

    % cat /etc/apt/sources.list
    deb http://mirror-apt.example.com/archive.ubuntu.com/ubuntu noble main restricted universe multiverse
    deb http://mirror-apt.example.com/archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse
    deb http://mirror-apt.example.com/archive.ubuntu.com/ubuntu noble-backports main restricted universe multiverse
    deb http://mirror-apt.example.com/archive.ubuntu.com/ubuntu noble-security main restricted universe multiverse

The gold standard is to have a DNS alias for the upstream mirror (e.g. <CODE>archive.ubuntu.com</CODE>) which points to your local webserver. Then you can leave all your clients configurations as shipped. It also stops them from going out to the internet inadvertently. Only the mirroring host needs to know the real remote IP number(s).<BR>

The <CODE>nginx</CODE> example files in this directory will enable a HTTP endpoint each for your APT, YUM and ISO files. Your Redhat/Centos machines use the YUM webserver, your APT hosts use the APT site etc.<BR>

My local directory tree keeps the package system types apart, to lower the admin burden. A seperate webserver serves each of these directories:<BR>

    /mirror/isos
    /mirror/repos
    /mirror/yum

Using the <CODE>apt-mirror-rsync</CODE> and the example configuration files you can have local copies of all of your chosen sites. It dramatically speeds up system updates and removes repeat downloading of the same vendor updates to each client over the internet. It will increase your security as you will update more often since the time involved is much less.<BR>

    host:/mirror/repos root# du -hs *
    52K     archive.canonical.com
    2.5T    archive.ubuntu.com
    3.3G    brave-browser-apt-release.s3.brave.com
    55M     cli.github.com
    120G    deb.debian.org
    88G     download.proxmox.com
    5.1G    esm.ubuntu.com
    21G     packages.mozilla.org
    1007G   ports.ubuntu.com
    162M    ppa.launchpadcontent.net
    21G     security.debian.org
    189M    security.ubuntu.com

If some vendors don't yet provide the ability to use <CODE>rsync</CODE> to mirror then you can still use the original <CODE>apt-mirror</CODE> tool to fetch the remote repositories via its underlying <CODE>wget</CODE> method. The remote files can still be mirrored to the same location that <CODE>nginx</CODE> shares out.<BR>

Some vendors have claimed they don't provide <CODE>rsync</CODE> mirrors because the process for mirroring is "complicated" <A HREF="https://forum.proxmox.com/threads/will-proxmox-provide-rsync-access-for-the-public-repository-download-proxmox-com.106742/post-832232">&#x1F517;</A>. Feel free to point them at this tool to help convince them it's time for agility and efficiency. And specifically if they put their debs in the repo pool directory instead of the dist directory that would vastly uncomplicate their workload. It enables the same binary syncing before the metadata files are presented to the clients.<BR>
  
  
