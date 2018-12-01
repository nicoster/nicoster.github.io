---
id: 160
title: Phases of Translation
date: 2006-06-27T17:19:00+00:00
author: nick
layout: post
permalink: /phases-of-translation/
categories:
  - Uncategorized
---
<h1><a name="_predir_phases_of_translation"></a>Phases of Translation</h1>
C and C++ programs consist of one or more source files, each of which contains some of the text of the program. A source file, together with its include files (files that are included using the #include preprocessor directive) but not including sections of code removed by conditional-compilation directives such as #if, is called a &ldquo;translation unit.&rdquo;
Source files can be translated at different times &mdash; in fact, it is common to translate only out-of-date files. The translated translation units can be kept either in separate object files or in object-code libraries. These separate translation units are then linked to form an executable program or a dynamic-link library (DLL).
Translation units can communicate using: 
<ul type="disc">
<li>Calls to functions that have external linkage.
</li>
<li>Calls to class member functions that have external linkage.
</li>
<li>Direct modification of objects that have external linkage.
</li>
<li>Direct modification of files.
</li>
<li>Interprocess communication (for Microsoft Windows-based applications only). </li>
</ul>
The following list describes the phases in which the compiler translates files:
<p class="dt"><em>Character mapping</em>
<p class="indent">Characters in the source file are mapped to the internal source representation. Trigraph sequences are converted to single-character internal representation in this phase.
<p class="dt"><em>Line splicing</em>
<p class="indent">All lines ending in a backslash () and immediately followed by a newline character are joined with the next line in the source file, forming logical lines from the physical lines. Unless it is empty, a source file must end in a newline character that is not preceded by a backslash.
<p class="dt"><em>Tokenization</em>
<p class="indent">The source file is broken into preprocessing tokens and white-space characters. Comments in the source file are replaced with one space character each. Newline characters are retained.
<p class="dt"><em>Preprocessing</em>
<p class="indent">Preprocessing directives are executed and macros are expanded into the source file. The #include statement invokes translation starting with the preceding three translation steps on any included text.
<p class="dt"><em>Character-set mapping</em>
<p class="indent">All source-character-set members and escape sequences are converted to their equivalents in the execution-character set. For Microsoft C and C++, both the source and the execution character sets are ASCII.
<p class="dt"><em>String concatenation</em>
<p class="indent">All adjacent string and wide-string literals are concatenated. For example,  "String  "  "concatenation " becomes  "String concatenation ".
<p class="dt"><em>Translation</em>
<p class="indent">All tokens are analyzed syntactically and semantically; these tokens are converted into object code.
<p class="dt"><em>Linkage</em>
<p class="indent">All external references are resolved to create an executable program or a dynamic-link library.
The compiler issues warnings or errors during phases of translation in which it encounters syntax errors.
The linker resolves all external references and creates an executable program or DLL by combining one or more separately processed translation units along with standard libraries.
<!--FOOTER_START-->
< language="JavaScript" src="MS-ITS:dsmsdn.chm::/html/msdn_footer.js" type="text/javascript"></script>
 
