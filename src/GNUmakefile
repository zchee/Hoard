# Commands to compile Hoard for various targets.
# Run make (with no arguments) to see the complete target list.

CPPFLAGS = -std=c++14 -flto -O3 -DNDEBUG -ffast-math -fno-builtin-malloc -Wall -Wextra -Wshadow -Wconversion -Wuninitialized # -DHL_NO_MALLOC_SIZE_CHECKS=1
CPPFLAGS_NOLTO = -std=c++14 -O3 -DNDEBUG -ffast-math -fno-builtin-malloc -Wall -Wextra -Wshadow -Wconversion -Wuninitialized # -DHL_NO_MALLOC_SIZE_CHECKS=1
#CPPFLAGS = -std=c++14 -g -O0 -ffast-math -fno-builtin-malloc -Wall -Wextra -Wshadow -Wconversion -Wuninitialized
CXX ?= clang++

# Compute platform (OS and architecture) and build accordingly.

ifeq ($(OS),Windows_NT)
windows: Heap-Layers
all: Heap-Layers windows
else
    UNAME_S := $(shell uname -s)
    UNAME_P := $(shell uname -p)
    ifeq ($(UNAME_S),SunOS)
      $(UNAME_S)-sunw-$(UNAME_P): Heap-Layers
      all: Heap-Layers $(UNAME_S)-sunw-$(UNAME_P)
      install: $(UNAME_S)-sunw-$(UNAME_P)-install
	@echo "To use Hoard, execute this command: export LD_PRELOAD=$(DESTDIR)$(PREFIX)/libhoard.so"
    else
      $(UNAME_S)-gcc-$(UNAME_P): Heap-Layers
      all: Heap-Layers $(UNAME_S)-gcc-$(UNAME_P)
      install: $(UNAME_S)-gcc-$(UNAME_P)-install
        ifeq ($(UNAME_S),Darwin)
	  @echo "To use Hoard, execute this command: export DYLD_INSERT_LIBRARIES=$(DESTDIR)$(PREFIX)/libhoard.dylib"
        else
	  @echo "To use Hoard, execute this command: export LD_PRELOAD=$(DESTDIR)$(PREFIX)/libhoard.so"
        endif
endif
endif

# Prefix for installations (Unix / Mac)

ifeq ($(UNAME_S),Darwin)
# Mac: can't copy into /usr/lib
PREFIX ?= /usr/local/lib
else
PREFIX ?= /usr/lib
  ifeq ($(UNAME_S),Linux)
    LINKER_N := $(shell ldd --version 2>&1 | head -n 1 | cut -f 1 -d " ")
    ifeq ($(LINKER_N),musl)
        CPPFLAGS = $(CPPFLAGS_NOLTO) -D__MUSL__
    endif
  endif
endif

help:
	@echo To build Hoard, specify the desired build target:
	@echo -------------------------------------------------
	@echo debian
	@echo freebsd
	@echo netbsd
	@echo Linux-gcc-arm
	@echo Linux-gcc-aarch64
	@echo Linux-gcc-x86
	@echo Linux-gcc-x86_64
	@echo Linux-gcc-riscv64
	@echo Darwin-gcc-i386
	@echo Darwin-gcc-arm
	@echo Darwin-gcc-powerpc
	@echo SunOS-sunw-sparc
	@echo SunOS-sunw-i386
	@echo SunOS-gcc-sparc
	@echo SunOS-gcc-i386
	@echo FreeBSD-gcc-amd64
	@echo generic-gcc
	@echo windows

.PHONY: Darwin-gcc-i386 Darwin-gcc-arm Darwin-gcc-powerpc debian freebsd netbsd Linux-gcc-x86 Linux-gcc-x86-debug SunOS-sunw-sparc SunOS-sunw-i386 SunOS-gcc-sparc generic-gcc Linux-gcc-arm Linux-gcc-aarch64 Linux-gcc-riscv64 Linux-gcc-x86_64 Linux-gcc-unknown windows windows-debug clean test release

#
# Source files
#

MAIN_SRC  = source/libhoard.cpp
UNIX_SRC  = $(MAIN_SRC) source/unixtls.cpp
SUNW_SRC  = $(UNIX_SRC) Heap-Layers/wrappers/wrapper.cpp
GNU_SRC   = $(UNIX_SRC)
MACOS_SRC = $(MAIN_SRC) Heap-Layers/wrappers/macwrapper.cpp source/mactls.cpp

#
# All dependencies.
#

DEPS = Heap-Layers $(MACOS_SRC) $(UNIX_SRC) source/libhoard.cpp

Heap-Layers:
	git clone https://github.com/emeryberger/Heap-Layers

release: Heap-Layers
	pushd ../.. ; tar -cf hoard-release.tar Hoard ;	gzip -f hoard-release.tar ; popd
	pushd ../.. ; zip -r hoard-release.zip Hoard ; popd

