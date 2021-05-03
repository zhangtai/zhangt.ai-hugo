---
title: "MindNode export as SVG"
date: 2018-01-04T20:20:48+08:00
---

## First, export FreeMind format

![](../img/mindnode-interface-mac-01.jpeg)

## Modify with freemind and export to SVG format

FreeMind says:

> Exporting from command line is impossible: it is impossible to tell FreeMind on the command line to export a given mind map using one of FreeMind's export functions.

![](../img/freemind-interface-mac-01.jpeg)

## Optimize the svg file

### What [SVGOMG](https://jakearchibald.github.io/svgomg/) can help in this case

- Reduce file size about 80%

### Remove background color and make the SVG responsible on web

Find those fills and change the values to "none"

![](../img/fill-remove-01.jpeg)

![](../img/fill-remove-02.jpeg)

Replace "width" and "height" with "viewBox"

![](../img/svg-viewbox-add.jpeg)

## Final

{{% svg "/img/MyTracks.svg" %}}
