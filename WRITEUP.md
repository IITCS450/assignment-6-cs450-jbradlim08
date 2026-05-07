# Write-Up

For this assignment, I added symbolic link support to xv6 by introducing a new inode type, `T_SYMLINK`, and a new system call, `symlink(target, linkpath)`. A symbolic link is stored as its own inode, and the target path is written into that inode's data blocks. This lets the file system treat a symlink as a normal file-system object while still preserving the pathname it should resolve to.

To support this, I updated the file-system type definitions and syscall plumbing so user programs can create symlinks. The main implementation lives in `sys_symlink()` in `xv6/sysfile.c`, where the kernel creates a `T_SYMLINK` inode and writes the target string into it. I also added the syscall number and user-facing wrapper in `xv6/syscall.h`, `xv6/syscall.c`, `xv6/usys.S`, and `xv6/user.h`.

I modified `open()` so that when a pathname refers to a symbolic link, xv6 reads the stored target path and follows it until it reaches a non-symlink inode. To avoid infinite loops, the implementation stops after 10 levels of traversal and returns failure if the path still resolves to a symlink at that point. This handles cyclic links like `a -> b` and `b -> a` safely.

For testing, I added `xv6/testsymlink.c` and included it in the build through `xv6/Makefile`. The test checks that creating a symlink works, opening and reading through the symlink returns the original file contents, and a symlink loop correctly causes `open()` to fail. I also verified that the `_testsymlink` build target compiles successfully.
