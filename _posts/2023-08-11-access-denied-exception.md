---
layout: post
title: "Concurrency puzzle: Access Denied Exception"
date: 2023-08-11
---

It is rather interesting to solve issues with concurrency. Although, it is not
so easy to find the root cause of the problem and you typically feel like a
detective who is trying to find a criminal. So, let's start our investigation.

# Evidences

Evidence #1: the exception stack trace.

[//]: # "@formatter:off"
```java
Caused by: java.nio.file.AccessDeniedException: D:\a\eo\eo\eo-maven-plugin\target\it\fibonacci\target\classes\EOorg\EOeolang\EOpositive_infinity$EOplus$EOplus_rec.class
     at sun.nio.fs.WindowsException.translateToIOException (WindowsException.java:89)
     at sun.nio.fs.WindowsException.rethrowAsIOException (WindowsException.java:103)
     at sun.nio.fs.WindowsException.rethrowAsIOException (WindowsException.java:108)
     at sun.nio.fs.WindowsFileSystemProvider.implDelete (WindowsFileSystemProvider.java:274)
     at sun.nio.fs.AbstractFileSystemProvider.deleteIfExists (AbstractFileSystemProvider.java:110)
     at java.nio.file.Files.deleteIfExists (Files.java:1181)
     at org.eolang.maven.TranspileMojo.cleanUpClasses (TranspileMojo.java:263)
```
[//]: # "@formatter:on"

Evidence #2: the code snippet which runs in many threads.

[//]: # "@formatter:off"
```java
for (final Path binary : unexpected) {
    try {
        Files.deleteIfExists(binary);
    } catch (final IOException cause) {
        throw new IllegalStateException(
            String.format("Can't delete file %s", binary),
            cause
        );
    }
}
```
[//]: # "@formatter:on"

Evidence #3: the code snippet which runs in many threads.
The problem happens time-to-time and it don't reproduce rather often.

# Hypothesis

Let's try think about all the possible reasons of the problem:
java.nio.file.AccessDeniedException when deleting a file can be caused by
several reasons, especially on Windows. Here are some common causes and
solutions:

* **File is in Use**: Windows locks files that are in use, which can cause this
  exception when trying to delete them.
  Solution: Make sure the file isn't in use by another process. If your program
  opened the file earlier, ensure you closed it properly using close() method.

* **File Permissions**: The process might not have the necessary permissions to
  delete the file.
  Solution: Check the file permissions. You may need to run the Java application
  with elevated permissions or change the file permissions to allow deletion.
  Read-only Attribute: Files marked as read-only can't be deleted until the
  attribute is changed.
  Solution: Change the file attribute to allow writing. This can be done
  manually
  via the file's properties or programmatically using Java.
  Directory Permissions: Even if you have permissions on the file, if you don't
  have the necessary permissions on the directory that contains the file, you
  might face this issue.
  Solution: Ensure your Java application has permissions to modify the directory
  contents.
  Anti-virus or Backup Software: Sometimes, these software can lock files
  temporarily.
  Solution: Temporarily disable the software and try the operation again. Make
  sure to re-enable your security software afterward.

* **Symbolic Links**: Sometimes, the actual file/directory you're
  trying to delete isn't what you think it is, due to symbolic links or junction
  points.
  Solution: Ensure you're deleting the correct file or directory.
  Try a Different Deletion Method: The Files.deleteIfExists() method is a
  convenient way to delete a file, but if it's causing issues, you could try
  using
  the old java.io.File.delete() method to see if it behaves differently.
  Debugging:
  To get more details about the exception, you can print the exception message
  and
  stack trace:
  [//]: # "@formatter:off"
  ```java
  try {
    Files.deleteIfExists(binary);
  } catch (IOException e) {
    e.printStackTrace();
  }
  ```
  [//]: # "@formatter:on"

  This might give more insights into the exact nature of the problem.
  Java Version & Bugs: It's worth checking if there's a known issue related to
  file deletion with the Java version you're using, especially if it's not the
  latest version.

* **Incorrect Path**: Double-check that the Path object binary is correctly
  pointing to the intended file and not mistakenly pointing to a directory or
  another protected system file.

# Investigation

...

And voila, it is. We are lucky, to be honest. It is rare case when the problem
can be found so easily.

# Solution

Ok, what about solution? We have several options:

1. We can use exceptions to catch the problem and ignore
   it. `AccessDeniedException`, `NoSuchFileException`.
2. Locking. Static, non-static, read-write, read-only.
3. Collect all the files which should be deleted and delete them in the end of
   the process.

I choosed the second option. Why?
