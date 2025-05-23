# ffmpeg-cxc-build
Fast cross-compile ffmpeg for Windows with MinGW on Linux and Cygwin to produce a statically linked ffmpeg.

**Enabled Features**

amf aom ass/ssa avisynth bzip2 dav1d decklink fdk-aac fontconfig freetype frei0r fribidi harfbuzz lame mfx/qsv nvenc/nvdec ogg openssl opus placebo png rubberband sdl sofalizer soxr srt svt-av1 theora vmaf vorbis vpx webp x264 x265(8/10/12-bit) xml2 zlib

**Package Requirements**

Debian/Ubuntu/Mint:

sudo apt-get -y install autoconf automake autopoint build-essential libarchive-tools cmake git-core gperf g++-mingw-w64 libssl-dev libtool libunwind-dev mercurial meson nasm pkg-config python3-lxml ragel subversion texinfo yasm wget win-iconv-mingw-w64-dev
  
Fedora:

sudo yum install make autogen automake bsdtar cmake gcc gcc-c++ git gettext-devel gperf kernel-devel libtool libunwind-devel mercurial meson mingw64-gcc mingw64-gcc-c++ mingw64-libgomp mingw64-winpthreads-static mingw64-win-iconv-static nasm openssl-devel perl-FindBin python3-lxml ragel subversion uuid-devel yasm

*Fedora Preconditions:*

With FC39 and later, a link from the mingw sys-root directory to the sandbox build path must be created (not pretty but works for now). Assuming the default configuration is used for the ROOT_PATH variable:

	sudo ln -s /home /usr/x86_64-w64-mingw32/sys-root/mingw/home

Arch/Manjaro:

autoconf automake bsdtar cmake git gperf libtool mercurial meson mingw-w64-binutils mingw-w64-crt mingw-w64-gcc mingw-w64-headers mingw-w64-tools mingw-w64-winpthreads nasm openssl-1.1 python-lxml ragel subversion wget yasm
  
Cygwin:
  
 After a baseline install, run setup and install the packages: autoconf autoconf2.1 autoconf2.5 autogen automake automake1.10 automake1.11 automake1.12 automake1.13 automake1.14 automake1.15 automake1.9 binutils bsdtar cmake gcc-core gcc-g++ gettext gettext-devel gettext-doc git gperf libtool libuuid-devel make mercurial meson mingw64-x86_64-binutils mingw64-x86_64-gcc-core mingw64-x86_64-gcc-g++ mingw64-x86_64-gettext mingw64-x86_64-headers mingw64-x86_64-runtime mingw64-x86_64-win-iconv mingw64-x86_64-windows-default-manifest mingw64-x86_64-winpthreads nasm python27 python27-lxml python27-six ragel rsync subversion wget yasm
 
*Cygwin Preconditions:*
 
The mingw toolchain files must be in the path, and a link from the mingw sys-root directory to the sandbox build path must be created.  As an example, assuming the default configuration is used for the ROOT_PATH variable,  *ffmpeg-cxc-build-hints* has been placed in the user's home directory, and *ffmpeg-cxc-mingw64* is executable in the environment's path:
 
	ln -s /home /usr/x86_64-w64-mingw32/sys-root/home
	export PATH=$PATH:/usr/x86_64-w64-mingw32/sys-root/mingw/bin
	HINTS_FILE=~/ffmpeg-cxc-build-hints ffmpeg-cxc-mingw64

**rav1e/cargo Installation**

If you want to enable the rav1e AV1 encoder, you will need to install the rust toolchain. A minimal working toolchain with support for the x86_64-pc-windows-gnu target can be installed as below.

	curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
	source "$HOME/.cargo/env"
	rustup install 1.87.0
	rustup default 1.87.0
	cargo install --version 0.10.7+cargo-0.84.0 cargo-c
	rustup target add x86_64-pc-windows-gnu
 	rustup target add x86_64-unknown-linux-musl

**Example Build Procedure**

Assumptions: ROOT_PATH is left to the default ($HOME) and *ffmpeg-cxc-mingw64* is executable in the environment's path.

	#fetch the source projects to the ~/src directory
	ROOT_PATH=~/ SRC_PATH=src HINTS_FILE=/dev/null CXC_FETCH_ONLY=1 ffmpeg-cxc-mingw64
	#copy ffmpeg-cxc-build-hints to ~/ and edit to fetch files from ~/src via URL override
	ffmpeg-cxc-mingw64

Similarly, a native (linux) version of ffmpeg can be built as below assuming the SRC_PATH has a complete set of repositories. If desired, the VPL package should be installed as per your distro, and the prefix path, usually /usr, should be specified as a variable before the script. The native build script should find the VPL files to copy and patch allowing the sanboxed build to work. An MFX (deprecated) build works essentially the same, but VPL should cover all options at this point.

	HINTS_FILE=~/ffmpeg-native-build-hints-slim VPL_DEV=/usr ffmpeg-native

Using these techniques, the script *update-repo* can be used to maintain up to date repositories by cd'ing into ~/src and running it.  Subsequent builds will then pull via the file URL override allowing up to date builds of ffmpeg and libraries if you edit the hints file and remove the commit hashes (empty means latest).  Note that fontconfig and openssl will likely require the hashes specified in the hints file to build and operate as expected.

**FFmpeg Latest Build Procedure**


	#using the ~/src directory as above - update and specify the path to ffmpeg-cxc-build-hints-file-head
	cd ~/src
	update-repo
	HINTS_FILE=~/ffmpeg-cxc-build-hints-file-head ffmpeg-cxc-mingw64

**Using frei0r**

Frei0r is a collection of video effect plugins which can be used as ffmpeg filters.  The effects are self-contained in individual DLLs/shared libraries and are not compiled into ffmepg.  After a successful build, the compiled modules will be at "$ROOT_PATH/$OUT_PATH/lib/frei0r".  After moving them to the desired path and setting the location of the plugins in an environment variable, they are easily enabled as an ffmpeg video filter (-vf).  Assuming *.frei0r* has been created in the user's home directory and the modules have been copied there, the FREI0R_PATH can be defined as below.

Sample:

	#Unixish: export FREI0R_PATH=$HOME/.frei0r-1
	#Windows: set FREI0R_PATH=%USERPROFILE%\.frei0r-1
	#Test the plugin
	ffmpeg -i in.mp4 -vf frei0r=vignette out.mp4

**Using fontconfig**

Fontconfig allows the user to specify fonts by name instead of file path making drawtext filters easier to construct.  To access user and system fonts on Windows, a minimal fonts.conf file should be specified in the environment. A sample fonts.conf and drawtext filter to test font access appear below.

fonts.conf:

	<?xml version="1.0"?>
	<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
	<!-- minimal fonts.conf file for ffmpeg on Windows -->
	<fontconfig>
		<dir>WINDOWSFONTDIR</dir>
		<dir>~/AppData/Local/Microsoft/Windows/Fonts</dir>
		<cachedir>LOCAL_APPDATA_FONTCONFIG_CACHE</cachedir>
	</fontconfig>

Sample:

	#Windows
	set FONTCONFIG_FILE=%USERPROFILE%\.fonts\fonts.conf
	#Test font access
	ffmpeg -y -t 10 -i in.mp4 -filter_complex "drawtext='font=Segoe UI:\
	fontsize=64:x=20:y=50:fontcolor=white:bordercolor=black:borderw=1:\
	shadowcolor=black:shadowx=2:shadowy=2:alpha=0.5:text=Testing 1, 2, 3'" -an out.mp4