#
# Include directories
#

INCLUDES = -I. -Iinclude -Iinclude/util -Iinclude/hoard -Iinclude/superblocks -IHeap-Layers

WIN_INCLUDES = /I. /Iinclude /Iinclude/util /Iinclude/hoard /Iinclude/superblocks /IHeap-Layers

#
# Compile commands for individual targets.
#

FREEBSD_COMPILE = $(CXX) -g $(CPPFLAGS) -DNDEBUG -fPIC $(INCLUDES) -D_REENTRANT=1 -shared $(SUNW_SRC) -Bsymbolic -o libhoard.so -lpthread

NETBSD_COMPILE = $(CXX) -g $(CPPFLAGS_NOLTO) -DNDEBUG -fPIC $(INCLUDES) -D_REENTRANT=1 -shared $(SUNW_SRC) -Bsymbolic -o libhoard.so -lpthread

DEBIAN_COMPILE = $(CXX) -g -O3 -fPIC -DNDEBUG -I. -Iinclude -Iinclude/util -Iinclude/hoard -Iinclude/superblocks -IHeap-Layers -D_REENTRANT=1 -shared source/libhoard.cpp source/unixtls.cpp Heap-Layers/wrappers/wrapper.cpp -Bsymbolic -o libhoard.so -lpthread -lstdc++ -ldl

# -ftls-model=local-dynamic
MACOS_COMPILE = $(CXX) -ftls-model=initial-exec -ftemplate-depth=1024 -arch x86_64 -arch arm64 -pipe -g $(CPPFLAGS) $(INCLUDES) -D_REENTRANT=1 -compatibility_version 1 -current_version 1 -D'CUSTOM_PREFIX(x)=xx\#\#x' $(MACOS_SRC) -dynamiclib -install_name $(DESTDIR)$(PREFIX)/libhoard.dylib -o libhoard.dylib -ldl -lpthread

MACOS_COMPILE_STATIC = $(CXX) -ftls-model=initial-exec -ftemplate-depth=1024 -arch x86_64 -pipe -static -fPIC -fno-builtin-malloc -g $(CPPFLAGS) -W -Wconversion -Wall -DNDEBUG $(INCLUDES) -D_REENTRANT=1 -D'CUSTOM_PREFIX(x)=xx\#\#x' -c $(MAIN_SRC) -fvisibility=hidden -Wpedantic -MD -MT libhoard.o -MF libhoard.d -MF libhoard.o -o libhoard.o; \
											 $(CXX) -ftls-model=initial-exec -ftemplate-depth=1024 -arch x86_64 -pipe -static -fPIC -fno-builtin-malloc -g $(CPPFLAGS) -W -Wconversion -Wall -DNDEBUG $(INCLUDES) -D_REENTRANT=1 -D'CUSTOM_PREFIX(x)=xx\#\#x' -c Heap-Layers/wrappers/macwrapper.cpp -fvisibility=hidden -Wpedantic -MD -MT macwrapper.o -MF macwrapper.d -MF macwrapper.o -o macwrapper.o; \
											 $(CXX) -ftls-model=initial-exec -ftemplate-depth=1024 -arch x86_64 -pipe -static -fPIC -fno-builtin-malloc -g $(CPPFLAGS) -W -Wconversion -Wall -DNDEBUG $(INCLUDES) -D_REENTRANT=1 -D'CUSTOM_PREFIX(x)=xx\#\#x' -c source/mactls.cpp -fvisibility=hidden -Wpedantic -MD -MT mactls.o -MF mactls.d -MF mactls.o -o mactls.o; \
											 ar cr libhoard.a libhoard.o macwrapper.o mactls.o ; ranlib libhoard.a
# [23/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/libc.c.o -MF CMakeFiles/mimalloc-static.dir/src/libc.c.o.d -o CMakeFiles/mimalloc-static.dir/src/libc.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/libc.c
# /usr/local/Cellar/cmake/3.27.8/bin/cmake -E rm -f libmimalloc.a && /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ar qc libmimalloc.a  CMakeFiles/mimalloc-static.dir/src/alloc.c.o CMakeFiles/mimalloc-static.dir/src/alloc-aligned.c.o CMakeFiles/mimalloc-static.dir/src/alloc-posix.c.o CMakeFiles/mimalloc-static.dir/src/arena.c.o CMakeFiles/mimalloc-static.dir/src/bitmap.c.o CMakeFiles/mimalloc-static.dir/src/heap.c.o CMakeFiles/mimalloc-static.dir/src/init.c.o CMakeFiles/mimalloc-static.dir/src/libc.c.o CMakeFiles/mimalloc-static.dir/src/options.c.o CMakeFiles/mimalloc-static.dir/src/os.c.o CMakeFiles/mimalloc-static.dir/src/page.c.o CMakeFiles/mimalloc-static.dir/src/random.c.o CMakeFiles/mimalloc-static.dir/src/segment.c.o CMakeFiles/mimalloc-static.dir/src/segment-map.c.o CMakeFiles/mimalloc-static.dir/src/stats.c.o CMakeFiles/mimalloc-static.dir/src/prim/prim.c.o CMakeFiles/mimalloc-static.dir/src/prim/osx/alloc-override-zone.c.o && /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib libmimalloc.a && /usr/local/Cellar/cmake/3.27.8/bin/cmake -E touch libmimalloc.a

