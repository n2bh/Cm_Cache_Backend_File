Cm_Cache_Backend_File
=====================

The stock Zend_Cache_Backend_File backend has extremely poor performance for
cleaning by tags making it become unusable as the number of cached items
increases. This backend makes many changes resulting in a huge performance boost,
especially for tag cleaning.

This cache backend works by indexing tags in files so that tag operations
do not require a full scan of every cache file. The ids are written to the
tag files in append-only mode and only when files exceed 4k and only randomly
are the tag files compacted to prevent endless growth in edge cases.

The metadata and the cache record are stored in the same file rather than separate
files resulting in fewer inodes and fewer file stat/read/write/lock/unlink operations.
Also, the original hashed directory structure had very poor distribution due to
the adler32 hashing algorithm and prefixes. The multi-level nested directories
have been dropped in favor of single-level nesting made from multiple characters.

Is the improvement substantial? Definitely. Tag cleaning is literally thousands of
times faster, loading is twice as fast, and saving is slightly slower dependent on
the number of tags being saved.

Test it for yourself with the [Magento Cache Benchmark](https://github.com/colinmollenhour/magento-cache-benchmark).

Installation
------------

1. Clone module with [modman](https://github.com/colinmollenhour/modman)
2. Edit `app/etc/local.xml` changing `global/cache/backend` to `Cm_Cache_Backend_File`
3. Delete all contents of the cache directory

Example Configuration
---------------------

By default, Cm_Cache_Backend_File is configured *not* to use chmod to set file permissions. The
proper way to do file permissions is to respect the umask and not set any permissions. This way
the file permissions can be properly inherited such as when the setgid bit is used on the parent
directory. To improve security the umask should be properly set. In Magento the umask is set in
index.php as '0' which means no restrictions. To make files and directories no longer public
change this to umask(0007).

```
<config>
    <global>
        <cache>
            <backend>Cm_Cache_Backend_File</backend>
        </cache>
        ...
    </global>
    ...
</config>
```

If umasks are too complicated and you prefer the sub-optimal (less-secure, needless system calls)
approach you can enable the old chmod usage like so:

```
<config>
    <global>
        <cache>
            <backend>Cm_Cache_Backend_File</backend>
            <backend_options>
                <use_chmod>1</use_chmod>
                <directory_mode>0777</directory_mode>
                <file_mode>0666</file_mode>
            </backend_options>
        </cache>
        ...
    </global>
    ...
</config>
```

Special Thanks
--------------

Thanks to Vinai Kopp for the inspiring this backend with your symlink rendition!

```
@copyright  Copyright (c) 2012 Colin Mollenhour (http://colin.mollenhour.com)
This project is licensed under the "New BSD" license (see source).
```
