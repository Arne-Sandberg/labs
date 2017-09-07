Makerbot Industries RFC 03: Multi-object file with object meta-info 

* Doc Revision: 0.2.2.1
* Protocol Revision: 0.1.1.1

# Abstract:
A file format that can be used to facilitate transfer of properly
-attributed designs for physical objects, and how they should be constructed
. This may include information at various levels of detail, from simple shape
data, to material types, temperatures, full toolpaths, etc. It may also contain
information about the chain of attribution for objects, and possibly the other
meta-info as well.

## 0. Terms (just a few):
* `.json` - A JavaScript Object Notation file (see
  [RFC-4627](http://www.ietf.org/rfc/rfc4627.txt))
* meta-info - I'm using this to refer to any information that describes the
  process of making the object, but there could be room for specifying expected
  properties of objects (for smarter slicing) or other things.

## 1. Goals: 
* To have a way to store objects arranged on a build plate
* To store information about how those objects should be created
* To track the chains of attribution for objects
* Openness? By keeping the format clear/simple we make it easier for us & them
  to develop against it.
* Extensibility. This RFC should help define version 0.1.1.1 of this file
  format, but should leave the format flexible enough that future details can be
  added.

## 2. Uses Cases:
* User uploads a `.thing` file to thingiverse to share.
* User uploads an `.stl` to thingiverse which is converted to `.thing`.
* User downloads a `.thing` file from thingiverse for printing.
* User downloads a `.thing` file from thingiverse for editing.
* User creates a plate of things in MakerWare, assigns colors, toolheads, etc., saves as `.thing`.

## 3. Overview:
This format consists of files stored in a compressed folder, using zip
compression.  The folder MAY contain one or more files describing one or more 3D
solid geometries and one or more `.json` files with details about those
geometries. The file MUST contain exactly one well specified `.json` file named
`manifest.json` with information about the other files stored in the `.
thing` zipped folder.

For forward/backward compatibility the meanings of JSON names (names of `.json
files` and the names of name/value pairs within `.json` files) should never
change. If, when parsing a `.json` file, an unrecognized name is encountered it
MUST be ignored *loudly* via logging or displaying a warning message. 

## 4. Current Version Overview (Version 0.1.1.1):
* A `.thing` file can hold `.stl`, `.obj`, and `.json` files.
* A `.thing` MUST contain a well defined `manifest.json` ( section 4.1 )
* A `.thing` MUST contain one or more objects in `.stl` or `.obj` format
  (section 4.5) 
* A `.thing` MAY contain other files (unspecified)

### 4.1 Example "manifest.json"
	
    { "namespace": "http://spec.makerbot.com/ns/thing.0.1.1.1"
    , "objects":
        { "bunny.stl": {}
        , "bunny2.stl": {}
        }
    , "constructions":
        { "plastic A": {}
        , "plastic B": {}
        }
    , "instances":
        { "NameA":
            { "object": "bunny.stl"
            , "scale": "mm"
            , "construction": "plastic A"
            , "xform" : "transform1"
            }
        , "NameB":
            { "object": "bunny2.stl"
            , "scale": "mm"
            , "construction": "plastic B"
            , "xform" : "transform2"
            }
        }
    , "transformations":
        { "transform1":
            { "matrix":
                [ [ 1.0, 0.0, 0.0, 23.1 ]
                , [ 0.0, 1.0, 0.0, 20.0 ]
                , [ 0.0, 0.0, 1.0, 9.9 ]
                , [ 0.0, 0.0, 0.0, 1.0 ]
                ]
            }
        , "transform2":
            { "matrix":
                [ [ 1.0, 0.0, 0.0, 23.0 ]
                , [ 0.0, 1.0, 0.0, 0.0 ]
                , [ 0.0, 0.0, 1.0, 0.0 ]
                , [ 0.0, 0.0, 0.0, 1.0 ]
                ]
            }
        }
    }

#### 4.1.1 Example Details
The above `manifest.json` entry expects to find a namespace at that URL which
contains DOM info and human readable info for what this specification is, etc
. It also expects to find 2 `.stl` files (`bunny.stl`, `bunny2.stl`) at the same
directory level as the file is.  The above JSON also defines two construction
types, `Tool 0`, and `Tool 1`, which it assumes the calling/using program or
person using the file can decode and use.  The file then defines a single build
plate with two objects (`NameA`, and `NameB`) to create, each item based on an
`.stl` object defined in the object section. 

### 4.2 namespace entry: 
Version information on this file format. Used to verify compatibility and
namespace/specification URI. each manifest MUST contain exactly one namespace
definition. 

### 4.3 objects entry:
List of objects bundled into this package. `.obj` and `.stl` files are located
relative to the `manifest.json` directory.  May support external links and URI
in the future. Each manifest MUST contain at least one object key.

### 4.4 construction entry:
List of construction systems. For version 0.1.1.1 these contain no information,
and only the name is significant. Future revisions may specify a JSON file of
construction methodology, or slice/tool control details.

### 4.5 instances entry:
List of all items that bundle together to make this print grouping. List of how
many and what objects we want instances of, in key-value pairs. Each key is a unique
id string containing one object name, each matched value in `instances` contains
a dictionary of details on the instance. Details MUST include at least one
object source, and MAY contain contain more construction or metadata. Unique key
names SHOULD be used as display names by the program using them. 

The instance dictionary MUST contain at least one object, with a matching object
in the `objects` list; MAY contain at most one scale entry, default to mm; MAY
contain at most one construction entry; and MAY contain at most one
transformation entry, which defaults to an identity matrix.

### 4.6 transformations entry:
Describes a 3D transformation for any objects in this .thing file
. Transformations are expressed as 4x4 matrices, stored row majorly
. Transformations must be Affine (only describing rotation, scale, and
translation). Each transformation is stored via a key, which MAY be randomly
generated, which is used for readability of the `manifest.json` file.
		
	"transformations":
	{ "xform1":
		{ "matrix": [[1,0,0,0],[0,1,0,0],[0,0,1,0],[0,0,0,1]]
		}
	, "xform2":
		{ "matrix": [[X,X,X,X],[X,X,X,X],[X,X,X,X],[X,X,X,X]]
		}
	, "xform3":
		{ "matrix": [[X,X,X,X],[X,X,X,X],[X,X,X,X],[X,X,X,X]]
		}
	}
  
## 5 Examples manifest files:

### 5.1 Total Minimum. Specify one object, assume the program can use the file
        as it wants.
    { "namespace": "http://spec.makerbot.com/ns/thing.0.1.1.1"
    , "objects":
        { "bunny.stl": {}
        }
    , "instances":
        { "bunny":
            { "object": "bunny.stl"
            , "scale": "mm"
            }
        }
    }

### 5.2 Material Minimum. Specify two objects, two materials, one object of both.
    { "namespace": "http://spec.makerbot.com/ns/thing.0.1.1.1"
    , "objects":
        { "bunny.stl": {}
        , "bunny2.stl": {}
        }
    , "constructions":
        { "plastic A": {}
        , "plastic B": {}
        }
    , "instances":
        { "NameA":
            { "object": "bunny.stl"
            , "scale": "mm"
            , "construction": "plastic B"
            }
        , "NameB":
            { "object": "bunny2.stl"
            , "scale": "mm"
            , "construction": "plastic B"
            }
        }
    }

### 5.3 Material Minimum. Specify two objects, two materials, one object of
        each.
    { "namespace": "http://spec.makerbot.com/ns/thing.0.1.1.1"
    , "objects":
        { "bunny.stl": {}
        , "bunny2.stl": {}
        }
    , "constructions":
        { "plastic A": {}
        , "plastic B": {}
        }
    , "instances":
        { "NameA":
            { "object": "bunny.stl"
            , "scale": "mm"
            , "construction": "plastic A"
            }
        , "NameB":
            { "object": "bunny2.stl"
            , "scale": "mm"
            , "construction": "plastic B"
            }
        }
    }

### 5.4 Attribution Minimum. Specify one object, creator, license.
    { "namespace": "http://spec.makerbot.com/ns/thing.0.1.1.1"
    , "objects":
        { "bunny.stl": {}
        }
    , "instances":
        { "bunny":
            { "object": "bunny.stl"
            , "scale": "mm"
            }
        }
    , "attribution":
        { "author": "Bob"
        , "license": "foo"
        }
    }

### 5.5  Example of load with transformations    
    {"namespace": "http://spec.makerbot.com/ns/thing.0.1.1.1"
	, "objects": 
		{ "bunny.stl": {}
		, "bunny2.stl": {}
		}
	, "constructions":
		{ "plasticA": {}
		, "plasticB": {}
		}
    , "instances":
		{ "NameA":
			{ "object": "bunny.stl"
			, "scale": "mm"
			, "construction": "plastic A"
			, "xform": "RANDNAME"
			}
    	, "NameB":
			{ "object": "bunny.stl"
			, "scale": "mm"
			, "construction": "plasticB"
			, "xform": "RANDNAME2"
			}
    	}
	,"transformations": 
		{ "RANDNAME":
			{ "matrix": [[1,0,0,0],[0,1,0,0],[0,0,1,0],[0,0,0,1]]
			}
    	, "RANDNAME2":
			{ "matrix": [[X,X,X,X],[X,X,X,X],[X,X,X,X],[X,X,X,X]]
			}
    	}
    }

## 5 Container format

### 5.1 ZIP format
`.thing` files are archived and compressed using standard modern ZIP archive
formats.

### 5.2 Implementations
In C++, the ZIP file handling is performed by the minizip library. This is part
of the zlib compression library.

In Python, the ZIP file handling is performed by the zipfile package. This is
part of the standard python distribution.

### 5.3 Directory structure
The manifest.json must be in the root directory of the zip file. The paths of
all files included in the zip should be relative to the manifest.json file. In
general it's good practice to put the model files in a "models/" subdirectory
, but this is not currently required.

# OLD  NOTES (do not remove until 1.0)

Z.1 "manifest.json"
.thing files have a single 'manifest.json' file containing a single json object
describing the contents of the archive in the format:
    
	{
		"version": "0.1.X.X",
		"attribution": {
			????????,
			{"author": "yournamehere"}
		}
		"files": [
			{"path": "bunny.stl"},
			{"path": "20mm-box.stl"},
			{"path": "teapot.obj"},
			{"path": "transformations.json"},
			{"path": "attributions.json"}
		]
	}

Z.1.2 The "attribution" Section
Decribes the chain of people involved in creating this file, and refers to other
files used to create this one. The references use some sort of checksum system,
I think Far has this partially worked out?  "author" should be self explanatory.

Z.1.3 The "files" Section
Lists the files contained as an array of paths to those files. "path"s double as
string identifiers for referring to files in other .json objects.

Z.2.1 "transformations.json"
Describes a 3D transformation on any objects in this .thing file
. Transformations are expressed as 4x4 matrices, stored row majorly
. Transformations must be Affine (only describing rotation, scale, and
translation). If there are multiple transformations on a single file, they
express that there are multiple of these objects in different locations. .things
offer no guarantees about whether the objects overlap. If no transformation is
specified for an object, an identity transformation is assumed.
	
	The transformations json file has the form:
	
	{
		"transformations": [
			{"object": "bunny.stl", "matrix": [[1,0,0,0],[0,1,0,0],[0,0,1,0],[0,0,0,1]]},
			{"object": "bunny.stl", "matrix": [[X,X,X,X],[X,X,X,X],[X,X,X,X],[X,X,X,X]]},
			{"object": "teapot.obj", "matrix": [[X,X,X,X],[X,X,X,X],[X,X,X,X],[X,X,X,X]]}
		]
	}
	
	where "object" refers to the path of a 3d model specified in the "files" section of "manifest.json".
	
X. Additional Comments:

X.1 Affine-ness of Transformations
For future versions we may wish to remove this constraint? I believe this is
to simplify coding for initial versions.
