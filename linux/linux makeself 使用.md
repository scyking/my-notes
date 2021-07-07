# linux makeself 使用

标签（空格分隔）： linux

---

[toc]

## 概述
> 可以把特定目录下的文件打包成为一个shell脚本类似的文件，并且能够像shell脚本一样执行。

### [源码地址](https://github.com/megastep/makeself)

## 安装

```
yum install -y makeself
```

## 使用

```
Usage: /usr/bin/makeself [params] archive_dir file_name label startup_script [args]

params can be one or more of the following :
    --version | -v  : Print out Makeself version number and exit
    --help | -h     : Print out this help message
    --quiet | -q    : Do not print any messages other than errors.
    --gzip          : Compress using gzip (default if detected)
    --bzip2         : Compress using bzip2 instead of gzip
    --pbzip2        : Compress using pbzip2 instead of gzip
    --xz            : Compress using xz instead of gzip
    --compress      : Compress using the UNIX 'compress' command
    --complevel lvl : Compression level for gzip xz bzip2 and pbzip2 (default 9)
    --base64        : Instead of compressing, encode the data using base64
    --nocomp        : Do not compress the data
    --notemp        : The archive will create archive_dir in the
                      current directory and uncompress in ./archive_dir
    --copy          : Upon extraction, the archive will first copy itself to
                      a temporary directory
    --append        : Append more files to an existing Makeself archive
                      The label and startup scripts will then be ignored
    --target dir    : Extract directly to a target directory
                      directory path can be either absolute or relative
    --current       : Files will be extracted to the current directory
                      Both --current and --target imply --notemp
    --tar-extra opt : Append more options to the tar command line
    --nomd5         : Don't calculate an MD5 for archive
    --nocrc         : Don't calculate a CRC for archive
    --header file   : Specify location of the header script
    --follow        : Follow the symlinks in the archive
    --noprogress    : Do not show the progress during the decompression
    --nox11         : Disable automatic spawn of a xterm
    --nowait        : Do not wait for user input after executing embedded
                      program from an xterm
    --lsm file      : LSM file describing the package
    --license file  : Append a license file

Do not forget to give a fully qualified startup script name
(i.e. with a ./ prefix if inside the archive).
```



