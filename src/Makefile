# SPDX-License-Identifier: BSD-3-Clause
# Copyright 2014-2023, Intel Corporation

#
# src/Makefile -- Makefile for PMDK
#
TOP := $(dir $(lastword $(MAKEFILE_LIST)))..
include $(TOP)/src/common.inc

# core targets
TARGETS = libpmem libpmemblk libpmemlog libpmemobj libpmempool tools libpmem2
# make all targets
ALL_TARGETS = $(TARGETS) common core deps
# all available targets
POSSIBLE_TARGETS = $(TARGETS) common core deps test\
	examples benchmarks

SCOPE_DIRS = $(TARGETS) common core deps

DEBUG_RELEASE_TARGETS = common core deps libpmem libpmemblk libpmemlog libpmemobj\
	libpmempool libpmem2
RELEASE_TARGETS = tools

ifneq ($(BUILD_EXAMPLES),n)
	ALL_TARGETS += examples
	RELEASE_TARGETS += examples
endif

ifneq ($(BUILD_BENCHMARKS),n)
	ALL_TARGETS += benchmarks
	RELEASE_TARGETS += benchmarks
endif

CLEAN_TARGETS = $(POSSIBLE_TARGETS)
CLOBBER_TARGETS = $(POSSIBLE_TARGETS)
CSTYLE_TARGETS = $(POSSIBLE_TARGETS)
INSTALL_TARGETS = $(TARGETS)
SPARSE_TARGETS = $(POSSIBLE_TARGETS)
EXAMPLES_TARGETS = $(TARGETS)

HEADERS_DESTDIR = $(DESTDIR)$(includedir)
HEADERS_INSTALL = include/libpmem.h\
		include/libpmemobj.h include/libpmempool.h\
		include/libpmemblk.h include/libpmemlog.h\
		include/libpmem2.h

OBJ_HEADERS_INSTALL = include/libpmemobj/*.h
PMEM2_HEADERS_INSTALL = include/libpmem2/*.h

PKG_CONFIG_DESTDIR = $(DESTDIR)$(pkgconfigdir)
PKG_CONFIG_COMMON = common.pc
PKG_CONFIG_FILES = libpmem.pc libpmemobj.pc libpmemlog.pc libpmemblk.pc\
		libpmempool.pc libpmem2.pc
PMREORDER_BIN = $(DESTDIR)$(bindir)
PMREORDER_SRC = tools/pmreorder
PMREORDER_FILES = $(PMREORDER_SRC)/*.py
PMREORDER_DESTDIR = $(DESTDIR)$(datarootdir)/pmreorder
PMREORDER_ROOTDIR = $(datarootdir)/pmreorder

rwildcard=$(strip $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2)\
	$(filter $(subst *,%,$2),$d)))

SCOPE_SRC_DIRS = $(SCOPE_DIRS) include
SCOPE_HDR_DIRS = $(SCOPE_DIRS) include

SCOPE_SRC_FILES = $(foreach d, $(SCOPE_SRC_DIRS), $(wildcard $(d)/*.c))
SCOPE_HDR_FILES = $(foreach d, $(SCOPE_HDR_DIRS), $(wildcard $(D)/*.h))
SCOPEFILES = $(SCOPE_SRC_FILES) $(SCOPE_HDR_FILES)

# include/lib*.h - skip include/pmemcompat.h
HEADERS =\
	$(foreach f, $(wildcard\
		freebsd/include/*.h\
		freebsd/include/*/*.h\
		include/lib*.h\
		include/libpmemobj/*.h\
		include/libpmem2/*.h\
		windows/include/*.h\
		windows/include/*/*.h\
		), $(f))

ifneq ($(filter 1 2, $(CSTYLEON)),)
TMP_HEADERS := $(addprefix debug/, $(addsuffix tmp, $(HEADERS)))
endif

SCRIPTS = $(call rwildcard,,*.sh)

debug/%.htmp: %.h
	$(call check-cstyle, $<, $@)

debug/%.hpptmp: %.hpp
	$(call check-cstyle, $<, $@)

all: $(TMP_HEADERS) $(ALL_TARGETS)
install: $(INSTALL_TARGETS:=-install)
uninstall: $(INSTALL_TARGETS:=-uninstall)
clean: $(CLEAN_TARGETS:=-clean)
clobber: $(CLOBBER_TARGETS:=-clobber)
cstyle: $(CSTYLE_TARGETS:=-cstyle)
format: $(CSTYLE_TARGETS:=-format)
examples benchmarks: $(EXAMPLES_TARGETS)
benchmarks: examples
sparse: $(SPARSE_TARGETS:=-sparse)

custom_build = $(DEBUG)$(OBJDIR)

tools: libpmem libpmemblk libpmemlog libpmemobj libpmempool
libpmemblk libpmemlog libpmemobj: libpmem
libpmempool: libpmemblk
benchmarks test tools: common core

pkg-cfg-common:
	@printf "version=%s\nlibdir=%s\nprefix=%s\nrasdeps=%s\n" "$(SRCVERSION)" "$(libdir)" "$(prefix)" "$(LIBNDCTL_PKG_CONFIG_DEPS_VAR)" > $(PKG_CONFIG_COMMON)

