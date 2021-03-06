===================================================================
Proposal for extended Vi tags file format
===================================================================

.. note::

    The contents of next section is a copy of FORMAT file in exuberant
    ctags source code in its subversion repository at sourceforge.net.
    I(Masatake YAMATO) changed only its format in the most of all
    parts.  I added a subsection for showing the position of universal
    ctags.

.. contents:: `Table of contents`
	:depth: 3
	:local:

----

:Version: 0.06 DRAFT
:Date: 1998 Feb 8
:Author: Bram Moolenaar <Bram at vim.org> and Darren Hiebert <dhiebert at users.sourceforge.net>


Introduction
---------------------------------------------------------------------

The file format for the "tags" file, as used by Vi and many of its
descendants, has limited capabilities.

This additional functionality is desired:

1. Static or local tags.
   The scope of these tags is the file where they are defined.  The same tag
   can appear in several files, without really being a duplicate.
2. Duplicate tags.
   Allow the same tag to occur more then once.  They can be located in
   a different file and/or have a different command.
3. Support for C++.
   A tag is not only specified by its name, but also by the context (the
   class name).
4. Future extension.
   When even more additional functionality is desired, it must be possible to
   add this later, without breaking programs that don't support it.


From proposal to standard
-------------------------------------------------------------------------

To make this proposal into a standard for tags files, it needs to be supported
by most people working on versions of Vi, ctags, etc..  Currently this
standard is supported by:

Darren Hiebert <dhiebert at users.sourceforge.net>
	Exuberant ctags

Bram Moolenaar <Bram at vim.org>
	Vim (Vi IMproved)

These have been or will be asked to support this standard:

Nvi
		Keith Bostic <bostic at bsdi.com>

Vile
		Tom E. Dickey <dickey at clark.net>

NEdit
		Mark Edel <edel at ltx.com>

CRiSP
		Paul Fox <fox at crisp.demon.co.uk>

Lemmy
		James Iuliano <jai at accessone.com>

Zeus
		Jussi Jumppanen <jussij at ca.com.au>

Elvis
		Steve Kirkendall <kirkenda at cs.pdx.edu>

FTE
		Marko Macek <Marko.Macek at snet.fri.uni-lj.si>


Backwards compatibility
---------------------------------------------------------------------------

A tags file that is generated in the new format should still be usable by Vi.
This makes it possible to distribute tags files that are usable by all
versions and descendants of Vi.

This restricts the format to what Vi can handle.  The format is:

1. The tags file is a list of lines, each line in the format::

	{tagname}<Tab>{tagfile}<Tab>{tagaddress}


   {tagname}
	Any identifier, not containing white space..

	EXCEPTION: Universal ctags violates this item of the proposal;
	tagname may contain spaces. However, tabs are not allowed.

   <Tab>
	Exactly one TAB character (although many versions of Vi can
	handle any amount of white space).

   {tagfile}
	The name of the file where {tagname} is defined, relative to
	the current directory (or location of the tags file?).

   {tagaddress}
	Any Ex command.  When executed, it behaves like 'magic' was
	not set.

2. The tags file is sorted on {tagname}.  This allows for a binary search in
   the file.

3. Duplicate tags are allowed, but which one is actually used is
   unpredictable (because of the binary search).

The best way to add extra text to the line for the new functionality, without
breaking it for Vi, is to put a comment in the {tagaddress}.  This gives the
freedom to use any text, and should work in any traditional Vi implementation.

For example, when the old tags file contains::

	main	main.c	/^main(argc, argv)$/
	DEBUG	defines.c	89

The new lines can be::

	main	main.c	/^main(argc, argv)$/;"any additional text
	DEBUG	defines.c	89;"any additional text

Note that the ';' is required to put the cursor in the right line, and then
the '"' is recognized as the start of a comment.

For Posix compliant Vi versions this will NOT work, since only a line number
or a search command is recognized.  I hope Posix can be adjusted.  Nvi suffers
from this.


Security
------------------------------------------------------------------

Vi allows the use of any Ex command in a tags file.  This has the potential of
a trojan horse security leak.

The proposal is to allow only Ex commands that position the cursor in a single
file.  Other commands, like editing another file, quitting the editor,
changing a file or writing a file, are not allowed.  It is therefore logical
to call the command a tagaddress.

Specifically, these two Ex commands are allowed:

* A decimal line number::

	89

* A search command.  It is a regular expression pattern, as used by Vi,
  enclosed in // or ??::

	/^int c;$/
	?main()?

There are two combinations possible:

