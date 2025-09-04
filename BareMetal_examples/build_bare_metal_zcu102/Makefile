 #
 # Copyright 2020, Xilinx Inc
 #
 # Permission is hereby granted, free of charge, to any person obtaining a copy
 # of this software and associated documentation files (the "Software"), to deal
 # in the Software without restriction, including without limitation the rights
 # to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 # copies of the Software, and to permit persons to whom the Software is
 # furnished to do so, subject to the following conditions:
 #    * The above copyright notice and this permission notice shall be included
 #      in all copies or substantial portions of the Software.
 #
 # THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 # IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 # FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 # AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 # LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 # OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
 # SOFTWARE.


UNAME_S:=$(shell uname -s)
ARCH:=$(shell uname -m)
COMPILE=Native

TOOLCHAIN_PATH:=""

TOOL_CC:=gcc
TOOL_LD:=ld
TOOL_AS:=as
TOOL_OBJCOPY:=objcopy

LD_SCRIPT:=-Thello_world.ld

ifeq ($(UNAME_S),Darwin)
    OS:=MacOS
    ifeq ($(ARCH),arm64)
        TOOLCHAIN_PATH:="/opt/homebrew/opt/aarch64-unknown-linux-gnu/bin/"
        CROSS_PREFIX:=aarch64-linux-gnu-
    else ifeq ($(ARCH),x86_64)
        COMPILE:=Cross
        TOOLCHAIN_PATH:="/opt/homebrew/opt/aarch64-unknown-linux-gnu/bin/"
        CROSS_PREFIX:=aarch64-linux-gnu-
    else
        $(error unsupported architecture $(ARCH) for $(OS))
    endif
else ifeq ($(UNAME_S),Linux)
    OS:=Linux
    ifeq ($(ARCH),arm64)
        COMPILE:=Native
    else ifeq ($(ARCH),x86_64)
        COMPILE:=Cross
        CROSS_PREFIX:=aarch64-linux-gnu-
    else
        $(error unsupported architecture $(ARCH) for $(OS))
    endif
else ifeq ($(findstring MINGW64,$(UNAME_S)),MINGW64)
    OS:=Windows(MINGW64)
    ifeq ($(ARCH),arm64)
        COMPILE:=Native
    else ifeq ($(ARCH),x86_64)
        COMPILE:=Cross
            TOOLCHAIN_PATH:="/usr/bin"
            CROSS_PREFIX:=aarch64-none-elf-
    else
        $(error unsupported architecture $(ARCH) for $(OS))
    endif
else
    $(error Unsupported build environement)
endif

AS := $(TOOLCHAIN_PATH)$(CROSS_PREFIX)$(TOOL_AS)
CC := $(TOOLCHAIN_PATH)$(CROSS_PREFIX)$(TOOL_CC)
LD := $(TOOLCHAIN_PATH)$(CROSS_PREFIX)$(TOOL_LD)
OBJCOPY := $(TOOLCHAIN_PATH)$(CROSS_PREFIX)$(TOOL_OBJCOPY)


$(info $(COMPILE) compile to arm64 on $(OS)/$(ARCH))


define c_compiler
    $(info CC       $1)
    @$(CC) -c $1 -o $2
endef

define as_compiler
    $(info AS       $1)
    @$(AS) -c $1 -o $2
endef

define linker
    $(info LD       $2)
    @$(LD) $(LD_SCRIPT) $1 $2 -o $3
endef

define elf2bin
    $(info ELF2BIN  $1)
    @$(OBJCOPY) -O binary $1 $2
endef

all: hello_world.bin

hello_world.o: hello_world.c
	$(call c_compiler,$<,$@)

startup64.o: startup64.s
	$(call as_compiler,$<,$@)

hello_world.elf: hello_world.o startup64.o
	$(call linker,$(LD_SCRIPT),$<,$@)

hello_world.bin: hello_world.elf
	$(call elf2bin,$<,$@)

clean:
	rm -f hello_world.bin hello_world.elf startup64.o hello_world.o
