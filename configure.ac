m4_define([cinnamon_control_center_version], 4.0.1)
AC_INIT([cinnamon-control-center], [cinnamon_control_center_version],
        [https://github.com/linuxmint/cinnamon-control-center/issues])

AC_CONFIG_SRCDIR([shell])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])

m4_ifdef([AX_IS_RELEASE], [AX_IS_RELEASE([always])])

AM_INIT_AUTOMAKE([1.11 no-dist-gzip dist-xz tar-ustar check-news])
AM_MAINTAINER_MODE([enable])
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

m4_ifdef([AX_CHECK_ENABLE_DEBUG],
         [AX_CHECK_ENABLE_DEBUG([yes])])
m4_ifdef([AX_COMPILER_FLAGS],
         [AX_COMPILER_FLAGS([WARN_CFLAGS],[WARN_LDFLAGS])])

# Check for programs
AC_PROG_CC
AM_PROG_CC_C_O
AC_HEADER_STDC

# Initialize libtool
LT_PREREQ([2.2])
LT_INIT

# .so version for libcinnamon-control-center
LIBCINNAMONCONTROLCENTER_CURRENT=1
LIBCINNAMONCONTROLCENTER_REVISION=0
LIBCINNAMONCONTROLCENTER_AGE=0
AC_SUBST(LIBCINNAMONCONTROLCENTER_CURRENT)
AC_SUBST(LIBCINNAMONCONTROLCENTER_REVISION)
AC_SUBST(LIBCINNAMONCONTROLCENTER_AGE)

# Internationalization support

IT_PROG_INTLTOOL([0.40.1])

GETTEXT_PACKAGE=cinnamon-control-center
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, "$GETTEXT_PACKAGE", [Gettext package])

AC_PATH_XTRA
x_libs="$X_PRE_LIBS $X_LIBS -lX11 $X_EXTRA_LIBS"

AC_PATH_PROG([GLIB_MKENUMS],[glib-mkenums])

AC_ARG_ENABLE(documentation,
              AC_HELP_STRING([--enable-documentation],
                             [build documentation]),,
              enable_documentation=yes)
if test x$enable_documentation = xyes; then
   AC_PATH_PROG([XSLTPROC], [xsltproc])
   if test x$XSLTPROC = x; then
      AC_MSG_ERROR([xsltproc is required to build documentation])
   fi
fi
AM_CONDITIONAL(BUILD_DOCUMENTATION, test x$enable_documentation = xyes)

dnl Region panel
savecppflags=$CPPFLAGS
CPPFLAGS="$CPPFLAGS $X_CFLAGS"
AC_CHECK_HEADERS([X11/Xlib.h])
AC_CHECK_LIB(Xxf86misc, XF86MiscQueryExtension, [
  AC_CHECK_HEADERS([X11/extensions/xf86misc.h], [XF86MISC_LIBS="-lXxf86misc"],[],
[#if HAVE_X11_XLIB_H
#include <X11/Xlib.h>
#endif
])])
AC_SUBST(XF86MISC_LIBS)
AC_CHECK_HEADERS(X11/extensions/XKB.h)
CPPFLAGS=$savecppflags

AC_CHECK_LIB(m, floor)

AC_ARG_ENABLE([systemd],
              AS_HELP_STRING([--enable-systemd], [Use systemd]),
              [with_systemd=$enableval],
              [with_systemd=no])
if test "$with_systemd" = "yes" ; then
  SYSTEMD=libsystemd-login
  AC_DEFINE(HAVE_SYSTEMD, 1, [Define to 1 if systemd is available])
else
  SYSTEMD=
fi

dnl ==============================================
dnl Check that we meet the  dependencies
dnl ==============================================

GLIB_REQUIRED_VERSION=2.31.0
GTK_REQUIRED_VERSION=3.4.1
GDKPIXBUF_REQUIRED_VERSION=2.23.0
POLKIT_REQUIRED_VERSION=0.103
NETWORK_MANAGER_REQUIRED_VERSION=1.2.0
NETWORK_MANAGER_APPLET_REQUIRED_VERSION=1.2.0
MODEM_MANAGER_REQUIRED_VERSION=0.7
LIBNOTIFY_REQUIRED_VERSION=0.7.3
CINNAMON_DESKTOP_REQUIRED_VERSION=1.0.0
CSD_REQUIRED_VERSION=1.0.0
LIBWACOM_REQUIRED_VERSION=0.7
GOA_REQUIRED_VERSION=3.18.0

