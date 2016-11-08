---
layout: post
title: P/Invoking Native Calls
---

There was a great question the other day. I figured I'd blog about this for others trying to figure out the answer.

> How does one invoke a `SIGABRT` on a device?

For those unfamiliar with what a `SIGABRT` is, here's the definition from [wikipedia](https://en.wikipedia.org/wiki/Unix_signal#POSIX_signals):

> `SIGABRT` is sent by the process to itself when it calls the `abort` libc function, defined in `stdlib.h`. The `SIGABRT` signal can be caught, but it cannot be blocked; if the signal handler returns then all open streams are closed and flushed and the program terminates (dumping core if appropriate). This means that the `abort` call never returns. Because of this characteristic, it is often used to signal fatal conditions in support libraries, situations where the current operation cannot be completed but the main program can perform cleanup before exiting. It is used when an assertion fails.

Here's what the community has to say about it:

[http://stackoverflow.com/questions/3413166/when-does-a-process-get-sigabrt-signal-6](http://stackoverflow.com/questions/3413166/when-does-a-process-get-sigabrt-signal-6)

[http://stackoverflow.com/questions/11161126/what-causes-a-sigabrt-fault](http://stackoverflow.com/questions/11161126/what-causes-a-sigabrt-fault)

Well now that question becomes a bit morphed into:

> How does one invoke a `SIGABRT` signal in Xamarin on both iOS and Android?

Seeing that we already found out the answer via these stack overflow links, we might want to dig in a little further to the actual potential library call we want to use. In our case, we want to look at `abort()`:

[http://www.cplusplus.com/reference/cstdlib/abort/](http://www.cplusplus.com/reference/cstdlib/abort/)

> Aborts the current process, producing an abnormal program termination.

> The function raises the SIGABRT signal (as if raise(SIGABRT) was called). This, if uncaught, causes the program to terminate returning a platform-dependent unsuccessful termination error code to the host environment.

Perfect! This is exactly what we need to call into. The next question is...How do we do that?

Based on the Xamarin iOS documentation, we are introduced into the world of P/Invoke(We can assume the same for Xamarin Android as well):

[https://developer.xamarin.com/guides/ios/advanced_topics/native_interop/#Accessing_C_Methods_from_C](https://developer.xamarin.com/guides/ios/advanced_topics/native_interop/#Accessing_C_Methods_from_C)

There's four steps to doing just this:

- Determine which C function you want to invoke
- Determine its signature
- Determine which library it lives in
- Write the appropriate P/Invoke declaration

Let's take a look at this first step now.

### Determine which C function you want to invoke

We already determined that we need to invoke `abort()`. 

### Determine its signature

The signature for this call is fairly straight forward. It's `void` and it has `0` arguments. We can define the structure as the following:

`static extern void abort ();`

### Determine which library it lives in

This is the tough one. Especially for those of us who have not dwelled into the native library side of things for awhile. Let's use Github as a tool here to see if there's previous examples. My first search is using `DllImport` to see examples of this in the Xamarin.iOS / Mac repository.

[https://github.com/xamarin/xamarin-macios/search?p=2&q=DllImport&type=Code&utf8=%E2%9C%93](https://github.com/xamarin/xamarin-macios/search?p=2&q=DllImport&type=Code&utf8=%E2%9C%93)

Unfortunately that's not what I want. I want to invoke this method from the standard `C` library. Let's refine again, this time with the following query: `DllImport libc`

[https://github.com/xamarin/xamarin-macios/search?utf8=%E2%9C%93&q=DllImport+libc&type=Code](https://github.com/xamarin/xamarin-macios/search?utf8=%E2%9C%93&q=DllImport+libc&type=Code)

Oh perfect! I'm starting to see some `libc` paths. There's two that seem to be useful here:

`[DllImport ("/usr/lib/libc.dylib")]`

and

`[DllImport("libc")]`

However what about Xamarin.Android? Let's do the same thing using our refined query of `DllImport libc`:

[https://github.com/xamarin/xamarin-android/search?utf8=%E2%9C%93&q=DllImport+libc](https://github.com/xamarin/xamarin-android/search?utf8=%E2%9C%93&q=DllImport+libc)

Perfect! It looks like both Xamarin.iOS and Xamarin.Android can use the path of:

`[DllImport("libc")]`

**Note:** You can make use of the following command to see the symbol table

`nm -a <path to .dylib>`

If you need to find dependencies of a `.dylib` you can use:

`otool -L <path to .dylib>`

### Write the appropriate P/Invoke declaration

Now to figure out "How many licks does it take to get to the Tootsie Roll center of a Tootsie Pop?"

A one...A two-Hoo!...A three...

	[DllImport ("libc")]
	static extern void abort ();

**Android:**

Calling this function in the `OnCreate` method:

    11-08 12:48:36.172 D/Mono( 6570): DllImport attempting to load: '/system/lib/libc.so'.
    11-08 12:48:36.172 D/Mono( 6570): DllImport loaded library '/system/lib/libc.so'.
    11-08 12:48:36.172 D/Mono( 6570): DllImport searching in: '/system/lib/libc.so' ('/system/lib/libc.so').
    11-08 12:48:36.172 D/Mono( 6570): Searching for 'abort'.
    11-08 12:48:36.172 D/Mono( 6570): Probing 'abort'.
    11-08 12:48:36.172 D/Mono( 6570): Found as 'abort'.
    11-08 12:48:36.180 I/monodroid-gref( 6570): +g+ grefc 9 gwrefc 0 obj-handle 0x100019/L -> new-handle 0x1004ae/G from thread '(null)'(1)
    11-08 12:48:36.212 F/libc( 6570): Fatal signal 6 (SIGABRT), code -6 in tid 6570 (App23.App23)

**iOS:**

Calling this function in the `FinishedLaunching` method:

    Exception Type:  EXC_CRASH (SIGABRT)
    Exception Codes: 0x0000000000000000, 0x0000000000000000
    Exception Note:  EXC_CORPSE_NOTIFY
    Triggered by Thread:  6


Big thanks to `Brendan Zagaeski` for helping make my initial theory turn into a reality and blog post!