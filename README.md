# THPS2X-Formats

## The THPS2X Levels have 4 files
* .DDM - containing material information
* .DDX - containing textures
* .psx - containing mesh data
* .trg - containing script data


# .DDM

## Header

* 4 bytes - version
* 4 bytes - unknown, maybe number of materials?
* 4 bytes - number of objects

## Object Header
* 4 bytes - checksum
* 4 bytes - index, this is the index of the object in .psx, however sometimes it seems cannot find the ddm object by index so when loading the objects in .psx should first loop throught ddm objects and check if find the index, else check if find the checksum.
* 4 bytes - offset to object, counting from begining of file
* 4 bytes - size

## DDM Flags
* 0xE - fake 3d grass. It uses textures grass[a-g][**] where a-g depends on grass type and ** is the grass layer.
The grass is always 15 layers.

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
          

## Object
* 4 bytes - index
* 4 bytes - checksum
* float - anim speed X
* float - anim speed Y
* float - unknown
* 4 bytes - unknown
* 4 bytes - DDM Flags, see above
* 64 bytes - object name
* 32 bytes - unknown, maybe some bounding box?
* 4 bytes - numeber of vertices
* 4 bytes - number of indices
* 4 bytes - number of materials
* Array of materials - check Material below
* Array of vertices - check DDM Vertex below
* Array of indices - each index is 2 bytes
* Array of Material Splits - check Material Split below

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
2 bytes - material index
2 bytes - offset, relative to array of indices
2 bytes - number of indices







