# GNU nano (6.2) - nosearchwrap
[![build](https://github.com/davidhcefx/GNU_nano_5.4_nosearchwrap/actions/workflows/build.yml/badge.svg)](https://github.com/davidhcefx/GNU_nano_5.4_nosearchwrap/actions/workflows/build.yml)

- While performing textual search, many editors, for instance [Sublime](https://www.sublimetext.com/), support toggling the option **Wrap**, which is to wrap past end-of-file and proceed searching. It will be nice if Nano can have this too!


### New rcfile option:
```
set nosearchwrap
    Don't wrap around to the start or end of the buffer when performing search or replace.
```

### New program option:
```
--nosearchwrap
    Don't wrap around to the start or end of the buffer when performing search or replace.
```

## How to compile

*Please refer to [README.GIT](/README.GIT) for more details.*

1. Install the dependencies: `apt install autoconf automake autopoint gcc gettext git groff make pkg-config texinfo`

2. Run `./autogen.sh`.

3. After that, just do the normal `./configure`, `make` and `make install`.


## Patch for Nano v6.2
```patch
From 959b04f862511dc7aecb4dccccd6ff1acf3ce3aa Mon Sep 17 00:00:00 2001
From: davidhcefx <davidhu0903ex3@gmail.com>
Date: Mon, 27 May 2024 15:59:43 +0800
Subject: [PATCH] Feature nosearchwrap

---
 doc/nano.1           | 3 +++
 doc/nanorc.5         | 3 +++
 src/definitions.h    | 1 +
 src/nano.c           | 5 +++++
 src/rcfile.c         | 1 +
 src/search.c         | 2 +-
 syntax/nanorc.nanorc | 2 +-
 7 files changed, 15 insertions(+), 2 deletions(-)

diff --git a/doc/nano.1 b/doc/nano.1
index 0a547929..89cb0794 100644
--- a/doc/nano.1
+++ b/doc/nano.1
@@ -373,6 +373,9 @@ With \fBM\-X\fR the help lines.
 .BR \-! ", " \-\-magic
 When neither the file's name nor its first line give a clue,
 try using libmagic to determine the applicable syntax.
+.TP
+.BR \-\-nosearchwrap
+Don't wrap around to the start or end of the buffer when performing search or replace.
 
 .SH TOGGLES
 Several of the above options can be switched on and off also while
diff --git a/doc/nanorc.5 b/doc/nanorc.5
index ad712e30..50765229 100644
--- a/doc/nanorc.5
+++ b/doc/nanorc.5
@@ -224,6 +224,9 @@ Don't display the two help lines at the bottom of the screen.
 Don't automatically add a newline when a text does not end with one.
 (This can cause you to save non-POSIX text files.)
 .TP
+.B set nosearchwrap
+Don't wrap around to the start or end of the buffer when performing search or replace.
+.TP
 .B set nowrap
 Deprecated option since it has become the default setting.
 When needed, use \fBunset breaklonglines\fR instead.
diff --git a/src/definitions.h b/src/definitions.h
index 2bdc782f..8acb06ce 100644
--- a/src/definitions.h
+++ b/src/definitions.h
@@ -322,6 +322,7 @@ enum {
 	CONSTANT_SHOW,
 	NO_HELP,
 	SUSPENDABLE,
+	NO_SEARCH_WRAP,
 	NO_WRAP,
 	AUTOINDENT,
 	VIEW_MODE,
diff --git a/src/nano.c b/src/nano.c
index 04ecdbbf..a29d38e6 100644
--- a/src/nano.c
+++ b/src/nano.c
@@ -658,6 +658,7 @@ void usage(void)
 #ifdef HAVE_LIBMAGIC
 	print_opt("-!", "--magic", N_("Also try magic to determine syntax"));
 #endif
+	print_opt("", "--nosearchwrap", N_("Don't wrap past EOF when search/replace"));
 }
 
 /* Display the version number of this nano, a copyright notice, some contact
@@ -1773,6 +1774,7 @@ int main(int argc, char **argv)
 #ifdef HAVE_LIBMAGIC
 		{"magic", 0, NULL, '!'},
 #endif
+		{"nosearchwrap", 0, NULL, '\x01'},
 		{NULL, 0, NULL, 0}
 	};
 
@@ -2064,6 +2066,9 @@ int main(int argc, char **argv)
 				SET(USE_MAGIC);
 				break;
 #endif
+			case '\x01':
+				SET(NO_SEARCH_WRAP);
+				break;
 			default:
 				printf(_("Type '%s -h' for a list of available options.\n"), argv[0]);
 				exit(1);
diff --git a/src/rcfile.c b/src/rcfile.c
index 049a2886..1eed4f07 100644
--- a/src/rcfile.c
+++ b/src/rcfile.c
@@ -68,6 +68,7 @@ static const rcoption rcopts[] = {
 #endif
 	{"nohelp", NO_HELP},
 	{"nonewlines", NO_NEWLINES},
+	{"nosearchwrap", NO_SEARCH_WRAP},
 #ifdef ENABLE_WRAPPING
 	{"nowrap", NO_WRAP},  /* Deprecated; remove in 2024. */
 #endif
diff --git a/src/search.c b/src/search.c
index 5b68192d..2b36108d 100644
--- a/src/search.c
+++ b/src/search.c
@@ -243,7 +243,7 @@ int findnextstr(const char *needle, bool whole_word_only, int modus,
 		/* If we've reached the start or end of the buffer, wrap around;
 		 * but stop when spell-checking or replacing in a region. */
 		if (line == NULL) {
-			if (whole_word_only || modus == INREGION) {
+			if (whole_word_only || modus == INREGION || ISSET(NO_SEARCH_WRAP)) {
 				nodelay(midwin, FALSE);
 				return 0;
 			}
diff --git a/syntax/nanorc.nanorc b/syntax/nanorc.nanorc
index 9d7d9c0e..556b8a3f 100644
--- a/syntax/nanorc.nanorc
+++ b/syntax/nanorc.nanorc
@@ -14,7 +14,7 @@ color bold,purple "^[[:blank:]]*include[[:blank:]][^"]*([[:blank:]]|$)"
 color lime "^[[:blank:]]*extendsyntax[[:blank:]]+[[:alpha:]]+[[:blank:]]+(i?color|header|magic|comment|formatter|linter|tabgives)[[:blank:]]+.*"
 
 # The arguments of commands
-color brightgreen "^[[:blank:]]*(set|unset)[[:blank:]]+(afterends|allow_insecure_backup|atblanks|autoindent|backup|boldtext|bookstyle|breaklonglines|casesensitive|constantshow|cutfromcursor|emptyline|historylog|indicator|jumpyscrolling|linenumbers|locking|magic|minibar|mouse|multibuffer|noconvert|nohelp|nonewlines|positionlog|preserve|quickblank|rawsequences|rebinddelete|regexp|saveonexit|showcursor|smarthome|softwrap|stateflags|tabstospaces|trimblanks|unix|wordbounds|zap|zero)\>"
+color brightgreen "^[[:blank:]]*(set|unset)[[:blank:]]+(afterends|allow_insecure_backup|atblanks|autoindent|backup|boldtext|bookstyle|breaklonglines|casesensitive|constantshow|cutfromcursor|emptyline|historylog|indicator|jumpyscrolling|linenumbers|locking|magic|minibar|mouse|multibuffer|noconvert|nohelp|nosearchwrap|nonewlines|positionlog|preserve|quickblank|rawsequences|rebinddelete|regexp|saveonexit|showcursor|smarthome|softwrap|stateflags|tabstospaces|trimblanks|unix|wordbounds|zap|zero)\>"
 color brightgreen "^[[:blank:]]*set[[:blank:]]+(backupdir|brackets|errorcolor|functioncolor|keycolor|matchbrackets|minicolor|numbercolor|operatingdir|promptcolor|punct|quotestr|scrollercolor|selectedcolor|speller|spotlightcolor|statuscolor|stripecolor|titlecolor|whitespace|wordchars)[[:blank:]]+"
 color brightgreen "^[[:blank:]]*set[[:blank:]]+(fill[[:blank:]]+-?[[:digit:]]+|(guidestripe|tabsize)[[:blank:]]+[1-9][0-9]*)\>"
 color brightgreen "^[[:blank:]]*bind[[:blank:]]+((\^([A-Za-z]|[]/@\^_`-]|Space)|([Ss][Hh]-)?[Mm]-[A-Za-z]|[Mm]-([][!"#$%&'()*+,./0-9:;<=>?@\^_`{|}~-]|Space))|F([1-9]|1[0-9]|2[0-4])|Ins|Del)[[:blank:]]+([a-z]+|".*")[[:blank:]]+(main|help|search|replace(with)?|yesno|gotoline|writeout|insert|execute|browser|whereisfile|gotodir|spell|linter|all)\>"
-- 
2.30.2
```
