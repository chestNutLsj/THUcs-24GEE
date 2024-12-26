## Intro

If a resource is duplicated but not modified, it is not necessary to create a new resource; the resource can be shared between the copy and the original. Modifications must still create a copy, hence the technique: the copy operation is deferred until the first write. By sharing resources in this way, it is possible to significantly reduce the resource consumption of unmodified copies, while adding a small overhead to resource-modifying operations.

## In virtual memory management

Copy-on-write finds its main use in sharing the virtual memory of operating system processes, in the implementation of the fork system call. Typically, the process does not modify any memory and immediately executes a new process, replacing the address space entirely. Thus, it would be wasteful to copy all of the process's memory during a fork, and instead the copy-on-write technique is used.

Copy-on-write can be implemented efficiently using the page table by marking certain pages of memory as read-only and keeping a count of the number of references to the page. When data is written to these pages, the operating-system kernel intercepts the write attempt and allocates a new physical page, initialized with the copy-on-write data, although the allocation can be skipped if there is only one reference. The kernel then updates the page table with the new (writable) page, decrements the number of references, and performs the write. The new allocation ensures that a change in the memory of one process is not visible in another's.

The copy-on-write technique can be extended to support efficient memory allocation by having a page of physical memory filled with zeros. When the memory is allocated, all the pages returned refer to the page of zeros and are all marked copy-on-write. This way, physical memory is not allocated for the process until data is written, allowing processes to reserve more virtual memory than physical memory and use memory sparsely, at the risk of running out of virtual address space. The combined algorithm is similar to demand paging.

Copy-on-write pages are also used in the Linux kernel's same-page merging feature.

## In software

COW is also used in library, application and system code.
### Examples

The string class provided by the C++ standard library was specifically designed to allow copy-on-write implementations in the initial C++98 standard,  but not in the newer C++11 standard: 

```
std::string x("Hello");

std::string y = x;  // x and y use the same buffer.

y += ", World!";    // Now y uses a different buffer; x still uses the same old buffer.
```

In the PHP programming language, all types except references are implemented as copy-on-write. For example, strings and arrays are passed by reference, but when modified, they are duplicated if they have non-zero reference counts. This allows them to act as value types without the performance problems of copying on assignment or making them immutable. 

In the Qt framework, many types are copy-on-write ("implicitly shared" in Qt's terms). Qt uses atomic [[30-并发#compare&swap 指令|compare-and-swap]] operations to increment or decrement the internal reference counter. Since the copies are cheap, Qt types can often be safely used by multiple threads without the need of locking mechanisms such as mutexes. The benefits of COW are thus valid in both single- and multithreaded systems.

## In computer storage

COW may also be used as the underlying mechanism for snapshots, such as those provided by logical volume management, file systems such as Btrfs and ZFS, and database servers such as Microsoft SQL Server. Typically, the snapshots store only the modified data, and are stored close to the original, so they are only a weak form of incremental backup and cannot substitute for a full backup.
