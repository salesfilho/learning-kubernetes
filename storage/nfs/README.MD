## Configure NFS Server

Let's configure a NFS Server to be used as Storage Area by Kubernetes. In this lab, let's setup the NFS Server in master01 node server.

## Install

At a terminal prompt enter the following command to install the NFS Server:

```
sudo apt install nfs-kernel-server
```

To start the NFS server, you can run the following command at a terminal prompt:

```
sudo systemctl start nfs-kernel-server.service
```

### Configuration

You can configure the directories to be exported by adding them to the /etc/exports file. For example:

```
/storage  master01(rw,async,no_subtree_check,no_root_squash)  master02(rw,async,no_subtree_check,no_root_squash) dcn01(rw,async,no_subtree_check,no_root_squash)
```
