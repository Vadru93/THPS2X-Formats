# THPS2X-Formats

## The THPS2X Levels have 4 files
* .DDM - containing material information
* .DDX - containing textures
* .psx - containing mesh data
* .trg - containing script data


# .DDM
* [DDM Header](#ddm-header)
* Array of [DDM Object Header](#ddm-object-header)
* Array of [DDM Object](#ddm-object)

## DDM Header

* 4 bytes - version
* 4 bytes - unknown, maybe number of materials?
* 4 bytes - number of [DDM Object Header](#ddm-object-header), [DDM Object](#ddm-object)

## DDM Object Header
* 4 bytes - offset to object, counting from begining of file
* 4 bytes - size

## DDM Flags
* 0xE - fake 3d grass. It uses textures grass[a-g][**] where a-g depends on grass type and ** is the grass layer.
The grass is always 16 layers starting from 00.

Example code in c++:

       `
       switch(flags & 0xE)
       {
       case 2:
          sprintf(material_name, "grassa%02u", layer);
          break;
        case 4:
          sprintf(material_name, "grassb%02u", layer);
          break;
        case 6:
          sprintf(material_name, "grassc%02u", layer);
          break;
        case 8:
          sprintf(material_name, "grassd%02u", layer);
          break;
        case 10:
          sprintf(material_name, "grasse%02u", layer);
          break;
        case 12:
          sprintf(material_name, "grassf%02u", layer);
          break;
        case 14:
          sprintf(material_name, "grassg%02u", layer);
          break;
        }
`
          

## DDM Object
* 4 bytes - index, this is the index linking the object to .psx, however sometimes it seems cannot find the ddm object by index so when loading the objects in .psx should first loop throught ddm objects and check if find the index, else check if find the checksum.
* 4 bytes - checksum
* float - anim speed X
* float - anim speed Y
* float - unknown
* 4 bytes - unknown
* 4 bytes - [DDM Flags](#ddm-flags)
* 64 bytes - object name
* 32 bytes - unknown, maybe some bounding box?
* 4 bytes - Number of [Vertices](#ddm-vertex)
* 4 bytes - Number of indices
* 4 bytes - Number of [Materials](#material)
* Array of [Materials](#material)
* Array of [Vertices](#ddm-vertex)
* Array of Indices - each index is 2 bytes
* Array of [Material Splits](#material-split)

## Material
* 64 bytes - name
* 64 bytes - texture name
* 4 bytes - draw order
* 4 bytes - material id
* float[3] - unknown maybe some color?
* 4 bytes - unknown

## DDM Vertex
* float - x need to multiply by -1 because level is flipped
* float - y need to multiply by -1 because level is flipped
* float - z
* float - x normal
* float - y normal
* float - z normal
* 4 bytes - RGBX;
* float - u;
* float - v;

## Material Split
* 2 bytes - material index
* 2 bytes - offset, relative to array of indices
* 2 bytes - number of indices



# .PSX
The `.PSX` contains collision data and since in THPS3 collision data and mesh data use shared vertices I use the [PSX Object](#psx-object) vertices when importing.
However for later games you can probably use [DDM Object](#ddm-object) for collision and [PSX Object](#psx-object) for mesh.
* [PSX Header](#psx-header)
* Array of [Position](#fixed-vertex) -  the position of the object
* Array of [PSX Objects](#psx-object)


## PSX Header
* 4 bytes - version
* 4 bytes - RGB offset
* 4 bytes - number of [PSX Object](#psx-object)

## PSX Object





