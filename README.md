# GNU nano (5.4) - nosearchwrap

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


## Modification Summary
```diff
doc/nano.1
325a326,328
> .TP
> .BR \-\-nosearchwrap
> Don't wrap around to the start or end of the buffer when performing search or replace.

doc/nanorc.5
201a202,204
> .B set nosearchwrap
> Don't wrap around to the start or end of the buffer when performing search or replace.
> .TP

src/nano.c
639a640
> 	print_opt("", "--nosearchwrap", N_("Don't wrap past EOF when search/replace"));
1750a1752
> 		{"nosearchwrap", 0, NULL, '\x01'},
2036a2039,2041
> 				break;
> 			case 0x01:
> 				SET(NO_SEARCH_WRAP);

src/nano.h
546c546,547
< 	INDICATOR
---
> 	INDICATOR,
> 	NO_SEARCH_WRAP

src/rcfile.c
111a112
> 	{"nosearchwrap", NO_SEARCH_WRAP},

src/search.c
265c265
< 			if (whole_word_only || modus == INREGION) {
---
> 			if (whole_word_only || modus == INREGION || ISSET(NO_SEARCH_WRAP)) {

syntax/nanorc.nanorc
10c10
< color brightgreen "^[[:space:]]*(set|unset)[[:space:]]+(afterends|allow_insecure_backup|atblanks|autoindent|backup|backwards|boldtext|breaklonglines|casesensitive|constantshow|cutfromcursor|emptyline|finalnewline|historylog|indicator|jumpyscrolling|linenumbers|locking|morespace|mouse|multibuffer|noconvert|nohelp|nopauses|nonewlines|nowrap|positionlog|preserve|quickblank|quiet|rawsequences|rebinddelete|regexp|saveonexit|showcursor|smarthome|smooth|softwrap|suspendable|tabstospaces|trimblanks|unix|view|wordbounds|zap)\>"
---
> color brightgreen "^[[:space:]]*(set|unset)[[:space:]]+(afterends|allow_insecure_backup|atblanks|autoindent|backup|backwards|boldtext|breaklonglines|casesensitive|constantshow|cutfromcursor|emptyline|finalnewline|historylog|indicator|jumpyscrolling|linenumbers|locking|morespace|mouse|multibuffer|noconvert|nohelp|nopauses|nosearchwrap|nonewlines|nowrap|positionlog|preserve|quickblank|quiet|rawsequences|rebinddelete|regexp|saveonexit|showcursor|smarthome|smooth|softwrap|suspendable|tabstospaces|trimblanks|unix|view|wordbounds|zap)\>"
```
