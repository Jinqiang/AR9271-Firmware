# Makefile for XTOS in <xtensa_tools_root>/xtensa-elf/src/xtos

# Copyright (c) 2000-2007 by Tensilica Inc.  ALL RIGHTS RESERVED.
# These coded instructions, statements, and computer programs are the
# copyrighted works and confidential proprietary information of Tensilica Inc.
# They may not be modified, copied, reproduced, distributed, or disclosed to
# third parties in any manner, medium, or form, in whole or in part, without
# the prior written consent of Tensilica Inc.

#
#  This makefile assumes GNU make features
#
#  Invoke this Makefile like this:
#	cd <some build directory>
#	make -f <this file> \
#		MAKEFILE_SRC=<this file> \
#		XTENSA_TOOLS_ROOT=<path to Xtensa Tools> \
#		XTENSA_ROOT=<path to Xtensa core package> \
#		INSTLIBDIR=<path to installation directory>
#

ifndef XTENSA_TOOLS_ROOT
$(error Please set XTENSA_TOOLS_ROOT to the path to Xtensa Tools)
endif
#  NOTE:  For now, we assume $(XTENSA_TOOLS_ROOT)/bin is on the PATH.
ifndef XTENSA_ROOT
$(error Please set XTENSA_ROOT to the path to your specific Xtensa core package)
endif

#ifndef INSTLIBDIR
#$(error Please set INSTLIBDIR to the path where libraries and objects are installed)
#INSTLIBDIR = $(call fixpath,$(XTENSA_ROOT)/xtensa-elf/arch/lib)
#endif

#  Select the specified Xtensa configuration:
export XTENSA_SYSTEM = $(XTENSA_ROOT)/config
export XTENSA_CORE = default

include $(XTENSA_TOOLS_ROOT)/misc/defs.mk

ifndef SRCDIR
$(error Please set MAKEFILE_SRC to the path to the XTOS Makefile.src)
endif

#  Native commands like $(MKPATH) and $(CP) need native directory separators:
#fixpath = $(subst /,$S,$(1))

XTCC  = $(CC_FOR_TARGET)
XTAR  = xt-ar

.PHONY: all clean

# Compilation flags
ASFLAGS  = -O2 -g -mlongcalls
CFLAGS   = -O2 -g -mlongcalls

#  File splitting:
#  Compile "split" files in separate parts that can be linked independently.
#  This allows keeping multiple related functions/etc together in one file
#  without forcing linking of unused functions (the GNU linker takes entire
#  object files; it does not currently provide automated dead code removal).
#  Files are split with "#[el]if defined(__SPLIT__<tag>)" lines using normal
#  C preprocessor syntax, where each <tag> must be unique and consist of
#  lowercase alphanumeric and underscore characters only (no dash etc).
#  The makefile function $(split_objs ...) returns the set of objects that
#  result for a given list of splittable source files; each ends in a -.o
#  suffix recognized by special rules further below.
#
split_objs = $(shell cd $(SRCDIR) && $(PERL) -ne '/__SPLIT__(\w+)/ and $$h{$$ARGV."--".$$1}++;\
 END {foreach (sort keys %h) {s/\.(.)--/-$$1--/; print "$$_-.o\n";}}' $(1))

#  Vectors, handlers, and other code to build:
PERLEVEL_INTVECTOR_OBJS  =   $(foreach N,2 3 4 5 6,int-vector-level$(N).o)
PERLEVEL_INTHANDLER_OBJS =   $(foreach N,2 3 4 5 6,int-handler-level$(N).o)
PERLEVEL_INTINITLEV_OBJS = $(foreach N,1 2 3 4 5 6,int-init-level$(N).o)
XTOS_COMMON_OBJS = \
	reset-vector-unpack.o \
	user-vector.o \
	nmi-vector.o \
	window-vectors.o \
	reloc-vectors.o \
	$(call split_objs, memerror-vector.S) \
	$(PERLEVEL_INTVECTOR_OBJS) \
	exc-alloca-handler.o \
	exc-return.o \
	exc-sethandler.o \
	exc-syscall-handler.o \
	exc-syscall-c-handler.o \
	exc-table.o \
	exit.o \
	init.o \
	int-lowpri-dispatcher.o \
	int-sethandler.o \
	intlevel-set.o \
	intlevel-setmin.o \
	intlevel-restore.o \
	ints-on.o \
	ints-off.o \
	switch_context.o \
	$(call split_objs, deprecated.S tiny-refs.S) \
	$(PERLEVEL_INTHANDLER_OBJS) \
	$(PERLEVEL_INTINITLEV_OBJS)
#  These objects have simulator and board variants:
XTOS_SIMBOARD_OBJS = \
	debug-vector.o \
	double-vector.o \
	kernel-vector.o \
	interrupt-table.o \
	exc-unhandled.o \
	exc-c-wrapper-handler.o \
	memep-initrams.o \
	memep-enable.o
XTOS_BOARD_OBJS = $(XTOS_SIMBOARD_OBJS)
XTOS_SIM_OBJS = $(XTOS_SIMBOARD_OBJS:.o=-sim.o)

#  Optimize a few things for size rather than speed:
#FLAGS_exc-syscall-handler = -Os --no-target-align

