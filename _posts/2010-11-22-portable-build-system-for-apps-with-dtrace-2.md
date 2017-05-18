---
id: 93
title: portable build system for apps with DTrace
date: 2010-11-22T22:14:17+00:00
author: mharsch
layout: post
guid: http://blog.harschsystems.com/?p=93
permalink: /2010/11/22/portable-build-system-for-apps-with-dtrace-2/
categories:
  - How-To
tags:
  - DTrace
---
Having [recently been bitten](http://bugs.php.net/bug.php?id=53338) by this issue, I thought I&#8217;d share a trivial example showing how to use Autoconf to stage a platform-independent build system for DTrace-enabled applications.

Before going any further, I should call out some pre-requisite reading: [Adam](http://dtrace.org/blogs/ahl) covers [usage of USDT probes here](http://dtrace.org/blogs/ahl/2006/05/08/user-land-tracing-gets-better-and-better/), while [Dave](http://dtrace.org/blogs/dap/) digs deeper [into the subject here](http://blogs.sun.com/dap/entry/writing_a_dtrace_usdt_provider){.broken_link}. Autoconf is not intuitive, so if you haven&#8217;t worked with it before (as I hadn&#8217;t until a few days ago), you should read chapters 1 and 2 of John Calcote&#8217;s excellent online reference [Autotools: a practitioners guide&#8230;](http://www.freesoftwaremagazine.com/books/autotools_a_guide_to_autoconf_automake_libtool).

For those making use of USDT probes in applications, there are some differences in the steps needed to bake-in DTrace support at compile time (depending on the underlying platform). In this post I&#8217;ll discuss how to use GNU Autoconf to deliver a portable build system that will &#8216;do the right thing&#8217; on supported platforms, or allow you to disable DTrace on unsupported platforms.

**Differences between Solaris and Mac OS X**
  
Solaris requires that a provider object file be generated (using dtrace(1M) with the -G flag) and linked to the final executable. As explained in [Adam&#8217;s post](http://dtrace.org/blogs/ahl/2006/05/08/user-land-tracing-gets-better-and-better/), the steps are as follows:

    
    $ dtrace -h -s my_provider.d
    $ gcc -c my_app.c 
    $ dtrace -G -s my_provider.d my_app.o 
    $ gcc -o my_app my_provider.o my_app.o
    

OS X doesn&#8217;t require this intermediate step (in fact, [dtrace(1M)](http://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man1/dtrace.1.html) on OS X doesn&#8217;t have a &#8216;-G&#8217; flag). So, on a Mac, the steps for accomplishing the same thing look like this:

    
    $ dtrace -h -s my_provider.d
    $ gcc -o my_app my_app.c
    

Of course, we can split out the compiling and linking steps (perhaps more realistic for anything beyond a toy example):

    
    $ dtrace -h -s my_provider.d
    $ gcc -c -o my_app.c
    $ gcc -o my_app my_app.o
    

Looking at what needs to be run on each platform, we can see that our Makefiles will have to be different. A `Makefile` for Solaris might look like this:

    
    my_app: my_app.o my_provider.o
            gcc -o my_app my_app.o my_provider.o
    my_app.o: my_app.c
            gcc -c -o my_app.o my_app.c
    my_provider.h: my_provider.d
            dtrace -h -s my_provider.d
    my_provider.o: my_app.o
            dtrace -G -s my_provider.d my_app.o
    

Whereas a Mac OS X `Makefile` might look like this.

    
    my_app: my_app.o
            gcc -o my_app my_app.o
    my_app.o: my_app.c
            gcc -c -o my_app.o my_app.c
    my_provider.h: my_provider.d
            dtrace -h -s my_provider.d
    

Enter GNU Autoconf. Autoconf allows us to specify a template file (`Makefile.in`) which has placeholders for certain parts that can vary across target platforms or compile-time configuration options. The placeholders are filled in by the `configure` script, which checks the environment and specified options before generating a `Makefile`. So, in or example here: `Makefile.in` needs to be general enough to allow for the inclusion (or exclusion) of the &#8216;`dtrace -G ...`&#8216; line, as well provide the option of listing the provider object file `provider.o` on the final linker line. Here&#8217;s one way to meet these requirements in `Makefile.in`:

    
    PROVIDER_DESC  = my_provider.d
    PROVIDER_HDR   = my_provider.h
    PROVIDER_OBJ   = my_provider.o
    MAIN_OBJ       = my_app.o
    OTHER_OBJS     = @OTHER_OBJS@
    GENHDRS        = @GENHDRS@
    DTFLAG         = @DTFLAG@
    
    all: my_app
    
    my_app: $(MAIN_OBJ) $(OTHER_OBJS)
            gcc $(DTFLAG) -o my_app $?
    
    $(MAIN_OBJ): $(GENHDRS) my_app.c
            gcc $(DTFLAG) -c -o $@ my_app.c
    
    $(PROVIDER_HDR): $(PROVIDER_DESC)
            dtrace -h -s $?
    
    $(PROVIDER_OBJ): $(MAIN_OBJ)
            dtrace -G -s $(PROVIDER_DESC) $?
    

The variables surrounded by &#8216;@&#8217; signs will be replaced by the `configure` script in it&#8217;s output `Makefile`. Depending on the needs of the target platform, these variables may contain valid values, or may be empty. For instance, a Solaris host will need &#8216;`$OTHER_OBJS`&#8216; to include &#8216;`$(PROVIDER_OBJ)`&#8216; so that the main target depends on the the provider object target (triggering it to be built). A Mac on the other hand, will need `$OTHER_OBJS` to be empty so that the `$(PROVIDER_OBJ)` target is not built.

The `$(DTFLAG)` variable holds a C preprocessor flag &#8216;-DHAVE_DTRACE&#8217; &#8230; or not. This allows DTrace probe code to be turned on/off in the compiled program like this:

    
    #ifdef HAVE_DTRACE MYAPP_TEST_PROBE("Hello from DTrace.n");
    #endif /* HAVE_DTRACE */
    

The `configure` script is generated by `autoconf` (or `autoreconf`) from the `configure.ac` file. This is where we add the &#8216;&#8211;enable-dtrace&#8217; flag, and the logic needed to set the substitution variables appropriately (according to the build host type). See the [Autoconf reference](http://www.freesoftwaremagazine.com/books/autotools_a_guide_to_autoconf_automake_libtool) mentioned earlier for coverage of the AC_ macros used here.

    
    AC_INIT([my_app], [0.1])
    AC_CONFIG_FILES([Makefile])
    AC_CONFIG_SRCDIR([my_app.c])
    
    dnl The GENHDRS variable will be used to determine whether to generate the
    dnl provider header file (only if DTrace is enabled).
    GENHDRS=""
    
    dnl The DTFLAG variable will determine if "-DHAVE_DTRACE" is passed
    dnl to the C Preprocessor at compile time (only if DTrace is enabled).
    DTFLAG=""
    
    AC_ARG_ENABLE([dtrace],
      [  --enable-dtrace     enable built-in DTrace USDT probes],
      [dtrace_requested=${enableval}],
      [dtrace_requested="no"])
    
    if test ${dtrace_requested} = "yes"; then
      echo "*******************************"
      echo "DTrace functionality requested."
      echo "*******************************"
      AC_CHECK_HEADERS(
        [sys/sdt.h], [dtrace_supported=yes],
        [dtrace_supported=no]
      )
      if test "${dtrace_supported}" = "yes"; then
        DTFLAG="-DHAVE_DTRACE"
        GENHDRS="$(PROVIDER_HDR)"
        dtrace_enabled=yes
        echo "***************"
        echo "DTrace engaged."
        echo "***************"
      else
        AC_MSG_ERROR([No DTrace here.])
      fi
    else
      dtrace_enabled=no
    fi
    
    AC_SUBST(GENHDRS)
    AC_SUBST(DTFLAG)
    
    OTHER_OBJS=""
    
    AC_CANONICAL_HOST
    case $host in
      *solaris*)
          dnl Add the provider object to OTHER_OBJS so it will be built
          if test ${dtrace_enabled} = "yes"; then
            OTHER_OBJS="$(PROVIDER_OBJ)"
          fi
        ;;
      *darwin*)
        dnl No extra modification needed here
        ;;
    esac
    
    AC_SUBST(OTHER_OBJS)
    
    AC_OUTPUT
    

With all this in place (plus a couple helper scripts copied over from automake), we can now run &#8216;`./configure`&#8216; and &#8216;`./configure --enable-dtrace`&#8216; on both platforms &#8211; giving us correctly formatted Makefiles for our current platform.

    
    ./configure --enable-dtrace
    *******************************
    DTrace functionality requested.
    *******************************
    checking for gcc... gcc
    checking for C compiler default output file name... a.out
    checking whether the C compiler works... yes
    checking whether we are cross compiling... no
    checking for suffix of executables...
    checking for suffix of object files... o
    checking whether we are using the GNU C compiler... yes
    checking whether gcc accepts -g... yes
    checking for gcc option to accept ISO C89... none needed
    checking how to run the C preprocessor... gcc -E
    checking for grep that handles long lines and -e... /opt/local/bin/ggrep
    checking for egrep... /opt/local/bin/ggrep -E
    checking for ANSI C header files... yes
    checking for sys/types.h... yes
    checking for sys/stat.h... yes
    checking for stdlib.h... yes
    checking for string.h... yes
    checking for memory.h... yes
    checking for strings.h... yes
    checking for inttypes.h... yes
    checking for stdint.h... yes
    checking for unistd.h... yes
    checking sys/sdt.h usability... yes
    checking sys/sdt.h presence... yes
    checking for sys/sdt.h... yes
    ***************
    DTrace engaged.
    ***************
    checking build system type... i386-pc-solaris2.11
    checking host system type... i386-pc-solaris2.11
    configure: creating ./config.status
    config.status: creating Makefile
    
    
    mac-host$ ./configure --enable-dtrace
    *******************************
    DTrace functionality requested.
    *******************************
    checking for gcc... gcc
    snip
    checking sys/sdt.h usability... yes
    checking sys/sdt.h presence... yes
    checking for sys/sdt.h... yes
    ***************
    DTrace engaged.
    ***************
    checking build system type... i386-apple-darwin10.5.0
    checking host system type... i386-apple-darwin10.5.0
    configure: creating ./config.status
    config.status: creating Makefile
    
    mac-host$ make dtrace -h -s my_provider.d
    gcc -DHAVE_DTRACE -c -o my_app.o my_app.c
    gcc -DHAVE_DTRACE -o my_app my_app.o
    mac-host$