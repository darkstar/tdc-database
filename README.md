This is the database from the Total DOS Collection (TDC) project, converted to yaml.

## Origin

The data is generated by an (ugly) Python script that takes the official Access database
and the official DAT file as input, and (slowly) produces the yaml files from that.

The Access database contains the whole metadata of each game, like names, languages, 
genres, links to MobyGames, parent-child relationships, etc.

The DAT file on the other hand contains the actual filenames, file sizes, timestamps
and checksums of the files that make up that particular game.

## Schema

The yaml files consist of various fields, which are described in the following paragraphs.
This will also document and describe the various conventions being used by TDC.

All fields that are not marked as *optional* are present in every entry.

- `filename`:
  This is the file name as taken from the DAT file. It includes tags for the publisher,
  the year, the genre and some other details. It is modeled after the TOSEC naming
  convention, with all the disadvantages that come with that (almost impossible to parse
  correctly, extremely long names for some entries, etc.)

- `tdc_id`:
  Contains a unique ID that identifies the entry within TDC. It consists of two parts,
  the major ID and the minor ID, separated by a dot. The major ID identifies a particular
  game, while the minor ID identifies variants of that game (like different languages,
  different re-releases, etc.)
  It is a sparse list, which means that some major IDs (and some minor IDs within a single
  major ID) are not used/empty. This can happen if entries later get re-classified and
  re-assigned a different major ID (e.g. when it is found out that they are the same game,
  just with a different language title).
  This is by design, and usually, such IDs will not be re-used.

- `parent_id`:
  This *optional* field, when present, determines the "parent" of the current game. What
  constitutes a "parent" can vary a bit, but it usually denotes that the parent and this
  entry are closely related. For example if it is a fan translation, the parent would be
  the base game that was used for the translation. For an installer, the parent would be
  the installed version of that exact game version.
  It is more an "organizational parent" rather than a "development parent", so releases
  of the same game that have different version numbers are usually not set up to be parents
  of each other.

- `parent_id_valid`:
  This *optional* field is used for checking purposes. Usually the parent ID of a game should
  share the same major ID number than the game itself. But in some cases, the parent ID
  for a game is a completely different major ID for a particular reason. In these cases,
  this field is present and set to `true` to show that the different major ID is in fact
  correct and not a typo.
  Typos happen due to the manual nature of the database, and these are regularly checked
  by scripts, so to help these scripts in not flagging "false positives", this field was
  introduced. We expect that at some point it will go away.

- `title`:
  This is a container element that contains at least one (but possibly multiple) of the
  following sub-elements, each describing the title of the game in question. The title does
  not follow the common practice (for sorting) of putting articles after the main title.
  Instead the full title is taken as-is. So it's not "7th Guest, The" but "The 7th Guest".

  - `box`: The *optional* title as written on the box (if the game came in a box). Note
    that this can be in a language that's not English
  - `screen`: The *optional* title as written on the title screen. This entry might be
    missing if it is the same as the `box` title. But at least one of these two is
    available for every game. This title can also be in a language other than English.
  - `translation`: *optional* A translation of the title into English. This is usually used
    for titles that are not in English (or not even in latin script) so that at least some
    sense can be made of the title
  - `transliteration`: (*optional*) Especially for asian titles, this entry contains the
    transliteration ('pronounciation') of the title. For Japanese titles, the Revised
    Hepburn romanization is used.
  - `qualifier`: An *optional* additional part of the title that describes the game.
    For example "Deluxe Edition", "Demo", "Installer", or similar

- `publisher`:
  Contains the publisher of the game. As with titles, the publisher is not modified for better
  sorting, so any articles are simply left where they are.

- `year`:
  Contains the year the game was published, if known. If it is not known, the unknown parts
  of the year will be replaced by an "x", as in `198x` or `19xx`. So this is not always an
  integer. One of the goals of TDC is to eliminate these non-integer years, or at least reduce
  them as much as possible, but sometimes it's simply not doable.