$(PKG_CONFIG_COMMON): pkg-cfg-common

%.pc: $(PKG_CONFIG_COMMON) $(TOP)/utils/%.pc.in
	@echo Generating $@
	@cat $(PKG_CONFIG_COMMON) > $@
	@cat $(TOP)/utils/$@.in  >> $@

pkg-config: $(PKG_CONFIG_FILES)

%-install: %
	$(MAKE) -C $^ install
	$(MAKE) -C $^ install DEBUG=1

$(eval $(call sub-target,$(INSTALL_TARGETS),uninstall,y))
$(eval $(call sub-target,$(CLEAN_TARGETS),clean,y))
$(eval $(call sub-target,$(CLOBBER_TARGETS),clobber,y))
$(eval $(call sub-target,$(CSTYLE_TARGETS),cstyle,n))
$(eval $(call sub-target,$(CSTYLE_TARGETS),format,n))
$(eval $(call sub-target,$(SPARSE_TARGETS),sparse,n))

$(DEBUG_RELEASE_TARGETS):
	$(MAKE) -C $@
ifeq ($(custom_build),)
	$(MAKE) -C $@ DEBUG=1
endif

$(RELEASE_TARGETS):
	$(MAKE) -C $@

df:
	@echo ====================================================================================
	@if [ -e test/testconfig.sh ]; then \
		bash -c '. test/testconfig.sh; \
		if [ -n "$$PMEM_FS_DIR" ]; then df -h "$$PMEM_FS_DIR"; stat -f "$$PMEM_FS_DIR"; fi; \
		if [ -n "$$NON_PMEM_FS_DIR" ]; then df -h "$$NON_PMEM_FS_DIR"; stat -f "$$NON_PMEM_FS_DIR";fi'; \
	fi
	@echo ====================================================================================

test: all
	$(MAKE) -C test test

check pcheck pycheck: test df
	$(MAKE) -C test $@

# Re-generate pkg-config files on 'make install' (not on 'make all'),
# to handle the case when prefix is specified only for 'install'.
# Clean up generated files when done.
install: all pkg-config
	install -d $(HEADERS_DESTDIR)
	install -p -m 0644 $(HEADERS_INSTALL) $(HEADERS_DESTDIR)
	install -d $(HEADERS_DESTDIR)/libpmemobj
	install -d $(HEADERS_DESTDIR)/libpmem2
	install -p -m 0644 $(OBJ_HEADERS_INSTALL) $(HEADERS_DESTDIR)/libpmemobj
	install -p -m 0644 $(PMEM2_HEADERS_INSTALL) $(HEADERS_DESTDIR)/libpmem2
	install -d $(PKG_CONFIG_DESTDIR)
	install -p -m 0644 $(PKG_CONFIG_FILES) $(PKG_CONFIG_DESTDIR)
	install -d $(PMREORDER_DESTDIR)
	install -p -m 0644 $(PMREORDER_FILES) $(PMREORDER_DESTDIR)
	@echo "#!/bin/sh" > $(PMREORDER_SRC)/pmreorder
	@echo "exec python3 $(PMREORDER_ROOTDIR)/pmreorder.py $(SRCVERSION) \$$*" >> $(PMREORDER_SRC)/pmreorder
	install -d $(PMREORDER_BIN)
	install -p $(PMREORDER_SRC)/pmreorder $(PMREORDER_BIN)
	$(RM) $(PKG_CONFIG_FILES)

uninstall:
	$(foreach f, $(HEADERS_INSTALL), $(RM) $(HEADERS_DESTDIR)/$(notdir $(f)))
	$(foreach f, $(OBJ_HEADERS_INSTALL), $(RM) $(HEADERS_DESTDIR)/libpmemobj/$(notdir $(f)))
	$(foreach f, $(PMEM2_HEADERS_INSTALL), $(RM) $(HEADERS_DESTDIR)/libpmem2/$(notdir $(f)))
	$(foreach f, $(PKG_CONFIG_FILES), $(RM) $(PKG_CONFIG_DESTDIR)/$(notdir $(f)))
	$(foreach f, $(PMREORDER_FILES), $(RM) $(PMREORDER_DESTDIR)/$(notdir $(f)))
	$(RM) $(PMREORDER_BIN)/pmreorder

cstyle:
	$(STYLE_CHECK) check $(HEADERS)
	$(CHECK_SHEBANG) $(SCRIPTS)

format:
	$(STYLE_CHECK) format $(HEADERS)

cscope:
	cscope -q -b $(SCOPEFILES)
	ctags -e $(SCOPEFILES)

clean-here:
	$(RM) tags cscope.in.out cscope.out cscope.po.out *.pc $(TMP_HEADERS)

clean: clean-here

clobber: clean-here

.PHONY: all install uninstall clean clobber cstyle format test check pcheck\
	cscope $(ALL_TARGETS)\
	pkg-config clean-here pkg-cfg-common