COMMON_MODULES="gtk+-3.0 >= $GTK_REQUIRED_VERSION
 glib-2.0 >= $GLIB_REQUIRED_VERSION
 gthread-2.0
 gio-2.0
 gio-unix-2.0
 cinnamon-desktop >= $CINNAMON_DESKTOP_REQUIRED_VERSION
 libnotify >= $LIBNOTIFY_REQUIRED_VERSION"

PKG_CHECK_MODULES(LIBCINNAMON_CONTROL_CENTER, $COMMON_MODULES)
PKG_CHECK_MODULES(LIBLANGUAGE, $COMMON_MODULES cinnamon-desktop fontconfig)
PKG_CHECK_MODULES(LIBSHORTCUTS, $COMMON_MODULES x11)
PKG_CHECK_MODULES(SHELL, $COMMON_MODULES libcinnamon-menu-3.0 gio-unix-2.0 x11)
PKG_CHECK_MODULES(BACKGROUND_PANEL, $COMMON_MODULES libxml-2.0 cinnamon-desktop
                  gdk-pixbuf-2.0 >= $GDKPIXBUF_REQUIRED_VERSION)
PKG_CHECK_MODULES(DATETIME_PANEL, $COMMON_MODULES dbus-glib-1
                  polkit-gobject-1 >= $POLKIT_REQUIRED_VERSION
                  gdk-pixbuf-2.0 >= $GDKPIXBUF_REQUIRED_VERSION)
PKG_CHECK_MODULES(DISPLAY_PANEL, $COMMON_MODULES cinnamon-desktop x11)
PKG_CHECK_MODULES(MEDIA_PANEL, $COMMON_MODULES)

PKG_CHECK_MODULES(NETWORK_PANEL, $COMMON_MODULES gmodule-2.0
                  polkit-gobject-1 >= $POLKIT_REQUIRED_VERSION)
PKG_CHECK_MODULES(REGION_PANEL, $COMMON_MODULES libgnomekbd >= 2.91.91
                  polkit-gobject-1 >= $POLKIT_REQUIRED_VERSION
                  libxklavier >= 5.1 libgnomekbdui >= 2.91.91)

#----------------------------------------------
# Network Manager Stuff

AC_ARG_ENABLE(networkmanager,
              AC_HELP_STRING([--disable-networkmanager],
                             [build NetworkManager support]),
              [case "${enableval}" in
                      yes) enable_networkmanager=yes ;;
                      no) enable_networkmanager=no ;;
                      *) AC_MSG_ERROR(bad value ${enableval} for --disable-networkmanager) ;;
              esac],
              [enable_networkmanager=yes]) dnl Enabled by default

if test "x$enable_networkmanager" = xyes; then
    PKG_CHECK_MODULES(NETWORK_MANAGER,
                      libnm >= $NETWORK_MANAGER_REQUIRED_VERSION
                      libnma >= $NETWORK_MANAGER_APPLET_REQUIRED_VERSION,
                      [have_networkmanager=yes], have_networkmanager=no)
    PKG_CHECK_MODULES(LIBNMA_1_8_0, libnma >= 1.8.0,
                      [have_nma_18=yes], have_nma_18=no)
    if test "x$have_networkmanager" = xno ; then
        AC_MSG_ERROR(*** NetworkManager not found ***)
    fi

    AC_DEFINE(BUILD_NETWORK, 1, [Define to 1 to build the Network panel])

    if test x${have_nma_18} = xyes; then
      AC_DEFINE(HAVE_NMA_18, 1, [Define to 1 if libnma ist >= 1.8.0])
    fi
fi

AM_CONDITIONAL(BUILD_NETWORK, [test x$have_networkmanager = xyes])

AC_ARG_ENABLE(modemmanager,
              AC_HELP_STRING([--disable-modemmanager],
                             [build ModemManager support]),
              [case "${enableval}" in
                      yes) enable_modemmanager=yes ;;
                      no) enable_modemmanager=no ;;
                      *) AC_MSG_ERROR(bad value ${enableval} for --disable-modemmanager) ;;
              esac],
              [enable_modemmanager=yes]) dnl Enable by default

if test "x$enable_modemmanager" = xyes; then
    if test "x$enable_networkmanager" = xno; then
        AC_MSG_ERROR(*** NetworkManager is required by ModemManager ***)
    fi
fi

