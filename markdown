#!/bin/sh

FILE="$1"
tmp="${FILE##*/}"
FILENAME="${tmp%.*}.html"

awk '
BEGIN {
    # Flags
	UN_LIST = 0
    O_LIST = 0
    CODE = 0
}

# Headings
/^#/ { c = length($1); $1 = "<h"c">"; $0 = $0 " </h"c">" }

# Paragraphs
/^\t|^[[:alpha:]]/ { sub(/^\t/,""); $0 = "<p> " $0 " </p>" }

# Unordered List
/^\* |^\+ |^- / { if (!UN_LIST) { print "<ul>"; UN_LIST = 1 }; $1 = "\t<li>"; $0 = $0 " </li>" }

# Ordered List
/^[[:alnum:]]\./ { if (!OR_LIST) { print "<ol>"; OR_LIST = 1 }; $1 = "\t<li>"; $0 = $0 " </li>" }

# Blockquotes
/^> / { $1 = "\<blockquote>"; $0 = $0 " </blockquote>" }

# Code 
/^```/ {
    CODE = 1; $0 = "<code>"; print
    while(getline != 0) {
        if ($0 ~ /```/) { CODE = 0; sub(/```/,"</code>"); break }
		print $0
	}
}

{
        # Flags
        if ($1 !~ /^<li>/ && UN_LIST && !OR_LIST) { print "</ul>"; UN_LIST = 0 }
        if ($1 !~ /^<li>/ && OR_LIST && !UN_LIST) { print "</ol>"; OR_LIST = 0 }
    
        # Where does the magic happen.
        print
}

END {
    # Flags
    if ($1 !~ /^<li>/ && UN_LIST && !OR_LIST) { print "</ul>"; UN_LIST = 0 }
    if ($1 !~ /^<li>/ && OR_LIST && !UN_LIST) { print "</ol>"; OR_LIST = 0 }
    
}'
< "$FILE" > ./"$FILENAME"
