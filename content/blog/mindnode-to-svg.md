---
title: "MindNode export as SVG"
date: 2018-01-04T20:20:48+08:00
---

## First, export FreeMind format

{{% center %}}
    {{< figure src="../img/mindnode-interface-mac-01.jpeg" width="100%" alt="">}}
{{% /center %}}

## Modify with freemind and export to SVG format

FreeMind says:

> Exporting from command line is impossible: it is impossible to tell FreeMind on the command line to export a given mind map using one of FreeMind's export functions.

{{% center %}}
    {{< figure src="../img/freemind-interface-mac-01.jpeg" width="100%" alt="">}}
{{% /center %}}

## Optimize the svg file

### What [SVGOMG](https://jakearchibald.github.io/svgomg/) can help in this case

- Reduce file size about 80%

### Remove background color and make the SVG responsible on web

Find those fills and change the values to "none"

{{% center %}}
    {{< figure src="../img/fill-remove-01.jpeg" width="100%" alt="">}}
{{% /center %}}

{{% center %}}
    {{< figure src="../img/fill-remove-02.jpeg" width="100%" alt="">}}
{{% /center %}}

Replace "width" and "height" with "viewBox"

{{% center %}}
    {{< figure src="../img/svg-viewbox-add.jpeg" width="100%" alt="">}}
{{% /center %}}

## Final

{{% svg "/img/MyTracks.svg" "/img/MyTracks.svg" %}}