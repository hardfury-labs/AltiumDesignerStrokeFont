# Altium Designer Stroke Font

![Altium Designer Stroke](snapshots/altium-designer-stroke.png)

Glyphs sketch

![Glyphs sketch](snapshots/glyphs-sketch.png)

## Font Variants

- Light 300
- Regular 400
- Medium 500
- Bold 700

## Using fonts in LCEDA

![LCEDA](snapshots/lceda.png)

List the font variants via browser console.

```javascript
for (const { postscriptName } of await queryLocalFonts()) {
  if (postscriptName.includes("AltiumDesignerStroke"))
    console.log(postscriptName);
}
```

## How it's developed

### 1. Generate charset

List the glyphs from ASCII 32 to 126.

```javascript
const chars = [];

for (let i = 32; i <= 126; i++) {
  chars.push(String.fromCharCode(i));
}

console.log(chars.join(""));
console.log(`total ${chars.length} chars`);
```

Output:

```
 !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~

total 95 chars
```

### 2. Generate PDF with Smart PDF

Display the glyphs in Altium Designer PCB and export to Smart PDF.

![Smart PDF](snapshots/smart-pdf.png)

### 3. Extract SVG from PDF

- Adobe Illustrator
- or [Vector Magic](https://vectormagic.com)
- or [Inkscape](https://inkscape.org) `/Applications/Inkscape.app/Contents/MacOS/inkscape --export-type="svg" --export-area-drawing PCB1.pdf`
- or [IDRsolutions](https://www.idrsolutions.com/online-pdf-to-html5-converter)

### 4. Guides marks

![Guides marks](snapshots/guides-marks.png)

### 5. Build font via [Glyphs](https://glyphsapp.com)

Set layer margin

```python
# current layer
layer = Glyphs.font.selectedLayers[0]

layer.LSB = 0
layer.RSB = 0
```

Check font glyphs

```python
# check font glyphs
for glyph in Glyphs.font.glyphs:
    layer = glyph.layers[0]

    if layer.LSB != 0 or layer.RSB != 0:
        print("{} except margin! left: {}, right: {}".format(glyph.name, layer.LSB, layer.RSB))

    if layer.width != 400:
        print("{} except width! {}".format(glyph.name, layer.width))
```

Listing attributes of paths

```python
layer = Glyphs.font.selectedLayers[0]

for path in layer.paths:
    print("attributes: {}".format(path.attributes))
```

Generate font family

```python
import re

for family_name in [
    "Light 300",
    # "Regular 400",
    # "Medium 500",
    # "Bold 700",
]:
    # config
    stroke_width = int(re.findall(r"\d+", family_name)[0]) / 10
    margin = stroke_width

    # get all glyphs
    for glyph in Glyphs.font.glyphs:
        stroke_layer = None
        target_layer = None

        # get all layer of glyph
        for layer in glyph.layers:
            # print("glyph: {}, layer: {}, layer paths: {}".format(glyph.name, layer.name, len(layer.paths)))

            if layer.name == "Stroke":
                stroke_layer = layer
            if layer.name == family_name:
                target_layer = layer

        # copy paths to target layer
        if len(target_layer.paths) == 0:
            for path in stroke_layer.paths:
                target_layer.paths.append(path.copy())

            print("copy paths to {}/{}".format(glyph.name, family_name))


        # set attributes
        for path in target_layer.paths:
            # path.attributes["lineCapStart"] = 1
            # path.attributes["lineCapEnd"] = 1
            path.attributes["strokeWidth"] = stroke_width

        # set layer margin
        if glyph.name == "space":
            target_layer.width = 400 + stroke_width * 3
        else:
            target_layer.LSB = margin
            target_layer.RSB = margin
```

## Similar fonts

- [Proto Mono](https://atktype.gumroad.com/l/protomono) Designed by Radinal Riki, without lowercase letters
- [Chakra Petch](https://fonts.google.com/specimen/Chakra+Petch) Designed by Cadson Demak
