---
title: RtlCopyVolatileMemory Function (Wdm.H)
ms.date: 01/04/2024
description: This article describes the RtlCopyVolatileMemory function (wdm.h).
---

# RtlCopyVolatileMemory function (wdm.h)

> [!IMPORTANT]
> Some information relates to a prerelease product which may be substantially modified before it's commercially released. Microsoft makes no warranties, express or implied, with respect to the information provided here.

**RtlCopyVolatileMemory** function provides [**RtlCopyMemory**](/windows-hardware/drivers/ddi/wdm/nf-wdm-rtlcopymemory) behavior (for example, copying memory from one location to another) in situations where the developer needs to be sure that the copy operation occurs (for example, isn't subject to compiler optimizations).

## Syntax

```cpp
volatile void * RtlCopyVolatileMemory(
    [out] volatile void       *Destination,
    [in]  volatile const void *Source,
    [in]  size_t              Length
);
```

## Parameters

### Destination [out]

A pointer to the starting address of the copied block's destination.

### Source [in]

A pointer to the starting address of the block of memory to copy.

### Length [in]

The size of the block of memory to copy, in bytes.

## Returns

Returns the value of *Destination*.

## Remarks

The **RtlCopyVolatileMemory** function has the following properties:

- The function isn't recognized as a compiler intrinsic so the compiler will never optimize away the call (either entirely or replace the call with an equivalent sequence of instructions). This differs from **RtlCopyMemory** which is subject to various compiler optimizations.

- When the call returns, the data has been copied from *Source* to *Destination*. This functions memory accesses to the *Source* and *Destination* will only be performed within the function (for example, the compiler can't move memory accesses out of this function).

- The function may perform unaligned memory accesses if the platform allows for it.

- The function may access memory locations more than once as part of its copy operation.

- It's similar to **RtlCopyMemory** in that it doesn't support copy operations when *Source* and *Destination* overlap each other.

> [!NOTE]
> This function works on all versions of Windows, not just the latest. You need to consume the latest WDK to get the function declaration from the wdm.h header. You also need the library (volatileaccessk.lib) from the latest WDK. However, the resulting driver will run fine on older versions of Windows.

### Example

```cpp
HEADER MyHeader;

UCHAR RawBuffer[100];

// Ensure that the shared memory (which could be constantly changing)
// is copied in to the local MyHeader variable.
// Note that the compiler is allowed to optimize away calls to
// RtlCopyMemory so those functions are not guaranteed to actually
// make a local copy of data.
//
// RtlCopyVolatileMemory does not handle a source/dest buffer that overlap
// with each other (RtlCopyMemory semantics).
//
// Assume SharedMemory points to virtual memory that is also mapped in an untrusted process.
// Assume that untrusted process is changing the memory contents while you are accessing it.

PVOID SharedMemory;

RtlCopyVolatileMemory(&MyHeader, SharedMemory, sizeof(MyHeader));

if (MyHeader.Size < 100) {

    // Because MyHeader is local and we are guaranteed we actually made
    // a local copy, we can be sure that the "Size" value will not change
    // between the previous bounds check and the below call to RtlFillMemory.
    // If RtlCopyMemory had been used to copy the data, it is possible
    // that a compiler may optimize away the call to RtlCopyMemory and instead fetch
    // the "size" field of MyHeader directly from untrusted memory two times.
    // The first time it would be fetched for the bounds check, and the second
    // time it is fetched is for the call to RtlFillMemory. It is possible the memory
    // could have changed between the two accesses resulting in the size check
    // being ineffective.
    
    RtlFillMemory (RawBuffer, MyHeader.Size, 0);

}
```

## Requirements

**Minimum supported client:** Windows 11, version 24H2

**Header:** wdm.h (include Wdm.h)

**Kernel-mode library:** volatileaccessk.lib

**User-mode library:** volatileaccessu.lib

## Related articles

[Bulk memory volatile accessor functions (v3)](bulk-memory-volatile-accessor-functions-v3.md)

[**RtlCopyMemory**](/windows-hardware/drivers/ddi/wdm/nf-wdm-rtlcopymemory)
