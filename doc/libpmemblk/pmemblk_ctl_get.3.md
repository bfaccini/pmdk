---
draft: false
slider_enable: true
description: ""
disclaimer: "The contents of this web site and the associated <a href=\"https://github.com/pmem\">GitHub repositories</a> are BSD-licensed open source."
aliases: ["pmemblk_ctl_get.3.html"]
title: "libpmemblk | PMDK"
header: "pmemblk API version 1.1"
---

[comment]: <> (SPDX-License-Identifier: BSD-3-Clause)
[comment]: <> (Copyright 2018-2019, Intel Corporation)

[comment]: <> (pmemblk_ctl_get.3 -- man page for libpmemblk CTL)

[NAME](#name)<br />
[SYNOPSIS](#synopsis)<br />
[DESCRIPTION](#description)<br />
[CTL NAMESPACE](#ctl-namespace)<br />
[CTL EXTERNAL CONFIGURATION](#ctl-external-configuration)<br />
[SEE ALSO](#see-also)<br />

# NAME #

_UW(pmemblk_ctl_get),
_UW(pmemblk_ctl_set),
_UW(pmemblk_ctl_exec)
- Query and modify libpmemblk internal behavior (EXPERIMENTAL)

# SYNOPSIS #

```c
#include <libpmemblk.h>

_UWFUNCR2(int, pmemblk_ctl_get, PMEMblkpool *pbp, *name, void *arg,
	=q= (EXPERIMENTAL)=e=)
_UWFUNCR2(int, pmemblk_ctl_set, PMEMblkpool *pbp, *name, void *arg,
	=q= (EXPERIMENTAL)=e=)
_UWFUNCR2(int, pmemblk_ctl_exec, PMEMblkpool *pbp, *name, void *arg,
	=q= (EXPERIMENTAL)=e=)
```

_UNICODE()

# DESCRIPTION #

The _UW(pmemblk_ctl_get), _UW(pmemblk_ctl_set) and _UW(pmemblk_ctl_exec)
functions provide a uniform interface for querying and modifying the internal
behavior of **libpmemblk**(7) through the control (CTL) namespace.

The *name* argument specifies an entry point as defined in the CTL namespace
specification. The entry point description specifies whether the extra *arg* is
required. Those two parameters together create a CTL query. The functions and
the entry points are thread-safe unless
indicated otherwise below. If there are special conditions for calling an entry
point, they are explicitly stated in its description. The functions propagate
the return value of the entry point. If either *name* or *arg* is invalid, -1
is returned.

If the provided ctl query is valid, the CTL functions will always return 0
on success and -1 on failure, unless otherwise specified in the entry point
description.

See more in **pmem_ctl**(5) man page.

# CTL NAMESPACE #

prefault.at_create | rw | global | int | int | - | boolean

If set, every page of the pool will be touched and written to when the pool
is created, in order to trigger page allocation and minimize the performance
impact of pagefaults. Affects only the _UW(pmemblk_create) function.

Always returns 0.

prefault.at_open | rw | global | int | int | - | boolean

If set, every page of the pool will be touched and written to when the pool
is opened, in order to trigger page allocation and minimize the performance
impact of pagefaults. Affects only the _UW(pmemblk_open) function.

Always returns 0.

sds.at_create | rw | global | int | int | - | boolean

If set, force-enables or force-disables SDS feature during pool creation.
Affects only the _UW(pmemblk_create) function. See **pmempool_feature_query**(3)
for information about SDS (SHUTDOWN_STATE) feature.

Always returns 0.

copy_on_write.at_open | rw | global | int | int | - | boolean

If set, pool is mapped in such a way that modifications don't reach the
underlying medium. From the user's perspective this means that when the pool
is closed all changes are reverted. This feature is not supported for pools
located on Device DAX.

Always returns 0.

# CTL EXTERNAL CONFIGURATION #

In addition to direct function call, each write entry point can also be set
using two alternative methods.

The first method is to load a configuration directly from the **PMEMBLK_CONF**
environment variable.

The second method of loading an external configuration is to set the
**PMEMBLK_CONF_FILE** environment variable to point to a file that contains
a sequence of ctl queries.

See more in **pmem_ctl**(5) man page.

# SEE ALSO #

**libpmemblk**(7), **pmem_ctl**(5) and **<https://pmem.io>**
