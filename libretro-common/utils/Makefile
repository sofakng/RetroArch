compiler    := gcc
extra_flags :=
use_neon    := 0
release	   := release
DYLIB	      := 

ifeq ($(platform),)
platform = unix
ifeq ($(shell uname -a),)
   platform = win
else ifneq ($(findstring MINGW,$(shell uname -a)),)
   platform = win
else ifneq ($(findstring Darwin,$(shell uname -a)),)
   platform = osx
   arch = intel
ifeq ($(shell uname -p),powerpc)
   arch = ppc
endif
else ifneq ($(findstring win,$(shell uname -a)),)
   platform = win
endif
endif

ifeq ($(platform),gcc)
extra_rules_gcc := $(shell $(compiler) -dumpmachine)
endif

ifneq (,$(findstring armv7,$(extra_rules_gcc)))
extra_flags += -mcpu=cortex-a9 -mtune=cortex-a9 -mfpu=neon
use_neon := 1
endif

ifneq (,$(findstring hardfloat,$(extra_rules_gcc)))
extra_flags += -mfloat-abi=hard
endif

ifeq (release,$(build))
extra_flags += -O2
endif

ifeq (debug,$(build))
extra_flags += -O0 -g
endif

ldflags := -shared -Wl,--version-script=link.T

ifeq ($(platform), unix)
DYLIB =
else ifeq ($(platform), osx)
compiler := $(CC)
DYLIB = 
ldflags := -dynamiclib
else
extra_flags += -static-libgcc -static-libstdc++
DYLIB = exe
endif

CC      := $(compiler)
CXX     := $(subst CC,++,$(compiler))
flags   := -fPIC $(extra_flags) -I../../libretro-common/include
asflags := -fPIC  $(extra_flags)
objects :=
LDFLAGS := -lz
flags   += -std=c99


ifeq (1,$(use_neon))
ASMFLAGS := -INEON/asm
asflags += -mfpu=neon
endif
           
objects += crc32$(DYLIB) djb2$(DYLIB) md5$(DYLIB) sha1$(DYLIB)

all: build;

%.o: %.S
	$(CC) -c -o $@ $(asflags) $(LDFLAGS)  $(ASMFLAGS)  $<

%.o: %.c
	$(CC) -c -o $@ $(flags) $<

%.$(DYLIB): %.o
	$(CC) -o $@ $(ldflags) $(flags) $^

build: $(objects)

clean:
	rm -f *.o
	rm -f *.$(DYLIB)

strip:
	strip -s *.$(DYLIB)
