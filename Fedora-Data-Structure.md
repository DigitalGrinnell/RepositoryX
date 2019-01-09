What follows is the Fedora data structure, with proper permissions and ownwership, as it should appear on RepositoryX.

```
vagrant@repositoryx:/usr/local/fedora/data$ ll
total 118
drwxrwxr-x  2 fedora fedora  4096 Jan  9 12:34 ./
drwxr-xr-x 11 fedora fedora  4096 Jan  9 12:54 ../
lrwxrwxrwx  1 fedora fedora     0 Oct  7  2017 activemq-data -> /data2/activemq-data/
drwxr-xr-x  2 fedora fedora 49152 Oct 10  2017 datastreamStore/
drwxrwxr-x  2 fedora fedora    64 Oct 10  2017 fedora-xacml-policies/
drwxrwx---  2 fedora fedora  4096 Oct  6  2017 ._nfs/                         <<-- Not needed?
drwxr-xr-x  2 fedora fedora  4096 Jan  9 12:34 nfsshare_rp7/                  <<-- Not needed? 
drwxr-xr-x  2 fedora fedora 49152 Oct 10  2017 objectStore/
drwx------  2 fedora fedora  4096 Jul 12 12:54 $RECYCLE.BIN/
drwxr-xr-x  2 fedora fedora    64 Oct 10  2017 resourceIndex/
d---rwx---  2 fedora fedora    64 Oct 10  2017 System Volume Information/
```