- `genre`:
  This field contains either one string, or a yaml list of multiple strings, to describe the
  genre of the game. There is currently no fixed list of what genres are possible in TDC.

- `language`:
  This contains either one or more strings defining the in-game language(s) of the game.
  Currently the list is from the following languages, but that is not a hard limit (it's just
  that no game was yet found that uses a different language): English, Spanish, French, Russian,
  Italian, Chinese, German, Polish, Hebrew, Finnish, Czech, Norwegian, Japanese, Dutch, Danish,
  Korean, Hungarian, Ukrainian, Swedish, Indonesian, Slovenian, Slovakian, Serbian, Belarusian,
  Arabic, Greek, Latin, Turkish, Icelandic, Croatian, Portuguese, Estonian, Persian, Catalan,
  Lithuanian, Bulgarian, Afrikaans.

  In the future it might be desirable to have these represented by proper ISO 639 language codes,
  since some of the Language names above are actually multiple languages (e.g. Norwegian or Arabic)

- `dates`:
  This *optional* entry only appears when at least one of its sub-entries has a value. It
  describes timestamps of the entry in the database, and as such is pure TDC metadata not directly
  related to the game in question

  - `added`: This *optional* field specifies when this entry was added to TDC
  - `last_modified`: This *optional* field specifies when the database entry was last modified

  Both timestamps are in ISO8601/RFC3339 format, to second precision and without local offset

- `version`:
  This *optional* field specifies the version of the game. It is only present if there is an
  actual version number visible anywhere (on the screen or in the game executable files). In the
  past, the version number "1.0" was ignored and implicit, but that has been changed and now
  even games that show a version 1.0 should have that reflected in this version tag. But since
  that was a recent change, it might be that not all "v1.0" games already have that version number
  reflected here. This is a bug in TDC and should be fixed!

- `release`:
  An *optional* field similar to the `version` field. Some games talk about a "release" instead of
  a "version", which can be represented here. There are also games which includ both, a version and
  a release.

- `type`:
  This *optional* field specifies what type of release this game is. In most cases it is "zip",
  meaning that it's a zip file containing the installed game's files. But it can also be "ima"
  for a floppy image, "iso" or "bin-cue" for CD ROM images, "kfx" for KryoFlux disk dumps, and
  some other formats.

- `tags`:
  This *optional* container element contains the tags describing the game. There are two types of
  tags, normal (binary, boolean, plain) tags that are either present or not, and "extended" tags
  that also come with a number. The number for those tags is simply a counter that increments with
  each variant of the game featuring that tag. For example the `hack` tag is `hack: 1` for the first
  hacked game variant, `hack: 2` for the second and so on. The order is completely random and has no
  significance.

  With the introduction of the stable TDC ID, it is expected that these tags will lose their number
  at some point, since the TDC ID can be used to uniquely identify different variants of the same game.

  The following tags are currently implemented, and each one will become a sub-element of the `tags`
  element when present (either as a simple list entry or as a tag:value entry):

  - `Shareware`: The game is shareware
  - `SharewareRegistered`: The game is the registered shareware version
  - `Freeware`: The game has been released as freeware
  - `DOSConversion`: The game is a DOS conversion of a PC booter game (i.e. the game was changed so
  that it can simply be launched from DOS instead of having to boot from a floppy disk)
  - `KnownGood`: The game was verified by some external means and is known to be a good dump
  - `Translated`: The game was a non-official or fan-made translation. The language specified in the
  `language` tag is the language it was translated into.
  - `Hack`: The game is some sort of hacked version
  - `Bad`: The game is in some way or form "bad" and does not work correctly. Usually these are only
  kept in TDC if no better dump is known, or to explicitly document them as bad
  - `Alternate`: An alternate version of the game where the exact differences are not necessarily
  known or documented.
  - `Overdump`: This tag specifies that at least one of the files for this game are larger than they
  need to be, usually with lots of zeroes appended at the end of e.g. the EXE file or the floppy image.
  - `Fixed`: This game has been fixed in some way to make it work. It could be a workaround for a bug,
  or a fix to make the game playable on newer DOS versions or similar. Usually the exact nature of
  the fix should be documented in the comments (or the TDC wiki)