TARGETS = \
	_vectors.o \
	crt0-app.o \
	crt1-boards.o \
	crt1-tiny.o \
	crt1-sim.o \
	libhandlers-sim.a \
	libhandlers-board.a \
	libhandlers-min.a \
	libhandlers-null.a \
	libnomovsp.a \
	libnosyscall.a \
	libhandler-reset.a
ifdef TENSILICA_INTERNAL
TARGETS += libhandler-reset-mp.a
endif

all: $(TARGETS)


#  Here's an example of how one could put a selected set of
#  objects into a specific section:
#
#XTOSSECFLAGS = \
#	-mrename-section-.text=.xtos.text \
#	-mrename-section-.literal=.xtos.literal \
#	-mrename-section-.rodata=.xtos.rodata \
#	-mrename-section-.data=.xtos.data
#$(XTOS_SIM_OBJS) : XTFLAGS = $(XTOSSECFLAGS)

libhandlers-sim.a: $(XTOS_SIM_OBJS) $(XTOS_COMMON_OBJS)

libhandlers-board.a: $(XTOS_BOARD_OBJS) $(XTOS_COMMON_OBJS)

libhandlers-min.a: user-vector-min.o tiny-refs-min.o

libhandlers-null.a: $(call split_objs, null-vectors.S) _vectors.o

libnomovsp.a: null-alloca.o

libnosyscall.a: null-syscall.o

$(PERLEVEL_INTVECTOR_OBJS): int-vector-level%.o: int-vector.S
	$(XTCC) -c $(ASFLAGS) $(FLAGS_int-vector-level$*) $(XTFLAGS) -D_INTERRUPT_LEVEL=$* -o $@ $<

$(PERLEVEL_INTHANDLER_OBJS): int-handler-level%.o: int-handler.S
	$(XTCC) -c $(ASFLAGS) $(FLAGS_int-handler-level$*) $(XTFLAGS) -D_INTERRUPT_LEVEL=$* -o $@ $<

$(PERLEVEL_INTINITLEV_OBJS): int-init-level%.o: int-initlevel.S
	$(XTCC) -c $(ASFLAGS) $(FLAGS_int-init-level$*) $(XTFLAGS) -D_INTERRUPT_LEVEL=$* -o $@ $<


#  For split-source rules, determine various things from $* (extended basename):
#
SPLIT_SRC = $(patsubst %/,%,$(dir $(subst -c--,.c/,$(subst -S--,.S/,$*))))
SPLIT_FLAGS = $(FLAGS_$(basename $(SPLIT_SRC))) \
		-D__SPLIT__$(notdir $(subst --,/,$*))

#  (Okay we cheat a bit, CFLAGS matches ASFLAGS so this works; it's also simpler:)
%-.o:
	$(XTCC) -c $(CFLAGS) $(SPLIT_FLAGS) $(XTFLAGS) -o $@ $(SRCDIR)/$(SPLIT_SRC)
%--sim.o:
	$(XTCC) -c $(CFLAGS) $(SPLIT_FLAGS) $(XTFLAGS) -DSIMULATOR -o $@ $(SRCDIR)/$(SPLIT_SRC)


%.o: %.S
	$(XTCC) -c $(ASFLAGS) $(FLAGS_$*) $(XTFLAGS) $<

%.o: %.c
	$(XTCC) -c $(CFLAGS)  $(FLAGS_$*) $(XTFLAGS) $<

%-sim.o: %.S
	$(XTCC) -c $(ASFLAGS) $(FLAGS_$*) $(XTFLAGS) -DSIMULATOR -o $@ $<

%-sim.o: %.c
	$(XTCC) -c $(CFLAGS)  $(FLAGS_$*) $(XTFLAGS) -DSIMULATOR -o $@ $<

#  Explicit rule for crt1-sim.o to avoid using %-sim.o rule above:
crt1-sim.o: crt1-sim.S
	$(XTCC) -c $(ASFLAGS) $(FLAGS_$*) $(XTFLAGS) $<


# When building the reset vector, leave the literals in
# the text section, so that the reset vector can be placed
# at address 0 (there's no lower address for the literals).
#
#FLAGS_reset-vector = -mtext-section-literals
#
reset-vector.o: reset-vector.S
	$(XTCC) -c $(ASFLAGS) $(XTFLAGS) -mtext-section-literals -o $@ $<
reset-vector-unpack.o: reset-vector.S
	$(XTCC) -c $(ASFLAGS) $(XTFLAGS) -mtext-section-literals -DXTOS_UNPACK -o $@ $<
reset-vector-mp.o: reset-vector.S
	$(XTCC) -c $(ASFLAGS) $(XTFLAGS) -mtext-section-literals -DXTOS_MP -o $@ $<
#
#  These libraries provide alternative reset vectors:
#  Without any unpacking:
libhandler-reset.a: reset-vector.o
#  With experimental MP unpacking:
libhandler-reset-mp.a: reset-vector-mp.o

%.a:
	-$(RM) $@
	$(XTAR) rs $@ $^

#install:
#	-$(MKPATH) $(INSTLIBDIR)
#	$(CP) $(TARGETS) $(INSTLIBDIR)

clean:
	-$(RM) *.o *.a

# NOTE: Header file dependencies not specified!

