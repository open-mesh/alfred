#!/usr/bin/make -f
# SPDX-License-Identifier: GPL-2.0
# -*- makefile -*-
#
# Copyright (C) B.A.T.M.A.N. contributors
#
# License-Filename: LICENSES/preferred/GPL-2.0

# alfred-gpsd build
BINARY_NAME = alfred-gpsd
OBJ += alfred-gpsd.o
MANPAGE = man/alfred-gpsd.8

# alfred flags and options
CFLAGS += -pedantic -Wall -W -std=gnu99 -fno-strict-aliasing -MD -MP
CPPFLAGS += -D_GNU_SOURCE

# disable verbose output
ifneq ($(findstring $(MAKEFLAGS),s),s)
ifndef V
	Q_CC = @echo '   ' CC $@;
	Q_LD = @echo '   ' LD $@;
	export Q_CC
	export Q_LD
endif
endif

ifeq ($(origin PKG_CONFIG), undefined)
  PKG_CONFIG = pkg-config
  ifeq ($(shell which $(PKG_CONFIG) 2>/dev/null),)
    $(error $(PKG_CONFIG) not found)
  endif
endif

ifeq ($(origin LIBGPS_CFLAGS) $(origin LIBGPS_LDLIBS), undefined undefined)
  LIBGPS_NAME ?= libgps
  ifeq ($(shell $(PKG_CONFIG) --modversion $(LIBGPS_NAME) 2>/dev/null),)
    $(error No $(LIBGPS_NAME) development libraries found!)
  endif
  LIBGPS_CFLAGS += $(shell $(PKG_CONFIG) --cflags $(LIBGPS_NAME))
  LIBGPS_LDLIBS +=  $(shell $(PKG_CONFIG) --libs $(LIBGPS_NAME)) -lm
endif
CFLAGS += $(LIBGPS_CFLAGS)
LDLIBS += $(LIBGPS_LDLIBS)

# standard build tools
CC ?= gcc
RM ?= rm -f
INSTALL ?= install
MKDIR ?= mkdir -p
COMPILE.c = $(Q_CC)$(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
LINK.o = $(Q_LD)$(CC) $(CFLAGS) $(LDFLAGS) $(TARGET_ARCH)

# standard install paths
PREFIX = /usr/local
SBINDIR = $(PREFIX)/sbin
MANDIR = $(PREFIX)/share/man

# try to generate revision
REVISION= $(shell	if [ -d ../.git ]; then \
				echo $$(git describe --always --dirty --match "v*" |sed 's/^v//' 2> /dev/null || echo "[unknown]"); \
			fi)
ifneq ($(REVISION),)
CPPFLAGS += -DSOURCE_VERSION=\"$(REVISION)\"
endif

# default target
all: $(BINARY_NAME)

# standard build rules
.SUFFIXES: .o .c
.c.o:
	$(COMPILE.c) -o $@ $<

$(BINARY_NAME): $(OBJ)
	$(LINK.o) $^ $(LDLIBS) -o $@

clean:
	$(RM) $(BINARY_NAME) $(OBJ) $(DEP)

install: $(BINARY_NAME)
	$(MKDIR) $(DESTDIR)$(SBINDIR)
	$(MKDIR) $(DESTDIR)$(MANDIR)/man8
	$(INSTALL) -m 0755 $(BINARY_NAME) $(DESTDIR)$(SBINDIR)
	$(INSTALL) -m 0644 $(MANPAGE) $(DESTDIR)$(MANDIR)/man8

# load dependencies
DEP = $(OBJ:.o=.d)
-include $(DEP)

.PHONY: all clean install
