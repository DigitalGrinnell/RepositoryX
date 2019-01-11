What follows is the *Fedora* data structure, with proper permissions and ownwership, as it should appear on *DGDocker1*.

The storage landscape of *DGDocker1*, as of 11-Jan-2019, looks like this:
```
[islandora@dgdocker1 ISLE]$ df -kh
Filesystem                   Size  Used Avail Use% Mounted on
/dev/mapper/centos-root       45G   24G   21G  54% /
devtmpfs                      12G     0   12G   0% /dev
tmpfs                         12G     0   12G   0% /dev/shm
tmpfs                         12G   13M   12G   1% /run
tmpfs                         12G     0   12G   0% /sys/fs/cgroup
/dev/sda1                    497M  270M  228M  55% /boot
storage2:/nfsshare_dgdocker  1.3T  1.1T  209G  84% /data
tmpfs                        2.4G     0  2.4G   0% /run/user/0
tmpfs                        2.4G     0  2.4G   0% /run/user/10000
```

The `storage2:/nfsshare_dgdocker` share, mounted as `/data/` is where our *Fedora* objects live, in the `datastreamStore/`, `objectStore/`, and `resourceIndex/` subdirectories.  *ActiveMQ* and *Fedora XACML* elements of *Fedora* are held in volumes managed solely by *Docker* and not explicitly mounted.

The structure of `/data` is as follows:
```
[islandora@dgdocker1 data]$ ll
total 99
drwxr-xr-x. 2 mcfatem  mcfatem  49152 Oct 10  2017 datastreamStore
drwxr-xr-x. 2 mcfatem  mcfatem  49152 Oct 10  2017 objectStore
-rw-rw-r--. 1 mcfatem  mcfatem    193 Nov 17 08:53 README.md
drwxr-xr-x. 2 mcfatem  mcfatem     64 Dec  4 14:28 resourceIndex
drwxr-xr-x. 2 connerms connerms    64 Jul 10  2017 solr
-rw-r--r--. 1 mcfatem  mcfatem      6 Jul  5  2017 test2.txt
-rw-r--r--. 1 mcfatem  mcfatem      8 Jul  3  2017 test.txt
```
Note that the ownership, the owner:group specifications, of directories in `/data` is largely inconsequential since all processes running in the `isle-fedora-dg` container are owned by `root`!
