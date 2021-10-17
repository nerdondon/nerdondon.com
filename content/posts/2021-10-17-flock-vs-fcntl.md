---
title: "[Notes]: `flock` vs fcntl`"
date: 2021-10-17T00:00:00-08:00
description: "Notes on *nix file locking"
tags: [notes, file-system, unix, posix, locking]
logoText: "cat flock-vs-fcntl.md"
---

## Prelude

In the course of developing RainDB, there was cause to get whole-file locks (as opposed to byte
ranges within a file). Here are some light notes on two Unix system calls for doing this: `flock`
and `fcntl`. Mostly from an [awesome post](https://apenwarr.ca/log/20101213) by
[@apenwarr](https://twitter.com/apenwarr)

## Notes

- `flock` locks an entire file at a time and is not standardized by POSIX

- Both `flock` and `fcntl` have shared locks and exclusive locks e.g. they are readers-writer locks

- `fcntl` is a POSIX standardized lock that allows locking of specific byte ranges

  - These byte ranges are not actually enforced e.g. you can lock bytes past the end of a file

  - Per @apenwarr:
    > You could lock bytes 9-12 and that might mean "records 9-12" if you want, even if records have
    > variable length. The meaning of a fcntl() byte range is up to whoever defines the file format
    > you're locking.

- Both `flock` and `fcntl` are advisory locks. This means that nothing prevents another process from
  reading a or writing to a locked file without first acquiring a lock. A well-behaved program will
  attempt to acquire a lock first.

- `fcntl` locks belong to a (pid, inode) pair so, if you close any file descriptor referring to the
  inode, all of the locks you have on the inode will be released. This is particularly insidious,
  because a library function that you use may open a file you have a lock on and close it as part of
  its implmentation. This would release your locks and you wouldn't really know it unless you dig
  into the implementation. Yikes.

  - This quote from @apenwarr speaks what we all think:

    > That behaviour is certifiably insane, and there's no possible justification for why it should
    > work that way. But it's how it's always worked, and POSIX standardized it, and now you're
    > stuck with it.

  - Aside, what's an inode?

    - It's a data structure that describes a file-system object (e.g. file or directory). It stores
      information like file ownership, file size, access timestamps, and etc. (paraphrased from
      [Wikipedia](https://en.wikipedia.org/wiki/Inode))

  - An API for `fcntl` (released in 2015) called "open file description locks" provides more sane
    behavior

    - `libc` docs:
      > Open file description locks are useful in the same sorts of situations as process-associated
      > locks. They can also be used to synchronize file access between threads within the same
      > process by having each thread perform its own open of the file, to obtain its own open file
      > description.
      >
      > Because open file description locks are automatically freed only upon closing the last file
      > descriptor that refers to the open file description, this locking mechanism avoids the
      > possibility that locks are inadvertently released due to a library routine opening and
      > closing a file without the application being aware.

### Thread safety

- According to @apenwarr:

  > Supposedly, flock() locks persist across fork() and (unlike fcntl locks, see below) won't
  > disappear if you close unrelated files. HOWEVER, you can't depend on this, because some
  > systems - notably earlier versions of Linux - emulated flock() using fcntl(), which has totally
  > different semantics. If you fork() or if you open the same file more than once, you should
  > assume the results with flock() are undefined.

  - This is particularly applicable in understanding usage of the
    [`fs2` crate](https://crates.io/crates/fs2) in Rust to acquire locks on files

  - The `fs2` crate does a whole-file lock with `flock` but, for systems that do not have `flock`
    (e.g. Oracle Solaris), it will
    [emulate `flock` using `fcntl`](https://github.com/danburkert/fs2-rs/blob/master/src/unix.rs#L56-L92)

- With `flock`, upgrading from a shared lock to an exclusive lock is racy because you have to
  release the shared lock first

- It seems like `fcntl` can atomically upgrade a shared lock to an exclusive lock

## Useful references

- [@apenwarr - Everything you never wanted to know about file locking](https://apenwarr.ca/log/20101213)
