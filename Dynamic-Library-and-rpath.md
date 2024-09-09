


https://amir.rachum.com/shared-libraries/


Order of search 

1. Directories listed in the executable’s rpath.
2. Directories in the LD_LIBRARY_PATH environment variable, which contains colon-separated list of directories (e.g., /path/to/libdir:/another/path)
3. Directories listed in the executable’s runpath.
4. The list of directories in the file /etc/ld.so.conf. This file can include other files, but it is basically a list of directories - one per line.
5. Default system libraries - usually /lib and /usr/lib (skipped if compiled with -z nodefaultlib).