MACOS_COMPILE_DEBUG = $(CXX) -std=c++14 -D_FORTIFY_SOURCE=2 -fstack-protector -ftrapv -fno-builtin-malloc -ftemplate-depth=1024 -arch x86_64 -arch arm64 -pipe -g -O0 -Wall $(INCLUDES) -D_REENTRANT=1 -compatibility_version 1 -current_version 1 -D'CUSTOM_PREFIX(x)=xx\#\#x' $(MACOS_SRC) -dynamiclib -o libhoard.dylib -ldl -lpthread

MACOS_COMPILE_PPC = $(CXX) -ftls-model=initial-exec -ftemplate-depth=1024 -arch ppc -pipe -g $(CPPFLAGS) $(INCLUDES) -D_REENTRANT=1 -compatibility_version 1 -current_version 1 -D'CUSTOM_PREFIX(x)=xx\#\#x' $(MACOS_SRC) -dynamiclib -install_name $(DESTDIR)$(PREFIX)/libhoard.dylib -o libhoard.dylib -ldl -lpthread

MACOS_COMPILE_PPC64 = $(CXX) -ftls-model=initial-exec -ftemplate-depth=1024 -arch ppc64 -pipe -g $(CPPFLAGS) $(INCLUDES) -D_REENTRANT=1 -compatibility_version 1 -current_version 1 -D'CUSTOM_PREFIX(x)=xx\#\#x' $(MACOS_SRC) -dynamiclib -install_name $(DESTDIR)$(PREFIX)/libhoard.dylib -o libhoard.dylib -ldl -lpthread

LINUX_GCC_ARM_COMPILE = arm-Linux-gnueabihf-g++ $(CPPFLAGS) -g -W -Wconversion -Wall -I/usr/include/nptl -fno-builtin-malloc -pipe -fPIC -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared   $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

LINUX_GCC_AARCH64_COMPILE = aarch64-linux-gnu-g++ $(CPPFLAGS) -g -W -Wconversion -Wall -I/usr/include/nptl -fno-builtin-malloc -pipe -fPIC -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared   $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

LINUX_GCC_RISCV64_COMPILE = riscv64-linux-gnu-g++ $(CPPFLAGS) -g -W -Wconversion -Wall -I/usr/include/nptl -fno-builtin-malloc -pipe -fPIC -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared   $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread -latomic

LINUX_GCC_x86_COMPILE = $(CXX) -m32 $(CPPFLAGS) -I/usr/include/nptl -ffast-math -g -fno-builtin-malloc -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared  $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

LINUX_GCC_x86_64_COMPILE = $(CXX) $(CPPFLAGS) -g -W -Wconversion -Wall -I/usr/include/nptl -fno-builtin-malloc -pipe -fPIC -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared   $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

LINUX_GCC_UNKNOWN_COMPILE = $(CXX) $(CPPFLAGS) -g -W -Wconversion -Wall -I/usr/include/nptl -fno-builtin-malloc -pipe -fPIC -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared   $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

LINUX_GCC_x86_64_COMPILE_DEBUG = g++ $(CPPFLAGS) -g -W -Wconversion -Wall -I/usr/include/nptl -fno-builtin-malloc -pipe -fPIC $(INCLUDES) -D_REENTRANT=1 -shared $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

LINUX_GCC_x86_COMPILE_STATIC = g++ $(CPPFLAGS) -g -I/usr/include/nptl -static -pipe -fno-builtin-malloc -DNDEBUG  $(INCLUDES) -D_REENTRANT=1  -c $(GNU_SRC) ; ar cr libhoard.a libhoard.o

LINUX_GCC_x86_64_COMPILE_STATIC = g++ $(CPPFLAGS) -g -W -Wconversion -Wall -I/usr/include/nptl -static -pipe -fPIC -fno-builtin-malloc -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared -c $(GNU_SRC) -Bsymbolic ; ar cr libhoard.a libhoard.o

LINUX_GCC_x86_COMPILE_DEBUG = g++ -m32 $(CPPFLAGS) -fPIC -fno-inline -I/usr/include/nptl -fno-builtin-malloc -g -pipe $(INCLUDES) -D_REENTRANT=1  -shared $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

