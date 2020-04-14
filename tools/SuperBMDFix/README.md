# SuperBMD
A library to import and export various 3D model formats into the Binary Model (BMD) format.

This API uses the Open Asset Import Library (AssImp). A list of supported model formats can be found [here](http://assimp.sourceforge.net/main_features_formats.html). Recommended: .dae/.fbx.

This is a fork of SuperBMD that supports generating triangle strips (a more space-efficient way to represent faces in a model) 
and extracting and applying BMD material data as JSON, maintained by Yoshi2 (RenolY2 on github). <br>
Please report any issues you find here: https://github.com/RenolY2/SuperBMD/issues <br>
New compiled releases (and their source code) are found here: https://github.com/RenolY2/SuperBMD/releases

# Usage

SuperBMD.exe can be used to convert models to the BMD format and BMD models to the DAE format.

To convert a model, drag it onto the executable or run the program via command line:

`SuperBMD.exe <model path>`

If model path is a BMD, it will be converted to DAE, its textures will be dumped as .PNG and 
its material data will be dumped as JSON in the same folder as the DAE and the textures.

If model path is a model but not BMD, it will be converted to BMD. By default triangle strips 
are generated for static meshes but not rigged meshes. See below for more options on this.

# Full usage
`SuperBMD.exe <input path> [output path] [--mat <material path>] [--texheader <texheader path>] [--tristrip (all/static/none)] [--rotate] [--dontFix] [--bdl] [--version]`

Arguments in square brackets are optional. If output path is left out it is created from the input 
path by replacing the extension either with `.bmd`, `.bdl` or `.dae`, depending on input. 
  
* If input path is BMD then the extracted DAE is written to output path. Textures are dumped to 
the same directory as the DAE. 
* If input path is not a BMD, the model is converted into a BMD and written to output path. If output path ends with ``.bdl`` and the ``--bdl`` option is set, or output path is empty and ``--bdl`` is set, the model is converted to BDL.  
* If input path is a BMD/BDL and output path is a BMD, behaviour is similar to if the input was not a BMD/BDL.

The ``--version`` option outputs the program's version and then exits.

Any model -> BMD:
* If material path is set, loads material data from that path (See "Materials" further down in this document)
* If texheader path is set, loads texture header info from that path (See "Texture header info" further down in this document)
* ``--tristrip none`` generates no triangle strips. ``--tristrip static`` generates triangle strips only for static models (no rig). 
``--tristrip all`` generates triangle strips for all models (static and rigged), currently triangle strips on rigged models are buggy though. 
If omitted, the default mode is generating triangle strips only for static models.
* ``--rotate`` rotates the model so that Y is up instead of Z. Default is no rotation.
* ``--dontFix`` disables (trying to) fix normals on rigged models. This is only relevant for using materials that add shading to the model and has no 
effect on whether a face is considered to be front or back-facing for culling purposes.
* ``--bdl`` is necessary for when you want to create a BDL instead of a BMD..

BMD -> DAE:
* If material path is set, write material data to that path. Otherwise, write it in the same place as the created DAE.
* If texheader path is set, write texture header info to that path. Otherwise, write it in the same place as the created DAE.
* None of the other options have an effect.


## Notes
### Modeling
* When exporting a model for conversion to BMD, rotate the model about the X axis by -90 degrees. 
Most modeling programs define the Z axis as the up axis, but Nintendo games use the Y axis instead. 
Rotating the model ensures that the model is not sideways when imported into a game. Alternatively, 
set the --rotate option when calling SuperBMD.exe which will rotate the model appropriately without you having to do it.

### Skinning
* SuperBMD supports both skinned and unskinned models.
* For skinned meshes, <b>make the root of the model's skeleton the child of a dummy object called 
  `skeleton_root`.</b> SuperBMD uses the name of this dummy object to find the root of the skeleton 
  so that it can process it.
* If a `skeleton_root` object is not found, then the model will be imported with a single root bone. 
This is recommended for models intended for maps. If your model

### Vertex Colors
* SuperBMD supports vertex colors.

### Textures
* SuperBMD supports models that have no textures. These models will appear white when imported into a game.
* It is recommended that the model's textures be in the same directory as the model being converted. <b>
If SuperBMD cannot find the model's textures, it will use a black and white checkerboard image instead.</b>
* Textures must be in BMP, JPG, PNG or TGA format.

### Converting BMD to DAE 
When you drop a BMD on SuperBMD.exe, it will be converted into DAE. When you import the DAE into 3DS Max,
in the Import options that appear you need to go into Advanced Options->Units, uncheck "Automatic" and set 
File units converted to Meters. Otherwise the imported dae will be very big.

### Materials
* This fork of SuperBMD allows dumping and inserting materials as JSON and it allows additional 
textures to be loaded without having to apply them in a 3D modelling program. Look at Full Usage 
above on how to extract and apply material files. While Drag&Drop works, writing bat files is recommended.

* The mat json file will contain one or more materials from the BMD file. When applying materials, the material 
names are exactly the name of the materials of the model when exported from a 3D modelling program. 
You do not have to cover every material, SuperBMD will generate material data for the rest. You can 
also use a default material setup for every material in the model by naming the material in the JSON ``__MatDefault``.

* The ``Textures`` section contains the names of the textures used by the material. Usually the 
first texture is the main texture, the rest are used for graphics effects. When using a ``__MatDefault`` 
material it can be useful to leave the first texture name as ``null`` so each material will retain the 
texture name it originally had.

* Additional textures can be loaded by adding their names in the ``Textures`` section of a material. 
The program will search for these textures in the folder of the material file by appending an extension to 
the name in this order: ``.png``, ``.jpg``, ``.tga``, ``.bmp``. (Example: If there are two textures 
``Texture.png`` and ``Texture.bmp``, the PNG texture will be loaded rather than the BMP texture)

* You can modify materials if you know what you are doing. Look on the internet for info on TEV and 
check the source code of SuperBMD for possible values for some of the enums (When viewing the source 
code in VS or other good IDEs, check ``SuperBMD/source/Materials/Material.cs``). A recap of all enums 
and useful info about materials will be available here https://github.com/RenolY2/SuperBMD/wiki when 
it is added.

### Material presets
This release has some bat scripts that apply some included materials.
* toonshading: This provides SMS-style toon shading. Known to work with SMS.
* shiny: Based on the toon shading material, this gives the model an interesting shine. Known to work with SMS.
* simpleshading: Creates shadows based on the direction a triangle is facing. Known to work with Pikmin 2, but looks dark in SMS.

These material presets are not always the best and you are encouraged to find other 
presets by experimenting with material data from models and making new scripts. 
You can combine options if wanted. For example, you can add the --rotate option 
from superbmd_rotatemodel.bat to the other bat scripts if you want shaded and rotated models, 
or you can add the --bdl option from superbmd_createbdl.bat to the other scripts for shaded BDL models.

### Texture header info
When dumping BMDs to DAE you will find a texheader json file. It contains some data from the BTI headers of 
the textures stored in the BMD. Of note are the Format (sets the texture format) and the WrapS/WrapT (affects 
the way UV maps will wrap when going beyond the edges) attributes. They are applied in a similar way to materials,
using --texheader to specify the path to such a file. Header data is applied to textures with a matching name. A default
header can be specified by setting the name to ``__TexDefault``.

Supported texture formats: I4, I8, IA4, IA8, RGB565, RGB5A3, RGBA32 and CMPR (in other words, all that GC hardware supports except 
for the color-indexed formats). Supported wrap modes: ClampToEdge, Repeat and MirroredRepeat 
When using I4 or I8, note that intensity equals alpha.

## Attribution
This project uses a number of external libraries or parts of them in the code which will be listed here 
together with their license if necessary:

* BrawlLib: Tristripping algorithm by ernie, blackjax & tfautre:  https://github.com/libertyernie/brawltools
<br>

* TGA reader by Dmitry Brant: http://dmitrybrant.com
<br>

* CTools by Chadderz, see https://github.com/RenolY2/SuperBMD/blob/master/SuperBMD/source/Materials/ImageDataFormat.cs
<br>

If you acquired a compiled release of SuperBMD, the following licenses also apply:

* Assimp: (Assimp32.dll, Assimp64.dll, AssimpNet.dll)
```
 Copyright (c) 2006-2015 assimp team
All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

    Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
    Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
    Neither the name of the assimp team nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
<br>

* Json.NET: (Newtonsoft.Json.dll)
```
The MIT License (MIT)

Copyright (c) 2007 James Newton-King

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```
<br>

* OpenTK (OpenTK.dll)
```
Further details about the license of OpenTK: https://github.com/andykorth/opentk/blob/master/Documentation/License.txt

The Open Toolkit library license

Copyright (c) 2006 - 2013 Stefanos Apostolopoulos <stapostol@gmail.com> for the Open Toolkit library.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

```
