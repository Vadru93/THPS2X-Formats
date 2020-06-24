# THPS2X-Formats
**Credit to DCxDemo for figuring out most of this**
## The THPS2X Levels have 4 files
* [.DDM](#ddm) - containing material information
* .DDX - containing textures
* [.psx](#psx) - containing mesh data
* [.trg](#trg) - containing script data


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
* 64 bytes - texture name there are 3 different logo textures that cannot be found in the .DDX file, it's `D_logo01`, `D_logo11` and `D_logo21`
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

## DDM Flags
* 0xE - fake 3d grass. It uses textures `grass[a-g][**]` where a-g depends on grass type and ** is the grass layer.
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
          







# .PSX
The `.PSX` contains collision data and since in THPS3 collision data and mesh data use shared vertices I use the [PSX Object](#psx-object) vertices when importing.
However for later games you can probably use [DDM Object](#ddm-object) for mesh and [PSX Object](#psx-object) for collision. If the [DDM Object](#ddm-object) points to the same index as [PSX Object](#psx-object) but has different checksum, you can rename the [PSX Object](#psx-object) to `"DDM Object Name_Collision"`
* [PSX Header](#psx-header)
* Array of [Object Position](#object-position)
* 4 bytes - Number of [PSX Object Header](#psx-object-header), [PSX Object](#psx-object)
* Array of [PSX Object Header](#psx-object-header)
* Array of [PSX Object](#psx-object)


## PSX Header
* 4 bytes - version
* 4 bytes - RGB offset
* 4 bytes - number of [Object Position](#object-position)

## Object Position
This is not looked into that much
* 4 bytes - unknown
* [Fixed Vertex](#fixed-vertex) - Position of object, should move both [DDM Object](#ddm-object) and [PSX Object](#psx-object)
* 12 bytes - unknown
* 4 bytes - index, this index links both to [DDM Object](#ddm-object) and [PSX Object](#psx-object)
* 10 bytes - unknown

## PSX Object Header
* 4 bytes - offset, relative to begining of file

## PSX Object
* 2 bytes - type
* 2 bytes - Number of [Fixed Vertex](#fixed-vertex)
* 2 bytes - number of Unknown each unknown is 8 bytes, maybe something for collision or sound
* 2 bytes - number of [Quad](#quad)
* 4 bytes - [PSX Flags](#psx-flags)



## Fixed Vertex
* The divider for Fixed Vertex is `4096` for scale I divide by `2.833`
* 2 bytes - x 
* 2 bytes - y
* 2 bytes - z
* When you have an angle it's calculated like this:
```
      //full circle is 4096(divider for Fixed Vertex) since level is flipped need - on all axes but y needs to be rotated 180 degree after.
      angle.x = -((D3DX_PI / 2048.0f) * ((float)(int)((short)x - 2048.0f));
      angle.y = -((D3DX_PI / 2048.0f) * ((float)(int)((short)y - 2048.0f)) + D3DX_PI;//180 degree
      angle.z = -((D3DX_PI / 2048.0f) * ((float)(int)((short)z - 2048.0f));
```


## Quad
* 4 bytes - [Quad Flags](#quad-flags)
* 4 bytes a, b, c, d
* 4 bytes - color index for a, b, c, d relative to RGB offset in [PSX Header](#psx-header)
* 4 bytes - [Collision Flags](#collision-flags)
* 4 bytes - Material Index, linked to the [Material](#material) in ddm
* 2 * 8 bytes - uv for a, b, c, d
* 4 bytes - zero padding

## PSX Flags

## Quad Flags
* QUAD 16 - if not set use triangle a, b, c
* TRANSPARENT 64
## Collision Flags
* It's not that well documentated yet so I will just provide the code I use to convert the flags.
* To make collision work corectly you also need to change the physics Wall_Non_Skatable_Angle
```
faceInfo.flags = 0x0001;//Set Skatable
          faceInfo.flags |= 0x0200;

          if (GetBit(collFlags, 8))
          {

            if (!GetBit(collFlags, 7))
            {
              faceInfo.flags = 0x0005;
            }
            else
            {
              faceInfo.flags = 0x0003;
            }
            if (collFlags & 0x0400)
              faceInfo.flags |= 0x0400;
            faceInfo.flags |= 0x0100;
          }
          if (GetBit(collFlags, 5))
          {
            faceInfo.flags |= 0x0009;
            faceInfo.flags &= ~0x0002;
          }
          else if (GetBit(collFlags, 6) && !GetBit(collFlags, 8))
          {
            faceInfo.flags |= 0x0018;
            faceInfo.flags &= ~0x0001;
            faceInfo.flags &= ~0x0002;
            //faceInfo.flags = 0x0510;
          }
          else if (GetBit(collFlags, 1))
          {
            faceInfo.flags &= ~0x0001;
            faceInfo.flags |= 0x0050;
          }
          else if (GetBit(collFlags, 0))
          {
            faceInfo.flags = 0x0510;
          }
          if (GetBit(collFlags, 3))
          {
            faceInfo.flags |= 0x0040;
          }
          mesh->AddFace(b, a, c, &faceInfo);
```

## .trg
The trigger nodes has 21 different types, these are the original names from Demo version of THPS
```
#define BADDY 0x1
#define CRATE 0x2
#define POINT 0x3
#define AUTOEXEC 0x4
#define POWERUP 0x5
#define COMMANDPOINT 0x6
#define SEEDABLEBADDY 0x7
#define RESTART 0x8
#define BARREL 0x9
#define RAILDEF 0xA
#define RAILPOINT 0xB
#define TRICKOB 0xC
#define CAMPT 0xD
#define GOALOB 0xE
#define AUTOEXEC2 0xF
#define MYST 0x10
#define TERMINATOR 0xFF
#define LIGHT 0x1F4
#define OFFLIGHT 0x1F5
#define SCRIPTPOINT 0x3E8
#define CAMERAPATH 0x3E9
```
* [TRG Header](#trg-header)
* Array of [Trigger Node Header ](#trigger-node-header)
* Array of [Trigger Node](#trigger-node)

## TRG Header
* 4 bytes - version
* 4 bytes - unknown, maybe size?
* 4 bytes - number of [Trigger Node Header](#trigger-node-header), [Trigger Node](#trigger-node)


## Trigger Node Header
* 4 bytes - offset


## Trigger Node
* 2 bytes - type, see above
* 2 bytes - number of links
* Array of links each link is 2 bytes
* 2 bytes - padding, only added if needed. After reading links check `if(file_position % 4 != 0)`, that means you have padding
* Extra data, depending on the [Node Type](#node-types)


## Node Types

## BADDY

## CRATE

## POINT

## AUTOEXEC

## POWERUP

## COMMANDPOINT

## SEEDABLEBADDY

## RESTART

## BARREL

## RAILDEF

## RAILPOINT

## TRICKOB

## CAMPT

## GOALOB

## AUTOEXEC2

## MYST

## TERMINATOR

## LIGHT

## OFFLIGHT

## SCRIPTPOINT

## CAMERAPATH


## Gaps
The gap names are stored in the exe, the format for how they are stored are:
* 2 bytes - [type](#gap-type)
* 2 bytes - unknown
* 2 bytes - id
* 2 bytes - score
* 39 bytes - Gap name - nullterminated string
Here is a complete list of the gaps:
```
Th2xGap Th2xGaps[] = {
  { 0x0013, 0, 0x03F2, 1000, "ACID DROP" },
  { 0x0013, 0, 0x044C, 100, "70FT" },
  { 0x0013, 0, 0x044D, 200, "80FT" },
  { 0x0013, 0, 0x044E, 300, "90FT" },
  { 0x0013, 0, 0x044F, 50, "HOPPIN' PLATFORM" },
  { 0x0019, 0, 0x04B2, 100, "1 POTATO" },
  { 0x0019, 0, 0x04B3, 200, "2 POTATO" },
  { 0x0019, 0, 0x04B4, 400, "3 POTATO" },
  { 0x0019, 0, 0x04B5, 300, "PIPE IT DOWN" },
  { 0x0019, 0, 0x04B6, 200, "CRANE-PLATFORM GAP" },
  { 0x0019, 0, 0x04B7, 500, "SLIDIN' UP" },
  { 0x0013, 0, 0x0BB8, 250, "OVER THE GATE" },
  { 0x0013, 0, 0x0BB9, 500, "OVER THE CROSSBAR" },
  { 0x0013, 0, 0x0BBC, 1000, "FREAKIN' HUGE HIP" },
  { 0x0013, 0, 0x0BBD, 200, "DUMPSTER POP" },
  { 0x0013, 0, 0x0BBE, 250, "TABLE POP" },
  { 0x0013, 0, 0x0BBF, 250, "2 THE BOX" },
  { 0x0013, 0, 0x0BC0, 250, "OVER THE TABLE" },
  { 0x0013, 0, 0x0BC3, 250, "BOX 2 BOX ACTION" },
  { 0x0013, 0, 0x0BC5, 1000, "HUMPTEY HUMPS!!!" },
  { 0x0013, 0, 0x0BC6, 150, "SHORTY DUMPSTER POP" },
  { 0x0013, 0, 0x0BC7, 150, "SHORTY TABLE POP" },
  { 0x0013, 0, 0x0BCA, 100, "OVER THE LIL' 4" },
  { 0x0013, 0, 0x0BCB, 250, "UP THE LIL' 4" },
  { 0x0013, 0, 0x0BCC, 500, "BIG OL' STANKY GAP" },
  { 0x0013, 0, 0x0BCD, 250, "WATER UP LE BACKSIDE" },
  { 0x0013, 0, 0x0BCE, 1000, "BIG MOUTH GAP" },
  { 0x0013, 0, 0x0BCF, 250, "UP!" },
  { 0x0013, 0, 0x0BD0, 500, "UP!!" },
  { 0x0013, 0, 0x0BD1, 1000, "AND AWAY!!!" },
  { 0x0019, 0, 0x0C1C, 250, "RAIL 2 LEDGE" },
  { 0x0019, 0, 0x0C1D, 250, "LEDGE 2 RAIL" },
  { 0x0019, 0, 0x0C1E, 2000, "LAMP STOMP" },
  { 0x0019, 0, 0x0C20, 50, "RAIL 2 RAIL" },
  { 0x0019, 0, 0x0C21, 2000, "KNUCKLIN' FUTS!!!" },
  { 0x0019, 0, 0x0C23, 1000, "DUMPSTER STOMP" },
  { 0x0019, 0, 0x0C24, 1000, "KINK CLANK" },
  { 0x0019, 0, 0x0C26, 1000, "KINK STOMP" },
  { 0x0019, 0, 0x0C27, 500, "THE HIDDEN 4 KINK!" },
  { 0x0019, 0, 0x0C28, 1500, "CROSSBAR STOMP" },
  { 0x0005, 0, 0x0C80, 250, "BOOMIN' EXTENSION" },
  { 0x0005, 0, 0x0C81, 250, "STANKY EXTENSION" },
  { 0x0005, 0, 0x0C82, 250, "U.U.A. EXTENSION" },
  { 0x0033, 0, 0x0C4E, 500, "WALL CRAWLER" },
  { 0x0013, 0, 0x0CE4, 500, "LEAP OF FAITH!!!" },
  { 0x0013, 0, 0x0CE5, 750, "CARLSBAD GAP" },
  { 0x0013, 0, 0x0CE6, 500, "DROP OUT ROOF GAP!" },
  { 0x0013, 0, 0x0CE7, 750, "CRAZY ROOF GAP!!" },
  { 0x0013, 2, 0x0CE8, 250, "TC'S ROOF GAP" },
  { 0x0013, 0, 0x0CE9, 1000, "SUICIDAL ROOF GAP!!!" },
  { 0x0013, 0, 0x0CEA, 500, "AWNING HOP" },
  { 0x0013, 0, 0x0CEB, 250, "TABLE TRANSFER" },
  { 0x0013, 0, 0x0CEE, 750, "2 DA ROOF!!!" },
  { 0x0013, 0, 0x0CEF, 750, "HUGE TRANSFER!!!" },
  { 0x0013, 0, 0x0CF0, 1000, "MAD SKEELZ ROOF GAP!!!" },
  { 0x0013, 0, 0x0CF1, 500, "OVERHANG AIR" },
  { 0x0013, 0, 0x0CF2, 1000, "BALCONY 2 AWNING!!!" },
  { 0x0013, 0, 0x0CF3, 2500, "ARE YOU SERIOUS?!!" },
  { 0x0013, 0, 0x0CF4, 250, "OVER THE WALL..." },
  { 0x0013, 0, 0x0CF5, 500, "AND DOWN THE BANK!" },
  { 0x0013, 0, 0x0CF6, 500, "CARLSBAD 11 SET" },
  { 0x0013, 0, 0x0CF7, 500, "3 POINTS!!!" },
  { 0x0019, 1, 0x0D48, 250, "ROLL CALL! GONZ RAIL!" },
  { 0x0019, 0, 0x0D49, 500, "BIG RANCHO BENCH GAP" },
  { 0x0019, 0, 0x0D4A, 250, "GYM RAIL 2 RAIL" },
  { 0x0019, 0, 0x0D4B, 250, "OVERHANG STOMP!" },
  { 0x0019, 0, 0x0D4C, 250, "RACK 'EM UP" },
  { 0x0019, 0, 0x0D4D, 250, "POLE STOMP!" },
  { 0x0019, 0, 0x0D4E, 500, "POLE 2 BRIX!" },
  { 0x0019, 0, 0x0D4F, 500, "BANK 2 LEDGE" },
  { 0x0019, 0, 0x0D50, 750, "FLYIN' THE FLAG!" },
  { 0x0019, 1, 0x0D51, 500, "ROLL CALL! NIGHTMARE RAIL!" },
  { 0x0019, 0, 0x0D52, 500, "BENDY'S CURB" },
  { 0x0019, 0, 0x0D53, 750, "STAGE RAIL 2 RAIL" },
  { 0x0019, 1, 0x0D54, 250, "ROLL CALL! OPUNSEZMEE RAIL!" },
  { 0x0019, 0, 0x0D55, 750, "KICKER 2 HOOK" },
  { 0x0019, 0, 0x0D56, 1000, "BACKBOARD DANCE!" },
  { 0x0047, 0, 0x0DAC, 500, "2 WHEELIN' TC'S ROOF" },
  { 0x0047, 0, 0x0DAD, 500, "LEDGE ON EDGE" },
  { 0x0047, 0, 0x0DAE, 500, "BENDY'S FLAT" },
  { 0x0047, 0, 0x0DAF, 250, "PLANTER ON EDGE" },
  { 0x0005, 0, 0x0D17, 500, "ARCH EXTENSION" },
  { 0x0005, 0, 0x0D18, 1000, "LIL' GUPPY EXTENSION!" },
  { 0x0005, 0, 0x0D19, 2500, "MID SQUID EXTENSION!!" },
  { 0x0005, 0, 0x0D1A, 5000, "HIGH DIVE EXTENSION!!!" },
  { 0x0005, 0, 0x0D1B, 500, "STARTING BLOCKS EXTENSION!!!" },
  { 0x0033, 0, 0x0D7A, 500, "ROCK THE BELLS!" },
  { 0x0013, 0, 0x0E10, 500, "MUSKA'S GAP" },
  { 0x0013, 0, 0x0E11, 200, "TABLE POP" },
  { 0x0013, 0, 0x0E12, 500, "TIGHT LANDING TRANSFER" },
  { 0x0013, 1, 0x0E13, 1000, "VB! HUGE TRANSFER!!!" },
  { 0x0013, 0, 0x0E14, 250, "CAKE TRANSFER" },
  { 0x0013, 0, 0x0E15, 250, "WEST SIDE TRANSFER" },
  { 0x0013, 0, 0x0E18, 500, "BIG DOUBLE 5 SET" },
  { 0x0013, 1, 0x0E19, 500, "VB! PIT TRANSFER" },
  { 0x0013, 0, 0x0E1A, 1000, "MASSIVE 20 SET!" },
  { 0x0013, 0, 0x0E1B, 250, "WEE LIL' ROOF GAP" },
  { 0x0013, 0, 0x0E1C, 500, "NICE MID SIZE ROOF GAP" },
  { 0x0013, 0, 0x0E1D, 1000, "SIIIIICK ROOF GAP!!!" },
  { 0x0013, 0, 0x0E1E, 200, "SHORTY PLANTER POP" },
  { 0x0013, 0, 0x0E1F, 500, "PLANTER POP" },
  { 0x0013, 0, 0x0E20, 500, "ROOF 2 RAMP" },
  { 0x0013, 0, 0x0E21, 750, "RAMP 2 ROOF" },
  { 0x0013, 0, 0x0E22, 1000, "HUGE ROOF 2 RAMP" },
  { 0x0013, 0, 0x0E23, 1500, "HUGE RAMP 2 ROOF" },
  { 0x0013, 1, 0x0E24, 100, "VB SKINNY TRANSFER" },
  { 0x0013, 0, 0x0E25, 1000, "FATTY TRANSFER" },
  { 0x0013, 0, 0x0E26, 100, "UP!" },
  { 0x0013, 0, 0x0E27, 250, "UP!!" },
  { 0x0013, 0, 0x0E28, 500, "AND AWAY!!!" },
  { 0x0013, 1, 0x0E29, 500, "VB! LEDGE TRANSFER" },
  { 0x0013, 0, 0x0E2A, 250, "CANYON JUMP" },
  { 0x0013, 0, 0x0E2B, 750, "UPHILL CANYON JUMP" },
  { 0x0013, 0, 0x0E2C, 500, "LIL' VENT GAP" },
  { 0x0013, 0, 0x0E2D, 1000, "BIG VENT GAP" },
  { 0x0013, 0, 0x0E2E, 750, "VENT 2 ROOF GAP" },
  { 0x0013, 0, 0x0E2F, 250, "LEDGE 9 SET" },
  { 0x0019, 0, 0x0E74, 500, "BENCH TRIPPIN'" },
  { 0x0019, 0, 0x0E75, 500, "LEDGE 2 LEDGE" },
  { 0x0019, 2, 0x0E76, 250, "THE VENICE LEDGE" },
  { 0x0019, 0, 0x0E77, 200, "SEASIDE HANDRAIL" },
  { 0x0019, 0, 0x0E78, 500, "10 POINT LANDING!" },
  { 0x0019, 0, 0x0E79, 2500, "'ROUND THE HORN!!!" },
  { 0x0019, 0, 0x0E7A, 1000, "THE HIGH WIRE" },
  { 0x0027, 0, 0x0ED8, 250, "HE COULD GO..." },
  { 0x0027, 0, 0x0ED9, 500, "ALL THE WAY..." },
  { 0x0027, 0, 0x0EDA, 1000, "TOUCHDOWN!!!" },
  { 0x0027, 0, 0x0EDC, 750, "CANDY CANE MANUAL" },
  { 0x0013, 0, 0x0FA0, 250, "DROPPING IN ON TONY" },
  { 0x0013, 0, 0x0FA1, 1000, "UP 2 COMBI" },
  { 0x0013, 0, 0x0FA2, 50, "DOWN 2 TONYS ISLAND" },
  { 0x0013, 0, 0x0FA3, 800, "GUTTER 2 SAN DIEGUITO ROOF" },
  { 0x0013, 0, 0x0FA4, 1500, "ZIG GAP" },
  { 0x0013, 0, 0x0FA5, 2000, "ZAG GAP" },
  { 0x0013, 0, 0x0FA6, 800, "WUSSY SNAKE GAP" },
  { 0x0013, 0, 0x0FA7, 2000, "REVERSE ZIG GAP" },
  { 0x0013, 0, 0x0FA8, 2000, "REVERSE ZAG GAP" },
  { 0x0013, 0, 0x0FA9, 1000, "REVERSE WUSSY SNAKE GAP" },
  { 0x0013, 0, 0x0FAA, 300, "ISLE OF TONY 2 SADLANDS" },
  { 0x0013, 0, 0x0FAB, 800, "SADLANDS PATH GAP" },
  { 0x0013, 0, 0x0FAC, 500, "HOUSE OF TONY 2 SADLANDS" },
  { 0x0013, 0, 0x0FAD, 50, "SAN DIEGUITO HALL 2 SADLANDS" },
  { 0x0013, 0, 0x0FAE, 500, "SAN DIEGUITO WINDOW 2 SADLANDS" },
  { 0x0013, 0, 0x0FAF, 1500, "TIGHT LANDING" },
  { 0x0013, 0, 0x0FB0, 100, "SAN DIEGUITO TEN SET" },
  { 0x0013, 0, 0x0FB1, 400, "BIG FAT GRASSY GAP" },
  { 0x0013, 0, 0x0FB2, 50, "GRASSY GAP" },
  { 0x0013, 0, 0x0FB3, 2000, "FEED ME!!!" },
  { 0x0013, 0, 0x0FB4, 800, "NORTHWEST SNAKE GAP" },
  { 0x0013, 0, 0x0FB5, 800, "NORTHEAST SNAKE GAP" },
  { 0x0013, 0, 0x0FB6, 1000, "SOUTHERN SNAKE GAP" },
  { 0x0013, 0, 0x0FB7, 50, "WEAK SAUCE ZIG GAP" },
  { 0x0013, 0, 0x0FB8, 50, "WEAK SAUCE ZAG GAP" },
  { 0x0013, 0, 0x0FB9, 50, "WEAK SAUCE WUSSY SNAKE GAP" },
  { 0x0013, 0, 0x0FBA, 300, "PLATFORM GAP" },
  { 0x0013, 0, 0x0FBB, 300, "AIRS HOLE" },
  { 0x0013, 0, 0x0FBC, 1000, "SADLANDS UP 2 ISLE OF TONY" },
  { 0x0013, 0, 0x0FBD, 1000, "OVER THE DOME" },
  { 0x0013, 0, 0x0FBE, 1000, "CLEARING THE SWINGS" },
  { 0x0013, 0, 0x0FBF, 1000, "JUMPIN DA HUB" },
  { 0x0002, 0, 0x0FC0, 1000, "TUNNEL OF LUVIN" },
  { 0x0013, 0, 0x0FC3, 500, "BLOWIN IT OUT THE HOLE!" },
  { 0x0013, 0, 0x0FC4, 4000, "PIT O DOOM!!!" },
  { 0x0013, 0, 0x0FC7, 500, "SADLANDS 2 SAN DIEGUITO HALL" },
  { 0x0013, 0, 0x0FC8, 500, "THE HOLY CRAIL" },
  { 0x0019, 0, 0x5DC0, 2000, "RADRAMP 2 SNAKERUN" },
  { 0x0019, 0, 0x5DC1, 400, "RADRAMP 2 ISLANDS EDGE" },
  { 0x0019, 0, 0x5DC2, 600, "GUTTER 2 SAN DIEGUITO ROOF" },
  { 0x0019, 0, 0x5DC3, 1000, "UP 2 PIPE RAIL" },
  { 0x0019, 0, 0x5DC4, 2000, "LONGRAIL" },
  { 0x0019, 0, 0x5DC5, 50, "RAIL 2 SNAKERUN" },
  { 0x0019, 0, 0x5DC6, 300, "RAMP 2 RAIL" },
  { 0x0019, 0, 0x5DC7, 300, "90 DEGREE SADLANDS RAIL GAP" },
  { 0x0019, 0, 0x5DC8, 75, "RAMP RAIL GAP" },
  { 0x0019, 0, 0x5DC9, 500, "90 DEGREE RAMP RAIL GAP" },
  { 0x0019, 0, 0x5DCA, 500, "FENCE 2 RADRAMP" },
  { 0x0019, 0, 0x5DCB, 2000, "OFF THE ROOF 2 RAIL" },
  { 0x0019, 0, 0x5DCC, 4000, "TIME 2 FEED THE VOLCANO!!!" },
  { 0x0019, 0, 0x5DCD, 600, "ISLE OF TONY 2 EDGE" },
  { 0x0019, 0, 0x5DCF, 500, "SAN DIEGUITO HALL 2 EDGE" },
  { 0x0019, 0, 0x5DD0, 1600, "SAN DIEGUITO ROOF 2 EDGE" },
  { 0x0019, 0, 0x5DD1, 2000, "BENCH GAP" },
  { 0x0019, 0, 0x5DD6, 450, "RIMRAIL GAP" },
  { 0x0019, 0, 0x5DD7, 2000, "SWINGING THE SET" },
  { 0x0019, 0, 0x5DD8, 5000, "BENCH GAP SERIES" },
  { 0x0019, 0, 0x5DD9, 900, "SOUTHERN SWINGRAIL" },
  { 0x0019, 0, 0x5DDA, 900, "NORTHERN SWINGRAIL" },
  { 0x0019, 0, 0x5DDB, 350, "MID INTERSECT SAD GAP" },
  { 0x0019, 0, 0x5DDC, 200, "SOUTHERN INTERSECT SAD GAP" },
  { 0x0019, 0, 0x5DDD, 200, "NORTHERN INTERSECT SAD GAP" },
  { 0x0019, 0, 0x5DDE, 300, "NORTHERN CROSSOVER SAD GAP" },
  { 0x0019, 0, 0x5DDF, 300, "SOUTHERN CROSSOVER SAD GAP" },
  { 0x0019, 0, 0x5DE1, 700, "TOP OF DA WORLD MA!!!" },
  { 0x0019, 0, 0x5DE2, 4000, "KICKER 2 RAIL" },
  { 0x0019, 0, 0x5DE3, 1500, "KICKER 2 RAILSPAN" },
  { 0x0019, 0, 0x5DE4, 1500, "RAIL 2 KICKER 2 RAIL 2 BENCH" },
  { 0x0019, 0, 0x5DE5, 300, "CHEN RAIL SERIES" },
  { 0x0013, 0, 0x0FC1, 2500, "WOOHOOO OH HO YEEHEE!!!" },
  { 0x0022, 0, 0x0FC2, 5000, "CLEANING THE PIPES" },
  { 0x0013, 0, 0x1194, 200, "LAUNCHIN ON UP" },
  { 0x0013, 0, 0x1195, 300, "LAUNCHIN THE PIPE" },
  { 0x0013, 0, 0x1196, 1000, "TIGHT GAP" },
  { 0x0013, 0, 0x1197, 100, "PLAT GAP" },
  { 0x0013, 0, 0x1198, 50, "WUSSY ROLLIN GAP" },
  { 0x0013, 0, 0x1199, 300, "ROLLIN GAP" },
  { 0x0013, 0, 0x119A, 600, "BIG ENCHILADA MAMA" },
  { 0x0013, 0, 0x119B, 1500, "JUMPIN DA HUMPS" },
  { 0x0013, 0, 0x119D, 400, "AIR TORO" },
  { 0x0013, 0, 0x119E, 150, "GATE GAP" },
  { 0x0019, 0, 0x5FB4, 500, "DONT LOOK DOWN!" },
  { 0x0019, 0, 0x5FB5, 250, "ENJOYIN THE VIEW" },
  { 0x0019, 0, 0x5FB6, 450, "GRINDIN THE PIPE" },
  { 0x0019, 0, 0x5FB7, 1000, "BOX TO BANANA" },
  { 0x0019, 0, 0x5FB9, 400, "KINK" },
  { 0x0019, 0, 0x5FBA, 750, "RAIL PLAT GAP" },
  { 0x0019, 0, 0x5FBB, 1, "LIL WEE WUSSY GAP" },
  { 0x0019, 0, 0x5FBC, 500, "RAMP RAIL TO BANANA" },
  { 0x0019, 0, 0x5FBD, 1000, "LAUNCH TO BANANA" },
  { 0x0019, 0, 0x5FBE, 2000, "LAUNCH TO RAIL" },
  { 0x0019, 0, 0x5FBF, 500, "BOX TO RAIL" },
  { 0x0019, 0, 0x5FC0, 500, "NICE FRIGGIN ANKLES" },
  { 0x0019, 0, 0x5FC1, 500, "NAILIN DA RAIL" },
  { 0x0019, 0, 0x5FC2, 650, "TAKIN THE HIGH ROAD" },
  { 0x0019, 0, 0x5FC3, 500, "WAY TO GO AMIGO" },
  { 0x0019, 0, 0x5FC4, 500, "RAMP RAIL TO RAIL" },
  { 0x0019, 0, 0x5FC5, 1500, "CLENCHFEST!" },
  { 0x0019, 0, 0x5FC6, 1500, "FINESSE TEST" },
  { 0x0011, 0, 0x38A4, 2000, "THREADIN THE NEEDLE" },
  { 0x0011, 0, 0x38A5, 50, "UP TO THE STANDS" },
  { 0x0022, 0, 0x11A0, 5000, "WAY TO GO GRINGO!!!" },
  { 0x0013, 0, 0x0834, 100, "HP TO BOWL" },
  { 0x0013, 0, 0x0835, 100, "BOWL TO HP" },
  { 0x0013, 0, 0x0836, 100, "BULLET BOWL HOP" },
  { 0x0013, 0, 0x0837, 100, "OVER THE DECK" },
  { 0x0013, 0, 0x0838, 100, "DAAAAAY TRIPPER" },
  { 0x0013, 0, 0x0839, 100, "GIMME GAP REDUX" },
  { 0x0013, 0, 0x083A, 100, "SODEE POP GAP" },
  { 0x0013, 0, 0x083C, 100, "CUT THE CORNER" },
  { 0x0013, 0, 0x083D, 100, "HIGH STICKER" },
  { 0x0013, 0, 0x0840, 50, "RAILING HOP" },
  { 0x0013, 0, 0x0841, 50, "OVER THE BRIDGE" },
  { 0x0013, 0, 0x0842, 10, "OVER THE WALL" },
  { 0x0013, 0, 0x0843, 100, "SHOOT THE GAP" },
  { 0x0013, 0, 0x0845, 150, "NO KIDDING AROUND" },
  { 0x0013, 0, 0x0846, 150, "STAIRSET" },
  { 0x0013, 0, 0x0847, 150, "HEXBOX GAP" },
  { 0x0013, 0, 0x084F, 250, "HIGH JUMPER" },
  { 0x0019, 0, 0x083B, 100, "VAN SECRET AREA KEY" },
  { 0x0019, 0, 0x083F, 50, "RAIL SECRET AREA KEY" },
  { 0x0019, 0, 0x5654, 100, "NAIL THE RAIL" },
  { 0x0019, 0, 0x5655, 100, "HP TO RAILBOX" },
  { 0x0019, 0, 0x5656, 100, "WAVE WALL MINIGAP" },
  { 0x0019, 0, 0x5657, 100, "SURFIN U.S.A." },
  { 0x0019, 0, 0x5658, 100, "SKATIN ON THE DOCK OF THE BAY" },
  { 0x0019, 0, 0x5659, 500, "CIRCLE THE POOL" },
  { 0x0019, 0, 0x565A, 100, "HAVIN A PICNIC" },
  { 0x0019, 0, 0x565B, 100, "EXTENSION TRANSFER" },
  { 0x0019, 0, 0x565C, 100, "BIG AIR RAILING GRIND" },
  { 0x0019, 0, 0x565D, 50, "RAIL TO RAIL" },
  { 0x0027, 0, 0x084B, 100, "FUNBOX WHEELIE" },
  { 0x0005, 0, 0x0848, 100, "BOWL LIP" },
  { 0x0005, 0, 0x0849, 100, "HP LIP" },
  { 0x0005, 0, 0x084A, 100, "RIDE THE WAVE" },
  { 0x0005, 0, 0x084C, 100, "GULLY LIP" },
  { 0x0005, 0, 0x084D, 100, "BOWL ENVY" },
  { 0x0005, 0, 0x084E, 100, "MR. SMALL LIPS" },
  { 0x0013, 0, 0x5781, 100, "PIGEON PUDDIN' GAP" },
  { 0x0013, 0, 0x578D, 100, "RAMP TO PARK GAP" },
  { 0x0013, 0, 0x578E, 250, "RAMP TO STATUE SHORTY GAP" },
  { 0x0013, 0, 0x578F, 250, "POUNCER WAS HERE" },
  { 0x0013, 0, 0x5792, 100, "AWNING AIR" },
  { 0x0013, 0, 0x5793, 50, "KICK IT" },
  { 0x0013, 0, 0x5799, 150, "TAKE IT TO THE BRIDGE" },
  { 0x0013, 0, 0x579B, 250, "OVER THE ROAD" },
  { 0x0013, 0, 0x579D, 250, "BIG AIR OUT OF THE BANKS" },
  { 0x0017, 0, 0x579F, 100, "OVER THE BANKS BARRIER" },
  { 0x0013, 0, 0x57A5, 500, "PILLAR AIR" },
  { 0x0013, 0, 0x57A7, 50, "ROCK IT AIR" },
  { 0x0019, 0, 0x5780, 100, "BENCH-HOPPIN" },
  { 0x0019, 0, 0x5782, 100, "LEFT SIDE PIT RAIL STOMP" },
  { 0x0019, 0, 0x5783, 100, "BANKS SPANK" },
  { 0x0019, 0, 0x5784, 100, "PARKING METER GAP" },
  { 0x0019, 0, 0x5785, 100, "YOU'RE NEXT IN LINE" },
  { 0x0011, 0, 0x5786, 100, "THE EASY WAY" },
  { 0x0011, 0, 0x5787, 500, "THE HARD WAY" },
  { 0x0019, 2, 0x5788, 100, "JOEY'S SCULPTURE" },
  { 0x0019, 0, 0x5789, 100, "RIGHT SIDE PIT RAIL STOMP" },
  { 0x0019, 0, 0x578A, 100, "JAMIE'S STEPS" },
  { 0x0019, 0, 0x578B, 100, "BANKS FENCE GAP" },
  { 0x0019, 0, 0x578C, 100, "BANKS ROAD GAP" },
  { 0x0019, 0, 0x5790, 100, "REBAR TO RAIL GAP" },
  { 0x0011, 1, 0x5795, 100, "RIDE THE RAILS" },
  { 0x0011, 0, 0x5791, 100, "ACROSS THE PIT" },
  { 0x0011, 0, 0x5794, 100, "CORNER CUT" },
  { 0x0019, 0, 0x5796, 500, "PATH LESS TRAVELED" },
  { 0x0019, 0, 0x5797, 100, "PARK ENTRANCE GAP" },
  { 0x0019, 0, 0x579C, 1000, "SIDEWALK BOMB" },
  { 0x0019, 0, 0x57A0, 1000, "CHANGIN TRAINS" },
  { 0x0019, 0, 0x57A2, 100, "GRAB A SNACK AND SIT DOWN." },
  { 0x0019, 0, 0x57A3, 100, "BUUURP!  NOW GO SKATE." },
  { 0x0019, 0, 0x57A4, 50, "RE-REBAR" },
  { 0x0019, 0, 0x57A8, 500, "SLAM DUNK" },
  { 0x0027, 0, 0x57A1, 250, "THE BRIDGE" },
  { 0x0027, 0, 0x57A6, 250, "GOING DOWN?" },
  { 0x0005, 0, 0x5798, 100, "PHAT LIP" },
  { 0x0005, 0, 0x579A, 100, "WAAAAY UP THERE" },
  { 0x0033, 0, 0x579E, 100, "BANKS BARRIER WALLRIDE" },
  { 0x0013, 0, 0x09C4, 500, "TH2X FOUNTAIN GAP" },
  { 0x0013, 0, 0x09C5, 500, "CHILLIN' ON THE BALCONY" },
  { 0x0013, 0, 0x09C6, 100, "STAIR SET" },
  { 0x0013, 0, 0x09C7, 100, "UP THE SMALL STEP SET" },
  { 0x0013, 0, 0x09C8, 100, "BENCH GAP" },
  { 0x0013, 0, 0x09C9, 250, "PHILLYSIDE HP TRANSFER" },
  { 0x0013, 0, 0x09CE, 100, "WORLDS MOST OBVIOUS GAP" },
  { 0x0013, 0, 0x09CF, 100, "PHILLYSIDE HOP" },
  { 0x0013, 0, 0x09D0, 50, "POST OLLIE" },
  { 0x0013, 0, 0x09D1, 10, "EASY POST OLLIE" },
  { 0x0013, 0, 0x09D2, 50, "STATUE HOP" },
  { 0x0013, 0, 0x09D3, 250, "PILLAR FIGHT" },
  { 0x0019, 0, 0x57E6, 250, "TELEPHONE CO. GAP" },
  { 0x0019, 0, 0x57E7, 750, "WORLDS SECOND MOST OBVIOUS GAP" },
  { 0x0019, 0, 0x57E8, 1500, "GRIND OF FAITH" },
  { 0x0011, 0, 0x30D4, 500, "GRIND UP DEM STAIRS" },
  { 0x0019, 2, 0x30D6, 500, "AWNING GRIND" },
  { 0x0019, 0, 0x30D7, 500, "LITTLE CORNER GRIND" },
  { 0x0019, 0, 0x30D8, 500, "FLY BY WIRE" },
  { 0x0019, 0, 0x30DA, 500, "DEATH FROM ABOVE" },
  { 0x0019, 0, 0x30DB, 50, "TRACK SMACK" },
  { 0x0019, 0, 0x30DC, 100, "HOBO GRIND" },
  { 0x0019, 0, 0x30DE, 100, "PLANTER TRANSFER" },
  { 0x0019, 0, 0x30DF, 100, "RAILING TO PLANTER" },
  { 0x0019, 0, 0x30E2, 500, "TRAIN HARD" },
  { 0x0019, 0, 0x30E3, 150, "PILLAR HOP" },
  { 0x0019, 0, 0x30E4, 250, "FUNBOX TRANSFER" },
  { 0x0019, 0, 0x30E5, 150, "PLANTER DOUBLE PILLAR GAP" },
  { 0x0019, 0, 0x30E6, 150, "JUST VISITING" },
  { 0x0019, 0, 0x30E7, 750, "FOUNTAIN PING!" },
  { 0x0019, 0, 0x30E8, 150, "SHORT STAIR" },
  { 0x0019, 0, 0x30E9, 250, "MEDIUM STAIR" },
  { 0x0019, 0, 0x30EA, 500, "LONG STAIR" },
  { 0x0027, 0, 0x30D9, 100, "FUNBOX WHEELIE" },
  { 0x0001, 0, 0x30DD, 500, "FLATLANDS TECHIN'" },
  { 0x0001, 0, 0x30E0, 2500, "ROCKIN' THE STAIRS" },
  { 0x0027, 0, 0x30E1, 500, "MANUAL STIMULATION" },
  { 0x0005, 1, 0x57F0, 100, "PHILLYSIDE NEW BOWL LIP" },
  { 0x0005, 1, 0x57F1, 100, "PHILLYSIDE HP LIP" },
  { 0x0005, 1, 0x57F2, 100, "PHILLYSIDE BIG BOWL LIP" },
  { 0x0005, 1, 0x57F3, 100, "PHILLYSIDE MID BOWL LIP" },
  { 0x0017, 1, 0x0A8D, 500, "HALFPIPE HANGTIME" },
  { 0x0019, 2, 0x0A8E, 250, "HALFPIPE GRIND" },
  { 0x0011, 0, 0x0A8F, 100, "ROLLIN GAP" },
  { 0x0013, 0, 0x0A92, 100, "CHOPPER HOP" },
  { 0x0013, 1, 0x0A95, 500, "WINGTIP HANGTIME" },
  { 0x0013, 1, 0x0A96, 500, "SKYCRANE HANGTIME" },
  { 0x0013, 0, 0x0A98, 250, "FLYIN HIGH" },
  { 0x0013, 0, 0x0A99, 500, "AIR OVER THE DOOR" },
  { 0x0013, 0, 0x0AA1, 250, "ITS COLD UP HERE" },
  { 0x0019, 0, 0x0A90, 250, "LIL LIGHT HOPPER" },
  { 0x0019, 0, 0x0A91, 500, "BIG LIGHT HOPPER" },
  { 0x0019, 0, 0x0A93, 100, "RAIL-GUIDED MISSILE" },
  { 0x0019, 0, 0x0A94, 100, "RAILDROP" },
  { 0x0019, 0, 0x0A9A, 500, "LIGHT CORNER" },
  { 0x0047, 0, 0x0A8C, 500, "INSTRUMENT LANDING" },
  { 0x0005, 0, 0x0A97, 100, "HIGH STEPPIN'" },
  { 0x0005, 0, 0x0A9C, 100, "ONE HALF PIPE LIP" },
  { 0x0005, 0, 0x0A9D, 100, "THE OTHER HALF PIPE LIP" },
  { 0x0005, 0, 0x0A9E, 100, "WIND TUNNEL BACK WALL" },
  { 0x0005, 0, 0x0A9F, 100, "UPWIND LIP" },
  { 0x0005, 0, 0x0AA0, 100, "DOWNWIND LIP" },
  { 0x0011, 0, 0x175C, 250, "WIMPY GAP" },
  { 0x0011, 0, 0x175D, 500, "GAP" },
  { 0x0011, 0, 0x175E, 1000, "PHAT GAP" },
  { 0x0011, 0, 0x1767, 200, "TRANSFER" },
  { 0x0011, 0, 0x1F40, 600, "TAXI GAP" },
  { 0x0011, 0, 0x1F41, 100, "KICKER GAP" },
  { 0x0011, 0, 0x1F42, 300, "OVER THE PIPE" },
  { 0x0011, 0, 0x1F43, 300, "SECRET ROOM" },
  { 0x0011, 0, 0x1F44, 400, "FACEPLANT" },
  { 0x0011, 0, 0x1770, 1000, "ACID DROP" },
  { 0x0011, 0, 0x20D0, 100, "KICKER 2 STREET" },
  { 0x0011, 0, 0x20D1, 500, "BS GAP" },
  { 0x0011, 0, 0x20D2, 500, "T 2 T GAP" },
  { 0x0011, 0, 0x20D3, 500, "SECRET TUNNEL ENTRANCE" },
  { 0x0011, 0, 0x20D4, 1000, "TUNNEL GAP" },
  { 0x0011, 0, 0x20D5, 2000, "OVER THE TUNNEL" },
  { 0x0011, 0, 0x20D6, 100, "CAR OLLIE" },
  { 0x0013, 0, 0x20D7, 50, "CHEESY DECK GAP" },
  { 0x0011, 0, 0x20D8, 250, "DECK GAP" },
  { 0x0011, 0, 0x20D9, 2500, "BURLY DECK GAP" },
  { 0x0011, 0, 0x20DA, 250, "TRUCK GAP" },
  { 0x0011, 0, 0x20DB, 2000, "ROOF 2 ROOF" },
  { 0x0011, 0, 0x20DC, 1500, "SUCKY ROOM GAP" },
  { 0x0011, 0, 0x20DD, 1500, "BIG ASS" },
  { 0x0011, 0, 0x20DE, 750, "GLASS GAP" },
  { 0x0011, 0, 0x206C, 1000, "WHOOP GAP" },
  { 0x0011, 0, 0x206D, 100, "WALL GAP" },
  { 0x0013, 0, 0x206E, 100, "OVER THE BOX" },
  { 0x0011, 0, 0x206F, 2000, "OVER THE RAFTERS" },
  { 0x0011, 0, 0x2070, 700, "OVER THE PIPE" },
  { 0x0013, 0, 0x2071, 500, "POOL HIP" },
  { 0x0011, 0, 0x2072, 700, "POOL 2 WALKWAY" },
  { 0x0011, 0, 0x2073, 250, "HP TRANSFER" },
  { 0x0011, 0, 0x219C, 1000, "BRIDGE GAP" },
  { 0x0011, 0, 0x2199, 700, "VERT WALL GAP" },
  { 0x0011, 0, 0x219A, 700, "TWINKIE TRANSFER" },
  { 0x0017, 0, 0x219B, 800, "OVER DA POOL" },
  { 0x0011, 0, 0x2008, 500, "ROOF 2 ROOF GAP" },
  { 0x0011, 0, 0x1FA5, 1000, "SWIM TEAM GAP" },
  { 0x0011, 0, 0x1FA7, 50, "GARBAGE OLLIE" },
  { 0x0011, 0, 0x1FA8, 750, "ROOF TO AWNING GAP" },
  { 0x0011, 0, 0x1FA9, 250, "DITCH SLAP" },
  { 0x0011, 0, 0x1FAA, 750, "OVER THE AIR CONDITIONER" },
  { 0x0011, 0, 0x1FAB, 1000, "OVER A FOOTBRIDGE" },
  { 0x0011, 0, 0x1FAC, 500, "PARK GAP" },
  { 0x0011, 0, 0x1FAD, 250, "MINI GAP" },
  { 0x0011, 0, 0x1FAE, 100, "PLANTER GAP" },
  { 0x0011, 0, 0x1FAF, 100, "KICKER GAP" },
  { 0x0013, 0, 0x2009, 250, "OVER A 16 STAIR SET" },
  { 0x0013, 0, 0x200A, 2000, "OVER A HUGE 32 STAIR GAP" },
  { 0x0013, 0, 0x200C, 500, "SKATER ESCALATOR GAP" },
  { 0x0013, 0, 0x200D, 2500, "32 STEPS OFF A MEZZANINE" },
  { 0x0011, 0, 0x200E, 100, "THE FLYING LEAP" },
  { 0x0011, 0, 0x200F, 100, "PLANTER GAP" },
  { 0x0011, 0, 0x2011, 250, "GOING UP GAP" },
  { 0x0011, 0, 0x2012, 250, "GOING DOWN GAP" },
  { 0x0011, 0, 0x2013, 250, "FOUNTAIN GAP" },
  { 0x0011, 0, 0x1FB0, 250, "RAGING GORGE GAP" },
  { 0x0011, 0, 0x2134, 1000, "HUGE WATER HAZARD GAP" },
  { 0x0011, 0, 0x2135, 50, "50 FEET" },
  { 0x0011, 0, 0x2136, 100, "100 FEET" },
  { 0x0011, 0, 0x2137, 150, "150 FEET" },
  { 0x0011, 0, 0x2138, 200, "200 FEET" },
  { 0x0011, 0, 0x2139, 250, "250 FEET" },
  { 0x0011, 0, 0x213A, 250, "SMALL WATER HAZARD GAP" },
  { 0x0011, 0, 0x213C, 25, "25 FEET" },
  { 0x0011, 0, 0x213D, 75, "75 FEET" },
  { 0x0011, 0, 0x213E, 125, "125 FEET" },
  { 0x0011, 0, 0x213F, 175, "175 FEET" },
  { 0x0011, 0, 0x2140, 225, "225 FEET" },
  { 0x0013, 0, 0x2262, 500, "LOW DECK GAP" },
  { 0x0013, 0, 0x2263, 1000, "HIGH DECK GAP" },
  { 0x0013, 0, 0x2264, 1500, "DECK GAP" },
  { 0x0011, 0, 0x21FC, 250, "PORCH GAP" },
  { 0x0011, 0, 0x21FD, 500, "PORCH GAP" },
  { 0x0013, 0, 0x21FE, 5000, "LOMBARD GAP" },
  { 0x0011, 0, 0x21FF, 500, "THE GONZ GAP" },
  { 0x0011, 0, 0x2200, 1000, "PAGODA GAP" },
  { 0x0011, 0, 0x2201, 100, "OVER THE SEVEN" },
  { 0x0011, 0, 0x2202, 750, "HUBBA GAP" },
  { 0x0011, 0, 0x2203, 250, "STREET GAP" },
  { 0x0011, 0, 0x2204, 500, "STREET GAP" },
  { 0x0011, 0, 0x2205, 1000, "HANDI GAP" },
  { 0x0011, 0, 0x2206, 500, "C BLOCK GAP" },
  { 0x0013, 0, 0x2207, 500, "PLANTER GAP" },
  { 0x0011, 0, 0x2208, 750, "FOUNTAIN GAP" },
  { 0x0013, 0, 0x2209, 1000, "SPINE GAP" },
  { 0x0013, 0, 0x220A, 500, "RAMP 2 RAMP" },
  { 0x0013, 0, 0x220B, 750, "RAMP 2 RAMP" },
  { 0x0013, 0, 0x220C, 500, "OVERSIZED 8 SET" },
  { 0x0011, 0, 0x220D, 1000, "ACID DROP-IN" },
  { 0x0011, 0, 0x220E, 250, "FOUNTAIN GAP" },
  { 0x0011, 0, 0x220F, 750, "PORCH GAP" },
  { 0x0011, 0, 0x2210, 500, "KICKER GAP" },
  { 0x0011, 0, 0x2211, 500, "PAGODA HOP" },
  { 0x0011, 0, 0x2212, 750, "PAGODA HOP" },
  { 0x0013, 0, 0x1F46, 250, "CHANNEL GAP" },
  { 0x0013, 0, 0x1F47, 200, "KICKER 2 LEDGE" },
  { 0x0013, 0, 0x2265, 1000, "ROLL IN CHANNEL GAP" },
  { 0x0013, 0, 0x2266, 500, "CHANNEL GAP" },
  { 0x0019, 0, 0x1FD6, 1000, "HALL PASS GAP" },
  { 0x0019, 0, 0x1F4A, 200, "BIG RAIL" },
  { 0x0019, 0, 0x1F4B, 300, "DECK 2 RAIL" },
  { 0x0019, 0, 0x1F4C, 500, "TAXI 2 LEDGE" },
  { 0x0019, 0, 0x1F4D, 1000, "TAXI 2 RAIL" },
  { 0x0019, 0, 0x1F4E, 200, "KICKER 2 LEDGE" },
  { 0x0019, 0, 0x1F4F, 500, "MONSTER GRIND" },
  { 0x0019, 0, 0x1F50, 200, "HIGH RAIL" },
  { 0x0019, 0, 0x1F51, 3000, "HOLY SHI..." },
  { 0x0019, 0, 0x1F52, 400, "TRANSITION GRIND" },
  { 0x0019, 0, 0x20E4, 100, "KICKER 2 EDGE" },
  { 0x0019, 0, 0x20E5, 200, "BS GRIND" },
  { 0x0019, 0, 0x20E6, 750, "RAIL 2 RAIL TRANSFER" },
  { 0x0019, 0, 0x20E7, 500, "BILLBOARD GRIND" },
  { 0x0019, 0, 0x20E8, 3000, "DIRTY RAIL" },
  { 0x0019, 0, 0x20E9, 2000, "DEATH GRIND" },
  { 0x0019, 0, 0x21A2, 1000, "TRIPLE RAIL" },
  { 0x0019, 0, 0x21A3, 800, "BRIDGE GRIND" },
  { 0x0019, 0, 0x21A4, 1000, "HAWK BRIDGE GRIND" },
  { 0x0019, 0, 0x2076, 1000, "RAFTER RAIL" },
  { 0x0019, 0, 0x2077, 1000, "PIPE 2 BOX GRIND" },
  { 0x0019, 0, 0x2078, 500, "LIGHT GRIND" },
  { 0x0019, 0, 0x2079, 700, "WALKWAY RAIL TRANS" },
  { 0x0019, 0, 0x207A, 1000, "POOL RAIL TRANS" },
  { 0x0019, 0, 0x226A, 1000, "ET GRIND" },
  { 0x0019, 0, 0x226B, 1000, "BHOUSE RAIL" },
  { 0x0019, 0, 0x226C, 2000, "POOL GRIND" },
  { 0x0019, 0, 0x226D, 800, "DECK GRIND" },
  { 0x0019, 0, 0x226E, 2000, "MB EMERSON GRIND" },
  { 0x0019, 0, 0x1FB8, 250, "DUMPSTER RAIL GAP" },
  { 0x0019, 0, 0x1FB9, 500, "PLAYGROUND RAIL" },
  { 0x0019, 0, 0x1FBA, 750, "RAIL TO RAIL TRANSFER" },
  { 0x0019, 0, 0x1FBB, 1000, "HUGE RAIL" },
  { 0x0019, 0, 0x1FBC, 2500, "LONG ASS RAIL" },
  { 0x0019, 0, 0x1FBD, 250, "FUNBOX TO RAIL TRANSFER" },
  { 0x0019, 0, 0x1FBE, 500, "FUNBOX TO TABLE TRANSFER" },
  { 0x0019, 0, 0x1FBF, 50, "GIMME GAP" },
  { 0x0019, 0, 0x1FC0, 500, "HANDICAP RAMP RAIL" },
  { 0x0019, 0, 0x201C, 1000, "COFFEE GRIND" },
  { 0x0019, 0, 0x201D, 500, "FOR THE WHOLE ATRIUM" },
  { 0x0019, 0, 0x201E, 500, "RAIL COMBO" },
  { 0x0019, 0, 0x2148, 1500, "NEVERSOFT ELEC CO GAP" },
  { 0x0019, 0, 0x221A, 2000, "DOWN THE SPIRAL" },
  { 0x0019, 0, 0x221B, 250, "LOMBARD LEDGE" },
  { 0x0019, 0, 0x221C, 500, "HUBBA LEDGE" },
  { 0x0019, 0, 0x221D, 750, "HOOK RAIL" },
  { 0x0019, 0, 0x221E, 500, "RAIL 2 RAIL" },
  { 0x0019, 0, 0x221F, 250, "BACKWOODS LEDGE" },
  { 0x0019, 0, 0x2220, 500, "BENDY'S LIP" },
  { 0x0019, 0, 0x2221, 3000, "ARE YOU KIDDING?" },
  { 0x0013, 1, 0x61A8, 250, "STINK FINGER" },
  { 0x0013, 1, 0x61A9, 250, "STINK FINGER" },
  { 0x0013, 1, 0x61AA, 100, "CROW'S NEST" },
  { 0x0013, 1, 0x61AB, 100, "EVEN BETTER" },
  { 0x0013, 1, 0x61AC, 5000, "JACKHOLE!" },
  { 0x0013, 1, 0x61AD, 5000, "HOLEJACK!" },
  { 0x0013, 1, 0x61AE, 5000, "STAYIN' ALIVE!" },
  { 0x0013, 1, 0x61AF, 400, "CLUB CLONE" },
  { 0x0013, 1, 0x61B0, 400, "ME TOO" },
  { 0x0013, 1, 0x61B1, 1000, "CHEATER" },
  { 0x0013, 1, 0x61B2, 750, "PERCH" },
  { 0x0013, 1, 0x61B3, 5000, "STAYIN' ALIVE!" },
  { 0x0013, 1, 0x61B5, 300, "PUNK GAP" },
  { 0x0013, 1, 0x61B6, 450, "TUFFY GAP" },
  { 0x0013, 1, 0x61B7, 100, "BUCK UP GAP" },
  { 0x0013, 1, 0x61B8, 350, "BUCK UP MORE GAP" },
  { 0x0013, 1, 0x61B9, 400, "YOU BUCKED UP NOW" },
  { 0x0013, 1, 0x61BA, 250, "TIGHT DROP" },
  { 0x0013, 1, 0x61BB, 250, "ALL BASS GAP" },
  { 0x0013, 1, 0x61BC, 250, "ATOMIC DROP" },
  { 0x0013, 1, 0x61BF, 250, "STAIR GAP" },
  { 0x0013, 1, 0x61C0, 100, "CROW'S NEST" },
  { 0x0013, 1, 0x61C1, 100, "POOL TRANSFER" },
  { 0x0019, 0, 0x639C, 400, "DINOGRIND!" },
  { 0x0019, 0, 0x639D, 400, "DINOGRIND!" },
  { 0x0019, 0, 0x639F, 250, "GRIND LT" },
  { 0x0019, 0, 0x63A0, 100, "ONE OF FOUR" },
  { 0x0019, 0, 0x63A1, 100, "TWO OF FOUR" },
  { 0x0019, 0, 0x63A2, 100, "THREE OF FOUR" },
  { 0x0019, 0, 0x63A3, 100, "FOUR OF FOUR" },
  { 0x0019, 0, 0x63A4, 500, "KING GOD" },
  { 0x0019, 0, 0x63A5, 100, "THE DRINK" },
  { 0x0019, 0, 0x63A6, 400, "ALL YOUR BASS ARE BELONG TO US!" },
  { 0x0019, 0, 0x63A7, 200, "TUNE IN GAP" },
  { 0x0019, 0, 0x63A8, 200, "TUNE IN GAP" },
  { 0x0019, 0, 0x63A9, 350, "TURN ON GAP" },
  { 0x0019, 0, 0x63AA, 350, "TURN ON GAP" },
  { 0x0019, 0, 0x63AB, 400, "DROP OUT GAP" },
  { 0x0019, 0, 0x63AC, 400, "DROP OUT GAP" },
  { 0x0019, 0, 0x63AD, 1000, "AIR DOMINATION GAP" },
  { 0x0019, 0, 0x63AE, 400, "LIGHT IN THE LOAFERS GAP" },
  { 0x0019, 0, 0x63AF, 200, "LIGHT STEPPER GAP" },
  { 0x0019, 0, 0x63B0, 300, "POOL SPAN GAP" },
  { 0x0019, 0, 0x63B1, 250, "FRONT AND CENTER GAP" },
  { 0x0019, 0, 0x63B2, 250, "COP A RIDE GAP" },
  { 0x0019, 0, 0x63B3, 250, "COP A RIDE GAP" },
  { 0x0019, 0, 0x63B4, 250, "COP A FEEL GAP" },
  { 0x0019, 0, 0x63B5, 250, "COP A FEEL GAP" },
  { 0x0019, 0, 0x63B6, 1000, "DJ GRIND" },
  { 0x0019, 0, 0x63B7, 250, "NIGEHOLE" },
  { 0x0019, 0, 0x63B8, 250, "GLOWSTICK GRIND" },
  { 0x0019, 0, 0x63B9, 400, "GLOWSTICK TRANSFER" },
  { 0x0019, 0, 0x63BA, 550, "BAR CRAWL" },
  { 0x0019, 0, 0x63BB, 550, "BRICK LINE GAP" },
  { 0x0047, 0, 0x6400, 500, "CENTER OF THE UNIVERSE" },
  { 0x0047, 0, 0x6401, 500, "BILLY J DANCEFLOOR" },
  { 0x0047, 0, 0x6402, 500, "BILLY J DANCEFLOOR" },
  { 0x0013, 1, 0x6465, 400, "DOORWAY GAP" },
  { 0x0013, 1, 0x6466, 300, "HALFPIPE TRANSFER" },
  { 0x0013, 1, 0x6467, 280, "OVER THE FLAT RAMP" },
  { 0x0013, 1, 0x6468, 200, "OVER RAMP NUMBER 2" },
  { 0x0013, 1, 0x6469, 1000, "CANYON GLIDE" },
  { 0x0013, 1, 0x646A, 290, "FLY HIGH" },
  { 0x0013, 1, 0x646B, 300, "ARMADILLO GAP" },
  { 0x0013, 1, 0x646C, 100, "LI'L KICKER GAP" },
  { 0x0013, 1, 0x646D, 100, "LOITER GAP" },
  { 0x0013, 1, 0x646E, 100, "OVER THE RAIL" },
  { 0x0013, 1, 0x646F, 300, "BUNT GAP" },
  { 0x0013, 1, 0x6470, 300, "CANYON GAP" },
  { 0x0013, 1, 0x6471, 5000, "CONGE OLLIE! LOST DOG FOUND!!" },
  { 0x0013, 1, 0x6472, 1200, "TRIPLE RAMP" },
  { 0x0013, 1, 0x6473, 900, "RAMP 2X GAP" },
  { 0x0013, 1, 0x6474, 500, "FUNKY DROP" },
  { 0x0013, 1, 0x6475, 1500, "RAMP 3X GAP" },
  { 0x0013, 1, 0x6476, 800, "DOUBLE RAMP" },
  { 0x0013, 1, 0x6477, 100, "OVERTOP" },
  { 0x0019, 0, 0x6478, 800, "BUFFET GAP" },
  { 0x0019, 0, 0x6479, 75, "FROG HOP!" },
  { 0x0019, 0, 0x647A, 950, "TARZAN GRIND!" },
  { 0x0019, 0, 0x647B, 800, "ROPE BRIDGE CROSSIN'" },
  { 0x0019, 0, 0x647C, 600, "FUNKY FUNBOX TRANSFER" },
  { 0x0019, 0, 0x647D, 1000, "TRICKY GATOR GRIND" },
  { 0x0019, 0, 0x647E, 700, "TRIPLE HI-LIGHTS" },
  { 0x0019, 0, 0x647F, 500, "BAR HOP" },
  { 0x0019, 0, 0x6480, 300, "STAIR-SLIDE TRANSFER" },
  { 0x0019, 0, 0x6481, 400, "STAIR-FENCE CROSSOVER" },
  { 0x0019, 0, 0x6482, 700, "CLEAR THE BLEACHERS!" },
  { 0x0019, 0, 0x6483, 2500, "FUNKY FLIER" },
  { 0x0019, 0, 0x6484, 280, "RAMP TO RAMP GRIND" },
  { 0x0019, 0, 0x6485, 4200, "TRIPLE SKIPPIN' AIR GRIND" },
  { 0x0019, 0, 0x6486, 1000, "VERT-IGO!" },
  { 0x0019, 0, 0x6487, 320, "FENCE 2 FENCE GAP SLIDE" },
  { 0x0019, 0, 0x6488, 400, "RAIL To RAIL HOP" },
  { 0x0019, 0, 0x6489, 1000, "AIR RAIL TO RAIL" },
  { 0x0019, 0, 0x648A, 1000, "FENCE TO RAIL TRANSFER" },
  { 0x0019, 0, 0x648B, 600, "TRIPLE BENCH SERIES" },
  { 0x0019, 0, 0x648C, 300, "BUNT GRIND HOP" },
  { 0x0019, 0, 0x648D, 400, "LOITER HOP" },
  { 0x0019, 0, 0x648E, 500, "ARMADILLO HOP" },
  { 0x0019, 0, 0x648F, 320, "CANYON HOP" },
  { 0x0019, 0, 0x6490, 500, "CUTTIN' CORNERS" },
  { 0x0019, 0, 0x6492, 700, "FLAT RAMP 2 LEDGE" },
  { 0x0019, 0, 0x6493, 400, "BUILDING VENT GAP" },
  { 0x0019, 0, 0x6494, 100, "LI'L KICKER HOP" },
  { 0x0019, 0, 0x6495, 3100, "DOUBLE TRIPLE FENCEGRIND COMBO" },
  { 0x0027, 0, 0x64B5, 350, "TAMPA FUNBOX MANUAL" },
  { 0x0047, 0, 0x64B6, 300, "CLEAN THE BAR MANUAL" },
  { 0x0027, 0, 0x64B8, 300, "HIGH KICK MANUAL" },
  { 0x0005, 1, 0x64BF, 320, "TAMPA HALFPIPE EXTENSION" },
  { 0x0005, 1, 0x64C0, 340, "RAMP PLANT" },
  { 0x0005, 1, 0x64C1, 1000, "LOOPY LIP" },
  { 0x0022, 0, 0x64AC, 5000, "JUS' WIN, BABY!" },
  { 0x0033, 0, 0x64AD, 600, "STAIR WALL-OVER" },
  { 0x0013, 1, 0x1772, 250, "STAIR GAP" },
  { 0x0013, 1, 0x1773, 200, "TRAIN AIR" },
  { 0x0013, 1, 0x1774, 800, "FULL TRAIN AIR!" },
  { 0x0013, 1, 0x1775, 250, "SHORT CUT" },
  { 0x0013, 1, 0x1776, 800, "SKY HOOK!" },
  { 0x0013, 1, 0x1777, 250, "S' PIPE" },
  { 0x0013, 1, 0x1778, 300, "OUTIE" },
  { 0x0013, 1, 0x1779, 200, "END OF THE LINE" },
  { 0x0013, 1, 0x177A, 100, "JUMPIN' THE TURNSTYLE" },
  { 0x0013, 1, 0x177B, 100, "JUMPIN' THE TURNSTYLE" },
  { 0x0013, 1, 0x177C, 200, "SKIP THE STAIRS GAP" },
  { 0x0013, 1, 0x177D, 100, "BENCH HOP GAP" },
  { 0x0013, 1, 0x177E, 100, "POOP CHUTE GAP" },
  { 0x0013, 1, 0x177F, 300, "RAMP TRANSFER GAP" },
  { 0x0013, 1, 0x1780, 100, "HOP THE TRAIN" },
  { 0x0013, 1, 0x1781, 300, "SEWER GAP" },
  { 0x0033, 0, 0x17A2, 100, "BOTTOM OF THE ECHO GAP" },
  { 0x0033, 0, 0x17A3, 100, "BLUE GIRL GAP" },
  { 0x0033, 0, 0x17A4, 100, "AFROQUATIC GAP" },
  { 0x0033, 0, 0x17A5, 100, "SILKY LOVE DROID GAP" },
  { 0x0033, 0, 0x17A6, 100, "YELOE GAP" },
  { 0x0033, 0, 0x17A7, 100, "D-MAN GAP" },
  { 0x0033, 0, 0x17A8, 400, "CRAOLA GAP" },
  { 0x0033, 0, 0x17A9, 100, "SAVE GAP" },
  { 0x0033, 0, 0x17AA, 100, "NACE GAP" },
  { 0x0033, 0, 0x17AB, 100, "SPLITOROUS REX GAP" },
  { 0x0033, 0, 0x17AC, 58, "BIG DUMB JOCK" },
  { 0x0019, 0, 0x17D4, 400, "LIGHT THE WAY" },
  { 0x0019, 0, 0x17D5, 200, "RAT WIRE" },
  { 0x0019, 0, 0x17D6, 400, "ON THE EDGE" },
  { 0x0019, 0, 0x17D7, 400, "FIRST RAIL" },
  { 0x0019, 0, 0x17D8, 400, "SECOND RAIL" },
  { 0x0019, 0, 0x17D9, 600, "!THIRD RAIL!" },
  { 0x0019, 0, 0x17DA, 600, "QUICK WAY" },
  { 0x0019, 0, 0x17DB, 400, "EDGE OF YOUR SEAT" },
  { 0x0019, 0, 0x17DC, 200, "IN THE S'" },
  { 0x0019, 0, 0x17DD, 200, "ABOVE THE S'" },
  { 0x0019, 0, 0x17DE, 400, "HARD ONE" },
  { 0x0019, 0, 0x17DF, 450, "HIGH HARD ONE" },
  { 0x0019, 0, 0x17E0, 250, "TOXIC" },
  { 0x0019, 0, 0x17E1, 400, "LONG PIPE" },
  { 0x0019, 0, 0x17E2, 400, "FAT PIPE" },
  { 0x0019, 0, 0x17E3, 400, "CROOKED PIPE" },
  { 0x0019, 0, 0x17E4, 400, "GATOR GAP" },
  { 0x0019, 0, 0x17E5, 400, "UP IN LIGHTS GAP" },
  { 0x0019, 0, 0x17E6, 400, "LIGHTS OUT GAP" },
  { 0x0019, 0, 0x17E7, 300, "BENCH GRIND GAP" },
  { 0x0019, 0, 0x17E8, 400, "SKIP IT" },
  { 0x0019, 0, 0x17E8, 600, "BENCH DROP" },
  { 0x0005, 0, 0x189C, 100, "HIGH STEPPIN'" },
  { 0x0027, 0, 0x1900, 500, "SPARE SOME CHANGE?" },
  { 0x0027, 0, 0x1901, 400, "RIDING BETWEEN THE TRAINS" },
  { 0x0013, 0, 0x1964, 250, "PORTA HOP" },
  { 0x0013, 0, 0x1965, 500, "DOZER TRANSFER" },
  { 0x0013, 0, 0x1966, 500, "OUTTA' THE PARK" },
  { 0x0013, 0, 0x1967, 750, "TWICE THE FUN" },
  { 0x0013, 0, 0x1968, 1000, "DEATH FROM ABOVE" },
  { 0x0013, 0, 0x1969, 200, "OBVIOUS GAP" },
  { 0x0013, 0, 0x196A, 250, "OLLEY UP" },
  { 0x0013, 0, 0x196B, 200, "FUNBOX HOP" },
  { 0x0013, 0, 0x196C, 500, "HIGH TRANSFER" },
  { 0x0013, 0, 0x196D, 1500, "TEAMSTER TRANSFER" },
  { 0x0013, 0, 0x196E, 1500, "TRUCK STOP HOP" },
  { 0x0013, 0, 0x196F, 2000, "OVER THE BOWL" },
  { 0x0013, 0, 0x1970, 250, "GOING DOWN" },
  { 0x0013, 0, 0x1971, 300, "2ND FLOOR" },
  { 0x0013, 0, 0x1972, 1000, "ACROSS THE GAP" },
  { 0x0013, 0, 0x1973, 1500, "TWO-THREE TRANSFER" },
  { 0x0013, 0, 0x1974, 2000, "TOP OF THE WORLD" },
  { 0x0013, 0, 0x1975, 2000, "HEADING BACK DOWN" },
  { 0x0013, 0, 0x1976, 2000, "HIGH POINT GAP" },
  { 0x0013, 0, 0x1977, 20000, "WAX ON" },
  { 0x0013, 0, 0x1978, 20000, "WAX OFF" },
  { 0x0019, 0, 0x19C8, 400, "B2B WALL RIDE" },
  { 0x0019, 0, 0x19C9, 30000, "NO CAN DEFEND" },
  { 0x0019, 0, 0x19CA, 2500, "HIGH FLYING RAIL" },
  { 0x0019, 0, 0x19CB, 500, "RAIL HOPPIN'" },
  { 0x0019, 0, 0x19CC, 1500, "PAINT THE FENCE" },
  { 0x0019, 0, 0x19CD, 2500, "STEALING CABLE" },
  { 0x0008, 0, 0x19CE, 2000, "PYRAMID TRANSFER" },
  { 0x0008, 0, 0x19CF, 500, "CUTTING IT CLOSE" },
  { 0x0019, 0, 0x19D0, 2500, "RIDE THE WIRE" },
  { 0x0019, 0, 0x19D1, 2500, "CLIMBING THE STAIRS" },
  { 0x0019, 0, 0x19D2, 500, "GRIND THE PIPE" },
  { 0x0019, 0, 0x19D3, 500, "RIDIN' THE RAILS" },
  { 0x0019, 0, 0x19D4, 2500, "HIGH POINT GRIND" },
  { 0x0019, 0, 0x19D5, 1000, "GRIND UP" },
  { 0x0019, 0, 0x19D6, 1250, "GRIND UP REDUX" },
  { 0x0019, 0, 0x19D7, 1500, "THE LAST GRIND UP" },
  { 0x0019, 0, 0x19D8, 500, "OVER THE DOZER" },
  { 0x0019, 0, 0x19D9, 1500, "SCAFFOLD GRIND" },
  { 0x0019, 0, 0x19DA, 800, "STEP DOWN GRIND" },
  { 0x0019, 0, 0x19DB, 2000, "FIRE GRIND" },
  { 0x0019, 0, 0x19DC, 10000, "EDIZ SEZ..." },
  { 0x0019, 0, 0x19DD, 10000, "TRY THE TRANSFORMERS" },
  { 0x0005, 0, 0x1A2C, 1000, "HOFFA'S LIP" },
  { 0x0005, 0, 0x1A2D, 500, "DABBA-DOO EXTENSION" },
  { 0x0005, 0, 0x1A2E, 500, "TOUCHE EXTENSION" },
  { 0x0005, 0, 0x1A2F, 2500, "ROOFTOP EXTENSION" },
  { 0x0022, 0, 0x1A90, 5000, "GRIND THE CRANE" },
  { 0x0013, 1, 0x64C9, 300, "CHIMNEY TRANSFER" },
  { 0x0013, 1, 0x64CA, 450, "CHIMNEY RUN" },
  { 0x0013, 1, 0x64CB, 900, "LEAP TALL BUILDINGS IN A SINGLE BOUND!" },
  { 0x0013, 1, 0x64CC, 300, "SINGIN' IN DA RAIN" },
  { 0x0013, 1, 0x64CD, 5000, "FREQUENT FLIER!" },
  { 0x0013, 1, 0x64CE, 400, "OFF THE DEEP END" },
  { 0x0013, 1, 0x64CF, 500, "REACH FOR THE SKY" },
  { 0x0013, 1, 0x64D0, 640, "FLY HIGH GAP" },
  { 0x0013, 1, 0x64D1, 2500, "THE CHIMNEY SWEEP!" },
  { 0x0013, 1, 0x64D2, 500, "RAMP TO RAMP GAP" },
  { 0x0013, 1, 0x64D3, 940, "AS LUCKY AS LUCKY CAN BE!" },
  { 0x0013, 1, 0x64D4, 150, "COOL AIR!" },
  { 0x0013, 1, 0x64D5, 450, "HIGH ANXIETY" },
  { 0x0013, 1, 0x64D6, 300, "BUILDING 2 BUILDING TRANSFER" },
  { 0x0013, 1, 0x64D7, 400, "FLOAT LIKE A BUTTERFLY" },
  { 0x0013, 1, 0x64D8, 250, "ROOFTOP 2 ROOFTOP GAP" },
  { 0x0013, 1, 0x64D9, 200, "GOIN' OVER" },
  { 0x0013, 1, 0x64DA, 120, "GOIN' UP" },
  { 0x0013, 1, 0x64DB, 100, "GOIN' DOWN" },
  { 0x0013, 1, 0x64FB, 3000, "AND DANCIN' IN DA RAIN!" },
  { 0x0013, 1, 0x64FC, 1000, "DOUBLE DOWN" },
  { 0x0013, 1, 0x64FD, 400, "REQUESTING FLY BY" },
  { 0x0013, 1, 0x64FE, 350, "HP ROOF LAUNCH" },
  { 0x0013, 1, 0x64FF, 250, "BUILDING SKIP TRANSFER" },
  { 0x0013, 1, 0x6500, 250, "FAN VENT HOP" },
  { 0x0013, 1, 0x6501, 100, "TO BE OR NOT TO BE" },
  { 0x0013, 1, 0x6502, 250, "BOURNOULI PRINCIPLE" },
  { 0x0019, 0, 0x64DD, 400, "HIGH GRIND" },
  { 0x0019, 0, 0x64DE, 250, "WAKE THE BIRDS" },
  { 0x0019, 0, 0x64DF, 500, "MOVIN' IT UP" },
  { 0x0019, 0, 0x64E0, 200, "ROOFTOP TO ROOFTOP GAP" },
  { 0x0019, 0, 0x64E1, 900, "SIGN GRIND" },
  { 0x0019, 0, 0x64E2, 1200, "FUNKY ROOFTOP RAIL SERIES" },
  { 0x0019, 0, 0x64E3, 300, "CHAIR AIR" },
  { 0x0019, 0, 0x64E4, 300, "WATERTOWER GRIND" },
  { 0x0019, 0, 0x64E5, 400, "HANGIN' LIGHT GRIND" },
  { 0x0019, 0, 0x64E6, 5000, "BILLBOARD SWEEP!" },
  { 0x0019, 0, 0x64E7, 3000, "MOTOWN MAD GRIND" },
  { 0x0019, 0, 0x64E8, 380, "ARRIVES JUST IN TIME" },
  { 0x0019, 0, 0x64E9, 800, "BIRD ON A WIRE" },
  { 0x0019, 0, 0x64EA, 190, "STING LIKE A BEE" },
  { 0x0019, 0, 0x64EB, 720, "VENT GRIND SERIES" },
  { 0x0019, 0, 0x64EC, 1234, "TRIPPIN' OUT" },
  { 0x0019, 0, 0x64ED, 800, "CHIMNEY GRIND" },
  { 0x0019, 0, 0x64EE, 600, "POOL 2 AWNING GAP" },
  { 0x0019, 0, 0x64EF, 250, "SKIPPIN THE SAFETY RAIL" },
  { 0x0019, 0, 0x6505, 350, "STAIR SLIDE" },
  { 0x0019, 0, 0x6506, 250, "ROPIN' DOWN" },
  { 0x0019, 0, 0x6507, 850, "LAND ON A WIRE" },
  { 0x0019, 0, 0x6508, 150, "BALANCING ACT" },
  { 0x0019, 0, 0x650A, 200, "CABLE GUY!" },
  { 0x0019, 0, 0x650B, 320, "OVEREASY" },
  { 0x0019, 0, 0x650C, 300, "STEP IT UP!" },
  { 0x0019, 0, 0x650D, 1000, "BILBOARD TO SIGN GRIND" },
  { 0x0019, 0, 0x6519, 100, "BUILDING HOP" },
  { 0x0019, 0, 0x651A, 200, "LIVIN' ON THE EDGE" },
  { 0x0019, 0, 0x651B, 100, "PIPE-VENT GAP" },
  { 0x0019, 0, 0x651C, 4000, "POOL PIGEON" },
  { 0x0019, 0, 0x651D, 1111, "SKY WALKER!" },
  { 0x0019, 0, 0x651E, 100, "SKYLIGHT GRIND" },
  { 0x0019, 0, 0x651F, 3000, "TO BE!" },
  { 0x0027, 0, 0x64F1, 250, "SKYLIGHT MANUAL" },
  { 0x0027, 0, 0x64F2, 250, "DANCIN' IN THE SKYLIGHT" },
  { 0x0005, 1, 0x64F4, 100, "HALFWAY HP LIP" },
  { 0x0005, 1, 0x64F5, 1000, "ODE TO JOY!" },
  { 0x0005, 1, 0x64F6, 100, "MOTOWN HP LIP" },
  { 0x0005, 1, 0x64F7, 540, "PIGEON COUP LIP" },
  { 0x0005, 1, 0x64F8, 200, "CHIMINY TIP LIP" },
  { 0x0005, 1, 0x64F9, 100, "OFFICE BEAK TRICK" },
  { 0x0033, 0, 0x650F, 5000, "RIDE THE BILBOARD" },
  { 0x0033, 0, 0x6511, 700, "SIGN WALLRIDE" }
};
```

## Gap Type
Example code to convert the gap types, check [Th2Scene.cpp](https://github.com/Vadru93/THPS-Level-Editor/blob/master/THPS%20Level%20Editor/Th2Scene.cpp) for more information:
```
switch (Th2xGaps[j].type)
{
  case 0x5:
    memcpy(&EndGap[12], &gapId, 4);
    memcpy(&EndGap[73], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGap[27], Th2xGaps[j].text);
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("PURE_LIP");
    script->Append(0x06);
    script->EndScript();
    break;
  case 0x11://air
  case 0x13:
  case 0x8:
  case 0x17:
    memcpy(&EndGap[12], &gapId, 4);
    memcpy(&EndGap[73], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGap[27], Th2xGaps[j].text);
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("PURE_AIR");
    script->Append(0x06);
    script->EndScript();
    break;
  case 0x19://grind
  case 0x22:
    gapType = 2;
    memcpy(&EndGapWithLoop[29], &gapId, 4);
    memcpy(&EndGapWithLoop[120], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGapWithLoop[57], Th2xGaps[j].text);
    sprintf(chc, "StartedGap%d", gapIndex - 1);
    EndId = Checksum(chc);
    globals.push_back(EndId);
    memcpy(&IsTrue[7], &EndId, 4);
    memcpy(&IsTrue[22], &EndId, 4);
    memcpy(&IsTrue[49], &EndId, 4);
    script->Append(IsTrue, sizeof(IsTrue));
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("CANCEL_GROUND");
    script->Script("CANCEL_MANUAL");
    script->Script("REQUIRE_RAIL");
    script->Append(0x06);
    script->EndScript();
    script->Append(EndIF, sizeof(EndIF));
    break;
  case 0x33://wall
    memcpy(&EndGap[12], &gapId, 4);
    memcpy(&EndGap[73], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGap[27], Th2xGaps[j].text);
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("PURE_WALL");
    script->Append(0x06);
    script->EndScript();
    break;
  case 0x47:
  case 0x1://manual can ollie
    memcpy(&EndGap[12], &gapId, 4);
    memcpy(&EndGap[73], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGap[27], Th2xGaps[j].text);
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("REQUIRE_MANUAL");
    script->Script("CANCEL_GROUND");
    script->Script("CANCEL_WALL");
    script->Script("CANCEL_RAIL");
    script->Append(0x06);
    script->EndScript();
    break;
  case 0x27://manual can't ollie
    memcpy(&EndGap[12], &gapId, 4);
    memcpy(&EndGap[73], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGap[27], Th2xGaps[j].text);
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("PURE_MANUAL");
    script->Append(0x06);
    script->EndScript();
    break;
  case 0x2://ground
    memcpy(&EndGap[12], &gapId, 4);
    memcpy(&EndGap[73], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGap[27], Th2xGaps[j].text);
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("PURE_GROUND");
    script->Append(0x06);
    script->EndScript();
    break;
  default:
    memcpy(&EndGap[12], &gapId, 4);
    memcpy(&EndGap[73], &Th2xGaps[j].score, 2);
    strcpy((char*)&EndGap[27], Th2xGaps[j].text);
    script->Script("StartGap");
    script->Script("GapID");
    script->Append(0x07);
    script->Script(gapId);
    script->Script("FLAGS");
    script->Append(0x07);
    script->Append(0x05);
    script->Script("PURE_AIR");
    script->Append(0x06);
    script->EndScript();
    break;
}
```