FREEBSD_GCC_AMD64_COMPILE = g++ -std=c++14 -g -O3 -fPIC -DNDEBUG -I. -Iinclude -Iinclude/util -Iinclude/hoard -Iinclude/superblocks -IHeap-Layers -D_REENTRANT=1 -shared source/libhoard.cpp source/unixtls.cpp Heap-Layers/wrappers/wrapper.cpp -Bsymbolic -o libhoard.so -lpthread -lstdc++

SUNOS_SUNW_SPARC_COMPILE_32_DEBUG = CC -dalign -xbuiltin=%all -fast -mt -g -xildoff -xthreadvar=dynamic -L/usr/lib/lwp -R/usr/lib/lwp -DNDEBUG $(INCLUDES) -D_REENTRANT=1 -G -PIC $(SUNW_SRC) Heap-Layers/wrappers/arch-specific/sparc-interchange.il -o libhoard_32.so -lthread -ldl -lCrun

SUNOS_SUNW_SPARC_COMPILE_32 = CC -dalign -xbuiltin=%all -fast -xO5 -DNDEBUG -mt -g -xildoff -xthreadvar=dynamic -L/usr/lib/lwp -R/usr/lib/lwp $(INCLUDES) -D_REENTRANT=1 -G -PIC $(SUNW_SRC) Heap-Layers/wrappers/arch-specific/sparc-interchange.il -o libhoard_32.so -lthread -ldl -lCrun 

SUNOS_SUNW_SPARC_COMPILE_64 = CC -g -xcode=pic13 -m64 -mt -fast -dalign -xbuiltin=%all -xO5 -xildoff -xthreadvar=dynamic -L/usr/lib/lwp -R/usr/lib/lwp -DNDEBUG $(INCLUDES) -D_REENTRANT=1 -G -PIC $(SUNW_SRC) Heap-Layers/wrappers/arch-specific/sparc-interchange.il -o libhoard_64.so -lthread -ldl -lCrun

SUNOS_SUNW_x86_COMPILE_32 = CC -g -fns -fsimple=2 -ftrap=%none -xbuiltin=%all -mt -xO5 -xildoff -xthreadvar=dynamic -L/usr/lib/lwp -R/usr/lib/lwp -DNDEBUG $(INCLUDES) -D_REENTRANT=1 -G -PIC $(SUNW_SRC) Heap-Layers/wrappers/arch-specific/x86-interchange.il -o libhoard_32.so -lthread -ldl -lCrun

SUNOS_SUNW_x86_COMPILE_32_DEBUG = CC -mt -g -xildoff -xthreadvar=dynamic -L/usr/lib/lwp -R/usr/lib/lwp $(INCLUDES) -D_REENTRANT=1 -G -PIC $(SUNW_SRC) Heap-Layers/wrappers/arch-specific/x86-interchange.il -o libhoard_32.so -lthread -ldl -lCrun

SUNOS_SUNW_x86_COMPILE_64 = CC -g -m64 -fns -fsimple=2 -ftrap=%none -xbuiltin=%all -xO5 -xildoff -xthreadvar=dynamic -L/usr/lib/lwp -R/usr/lib/lwp -DNDEBUG $(INCLUDES) -D_REENTRANT=1 -G -PIC $(SUNW_SRC) Heap-Layers/wrappers/arch-specific/x86_64-interchange.il -o libhoard_64.so -lthread -ldl -lCrun

SUNOS_GCC_SPARC_COMPILE_32 = g++ -g -fno-builtin-malloc -nostartfiles -pipe -DNDEBUG -mcpu=ultrasparc -m32 $(CPPFLAGS) -fPIC -ffast-math $(INCLUDES) -D_REENTRANT=1 -shared $(SUNW_SRC) -lthread -lpthread -ldl -o libhoard_32.so

SUNOS_GCC_SPARC_COMPILE_64 = g++ -g -fno-builtin-malloc -nostartfiles -pipe -DNDEBUG -mcpu=ultrasparc -m64 $(CPPFLAGS) -fPIC -fkeep-inline-functions -finline-functions -ffast-math $(INCLUDES) -D_REENTRANT=1 -shared $(SUNW_SRC) -lthread -lpthread -ldl -o libhoard_64.so

SUNOS_GCC_I386_COMPILE_32 = $(CXX) -g -fno-builtin-malloc -nostartfiles -pipe -DNDEBUG -m32 $(CPPFLAGS) -finline-limit=20000 -fPIC -fkeep-inline-functions -finline-functions -ffast-math $(INCLUDES) -D_REENTRANT=1 -shared $(SUNW_SRC) -lthread -ldl -fno-use-cxa-atexit -o libhoard_32.so

SUNOS_GCC_I386_COMPILE_64 = $(CXX) -g -fno-builtin-malloc -nostartfiles -pipe -DNDEBUG -m64 $(CPPFLAGS) -finline-limit=20000 -fPIC -fkeep-inline-functions -finline-functions -ffast-math $(INCLUDES) -D_REENTRANT=1 -shared $(SUNW_SRC) -lthread -ldl -fno-use-cxa-atexit -o libhoard_64.so