if test "x$enable_modemmanager" = xyes; then
    PKG_CHECK_MODULES(MODEM_MANAGER, mm-glib >= $MODEM_MANAGER_REQUIRED_VERSION,
                      [have_modemmanager=yes], have_modemmanager=no)

    if test "x$have_modemmanager" = xno ; then
        AC_MSG_ERROR(*** ModemManager not found ***)
    fi

    AC_DEFINE(BUILD_MODEM, 1, [Define 1 to build ModemManager support])
fi

AM_CONDITIONAL(BUILD_MODEM, [test x$have_modemmanager = xyes])

# Work-around for https://bugzilla.gnome.org/show_bug.cgi?id=723769
PKG_CHECK_MODULES(NM_UNSTABLE_CHECK, libnm-util >= 0.9.9.0, nm_unstable=yes, nm_unstable=no)
if test x$nm_unstable = xyes; then
  AC_DEFINE(HAVE_NM_UNSTABLE, 1, [Define to 1 if NetworkManager is available])
fi
AM_CONDITIONAL(HAVE_NM_UNSTABLE, [test x$nm_unstable = xyes])


# -----------------------------------------------
# Check for CUPS 1.4 or newer

AC_ARG_ENABLE([cups],
              AS_HELP_STRING([--disable-cups], [disable CUPS support (default: enabled)]),,
              [enable_cups=no])

if test x"$enable_cups" != x"no" ; then
  AC_PROG_SED

  AC_PATH_PROG(CUPS_CONFIG, cups-config)

  if test x$CUPS_CONFIG = x; then
    AC_MSG_ERROR([cups-config not found but CUPS support requested])
  fi

  CUPS_API_VERSION=`$CUPS_CONFIG --api-version`
  CUPS_API_MAJOR=`echo $ECHO_N $CUPS_API_VERSION | cut -d . -f 1`
  CUPS_API_MINOR=`echo $ECHO_N $CUPS_API_VERSION | cut -d . -f 2`

  AC_CHECK_HEADERS([cups/cups.h cups/http.h cups/ipp.h cups/ppd.h],,
                   AC_MSG_ERROR([CUPS headers not found but CUPS support requested]))

  if ! test $CUPS_API_MAJOR -gt 1 -o \
            $CUPS_API_MAJOR -eq 1 -a $CUPS_API_MINOR -ge 4 ; then
    AC_MSG_ERROR([CUPS 1.4 or newer not found, but CUPS support requested])
  fi

  CUPS_CFLAGS=`$CUPS_CONFIG --cflags | $SED -e 's/-O\w*//g' -e 's/-m\w*//g'`
  CUPS_LIBS=`$CUPS_CONFIG --libs`
  AC_SUBST(CUPS_CFLAGS)
  AC_SUBST(CUPS_LIBS)
fi

build_color=false
AC_ARG_ENABLE(color,
       AC_HELP_STRING([--disable-color],
               [disable color management panel]),
       [case "${enableval}" in
               yes) WANT_COLOR=yes ;;
               no) WANT_COLOR=no ;;
               *) AC_MSG_ERROR(bad value ${enableval} for --disable-color) ;;
       esac],
       [WANT_COLOR=yes]) dnl Default value

if test x$WANT_COLOR = xyes; then
       PKG_CHECK_MODULES(COLOR_PANEL, $COMMON_MODULES colord >= 0.1.14)
       build_color=true
fi
if test "x$build_color" = xtrue ; then
       AC_DEFINE(BUILD_COLOR, 1, [Define to 1 to build the Color panel])
fi
AM_CONDITIONAL(BUILD_COLOR, test "x$build_color" = "xtrue")
AC_SUBST(COLOR_PANEL_CFLAGS)
AC_SUBST(COLOR_PANEL_LIBS)

build_wacom=false
AC_ARG_ENABLE(wacom,
       AC_HELP_STRING([--disable-wacom],
               [disable wacom management panel]),
       [case "${enableval}" in
               yes) WANT_WACOM=yes ;;
               no) WANT_WACOM=no ;;
               *) AC_MSG_ERROR(bad value ${enableval} for --disable-wacom) ;;
       esac],
       [WANT_WACOM=yes]) dnl Default value

if test x$WANT_WACOM = xyes; then
        PKG_CHECK_MODULES(WACOM_PANEL, [$COMMON_MODULES
                  cinnamon-settings-daemon >= $CSD_REQUIRED_VERSION
                  xi >= 1.2 x11 libwacom >= $LIBWACOM_REQUIRED_VERSION
                  cinnamon-desktop
                  gtk+-3.0 >= 3.8.0])
        build_wacom="true"
        AC_DEFINE(HAVE_WACOM, 1, [Define if wacom is being build])

        PKG_CHECK_MODULES(NEWER_WACOM, libwacom >= 0.27, have_newer_wacom="yes", have_newer_wacom="no")
        if test x$have_newer_wacom = xyes; then
                    AC_DEFINE(HAVE_WACOM_3D_STYLUS, 1, [Define if wacom supports 3d stylus])
        fi