- `requirements`:
  This *optional* element, if present, specifies the minimum system requirements to play this
  game. It has multiple sub-elements that are each individual requirements that must all be met.

  - `cpu`: This specifies the minimum CPU required to play the game. One of 8086, 8088, 80286, 80386,
  80486, pentium.

  Other requirements are not yet implemented in the database, but it is a goal of TDC to add e.g.
  memory requirements or graphics adapter requirements too.

  This field might change significantly in the future!

- `media`:
  This *optional* field contains the media that this game came on. The following values are possible:
  360k, 720k, 1.2M, 1.44M, Cartridge, CD-ROM, Download, Type-In

  Theoretically, it is possible that there are multiple entries in the `media` field, but I don't think
  any games currently have more than one entry

- `installer`:
  This *optional* boolean field is `true` if the game in question is an installer, and not the
  ready-to-run game itself.

- `demo`:
  This *optional* boolean field is `true` if that game is just a (playable) demo

- `compilation`:
  This *optional* boolean field is `true` if that game is a compilation of multiple games

- `patch`:
  This *optional* boolean field is `true` if that game is a patch for another version of that game

- `cd_audio`:
  This *optional* boolean field is `true` if the game contains and uses CD (Redbook) Audio

- `commands`:
  This *optional* entry contains sub-entries of how to configure (setup) or start the game. This
  can be helpful for frontend development, or as a guide for games that contain multiple COM, EXE
  or BAT files.

  - `play`: (*optional*) If present, this field specifies the command to exceute to actually play
  the game

  - `setup`: (*optional*) If present, this field specifies the command to configure the game (change
  the sound card or similar)

- `references`:
  This *optional* element contains sub-elements that reference other projects or websites, to
  cross-reference the game. At the moment, the following sub-elements are defined:

  - `mobygames` contains an url to the MobyGames page for that game

  - `dosid` contains ...???

  - `exodos` contains the filename of the game in the eXoDOS collection. Note that this is always
  just an approximation, as eXoDOS is known to hack, patch and modify their games to make them
  "better", so an exact match might often be not at all possible.

- `notes`:
  This *optional* element contains any freeform text notes from the Access database

- `files`:
  This entry contains the actual file names, sizes and checksums from the DAT file. It is
  basically a list of entries, one for each file, in the following format. Note that, similar
  to ZIP files, directories are explicitly contained in the list to capture their timestamps.

  - `name`: The name of the file, relative to the game's base directory. The names are in DOS
  format and use a backslash as path separator. If the name ends with a backslash, it represents
  a directory.

  - `size`: The size of the file, in bytes. If the entry is a directory, the size is always 0.
  Note that the reverse is not true, there can definitely be regular files that are 0 bytes in size.

  - `date`: The DOS timestamp of the file. This is where it gets tricky, as DOS did not check the
  timestamps for validity, and so "impossible" timestamps could exist, like 25:34:62. To handle that,
  this field has two possible formats:

    - it is either a regular, valid ISO 8901 / RFC3339 timestamp, like `1995-11-11T06:11:10`

    - or it is an invalid timestamp that has the same format as the ISO timestamp, but with an "X"
    instead of the "T" of the ISO format, like `1983-00-24X29:55:04`

  When parsing this field you should wrap the ISO timestamp parsing in a try...catch block and
  handle the "invalid" case as is convenient for you (ignore, fix, ...)

  - `crc`: the CRC32 checksum of the complete file, in hex format with a leading `0x` string. If
  the entry is a directory or a 0-byte size file, the `crc` field is missing