SUNOS_GCC_SPARC_COMPILE_DEBUG = g++ -g -fno-builtin-malloc -nostartfiles -pipe -mcpu=ultrasparc -g -fPIC $(INCLUDES) -D_REENTRANT=1 -shared $(SUNW_SRC) -lthread -lpthread -ldl -o libhoard.so

GENERIC_GCC_COMPILE = g++ -I/usr/include/nptl -fno-builtin-malloc -pipe -g $(CPPFLAGS) -finline-limit=20000 -finline-functions  -DNDEBUG  $(INCLUDES) -D_REENTRANT=1 -shared $(GNU_SRC) -Bsymbolic -o libhoard.so -ldl -lpthread

WIN_DEFINES = /D "NDEBUG" /D "_WINDOWS" /D "_WINDLL" /D "_WINRT_DLL" /D "_UNICODE" /D "UNICODE"
WIN_DEBUG_DEFINES = /D "_WINDOWS" /D "_WINDLL" /D "_WINRT_DLL" /D "_UNICODE" /D "UNICODE"

WIN_FLAGS         = /Zi /Ox /MD /nologo /W1 /WX- /Ox /Oi /Oy- /Gm- /EHsc /MD /GS /Gy /Zc:wchar_t /Zc:forScope /Gd /errorReport:queue
WIN_DEBUG_FLAGS   = /Zi /MD /nologo /W1 /WX- /Gm- /EHsc /MD /GS /Gy /Zc:wchar_t /Zc:forScope /Gd /errorReport:queue

windows: $(DEPS)
	cl $(WIN_INCLUDES) $(WIN_DEFINES) $(WIN_FLAGS) "source\libhoard.cpp" "Heap-Layers\wrappers\winwrapper.cpp" "source\wintls.cpp" /GL /link /DLL /subsystem:console /OUT:libhoard.dll
	cl $(WIN_INCLUDES) $(WIN_DEFINES) $(WIN_FLAGS) /c "source\uselibhoard.cpp"

windows-debug: $(DEPS)
	cl /analyze /analyze:stacksize131072 $(WIN_INCLUDES) $(WIN_DEBUG_DEFINES) $(WIN_DEBUG_FLAGS) "source\libhoard.cpp" "Heap-Layers\wrappers\winwrapper.cpp" "source\wintls.cpp" /GL /link /DLL /subsystem:console /OUT:libhoard.dll
	cl $(WIN_INCLUDES) $(WIN_DEBUG_DEFINES) $(WIN_DEBUG_FLAGS) /c "source\uselibhoard.cpp"

Darwin-gcc-i386:
	$(MACOS_COMPILE)

Darwin-gcc-i386-install: Darwin-gcc-i386
	cp libhoard.dylib $(DESTDIR)$(PREFIX)

Darwin-gcc-i386-install-static: Darwin-gcc-i386-static
	cp libhoard.a $(DESTDIR)$(PREFIX)

Darwin-gcc-arm:
	$(MACOS_COMPILE)

Darwin-gcc-arm-install: Darwin-gcc-arm
	cp libhoard.dylib $(DESTDIR)$(PREFIX)

Darwin-gcc-powerpc:
	$(MACOS_COMPILE_PPC)

Darwin-gcc-powerpc-install: Darwin-gcc-powerpc
	cp libhoard.dylib $(DESTDIR)$(PREFIX)

generic-gcc:
	$(GENERIC_GCC_COMPILE)

generic-gcc-install: generic-gcc
	cp libhoard.so $(DESTDIR)$(PREFIX)

Linux-gcc-arm:
	$(LINUX_GCC_ARM_COMPILE)

Linux-gcc-arm-install: Linux-gcc-arm
	cp libhoard.so $(DESTDIR)$(PREFIX)

Linux-gcc-aarch64:
	$(LINUX_GCC_AARCH64_COMPILE)

Linux-gcc-aarch64-install: Linux-gcc-aarch64
	cp libhoard.so $(DESTDIR)$(PREFIX)

Linux-gcc-riscv64:
	$(LINUX_GCC_RISCV64_COMPILE)

Linux-gcc-riscv64-install: Linux-gcc-riscv64
	cp libhoard.so $(DESTDIR)$(PREFIX)

Linux-gcc-x86:
	$(LINUX_GCC_x86_COMPILE)

Linux-gcc-x86-install: Linux-gcc-x86
	cp libhoard.so $(DESTDIR)$(PREFIX)

FreeBSD-gcc-amd64:
	$(FREEBSD_GCC_AMD64_COMPILE)

Linux-gcc-x86_64:
	$(LINUX_GCC_x86_64_COMPILE)

Linux-gcc-x86_64-install: Linux-gcc-x86_64
	cp libhoard.so $(DESTDIR)$(PREFIX)