fi
AM_CONDITIONAL(BUILD_WACOM, test "x$build_wacom" = "xtrue")
AC_SUBST(WACOM_PANEL_CFLAGS)
AC_SUBST(WACOM_PANEL_LIBS)

# Online accounts
AC_ARG_ENABLE(onlineaccounts, AC_HELP_STRING([--disable-onlineaccounts], [disable online-accounts panel]),
              [case "${enableval}" in
                      yes) enable_onlineaccounts=yes ;;
                      no) enable_onlineaccounts=no ;;
                      *) AC_MSG_ERROR(bad value ${enableval} for --disable-onlineaccounts) ;;
              esac],
              [enable_onlineaccounts=yes]) dnl Enabled by default

if test "x$enable_onlineaccounts" = xyes; then
    PKG_CHECK_MODULES(ONLINE_ACCOUNTS_PANEL, $COMMON_MODULES goa-1.0
                  goa-backend-1.0 >= $GOA_REQUIRED_VERSION,
                      [have_onlineaccounts=yes], have_onlineaccounts=no)
    if test "x$have_onlineaccounts" = xno ; then
        AC_MSG_ERROR(*** Gnome Online Accounts not found ***)
    fi

    AC_DEFINE(BUILD_ONLINE_ACCOUNTS, 1, [Define to 1 to build the online-accounts panel])

    if test x${have_onlineaccounts} = xyes; then
      AC_DEFINE(HAVE_ONLINE_ACCOUNTS, 1, [Define to 1 if Gnome Online Accounts is available])
    fi
fi

AM_CONDITIONAL(BUILD_ONLINE_ACCOUNTS, [test x$have_onlineaccounts = xyes])
AC_SUBST(ONLINE_ACCOUNTS_CFLAGS)
AC_SUBST(ONLINE_ACCOUNTS_LIBS)

# This is a hard-dependency for the region and user-accounts panels
PKG_CHECK_MODULES(ISOCODES, iso-codes)

AC_DEFINE_UNQUOTED([ISO_CODES_PREFIX],["`$PKG_CONFIG --variable=prefix iso-codes`"],[ISO codes prefix])
ISO_CODES=iso-codes

dnl ==============================================
dnl End: Check that we meet the  dependencies
dnl ==============================================

AC_PATH_PROG(GLIB_GENMARSHAL, glib-genmarshal, no)

if test x"$GLIB_GENMARSHAL" = xno; then
  AC_MSG_ERROR([glib-genmarshal executable not found in your path - should be installed with glib])
fi

AC_SUBST(GLIB_GENMARSHAL)

dnl =======================================
dnl Panels
dnl =======================================

PANELS_DIR="${libdir}/cinnamon-control-center-1/panels"
AC_SUBST(PANELS_DIR)

PANEL_CFLAGS="-I\$(top_srcdir)/ -DG_LOG_DOMAIN=\"\\\"\$(cappletname)-cc-panel\\\"\""
AC_SUBST(PANEL_CFLAGS)

PANEL_LIBS="\$(top_builddir)/shell/libcinnamon-control-center.la"
AC_SUBST(PANEL_LIBS)

PANEL_LDFLAGS="-export_dynamic -avoid-version -module -no-undefined -export-symbols-regex '^g_io_module_(load|unload)'"
AC_SUBST(PANEL_LDFLAGS)


CONTROL_CENTER_VERSION=cinnamon_control_center_version
AC_SUBST(CONTROL_CENTER_VERSION)

dnl =======================================
dnl Finish
dnl =======================================

# Turn on the additional warnings last

AC_ARG_ENABLE(more-warnings,
              AS_HELP_STRING([--enable-more-warnings],
                             [Maximum compiler warnings]),
              set_more_warnings="$enableval",[
	      if test -d $srcdir/.git; then
	        set_more_warnings=yes
	      else
	        set_more_warnings=no
              fi])

