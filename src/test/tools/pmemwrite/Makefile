# SPDX-License-Identifier: BSD-3-Clause
# Copyright 2014-2023, Intel Corporation
#
# Makefile -- Makefile for pmemwrite tool
#

TOP = ../../../..

TARGET = pmemwrite

OBJS = write.o

LIBPMEM=y
LIBPMEMBLK=y
LIBPMEMLOG=y
TOOLS_COMMON=y
TOOLS_PMEMPOOL_COMMON=y
LIBPMEMBLK_PRIV=btt_info_convert2h

include $(TOP)/src/tools/Makefile.inc
