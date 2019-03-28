# ffmpeg-cxc-build
Fast cross-compile ffmpeg for Windows with MinGW on Linux.

**Enabled Features**

  aom fdk-aac mp3lame opus theora vorbis vpx x264 x265 nvenc nvdec sofalizer fontconfig ass/ssa (subtitles)

**Package Requirements**

Debian/Ubuntu/Mint:

  sudo apt-get -y install autoconf automake build-essential cmake git-core gperf g++-mingw-w64 libtool mercurial nasm pkg-config python-lxml subversion texinfo yasm wget
  
Fedora:
  sudo yum install make autogen automake cmake gcc gcc-c++ git gperf kernel-devel libtool mercurial mingw64-gcc mingw64-gcc-c++ mingw64-winpthreads-static nasm python2-lxml subversion yasm
