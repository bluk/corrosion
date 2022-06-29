# Magic Numbers

Magic numbers are values which may not have any real meaning and are just
"magically" chosen.

For instance, if there's a serialization format which requires the number `42`
to be written before any other data, then it is considered a magic number. The
`42` value itself does not have any significance. It could have been
`0x00043110`. The value may have an amusing meaning to the person choosing the
number, but other than being useful for identifying the format/protocol, the
number is usually not useful.

Strings are just bytes so magic "numbers" could be strings as well. Some
protocols require a "Hello" message to be sent to identify the protocol. The
string could be the protocol name or some other magic number.

In some cases, a magic number may be a known constant input into a function. The
value may have been chosen for specific reasons (such as cryptographic
properties).