Linux-gcc-unknown:
	$(LINUX_GCC_UNKNOWN_COMPILE)

Linux-gcc-unknown-install: Linux-gcc-unknown
	cp libhoard.so $(DESTDIR)$(PREFIX)

SunOS-sunw-sparc:
	$(SUNOS_SUNW_SPARC_COMPILE_32)
	$(SUNOS_SUNW_SPARC_COMPILE_64)

SunOS-sunw-sparc-install: SunOS-sunw-sparc
	cp libhoard_32.so $(DESTDIR)$(PREFIX)
	cp libhoard_64.so $(DESTDIR)$(PREFIX)

SunOS-gcc-sparc:
	$(SUNOS_GCC_SPARC_COMPILE_32)
	$(SUNOS_GCC_SPARC_COMPILE_64)

SunOS-gcc-sparc-install: SunOS-gcc-sparc
	cp libhoard_32.so $(DESTDIR)$(PREFIX)
	cp libhoard_64.so $(DESTDIR)$(PREFIX)

SunOS-gcc-i386:
	$(SUNOS_GCC_I386_COMPILE_32)
	$(SUNOS_GCC_I386_COMPILE_64)

SunOS-gcc-i386-install: SunOS-gcc-i386
	cp libhoard_32.so $(DESTDIR)$(PREFIX)
	cp libhoard_64.so $(DESTDIR)$(PREFIX)

SunOS-sunw-i386:
	$(SUNOS_SUNW_x86_COMPILE_32)
	$(SUNOS_SUNW_x86_COMPILE_64)

SunOS-sunw-i386-install: SunOS-sunw-i386
	cp libhoard_32.so $(DESTDIR)$(PREFIX)
	cp libhoard_64.so $(DESTDIR)$(PREFIX)

#
#
#

Linux-gcc-x86-static:
	$(LINUX_GCC_x86_COMPILE_STATIC)

Linux-gcc-x86-debug:
	$(LINUX_GCC_x86_COMPILE_DEBUG)

SunOS-gcc-sparc-debug:
	$(SUNOS_GCC_SPARC_COMPILE_DEBUG)

Darwin-gcc-i386-static: Heap-Layers
	$(MACOS_COMPILE_STATIC)

Darwin-gcc-i386-debug:
	$(MACOS_COMPILE_DEBUG)

Darwin-gcc-arm-debug:
	$(MACOS_COMPILE_DEBUG)

Linux-gcc-x86_64-static:
	$(LINUX_GCC_x86_64_COMPILE_STATIC)

Linux-gcc-x86_64-debug:
	$(LINUX_GCC_x86_64_COMPILE_DEBUG)

SunOS-sunw-sparc-debug:
	$(SUNOS_SUNW_SPARC_COMPILE_32_DEBUG)

freebsd:
	$(FREEBSD_COMPILE)

netbsd:
	$(NETBSD_COMPILE)

debian:
	$(DEBIAN_COMPILE)

clean:
	rm -rf libhoard.*