* Concatenation of the above, with ';' in between.  The meaning is that the
  first line number or search command is used, the cursor is positioned in
  that line, and then the second search command is used (a line number would
  not be useful).  This can be done multiple times.  This is useful when the
  information in a single line is not unique, and the search needs to start
  in a specified line.
  ::

	/struct xyz {/;/int count;/
	389;/struct foo/;/char *s;/

* A trailing comment can be added, starting with ';"' (two characters:
  semi-colon and double-quote).  This is used below.
  ::

	89;" foo bar

This might be extended in the future.  What is currently missing is a way to
position the cursor in a certain column.


Goals
--------

Now the usage of the comment text has to be defined.  The following is aimed
at:

1. Keep the text short, because:

   * The line length that Vi can handle is limited to 512 characters.
   * Tags files can contain thousands of tags.  I have seen tags files of
     several Mbytes.
   * More text makes searching slower.

2. Keep the text readable, because:

   * It is often necessary to check the output of a new ctags program.
   * Be able to edit the file by hand.
   * Make it easier to write a program to produce or parse the file.

3. Don't use special characters, because:

   * It should be possible to treat a tags file like any normal text file.

Proposal
-----------

Use a comment after the {tagaddress} field.  The format would be::

	{tagname}<Tab>{tagfile}<Tab>{tagaddress}[;"<Tab>{tagfield}..]


{tagname}
	Any identifier, not containing white space..

	EXCEPTION: Universal ctags violates this item of the proposal;
	name may contain spaces. However, tabs are not allowed.
	Conversion, for some characters including <Tab> in the "value",
	explained in the last of this section is applied.

<Tab>
	Exactly one TAB character (although many versions of Vi can
	handle any amount of white space).

{tagfile}
	The name of the file where {tagname} is defined, relative to
	the current directory (or location of the tags file?).

{tagaddress}
	Any Ex command.  When executed, it behaves like 'magic' was
	not set.  It may be restricted to a line number or a search
	pattern (Posix).

Optionally:

;"
		semicolon + doublequote: Ends the tagaddress in way that looks
		like the start of a comment to Vi.

{tagfield}
		See below.

A tagfield has a name, a colon, and a value: "name:value".

* The name consist only out of alphabetical characters.  Upper and lower case
  are allowed.  Lower case is recommended.  Case matters ("kind:" and "Kind:
  are different tagfields).

* The value may be empty.
  It cannot contain a <Tab>.

  - When a value contains a "\\t", this stands for a <Tab>.
  - When a value contains a "\\r", this stands for a <CR>.
  - When a value contains a "\\n", this stands for a <NL>.
  - When a value contains a "\\\\", this stands for a single '\\' character.

  Other use of the backslash character is reserved for future expansion.
  Warning: When a tagfield value holds an MS-DOS file name, the backslashes
  must be doubled!

  EXCEPTION: Universal ctags introduces more conversion rules.
  The characters in range 0 to 0x20 and 0x7F is converted
  to \x prefixed hexadecimal number if the characters are not handled
  in the abouve "value" rules.

Proposed tagfield names:

=============== =============================================================================
FIELD-NAME	DESCRIPTION
=============== =============================================================================
arity		Number of arguments for a function tag.

class		Name of the class for which this tag is a member or method.

enum		Name of the enumeration in which this tag is an enumerator.

file		Static (local) tag, with a scope of the specified file.  When
		the value is empty, {tagfile} is used.

function	Function in which this tag is defined.  Useful for local
		variables (and functions).  When functions nest (e.g., in
		Pascal), the function names are concatenated, separated with
		'/', so it looks like a path.

kind		Kind of tag.  The value depends on the language.  For C and
		C++ these kinds are recommended:

		c
			class name

		d
			define (from #define XXX)

		e
			enumerator

		f
			function or method name

		F
			file name

		g
			enumeration name

		m
			member (of structure or class data)

		p
			function prototype

		s
			structure name

		t
			typedef

		u
			union name

		v
			variable

		When this field is omitted, the kind of tag is undefined.

struct		Name of the struct in which this tag is a member.

union		Name of the union in which this tag is a member.
=============== =============================================================================


Note that these are mostly for C and C++.  When tags programs are written for
other languages, this list should be extended to include the used field names.
This will help users to be independent of the tags program used.

Examples::

	asdf	sub.cc	/^asdf()$/;"	new_field:some\svalue	file:
	foo_t	sub.h	/^typedef foo_t$/;"	kind:t
	func3	sub.p	/^func3()$/;"	function:/func1/func2	file:
	getflag	sub.c	/^getflag(arg)$/;"	kind:f	file:
	inc	sub.cc	/^inc()$/;"	file: class:PipeBuf


The name of the "kind:" field can be omitted.  This is to reduce the size of
the tags file by about 15%.  A program reading the tags file can recognize the
"kind:" field by the missing ':'.  Examples::

	foo_t	sub.h	/^typedef foo_t$/;"	t
	getflag	sub.c	/^getflag(arg)$/;"	f	file:


Additional remarks:

* When a tagfield appears twice in a tag line, only the last one is used.


Note about line separators:

Vi traditionally runs on Unix systems, where the line separator is a single
linefeed character <NL>.  On MS-DOS and compatible systems <CR><NL> is the
standard line separator.  To increase portability, this line separator is also
supported.

On the Macintosh a single <CR> is used for line separator.  Supporting this on
Unix systems causes problems, because most fgets() implementation don't see
the <CR> as a line separator.  Therefore the support for a <CR> as line
separator is limited to the Macintosh.

Summary:

==============  ======================  =========================
line separator	generated on		accepted on
==============  ======================  =========================
<LF>		Unix			Unix, MS-DOS, Macintosh
<CR>		Macintosh		Macintosh
<CR><LF>	MS-DOS			Unix, MS-DOS, Macintosh
==============  ======================  =========================

The characters <CR> and <LF> cannot be used inside a tag line.  This is not
mentioned elsewhere (because it's obvious).


Note about white space:

Vi allowed any white space to separate the tagname from the tagfile, and the
filename from the tagaddress.  This would need to be allowed for backwards
compatibility.  However, all known programs that generate tags use a single
<Tab> to separate fields.

There is a problem for using file names with embedded white space in the
tagfile field.  To work around this, the same special characters could be used
as in the new fields, for example "\\s".  But, unfortunately, in MS-DOS the
backslash character is used to separate file names.  The file name
"c:\\vim\\sap" contains "\\s", but this is not a <Space>.  The number of
backslashes could be doubled, but that will add a lot of characters, and make
parsing the tags file slower and clumsy.

To avoid these problems, we will only allow a <Tab> to separate fields, and
not support a file name or tagname that contains a <Tab> character.  This
means that we are not 100% Vi compatible.  However, there is no known tags
program that uses something else than a <Tab> to separate the fields.  Only
when a user typed the tags file himself, or made his own program to generate a
tags file, we could run into problems.  To solve this, the tags file should be
filtered, to replace the arbitrary white space with a single <Tab>.  This Vi
command can be used::

	:%s/^\([^ ^I]*\)[ ^I]*\([^ ^I]*\)[ ^I]*/\1^I\2^I/

(replace ^I with a real <Tab>).


TAG FILE INFORMATION:

Psuedo-tag lines can be used to encode information into the tag file regarding
details about its content (e.g. have the tags been sorted?, are the optional
tagfields present?), and regarding the program used to generate the tag file.
This information can be used both to optimize use of the tag file (e.g.
enable/disable binary searching) and provide general information (what version
of the generator was used).

The names of the tags used in these lines may be suitably chosen to ensure
that when sorted, they will always be located near the first lines of the tag
file.  The use of "!_TAG_" is recommended.  Note that a rare tag like "!"
can sort to before these lines.  The program reading the tags file should be
smart enough to skip over these tags.

The lines described below have been chosen to convey a select set of
information.

Tag lines providing information about the content of the tag file::

    !_TAG_FILE_FORMAT	{version-number}	/optional comment/
    !_TAG_FILE_SORTED	{0|1}			/0=unsorted, 1=sorted/

The {version-number} used in the tag file format line reserves the value of
"1" for tag files complying with the original UNIX vi/ctags format, and
reserves the value "2" for tag files complying with this proposal. This value
may be used to determine if the extended features described in this proposal
are present.

Tag lines providing information about the program used to generate the tag
file, and provided solely for documentation purposes::

    !_TAG_PROGRAM_AUTHOR	{author-name}	/{email-address}/
    !_TAG_PROGRAM_NAME	{program-name}	/optional comment/
    !_TAG_PROGRAM_URL	{URL}	/optional comment/
    !_TAG_PROGRAM_VERSION	{version-id}	/optional comment/

Universal ctags
--------------------------------

Universal ctags supports this proposal with some
exceptions.


Exceptions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. {tagname} in tags file generated by Universal ctags may contain
   spaces. Parsers for documents like Tex and reStructuredText need
   this exceptions. See {tagname} of Proposal section for more detail
   about the conversion.

.. _compat-output:

Compatible output and weakness
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. NOT REVIEWED YET

Default behavior (``--output-format=u-ctags`` option) has the
exceptions.  In ther hand, with ``--output-format=e-ctags`` option
ctags has no exception; Universal-ctags command may use the same file
format as Exuberant-ctags. However, ``--output-format=e-ctags`` throws
away a tag entry which name includes a space or a tab
character. ``TAG_OUTPUT_MODE`` psuedo tag tells which format is
used when ctags generating tags file.
