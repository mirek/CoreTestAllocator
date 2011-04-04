# Core Test Allocator Reference

Declared in | `CoreTestAllocator.h`
Copyright   | 2011 Mirek Rusin <mirek [at] me [dot] com>
     
## Overview

Test allocator helps spot memory leaks in your Core Foundation based frameworks.

## Functions by Task

### Creating Allocator

* [TestAllocatorCreate](#TestAllocatorCreate)

### Getting Information on Allocated Addresses and Backtraces

* [TestAllocatorGetAddresses](#TestAllocatorGetAddresses)
* [TestAllocatorGetBacktracesWithAddress](#TestAllocatorGetBacktracesWithAddress)
* [TestAllocatorCreateTraceString](#TestAllocatorCreateTraceString)
* [TestAllocatorCreateAddressesAndBacktracesDictionary](#TestAllocatorCreateAddressesAndBacktracesDictionary)
* [TestAllocatorPrintAddressesAndBacktraces](#TestAllocatorPrintAddressesAndBacktraces)

---

## Functions

<a name="TestAllocatorCreate"></a>
### TestAllocatorCreate

Creates test allocator.

<pre>
CFAllocatorRef TestAllocatorCreate (
  void
);
</pre>

#### Return Value

New test allocator. To release allocator use `CFRelease(allocator)`.

<a name="TestAllocatorGetAddresses"></a>
### TestAllocatorGetAddresses

Get currently allocated addresses.

<pre>
CFMutableArrayRef TestAllocatorGetAddresses(
  CFAllocatorRef allocator
);
</pre>

#### Return Value

An array of currently allocated addresses.

<a name="TestAllocatorGetBacktracesWithAddress"></a>
### TestAllocatorGetBacktracesWithAddress

Get a backtrace for the address.

<pre>
CFMutableArrayRef TestAllocatorGetBacktracesWithAddress(
  CFAllocatorRef allocator,
  CFStringRef address
);
</pre>

#### Return Value

A backtrace for the address.

<a name="TestAllocatorCreateTraceString"></a>
### TestAllocatorCreateTraceString

<pre>CFStringRef TestAllocatorCreateTraceString (
  void
);</pre>

<a name="TestAllocatorCreateAddressesAndBacktracesDictionary"></a>
### TestAllocatorCreateAddressesAndBacktracesDictionary

<pre>CFMutableDictionaryRef TestAllocatorCreateAddressesAndBacktracesDictionary (
  CFAllocatorRef allocator
);</pre>

<a name="TestAllocatorPrintAddressesAndBacktraces"></a>
### TestAllocatorPrintAddressesAndBacktraces

<pre>void TestAllocatorPrintAddressesAndBacktraces (
  CFAllocatorRef allocator
);</pre>

---

## Usage

In your project, create Unit Test Bundle if you haven't already.

Then add submodule to the Test Bundle the project:

    git submodule init
    git submodule add git://github.com/mirek/CoreTestAllocator.git YourProjectTests/CoreTestAllocator

Drag and drop `CoreTestAllocator.h` and `CoreTestAllocator.c` files into `YourProjectTests/Supporting Files` group in Xcode 4.

You're ready now to start using the allocator, in your test case file you can add:

    #import <SenTestingKit/SenTestingKit.h>
    #import "CoreTestAllocator/CoreTestAllocator.h"
    #import "CoreOSC.h"

    @interface CoreOSCTests : SenTestCase {
    @private
      CFAllocatorRef allocator;
    }

    @end

And the 

    #import "CoreOSCTests.h"

    @implementation CoreOSCTests

    - (void) setUp {
      [super setUp];

      allocator = TestAllocatorCreate();
    }

    - (void) tearDown {
      STAssertTrue(TestAllocatorGetAllocationsCount(allocator) > 0, @"Allocations count should be more than 0");
      STAssertTrue(TestAllocatorGetDeallocationsCount(allocator) > 0, @"Deallocations count should be more than 0");
      STAssertEquals(TestAllocatorGetAllocationsCount(allocator), TestAllocatorGetDeallocationsCount(allocator), @"Allocations/deallocations mismatch");

      if (TestAllocatorGetAllocationsCount(allocator) != TestAllocatorGetDeallocationsCount(allocator))
        TestAllocatorPrintAddressesAndBacktraces(allocator);

      CFRelease(allocator);

      [super tearDown];
    }

    - (void) testExample {
      OSCRef osc = OSCCreate(allocator, CFSTR("127.0.0.1"), CFSTR("60000"));
      CFIndex testFloatIndex = OSCAddressesAppendWithString(osc, CFSTR("/test/float"));
      OSCSendFloat(osc, testFloatIndex, 3.14);
      OSCRelease(osc);
    }

    @end

For each non-deallocated address `TestAllocatorPrintAddressesAndBacktraces(allocator)` prints the stack trace similar to:

    0x100504390 address not deallocated, allocated from 1 place(s):
    0   CoreOSCTests                        0x00000001000eef31 TestAllocatorCreateTraceString + 97
    1   CoreOSCTests                        0x00000001000eea8b TestAllocatorAllocateCall + 107
    2   CoreOSCTests                        0x00000001000ef840 OSCCreate + 80
    3   CoreOSCTests                        0x00000001000ee6ed -[CoreOSCTests testExample] + 77
    4   CoreFoundation                      0x00007fff80fdf96c __invoking___ + 140
    5   CoreFoundation                      0x00007fff80fdf83d -[NSInvocation invoke] + 141
    6   SenTestingKit                       0x0000000100487781 -[SenTestCase invokeTest] + 48
    7   SenTestingKit                       0x000000010048787e -[SenTestCase performTest:] + 177
    8   SenTestingKit                       0x000000010048721c -[SenTest run] + 68
    9   SenTestingKit                       0x000000010048a599 -[SenTestSuite performTest:] + 116
    10  SenTestingKit                       0x000000010048721c -[SenTest run] + 68
    11  SenTestingKit                       0x000000010048a599 -[SenTestSuite performTest:] + 116
    12  SenTestingKit                       0x000000010048721c -[SenTest run] + 68
    13  SenTestingKit                       0x000000010048952b +[SenTestProbe runTests:] + 151
    14  otest                               0x0000000100001ce8 0x0 + 4294974696
    15  otest                               0x00000001000020d8 0x0 + 4294975704
    16  otest                               0x0000000100001a00 0x0 + 4294973952
    17  otest                               0x00000001000019b4 0x0 + 4294973876
    18  ???                                 0x0000000000000006 0x0 + 6

Where the first two lines can be ignored. The third one shows that the `OSCCreate` function have allocated `0x100504390` address which
has never been released.
