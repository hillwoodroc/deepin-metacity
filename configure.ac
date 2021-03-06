dnl **************************************************************************
dnl Define version info
dnl **************************************************************************

m4_define([m_major_version], [3])
m4_define([m_minor_version], [18])
m4_define([m_micro_version], [0])
m4_define([m_version], [m_major_version.m_minor_version.m_micro_version])

dnl **************************************************************************
dnl Define bugzilla address
dnl **************************************************************************

m4_define([m_bugzilla_url],
          [https://bugzilla.gnome.org/enter_bug.cgi?product=metacity])

dnl **************************************************************************
dnl Initialize autoconf
dnl **************************************************************************

AC_INIT([deepin-metacity], [m_version], [m_bugzilla_url])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_SRCDIR([src/core/display.c])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([build-aux])

CFLAGS="${CFLAGS} -std=gnu99"

dnl **************************************************************************
dnl Set release policy
dnl **************************************************************************

# comment out for shenwei
# AX_IS_RELEASE([git-directory])

dnl **************************************************************************
dnl Enable debug
dnl **************************************************************************

AX_CHECK_ENABLE_DEBUG([yes])

dnl **************************************************************************
dnl Add warning flags
dnl **************************************************************************

#AX_COMPILER_FLAGS([WARN_CFLAGS], [WARN_LDFLAGS])

dnl **************************************************************************
dnl Initialize automake
dnl **************************************************************************

AM_INIT_AUTOMAKE([1.13 no-dist-gzip dist-xz tar-ustar -Wno-portability])
AM_MAINTAINER_MODE([enable])
AM_SILENT_RULES([yes])

dnl **************************************************************************
dnl Initialize libtool
dnl **************************************************************************

LT_INIT
LT_LIB_M

# Honor aclocal flags
AC_SUBST(ACLOCAL_AMFLAGS, "\${ACLOCAL_FLAGS}")

GETTEXT_PACKAGE=deepin-metacity
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE,"$GETTEXT_PACKAGE",[Name of default gettext domain])

IT_PROG_INTLTOOL([0.34.90])
AC_PROG_CC
AC_ISC_POSIX
AC_HEADER_STDC

#### Integer sizes 

AC_CHECK_SIZEOF(char)
AC_CHECK_SIZEOF(short)
AC_CHECK_SIZEOF(long)
AC_CHECK_SIZEOF(int)
AC_CHECK_SIZEOF(void *)
AC_CHECK_SIZEOF(long long)
AC_CHECK_SIZEOF(__int64)

## byte order
AC_C_BIGENDIAN

GTK_MIN_VERSION=3.20.0
GIO_MIN_VERSION=2.25.10
CANBERRA_GTK=libcanberra-gtk3

METACITY_PC_MODULES="gtk+-3.0 >= $GTK_MIN_VERSION gio-2.0 >= $GIO_MIN_VERSION pango >= 1.2.0 gsettings-desktop-schemas >= 3.3.0 xi >= 1.6.0 json-glib-1.0 >= 1.0.0 libbamf3 >= 0.2.118"

GLIB_GSETTINGS

AC_ARG_ENABLE(canberra,
  AC_HELP_STRING([--disable-canberra],
                 [disable use of canberra for event sounds, for embedded non-GNOME builds]),,
  enable_canberra=yes)

if test x$enable_canberra = xyes; then
    AC_DEFINE(HAVE_CANBERRA,1,[Build with canberra support])
    METACITY_PC_MODULES="$METACITY_PC_MODULES $CANBERRA_GTK"
fi

AC_ARG_ENABLE(verbose-mode,
  AC_HELP_STRING([--disable-verbose-mode],
                 [disable metacity's ability to do verbose logging, for embedded/size-sensitive custom builds]),,
  enable_verbose_mode=yes)

if test x$enable_verbose_mode = xyes; then
    AC_DEFINE(WITH_VERBOSE_MODE,1,[Build with verbose mode support])
fi

AC_ARG_ENABLE(sm,
  AC_HELP_STRING([--disable-sm],
                 [disable metacity's session management support, for embedded/size-sensitive custom non-GNOME builds]),,
  enable_sm=auto)

AC_ARG_ENABLE(startup-notification,
  AC_HELP_STRING([--disable-startup-notification],
                 [disable metacity's startup notification support, for embedded/size-sensitive custom non-GNOME builds]),,
  enable_startup_notification=auto)

AC_ARG_ENABLE(compositor,
  AC_HELP_STRING([--disable-compositor],
                 [disable metacity's compositing manager]),,
  enable_compositor=auto)

AC_ARG_ENABLE(render,
  AC_HELP_STRING([--disable-render],
                 [disable metacity's use of the RENDER extension]),,
  enable_render=auto)

AC_ARG_ENABLE(gprof,
              AS_HELP_STRING([--enable-gprof=@<:@no/yes@:>@],
                             [build with support for gprof]),,
                             [enable_gprof=no])
if test "x$enable_gprof" != "xno" ; then
    CFLAGS="$CFLAGS -pg"
    LDFLAGS="$LDFLAGS -pg"
fi

AC_SUBST(LDFLAGS)

## try definining HAVE_BACKTRACE
AC_CHECK_HEADERS(execinfo.h, [AC_CHECK_FUNCS(backtrace)])

AM_GLIB_GNU_GETTEXT

## here we get the flags we'll actually use
PKG_CHECK_MODULES(ALL, glib-2.0 >= 2.36.0 gthread-2.0 >= 2.32.0)
# gtk_window_set_icon_name requires gtk2+-2.60
PKG_CHECK_MODULES(METACITY_MESSAGE, gtk+-3.0 >= $GTK_MIN_VERSION)
PKG_CHECK_MODULES(METACITY_WINDOW_DEMO, gtk+-3.0 >= $GTK_MIN_VERSION)

if $PKG_CONFIG --atleast-version 1.2.0 pangoxft; then
   echo "pangoxft found"
else
   AC_MSG_ERROR("Pango 1.2.0 or greater based on Xft2 is required")
fi

# Unconditionally use this dir to avoid a circular dep with gnomecc
GNOME_KEYBINDINGS_KEYSDIR="${datadir}/gnome-control-center/keybindings"
AC_SUBST(GNOME_KEYBINDINGS_KEYSDIR)

STARTUP_NOTIFICATION_VERSION=0.7
AC_MSG_CHECKING([Startup notification library >= $STARTUP_NOTIFICATION_VERSION])
if $PKG_CONFIG --atleast-version $STARTUP_NOTIFICATION_VERSION libstartup-notification-1.0; then
   have_startup_notification=yes
else
   have_startup_notification=no
fi
AC_MSG_RESULT($have_startup_notification)

if test x$enable_startup_notification = xyes; then
   have_startup_notification=yes
   echo "startup-notification support forced on"
elif test x$enable_startup_notification = xauto; then
   true
else
   have_startup_notification=no
fi

if test x$have_startup_notification = xyes; then
  echo "Building with libstartup-notification"
  METACITY_PC_MODULES="$METACITY_PC_MODULES libstartup-notification-1.0 >= $STARTUP_NOTIFICATION_VERSION"
  AC_DEFINE(HAVE_STARTUP_NOTIFICATION, , [Building with startup notification support])
else
  echo "Building without libstartup-notification"
fi

## init this, it gets set either in the compositor check below 
## or the render-specific check later
have_xrender=no

XCOMPOSITE_VERSION=0.3
if test x$enable_compositor = xyes; then
   have_xcomposite=yes
elif test x$enable_compositor = xauto; then
   echo "Building compositing manager by default now."
   have_xcomposite=yes
else
   have_xcomposite=no
fi

if test x$have_xcomposite = xyes; then
   AC_MSG_CHECKING([Xcomposite >= $XCOMPOSITE_VERSION])
   if $PKG_CONFIG --atleast-version $XCOMPOSITE_VERSION xcomposite; then
      AC_MSG_RESULT([yes])
   else
      AC_MSG_ERROR([no. Use --disable-compositor to disable.])
   fi
fi

if test x$have_xcomposite = xyes; then
  METACITY_PC_MODULES="$METACITY_PC_MODULES xcomposite >= $XCOMPOSITE_VERSION xfixes xrender xdamage"
  AC_DEFINE(HAVE_COMPOSITE_EXTENSIONS, 1, [Building with compositing manager support])
  echo "Building with compositing manager"

  ## force on render also
  have_xrender=yes
else
  echo "Building without compositing manager"
fi

## if no compositor, still possibly enable render
if test x$have_xcomposite = xno; then
  XRENDER_VERSION=0.0
  AC_MSG_CHECKING([xrender >= $XRENDER_VERSION])
  if $PKG_CONFIG --atleast-version $XRENDER_VERSION xrender; then
     have_xrender=yes
  else
     have_xrender=no
  fi
  AC_MSG_RESULT($have_xrender)

  if test x$enable_render = xyes; then
     have_xrender=yes
     echo "Render support forced on"
  elif test x$enable_render = xauto; then
     true
  else
     have_xrender=no
  fi

  if test x$have_xrender = xyes; then
     echo "Building with Render"
     METACITY_PC_MODULES="$METACITY_PC_MODULES xrender >= $XRENDER_VERSION"
  fi
fi ## have_composite

if test x$have_xrender = xyes; then
  AC_DEFINE(HAVE_RENDER, , [Building with Render extension support])
fi

AC_MSG_CHECKING([Xcursor])
if $PKG_CONFIG xcursor; then
     have_xcursor=yes
  else
     have_xcursor=no
  fi
  AC_MSG_RESULT($have_xcursor)

if test x$have_xcursor = xyes; then
  echo "Building with Xcursor"
  METACITY_PC_MODULES="$METACITY_PC_MODULES xcursor"
  AC_DEFINE(HAVE_XCURSOR, , [Building with Xcursor support]) 
fi

AC_MSG_CHECKING([libgtop])
if $PKG_CONFIG libgtop-2.0; then
     have_gtop=yes
  else
     have_gtop=no
  fi
  AC_MSG_RESULT($have_gtop)

if test x$have_gtop = xyes; then
  echo "Building with libgtop"
  METACITY_PC_MODULES="$METACITY_PC_MODULES libgtop-2.0"
  AC_DEFINE(HAVE_GTOP, , [Building with libgtop]) 
fi

PKG_CHECK_MODULES(METACITY, $METACITY_PC_MODULES)

AC_PATH_XTRA

ALL_X_LIBS="-lX11 $X_EXTRA_LIBS"

# Check for Xinerama extension (Solaris impl or Xfree impl)
metacity_save_cppflags="$CPPFLAGS"
CPPFLAGS="$CPPFLAGS $X_CFLAGS"

AC_ARG_ENABLE(xinerama,
  AC_HELP_STRING([--disable-xinerama],
                 [disable metacity's use of the Xinerama extension]),
  try_xinerama=$enable_xinerama,try_xinerama=yes)

use_solaris_xinerama=no
use_xfree_xinerama=no
if test "${try_xinerama}" != no; then
    case "$host" in
        *-*-solaris*)
            # Check for solaris
            use_solaris_xinerama=yes
            AC_CHECK_LIB(Xext, XineramaGetInfo,
                         use_solaris_xinerama=yes, use_solaris_xinerama=no,
                         $ALL_X_LIBS)
            if test "x$use_solaris_xinerama" = "xyes"; then
                AC_CHECK_HEADER(X11/extensions/xinerama.h,
                                if test -z "`echo $ALL_X_LIBS | grep "\-lXext" 2> /dev/null`"; then
                                    X_EXTRA_LIBS="-lXext $X_EXTRA_LIBS"
                                fi
                                AC_DEFINE(HAVE_SOLARIS_XINERAMA, , [Have Solaris-style Xinerama])
                                AC_DEFINE(HAVE_XINERAMA, , [Have some version of Xinerama]),
                                use_solaris_xinerama=no,
                                [#include <X11/Xlib.h>])
            fi	
            AC_MSG_CHECKING(for Xinerama support on Solaris)
            AC_MSG_RESULT($use_solaris_xinerama);
            ;;
        *)
            # Check for XFree
            use_xfree_xinerama=yes
            AC_CHECK_LIB(Xinerama, XineramaQueryExtension,
                [AC_CHECK_HEADER(X11/extensions/Xinerama.h,
                                 X_EXTRA_LIBS="-lXinerama $X_EXTRA_LIBS"   
                                 if test -z "`echo $ALL_X_LIBS | grep "\-lXext" 2> /dev/null`"; then
                                     X_EXTRA_LIBS="-lXext $X_EXTRA_LIBS"
                                 fi
                                 AC_DEFINE(HAVE_XFREE_XINERAMA, , [Have XFree86-style Xinerama])
                                 AC_DEFINE(HAVE_XINERAMA,, [Have some version of Xinerama]), 
                                 use_xfree_xinerama=no,
                                 [#include <X11/Xlib.h>])],
                use_xfree_xinerama=no, -lXext $ALL_X_LIBS)
            AC_MSG_CHECKING(for Xinerama support on XFree86)
            AC_MSG_RESULT($use_xfree_xinerama);
            ;;
    esac
fi

CPPFLAGS="$metacity_save_cppflags"

found_xkb=no
AC_CHECK_LIB(X11, XkbQueryExtension,
               [AC_CHECK_HEADER(X11/XKBlib.h,
                                found_xkb=yes)],
	, $ALL_X_LIBS)

if test "x$found_xkb" = "xyes"; then
   AC_DEFINE(HAVE_XKB, , [Have keyboard extension library])
fi


RANDR_LIBS=
found_randr=no
AC_CHECK_LIB(Xrandr, XRRUpdateConfiguration,
               [AC_CHECK_HEADER(X11/extensions/Xrandr.h,
                                RANDR_LIBS=-lXrandr found_randr=yes,,
				[#include <X11/Xlib.h>])],
               , -lXrender -lXext $ALL_X_LIBS)

if test "x$found_randr" = "xyes"; then
   AC_DEFINE(HAVE_RANDR, , [Have the Xrandr extension library])
fi

METACITY_LIBS="$ALL_LIBS $METACITY_LIBS $RANDR_LIBS -lX11 -lXext $X_EXTRA_LIBS $LIBM"
METACITY_MESSAGE_LIBS="$METACITY_MESSAGE_LIBS -lX11 $X_EXTRA_LIBS"
METACITY_WINDOW_DEMO_LIBS="$METACITY_WINDOW_DEMO_LIBS -lX11 $X_EXTRA_LIBS $LIBM"
METACITY_PROPS_LIBS="$METACITY_PROPS_LIBS -lX11 $X_EXTRA_LIBS"

found_sm=no
case "$METACITY_LIBS" in
 *-lSM*)
    found_sm=yes
    ;;
 *)
  AC_CHECK_LIB(SM, SmcSaveYourselfDone, 
               [AC_CHECK_HEADERS(X11/SM/SMlib.h,
                                METACITY_LIBS="-lSM -lICE $METACITY_LIBS" found_sm=yes)],
               , $METACITY_LIBS)
    ;;
esac

if test x$enable_sm = xno; then
   found_sm=no
fi

if test x$enable_sm = xyes; then
   if test "$found_sm" = "no"; then
      AC_MSG_ERROR([--enable-sm forced and -lSM not found])
      exit 1
   fi
fi

if test "$found_sm" = "yes"; then
   AC_DEFINE(HAVE_SM, , [Building with SM support])
fi

AM_CONDITIONAL(HAVE_SM, test "$found_sm" = "yes") 

HOST_ALIAS=$host_alias
AC_SUBST(HOST_ALIAS)

AC_PATH_PROG(ZENITY, zenity, no)
if test x"$ZENITY" = xno; then
  AC_MSG_ERROR([zenity not found in your path - needed for dialogs])
fi

AC_ARG_ENABLE(themes-documentation,
  AC_HELP_STRING([--disable-themes-documentation],
                 [disable building themes documentation on system where yelp-tools is unavailable]),,
  enable_themes_documentation=yes)
AM_CONDITIONAL(BUILD_THEMES_DOCS, test "$enable_themes_documentation" = "yes")

if test x$enable_themes_documentation = xyes; then
  YELP_HELP_INIT
fi

dnl **************************************************************************
dnl Process .in files
dnl **************************************************************************

AC_CONFIG_FILES([
  Makefile

  doc/Makefile
  doc/creating_themes/Makefile
  doc/man/Makefile

  src/Makefile
  src/libmetacity-private.pc
  src/tools/Makefile
  src/themes/Makefile

  po/Makefile.in
])

AC_OUTPUT

dnl **************************************************************************
dnl Show summary
dnl **************************************************************************

echo ""
echo "  Deepin Metacity $VERSION"
echo "  `echo Deepin Metacity $VERSION | sed "s/./=/g"`"
echo ""
echo "  srcdir ......................: ${srcdir}"
echo ""
echo "  compiler ....................: ${CC}"
echo "  cflags ......................: ${CFLAGS}"
echo ""
echo "  prefix ......................: ${prefix}"
echo "  exec_prefix .................: ${exec_prefix}"
echo "  libdir ......................: ${libdir}"
echo "  sysconfdir ..................: ${sysconfdir}"
echo "  libexecdir ..................: ${libexecdir}"
echo ""
echo "  Canberra ....................: ${enable_canberra}"
echo "  XFree86 Xinerama ............: ${use_xfree_xinerama}"
echo "  Solaris Xinerama ............: ${use_solaris_xinerama}"
echo "  Startup notification ........: ${have_startup_notification}"
echo "  Compositing manager .........: ${have_xcomposite}"
echo "  Session management ..........: ${found_sm}"
echo "  Resize-and-rotate ...........: ${found_randr}"
echo "  Render ......................: ${have_xrender}"
echo "  Xcursor .....................: ${have_xcursor}"
echo ""