AC_MSG_CHECKING(for more warnings)
if test "$GCC" = "yes" -a "$set_more_warnings" != "no"; then
        AC_MSG_RESULT(yes)
        CFLAGS="\
        -Wall -Wclobbered -Wempty-body -Wignored-qualifiers \
        -Wmissing-field-initializers -Wmissing-parameter-type \
        -Wold-style-declaration -Woverride-init -Wtype-limits \
        -Wuninitialized \
        -Wchar-subscripts -Wmissing-declarations -Wmissing-prototypes \
        -Wnested-externs -Wpointer-arith \
        -Wcast-align -Wsign-compare \
        $CFLAGS"

	# Only add this when optimizing is enabled (default)
	AC_MSG_CHECKING([whether optimization is enabled])
	AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#if __OPTIMIZE__ == 0
	                                     #error No optimization
	                                     #endif
	                                   ]], [[]])],
	                  [has_optimization=yes],
	                  [has_optimization=no])
	if test $has_optimization = yes; then
		CFLAGS="$CFLAGS -Wp,-D_FORTIFY_SOURCE=2"
	fi
	AC_MSG_RESULT($has_optimization)

        for option in -Wno-strict-aliasing -Wno-sign-compare; do
                SAVE_CFLAGS="$CFLAGS"
                CFLAGS="$CFLAGS $option"
                AC_MSG_CHECKING([whether gcc understands $option])
                AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[]])],
                                  [has_option=yes],
                                  [has_option=no])
                if test $has_option = no; then
                        CFLAGS="$SAVE_CFLAGS"
                fi
                AC_MSG_RESULT($has_option)
                unset has_option
                unset SAVE_CFLAGS
        done
        unset option
else
        AC_MSG_RESULT(no)
fi


AC_OUTPUT([
Makefile
panels/Makefile
panels/common/Makefile
panels/display/Makefile
panels/display/cinnamon-display-panel.desktop.in
panels/region/Makefile
panels/region/cinnamon-region-panel.desktop.in
panels/color/Makefile
panels/color/cinnamon-color-panel.desktop.in
panels/color/icons/Makefile
panels/color/icons/16x16/Makefile
panels/color/icons/22x22/Makefile
panels/color/icons/24x24/Makefile
panels/color/icons/32x32/Makefile
panels/color/icons/48x48/Makefile
panels/color/icons/64x64/Makefile
panels/color/icons/256x256/Makefile
panels/color/icons/scalable/Makefile
panels/network/Makefile
panels/network/wireless-security/Makefile
panels/network/connection-editor/Makefile
panels/network/cinnamon-network-panel.desktop.in
panels/wacom/Makefile
panels/wacom/calibrator/Makefile
panels/wacom/cinnamon-wacom-panel.desktop.in
panels/datetime/Makefile
panels/datetime/po-timezones/Makefile
panels/online-accounts/Makefile
panels/online-accounts/cinnamon-online-accounts-panel.desktop.in
panels/online-accounts/icons/Makefile
panels/online-accounts/icons/16x16/Makefile
panels/online-accounts/icons/22x22/Makefile
panels/online-accounts/icons/24x24/Makefile
panels/online-accounts/icons/32x32/Makefile
panels/online-accounts/icons/48x48/Makefile
panels/online-accounts/icons/256x256/Makefile
po/Makefile.in
shell/Makefile
shell/cinnamon-control-center.desktop.in
shell/libcinnamon-control-center.pc
])

AC_MSG_NOTICE([cinnamon-control-center was configured with the following options:])
if test "x$have_networkmanager" = "xyes"; then
	AC_MSG_NOTICE([** NetworkManager (Network panel)])
else
	AC_MSG_NOTICE([   Network panel disabled])
fi
if test "x$have_modemmanager" = "xyes"; then
	AC_MSG_NOTICE([** ModemManager support enabled])
else
	AC_MSG_NOTICE([   ModemManager support disabled])
fi
#if test "x$with_systemd" = "xyes"; then
#	AC_MSG_NOTICE([** systemd (Systemd session tracking)])
#else
#	AC_MSG_NOTICE([   Using ConsoleKit for session tracking])
#fi

if test "x$build_color" = "xtrue"; then
	AC_MSG_NOTICE([** Colord support (Color management panel)])
else
	AC_MSG_NOTICE([   Colord support disabled])
fi

if test "x$build_wacom" = "xtrue"; then
	AC_MSG_NOTICE([** Wacom support (Wacom panel)])
else
	AC_MSG_NOTICE([   Wacom support disabled])
fi

if test "x$have_onlineaccounts" = "xyes"; then
	AC_MSG_NOTICE([** Online-Accounts support (Online-Accounts panel)])
else
	AC_MSG_NOTICE([   Online-Accounts support disabled])
fi

AC_MSG_NOTICE([End options])
