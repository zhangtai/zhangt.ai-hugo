---
title: Convert Geneos Docs into eBooks with Calibre
date: 2018-03-29T21:46:48+08:00
draft: false
---

Dash and DevDocs are both well designed and maintained development docs reference apps, you can find most common used language or apps there, they are pretty handy for ad-hoc checking. But for me, I need more features to study new skills. e.g.:

* add favorite
* highlight
* taking notes
* export
* ...

I know I am asking too much for those apps, so I am going to do myself for some techs I really serious want to learn, make them to ebook, read and take note.

The first doc start with Geneos, it's not listed on both DevDocs and Dash(as expected), because it's not widely known as I know. Thanks to the ITRS group, they update the docs regurlaly and open to download [HTML files](https://resources.itrsgroup.com/downloads/offline-documentation-bundle-geneos) to their registered members.

## Remove unwanted sections

After docs download and extracted into it's folder, we need to remove some sections we don't want to be shown in the eBook, e.g. register section, footer section, menus, bookmark indicator etc.

Job can be done by node.js, within your project install `cheerio` and `recursive-readdir`, they are used for parse and remove DOM elements in HTML, and list all HTML files to be edited in the folder.

```js
const cheerio = require('cheerio')
const fs = require('fs')
const recursive = require('recursive-readdir')
// this app take only one argument: the path of docs
const docPath = process.argv[2]

recursive(docPath, (err, files) => {
  if (!err) {
    for (const file of files) {
      // only search and replace in HTML file
      if (file.endsWith('html')) {
        trimFile(file)
      }
    }
  }
})

let trimFile = (filename) => {
  fs.readFile(filename, (err, data) => {
    if (!err) {
      let $ = cheerio.load(data)
      // removing all sections we don't want
      $('header#main-header').remove()
      $('footer#main-footer').remove()
      $('div#Modal').remove()
      $('div#MainNav').remove()
      $('a.headerlink').remove()
      // write back to the file
      fs.writeFile(filename, $('html'), () => {
        console.log('Write new index file done')
      })
    }
  })
}
```

## Convert HTML to eBooks with Calibre

Calibre is a cross-platform open-source suite of e-book software, which also can create and convert ebooks in many format.

There are some documents in the offline pages of geneos, I am picking one: Gateway 2, by click the Add book button at top right of the interface of Calibre and select the index file. After book added, click the convert book button and file in below config in the confiture interface. Click OK to let it go. The conversion should be around 1 minute.

``` yml
Metadata:
  'Input format': ZIP
  'Output format': EPUB

Look & feel:
  'Extra CSS': "#MainContent #Docs article p { line-height: 1.2rem; }"
  'Filter Style Information':
    - Margin
    - Padding

Structure detection:
  'Insert page breaks': "//*[name()='h1' or name()='h2' name()='h3']"

Table of Contents:
  'Level 1 TOC': "//h:h1"
  'Level 2 TOC': "//h:h2"
  'Level 3 TOC': "//h:h3"
```

After the conversion, you will get you eBook with structured ToC. Enjoy reading and learning!