# [1/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/alloc-posix.c.o -MF CMakeFiles/mimalloc.dir/src/alloc-posix.c.o.d -o CMakeFiles/mimalloc.dir/src/alloc-posix.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/alloc-posix.c
# [2/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimall
# oc.dir/src/segment-map.c.o -MF CMakeFiles/mimalloc.dir/src/segment-map.c.o.d -o CMakeFiles/mimalloc.dir/src/segment-map.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/segment-map.c
# [3/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/segment-map.c.o -MF CMakeFiles/mimalloc-static.dir/src/segment-map.c.o.d -o CMakeFiles/mimalloc-static.dir/src/segment-map.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/segment-map.c
# [4/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/prim/osx/alloc-override-zone.c.o -MF CMakeFiles/mimalloc.dir/src/prim/osx/alloc-override-zone.c.o.d -o CMakeFiles/mimalloc.dir/src/prim/osx/alloc-override-zone.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/prim/osx/alloc-override-zone.c
# [5/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/alloc-posix.c.o -MF CMakeFiles/mimalloc-static.dir/src/alloc-posix.c.o.d -o CMakeFiles/mimalloc-static.dir/src/alloc-posix.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/alloc-posix.c
# [6/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/prim/osx/alloc-override-zone.c.o -MF CMakeFiles/mimalloc-static.dir/src/prim/osx/alloc-override-zone.c.o.d -o CMakeFiles/mimalloc-static.dir/src/prim/osx/alloc-override-zone.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/prim/osx/alloc-override-zone.c
# [7/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/random.c.o -MF CMakeFiles/mimalloc.dir/src/random.c.o.d -o CMakeFiles/mimalloc.dir/src/random.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/random.c
# [8/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/prim/prim.c.o -MF CMakeFiles/mimalloc-static.dir/src/prim/prim.c.o.d -o CMakeFiles/mimalloc-static.dir/src/prim/prim.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/prim/prim.c
# [9/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/prim/prim.c.o -MF CMakeFiles/mimalloc.dir/src/prim/prim.c.o.d -o CMakeFiles/mimalloc.dir/src/prim/prim.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/prim/prim.c
# [10/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/random.c.o -MF CMakeFiles/mimalloc-static.dir/src/random.c.o.d -o CMakeFiles/mimalloc-static.dir/src/random.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/random.c
# [11/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/stats.c.o -MF CMakeFiles/mimalloc-static.dir/src/stats.c.o.d -o CMakeFiles/mimalloc-static.dir/src/stats.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/stats.c
# [12/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/stats.c.o -MF CMakeFiles/mimalloc.dir/src/stats.c.o.d -o CMakeFiles/mimalloc.dir/src/stats.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/stats.c
# [13/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/init.c.o -MF CMakeFiles/mimalloc-static.dir/src/init.c.o.d -o CMakeFiles/mimalloc-static.dir/src/init.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/init.c
# [14/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/init.c.o -MF CMakeFiles/mimalloc.dir/src/init.c.o.d -o CMakeFiles/mimalloc.dir/src/init.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/init.c
# [15/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/alloc-aligned.c.o -MF CMakeFiles/mimalloc.dir/src/alloc-aligned.c.o.d -o CMakeFiles/mimalloc.dir/src/alloc-aligned.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/alloc-aligned.c
# [16/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/options.c.o -MF CMakeFiles/mimalloc.dir/src/options.c.o.d -o CMakeFiles/mimalloc.dir/src/options.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/options.c
# [17/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/alloc-aligned.c.o -MF CMakeFiles/mimalloc-static.dir/src/alloc-aligned.c.o.d -o CMakeFiles/mimalloc-static.dir/src/alloc-aligned.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/alloc-aligned.c
# [18/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/options.c.o -MF CMakeFiles/mimalloc-static.dir/src/options.c.o.d -o CMakeFiles/mimalloc-static.dir/src/options.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/options.c
# [19/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/heap.c.o -MF CMakeFiles/mimalloc.dir/src/heap.c.o.d -o CMakeFiles/mimalloc.dir/src/heap.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/heap.c
# [20/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/heap.c.o -MF CMakeFiles/mimalloc-static.dir/src/heap.c.o.d -o CMakeFiles/mimalloc-static.dir/src/heap.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/heap.c
# [21/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/bitmap.c.o -MF CMakeFiles/mimalloc.dir/src/bitmap.c.o.d -o CMakeFiles/mimalloc.dir/src/bitmap.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/bitmap.c
# [22/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/libc.c.o -MF CMakeFiles/mimalloc.dir/src/libc.c.o.d -o CMakeFiles/mimalloc.dir/src/libc.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/libc.c
# [23/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/libc.c.o -MF CMakeFiles/mimalloc-static.dir/src/libc.c.o.d -o CMakeFiles/mimalloc-static.dir/src/libc.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/libc.c
# [24/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/bitmap.c.o -MF CMakeFiles/mimalloc-static.dir/src/bitmap.c.o.d -o CMakeFiles/mimalloc-static.dir/src/bitmap.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/bitmap.c
# [25/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/os.c.o -MF CMakeFiles/mimalloc-static.dir/src/os.c.o.d -o CMakeFiles/mimalloc-static.dir/src/os.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/os.c
# [26/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/os.c.o -MF CMakeFiles/mimalloc.dir/src/os.c.o.d -o CMakeFiles/mimalloc.dir/src/os.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/os.c
# [27/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/arena.c.o -MF CMakeFiles/mimalloc.dir/src/arena.c.o.d -o CMakeFiles/mimalloc.dir/src/arena.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/arena.c
# [28/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/arena.c.o -MF CMakeFiles/mimalloc-static.dir/src/arena.c.o.d -o CMakeFiles/mimalloc-static.dir/src/arena.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/arena.c
# [29/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/page.c.o -MF CMakeFiles/mimalloc-static.dir/src/page.c.o.d -o CMakeFiles/mimalloc-static.dir/src/page.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/page.c
# [30/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/alloc.c.o -MF CMakeFiles/mimalloc.dir/src/alloc.c.o.d -o CMakeFiles/mimalloc.dir/src/alloc.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/alloc.c
# [31/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/page.c.o -MF CMakeFiles/mimalloc.dir/src/page.c.o.d -o CMakeFiles/mimalloc.dir/src/page.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/page.c
# [32/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/alloc.c.o -MF CMakeFiles/mimalloc-static.dir/src/alloc.c.o.d -o CMakeFiles/mimalloc-static.dir/src/alloc.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/alloc.c
# [33/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_SHARED_LIB -DMI_SHARED_LIB_EXPORT -DMI_XMALLOC=1 -Dmimalloc_EXPORTS -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc.dir/src/segment.c.o -MF CMakeFiles/mimalloc.dir/src/segment.c.o.d -o CMakeFiles/mimalloc.dir/src/segment.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/segment.c
# [34/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_STATIC_LIB -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-static.dir/src/segment.c.o -MF CMakeFiles/mimalloc-static.dir/src/segment.c.o.d -o CMakeFiles/mimalloc-static.dir/src/segment.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/segment.c
# [35/39] : && /usr/local/Cellar/cmake/3.27.8/bin/cmake -E rm -f libmimalloc.a && /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ar qc libmimalloc.a  CMakeFiles/mimalloc-static.dir/src/alloc.c.o CMakeFiles/mimalloc-static.dir/src/alloc-aligned.c.o CMakeFiles/mimalloc-static.dir/src/alloc-posix.c.o CMakeFiles/mimalloc-static.dir/src/arena.c.o CMakeFiles/mimalloc-static.dir/src/bitmap.c.o CMakeFiles/mimalloc-static.dir/src/heap.c.o CMakeFiles/mimalloc-static.dir/src/init.c.o CMakeFiles/mimalloc-static.dir/src/libc.c.o CMakeFiles/mimalloc-static.dir/src/options.c.o CMakeFiles/mimalloc-static.dir/src/os.c.o CMakeFiles/mimalloc-static.dir/src/page.c.o CMakeFiles/mimalloc-static.dir/src/random.c.o CMakeFiles/mimalloc-static.dir/src/segment.c.o CMakeFiles/mimalloc-static.dir/src/segment-map.c.o CMakeFiles/mimalloc-static.dir/src/stats.c.o CMakeFiles/mimalloc-static.dir/src/prim/prim.c.o CMakeFiles/mimalloc-static.dir/src/prim/osx/alloc-override-zone.c.o && /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ranlib libmimalloc.a && /usr/local/Cellar/cmake/3.27.8/bin/cmake -E touch libmimalloc.a && :
# [36/39] : && /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -O3 -DNDEBUG -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -dynamiclib -Wl,-headerpad_max_install_names -compatibility_version 2.0.0 -current_version 2.1.0 -o libmimalloc.2.1.dylib -install_name @rpath/libmimalloc.2.dylib CMakeFiles/mimalloc.dir/src/alloc.c.o CMakeFiles/mimalloc.dir/src/alloc-aligned.c.o CMakeFiles/mimalloc.dir/src/alloc-posix.c.o CMakeFiles/mimalloc.dir/src/arena.c.o CMakeFiles/mimalloc.dir/src/bitmap.c.o CMakeFiles/mimalloc.dir/src/heap.c.o CMakeFiles/mimalloc.dir/src/init.c.o CMakeFiles/mimalloc.dir/src/libc.c.o CMakeFiles/mimalloc.dir/src/options.c.o CMakeFiles/mimalloc.dir/src/os.c.o CMakeFiles/mimalloc.dir/src/page.c.o CMakeFiles/mimalloc.dir/src/random.c.o CMakeFiles/mimalloc.dir/src/segment.c.o CMakeFiles/mimalloc.dir/src/segment-map.c.o CMakeFiles/mimalloc.dir/src/stats.c.o CMakeFiles/mimalloc.dir/src/prim/prim.c.o CMakeFiles/mimalloc.dir/src/prim/osx/alloc-override-zone.c.o  /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/lib/libpthread.tbd && :
# [37/39] /usr/local/Cellar/cmake/3.27.8/bin/cmake -E cmake_symlink_library libmimalloc.2.1.dylib  libmimalloc.2.dylib libmimalloc.dylib && :
# [38/39] /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -DMI_MALLOC_OVERRIDE -DMI_OSX_INTERPOSE=1 -DMI_OSX_ZONE=1 -DMI_XMALLOC=1 -I/Users/zchee/src/github.com/microsoft/mimalloc/include -O3 -DNDEBUG -std=gnu11 -arch x86_64 -isysroot /Applications/Xcode-15.1.0-Beta.2.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=13.3 -fPIC -Wall -Wextra -Wno-unknown-pragmas -fvisibility=hidden -Wstrict-prototypes -Wpedantic -Wno-static-in-inline -ftls-model=initial-exec -fno-builtin-malloc -MD -MT CMakeFiles/mimalloc-obj.dir/src/static.c.o -MF CMakeFiles/mimalloc-obj.dir/src/static.c.o.d -o CMakeFiles/mimalloc-obj.dir/src/static.c.o -c /Users/zchee/src/github.com/microsoft/mimalloc/src/static.c
# [39/39] cd /Users/zchee/src/github.com/microsoft/mimalloc/build && /usr/local/Cellar/cmake/3.27.8/bin/cmake -E copy /Users/zchee/src/github.com/microsoft/mimalloc/build/CMakeFiles/mimalloc-obj.dir/src/static.c.o /Users/zchee/src/github.com/microsoft/mimalloc/build/mimalloc.o
