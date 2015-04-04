---
layout: post
category: linux
title: Understand NFS
---
### Preface
最近工作当中经常使用到NFS辅助做些事情。然而，一次意外却让我对这个不起眼的东西产生了浓厚的兴趣。
我发现当Client的IP发生变化的时候我的NFS挂载点居然没有断开。我觉得这真是神奇，于是便有了这篇文章。
以下文章摘自NMU的一篇[文章](http://euclid.nmu.edu/~rappleto/Classes/CS442/Notes/nfs.html)，通俗易懂，所以就将其搬运了过来。(Don't tell me you don't know english)

### NFS(Network File System) Glance
LAN-users might want to access files on more disks than the one that is physically attached to the local computer.  NFS is one tool used to access disks located on remote computers.

NFS provides transparent file access for clients to files and filesystems on a server.  This means that a user need not know if the file (s)he wants to access is physically located on his/her local computer or not, as long as the filesystem containing the file is mounted to his/her local filesystem.

The figure below shows how files are accessed in a NFS-system.

![figure](http://euclid.nmu.edu/~rappleto/Classes/CS442/Notes/nfs.gif)

When a user-process wants to access a local file the client kernel uses a local file access, but when a user-process wants to access a remote file, the kernel sends RPC-calls to the NFS server over TCP/IP or (more likely) UDP/IP. The NFS server then accesses its local file, and sends the result back to the NFS client.

NFS is part of the operating system kernel. Nothing special need be done by the client program to use NFS . The kernel detects that the file is remote and automatically generates the RPC calls to access the file.

The RFC is at  http://www.faqs.org/rfcs/rfc1813.html.

### NFS File Handles
File hndles are opaque descriptors for each file and/or directory on the system.  To open, write, append, etc. a file you must have a file handle.  The lookup call returns a file handle given a file name.  It's the most frequent call in an NFS system.

### NFS mount protocol
The client must use the NFS mount protocol to mount a server's filesystem, before the client can access files on that filesystem. This is often done when the client is bootstrapped. The end result is for the client to obtain a file handle for the server's filesystem.

Mounting is done by client RPC calls to the mount deamon, which runs on the server. The mount deamon replies with the file handle for the given filesystem. The file handle is stored in the NFS client code, and from this point on any references by user procedures to files on that server's filesystem will use that file handle as the starting point.   Without this handle, clients cannot access files on the server.

Permissions are checked during mount time.

### NFS procedures
The NFS server provides 15 procedures:

  1. GETATTR. Return the attributes of a file.
  2. SETATTR. Set the attributes of a file.
  3. STATFS. Return the status of a filesystem. Like 'df'.
  4. LOOKUP. Opens a file and returns the file handle and attributes to the client.
  5. READ. Read from a file. (Up to 8192 bytes in v2, more in v3.)
  6. WRITE. Write to a file.
  7. CREATE. Create a file.
  8. REMOVE. Delete a file.
  9. RENAME. Rename a file.
 10. LINK. Make a hard link to a file.
 11. SYMLINK. Create a symbolic link to a file.
 12. READLINK. Read a symbolic link to a file. (Returning the name of the file pointed to.)
 13. MKDIR. Create a directory.
 14. RMDIR. Remove a directory.
 15. READDIR. Read a directory.
 16. NULL.  Does nothing.  Used for testing and timing.
