---
layout: post
category: linux
title: Understand NFS
---
## Preface
最近工作当中经常使用到NFS辅助做些事情。然而，一次意外却让我对这个不起眼的东西产生了浓厚的兴趣。
我发现当Client的IP发生变化的时候我的NFS挂载点居然没有断开。我觉得这真是神奇，于是便有了这篇文章。
以下文章摘自NMU的一篇[文章](http://euclid.nmu.edu/~rappleto/Classes/CS442/Notes/nfs.html)，通俗易懂，所以就将其搬运了过来。(Don't tell me you don't know english)

## NFS(Network File System) Glance
LAN-users might want to access files on more disks than the one that is physically attached to the local computer.  NFS is one tool used to access disks located on remote computers.

NFS provides transparent file access for clients to files and filesystems on a server.  This means that a user need not know if the file (s)he wants to access is physically located on his/her local computer or not, as long as the filesystem containing the file is mounted to his/her local filesystem.

The figure below shows how files are accessed in a NFS-system.
![figure](http://euclid.nmu.edu/~rappleto/Classes/CS442/Notes/nfs.gif)  
When a user-process wants to access a local file the client kernel uses a local file access, but when a user-process wants to access a remote file, the kernel sends RPC-calls to the NFS server over TCP/IP or (more likely) UDP/IP. The NFS server then accesses its local file, and sends the result back to the NFS client.

NFS is part of the operating system kernel. Nothing special need be done by the client program to use NFS . The kernel detects that the file is remote and automatically generates the RPC calls to access the file.

The RFC is at  http://www.faqs.org/rfcs/rfc1813.html.

## NFS File Handles
File hndles are opaque descriptors for each file and/or directory on the system.  To open, write, append, etc. a file you must have a file handle.  The lookup call returns a file handle given a file name.  It's the most frequent call in an NFS system.

## NFS mount protocol
The client must use the NFS mount protocol to mount a server's filesystem, before the client can access files on that filesystem. This is often done when the client is bootstrapped. The end result is for the client to obtain a file handle for the server's filesystem.

Mounting is done by client RPC calls to the mount deamon, which runs on the server. The mount deamon replies with the file handle for the given filesystem. The file handle is stored in the NFS client code, and from this point on any references by user procedures to files on that server's filesystem will use that file handle as the starting point.   Without this handle, clients cannot access files on the server.

Permissions are checked during mount time.

## NFS procedures
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
17. ACCESS.  Would a file operation be permitted.  Helps with client caching.
18. MKNOD.  Makes a device special file.
19. FSINFO.  Returns information about the server's capabilities.  Can it do mknod.  Prefered transfer size.   etc.
20. READDIRPLUS.  Does a multi-component path lookup in one swoop.
21. COMMIT.  Force the writing of asynchronus data.

(17 - 21) are included in V3.

NB! The procedure names actually starts with the prefix NFSPROC_.

## NFS V3.0
Classic NFS is NFS v2.0.  The latest version is NFS v3.0.  There are a couple of improvements, but the protocol is essencially the same.

- Support for very long files. (files more than 2^32 bytes long)
- Ability to look up more than one pathname component in a single lookup() call.
- Every operation returns the modified metadata for the file, which reduces the getattr calls.
- Added more support for server side caching of written data.

## NFS v4
Basically, NFSv4 moted to everything that NFSv2 stands for. It is stateful, not idempotent, and complex. The big things they added were

- An open call
- Compound statements: Run the list, stop at the first failure, return all the results
- Transient file handles
- Security, with encryption and such
- Built in locking, with timeouts, and revocations

## Server Crashes
The NFS protocol is stateless.  The NFS server does not need to keep any record of who has an NFS file system mounted.  When a server crashes, all NFS clients can just wait with their request until the server comes back up.  There is no need to remount the server's file system.

## Statelessness and Idempotency
Every NFS operation is designed to be stateless and hopefully idempotent.  That way, if the server crashes in the middle of an operation, the client can retry the operation without worrying if the operation has already happened.  For example, if the client sends a link() command, but the server crashes before getting a reply, the client has no way of knowing if the operations succeeded or not.  All the client can conclude for certain is that the server crashed before the reply could be sent.  Retrying the link() command after the server comes back up will ensure that the link is created, even though the first operation might have already made the link.

There is no open operation.  That would be stateful.

Because of this every read and write request must include the offset to be used and the authorization credentials of the actor.  Permissions are checked on every read/write, not on every open.

Note that because of this, there is no append() operations.  There is no way to make append() idempotent.

The create operation with the exclusive modifier is an example of a violation of idempotency.  It is included in the NFS protocol anyway.

## Server Caching
Someone, either the server or the client, must keep a copy of all data written by an application untill that data hits an actual hard drive.  Otherwise, and unfortunate system crash could cause data to be lost.

The NFS standard says that the NFS server cannot acknowedge a write() operation until that data has hit stable storage.  This safety feature greatly slows NFS writes.  Sometimes people enable a special feature of NFS servers such that the server acknowdges the write before the data hits stable storage, speeding the server at the cost of some safety.  Othes spend money on NVRAM for their write cache, giving safety and speed at the cost of money.

## Client Caching
NFS clients typically cache data for performance reasons.  However, if client 'a' has a cached copy of some data, and client 'b' performs a write changing that same data, there is no way for client 'a' to be notified of the changes.  There are no **callbacks** in NFS.  In practice, typical usage has each client modifing only it's own data, and such conflicts are rare (but can happen).

## Weird Things
An NFS server can implement any file name limitations it wants.  For example, it can forbid any filename with a '/' in the name.  It can also forbid any filename with a curvy letter in it.  Therefore one cannot assume any particular filename will work on an NFS server.  This is NOT a problem in practice.

There is no NFS way of storing extended file attributes (thumbnails, etc.).  Sometimes clients make a special subdirectory to do this (i.e.  something like .Appledouble).

In a typical UNIX filesystem, one can do the following things in this order:

1. Create a file
2. Delete the file.
3. Read and/or write to the file.
4. Close the file (or crash) and watch the space be free'd.

This is not guarenteed to work in NFS.  One should rename instead of delete in step two, and have a server side deamon clean up sometime around step 4.

One can page and swap to an NFS server, and this is often faster than paging and swapping to the local disk.

## Copying Files
There is no NFS file copy operation.  To copy a file from one place to another the data must go through the client, even if both locations are on the same server.

## Pros 'n' Cons
### Pros
- The user need not know where the file is physically located.   From the user point of view all files appear local, even the ones on the NFS server.
- If the user wants to access a file (s)he know is remote, (s)he need not get the whole file, but only the part of the file (s)he wants to access.   This is quite different that HTTP or FTP, and can improve performance in certain cases.
- Connection management. An NFS client can download multiple files over a single TCP connection. HTTP and FTP require a new TCP connection for every file. NFS reduces connection overhead.
- Concurrency. NFS clients can issue multiple, concurrent requests to an NFS server. The effect is better utilization of server and network resources, and better performance as perceived by the end-user. HTTP servers cannot support out-of-order HTTP requests, even  when batched over a single TCP connection.   HTTP/1.1 also knows this trick.
- Fault Tolerance. NFS is well-known for its fault tolerance in the face of network and server failures. While interrupted HTTP or FTP file downloads must be resumed from the beginning, and NFS client can  resume a download from where it left off.   The NFS client need not even make a new connection, but can just retry the old request until the request succeeds.
- Performance and Scalability. NFS servers currently handle many times the I/O load of an HTTP server on the same hardware. NFS servers are highly integrated with the operating system, tuned for  maximum system performance, and are easy to administer.
- Installed Base. NFS servers are already widely deployed within Intranets and are responsible for much of the network traffic on those nets. NFS servers already provide access to the bulk of file data within  an organization. Well, Novell does compete well here, actually.  There is generaly an NFS client and an NFS server available for every platform you've ever heard of, often free.
- There are no passwords. Ever.

### Cons
- NFS must trust the client to know the name of the person accessing any particular file. If the client lies, the servers is fooled.
- On large networks, where the lines are bad, NFS provides slow access to files/disks, because NFS is dependent on PRC andTCP/UDP/IP.
- The data is sent unencrypted.  Any net snooper can read it.   HUGE security hole.  There are versions of NFS that use encryption, but they incompataible with stanard NFS.
- There is nothing like a cgi-bin generally possible with NFS, and no accounting like http and FTP have.
