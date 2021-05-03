---
title: "How to use custom font in VS Code(Windows) without admin privilege to install the font?"
date: 2018-09-12T19:25:48+08:00
slug: vscode-windows
---

> I don't have Admin privilege on my working PC(Windows 7), so I can't install custom font(Fira Code) into my system. Is there a way to use such font without installing in VS Code?

The same question I asked on [stack overflow](https://stackoverflow.com/questions/52286889/how-to-use-custom-font-in-vs-codewindows-without-admin-privilege-to-install-th), since no perfect answers for it, I need to get my hands dirty on it. VS Code is a Chromium based application, so why not using web fonts?


1. Open `Help -> Toggle Developer Tools` in the menu
2. Paste below js codes and execute in the `Console` of DevTools to setup 'Fira Code' font face.

    ```js
    var styleNode = document.createElement('style');
    styleNode.type = "text/css";
    var styleText = document.createTextNode(`
        @font-face{
            font-family: 'Fira Code';
            src: url('https://raw.githubusercontent.com/tonsky/FiraCode/master/distr/eot/FiraCode-Regular.eot') format('embedded-opentype'),
                 url('https://raw.githubusercontent.com/tonsky/FiraCode/master/distr/woff2/FiraCode-Regular.woff2') format('woff2'),
                 url('https://raw.githubusercontent.com/tonsky/FiraCode/master/distr/woff/FiraCode-Regular.woff') format('woff'),
                 url('https://raw.githubusercontent.com/tonsky/FiraCode/master/distr/ttf/FiraCode-Regular.ttf') format('truetype');
            font-weight: normal;
            font-style: normal;
        }`);
    styleNode.appendChild(styleText);
    document.getElementsByTagName('head')[0].appendChild(styleNode);
    ```

3. Make sure `'Fira Code'` is the first parameter in the **Font Family** settings.
4. Enable **Font Ligatures** settings, otherwize ">=" won't be transferred to "≥"
5. (Optional) in the **Sources** tab of DevTools, create a new snippet with the codes, and right click the new created file to run. It can be preserved ever after restart the application.
