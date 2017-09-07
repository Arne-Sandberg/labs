# .makerbot File Spec

.makerbot files are the print files that are sent to MakerBot printers after a
3D model is sliced and prepared for printing.  They are somewhat analogous to
gcode files.

They are .zip files and can be renamed to .zip and extracted to see their
contents which are jsontoolpath, metadata, and thumbnail files.

## Versioning

.makerbot files have a version number, which is contained in the meta.json file. 
This version number follows [semantic versioning](http://semver.org/).

Version 1.0.0 is the first version with this version number. Any meta.json file 
that does not have a "version" key is by definition a version 0.0.3 file.

Version 2.0.0 allows for comment commands in JsonToolPath files.

Version 3.0.0 moves to a multi-extruder-by-default scheme for the meta file. Any 
key in the metafile that can be different on a per-extruder basis should always 
be a list, with a number of entries equivalent to the number of extruders on the 
machine - NOT the number of extruders used in a print.

## JsonToolpath File

filename: "print.jsontoolpath"

Describes the toolpath. See [the spec](jsontoolpath.md) in this directory for
more information regarding this file.  This file can be reduced to a gcode file.

## Thumbnails

.makerbot files contain 3 thumbnails:

    thumbnail_55x40.png
    thumbnail_110x80.png
    thumbnail_320x200.png

The thumbnails will be displayed for instance in the Bot LCD UI.

Thumbnails can be any image files matching these dimensions.

## Meta Data

filename: meta.json

Contains meta information about the file such as slicer settings used. See 
[the spec](metafile.md) in this directory for more information about this file.
