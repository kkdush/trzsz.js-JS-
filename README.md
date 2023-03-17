# trzsz.js

Making webshell and terminal supports [trzsz](https://trzsz.github.io/) ( trz / tsz ), which similar to ( rz / sz ), and compatible with tmux.

[![MIT License](https://img.shields.io/badge/license-MIT-green.svg?style=flat)](https://choosealicense.com/licenses/mit/)
[![npmjs trzsz](https://img.shields.io/npm/v/trzsz.svg?style=flat)](https://www.npmjs.com/package/trzsz)

#前言
我在实现 https://github.com/trzsz/trzsz.js （ 在浏览器中类似 rz / sz 那样上传和下载文件 ）时，发现了 Chrome 等浏览器支持 js 读写本地文件，详见 https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API

一开始实现的 trzsz ( trz / tsz ) 只支持上传和下载文件，不支持目录，所以没太关注目录方面的描述。等到后来 trzsz ( trz / tsz ) 开始支持上传和下载目录，想当然认为 js 能在浏览器中读写文件已经很好了，没想着 js 是否也可以在浏览器中读写目录。

直到有人问起 trzsz ( trz / tsz ) 在浏览器中是不支持目录传输吗，又再去看了一次 API 文档，才发现目录相关的 API 被我忽略了。js 有读写目录的权限，https://github.com/trzsz/trzsz.js 就可以和其他终端一样，上传和下载目录了，下载多个文件时也不用多次弹出对话框了。Cool…


## Why?

Considering `laptop -> hostA -> hostB -> docker -> tmux`, using `scp` or `sftp` is inconvenience.

In this case, `lrzsz` ( rz / sz ) is convenient to use, but unfortunately it's not compatible with `tmux`.

`tmux` is not going to support rz / sz ( [906](https://github.com/tmux/tmux/issues/906), [1439](https://github.com/tmux/tmux/issues/1439) ), and creating a new tools is much easier than patching `tmux`.

[trzsz.js](https://github.com/trzsz/trzsz.js) is a `js` version of [trzsz](https://github.com/trzsz/trzsz), which supports webshell running in browser, terminal built with electron, etc.


## Getting Started

* Install the module
  ```
  npm install trzsz
  ```
  or
  ```
  yarn add trzsz
  ```

* Use in Node.js
  ```js
  import { TrzszFilter } from "trzsz";
  ```
  or
  ```js
  const { TrzszFilter } = require("trzsz");
  ```

* Use in browser
  ```html
  <script src="node_modules/trzsz/lib/trzsz.js"></script>
  ```

* Create `TrzszFilter` object
  ```js
  const trzszFilter = new TrzszFilter({
    // The trzsz options, see below
  });
  ```

* Generally, the output of the server is forwarded to the terminal. Pass the output through `TrzszFilter`.
  ```js
  const trzszFilter = new TrzszFilter({
    // The output will be forwarded back by TrzszFilter, unless the user runs ( trz / tsz ) on the server.
    writeToTerminal: (data) => terminal.write(typeof data === "string" ? data : new Uint8Array(data)),
  });

  // forward the output to TrzszFilter
  webSocket.addEventListener("message", (ev) => trzszFilter.processServerOutput(ev.data));
  ```

* Generally, the user input is forwarded to the server. Pass the user input through `TrzszFilter`.
  ```js
  const trzszFilter = new TrzszFilter({
    // The user input will be forwarded back by TrzszFilter, unless there are files being transferred.
    sendToServer: (data) => webSocket.send(data),
  });

  // forward the user input to TrzszFilter
  terminal.onData((data) => trzszFilter.processTerminalInput(data));
  // forward binary input to TrzszFilter
  terminal.onBinary((data) => trzszFilter.processBinaryInput(data));
  ```

* Let `TrzszFilter` know the terminal columns for rendering progress bar.
  ```js
  const trzszFilter = new TrzszFilter({
    // initialize the terminal columns
    terminalColumns: terminal.cols,
  });

  // reset the terminal columns
  terminal.onResize((size) => trzszFilter.setTerminalColumns(size.cols));
  ```

* If there is a windows shell, such as `cmd` and `PowerShell`.
  ```js
  const trzszFilter = new TrzszFilter({
    // There is a windows shell
    isWindowsShell: true,
  });
  ```

* If running in `Node.js` and `TrzszFilter` can `require('fs')`, `chooseSendFiles` and `chooseSaveDirectory` are required. If running in web browser, they will be ignored. Note that they are `async` functions.
  ```js
  const trzszFilter = new TrzszFilter({
    // call on the user runs trz ( upload files ) on the server and no error on require('fs').
    chooseSendFiles: async (directory) => {
      // if `directory` is `true`, allow to choose multiple directories and files.
      // otherwise, only allow to choose multiple files.
      // return `undefined` if the user cancels.
      // return an array of file paths choosed by the user.
      return ["/path/to/file1", "/path/to/file2", "/path/to/directory3"];
    },
    // call on the user runs tsz ( download files ) on the server and no error on require('fs').
    chooseSaveDirectory: async () => {
      // return `undefined` if the user cancels.
      // return a directory path choosed by the user.
      return "/path/to/directory";
    },
  });
  ```

* Support dragging files or directories to upload.
  ```js
  terminalHtmlElement.addEventListener("dragover", (event) => event.preventDefault());
  terminalHtmlElement.addEventListener("drop", (event) => {
    event.preventDefault();
    trzszFilter
      .uploadFiles(event.dataTransfer.items)
      .then(() => console.log("upload success"))
      .catch((err) => console.log(err));
  });
  ```

* `TrzszAddon` is a wrapper for `TrzszFilter`. If you are using [xterm-addon-attach](https://www.npmjs.com/package/xterm-addon-attach), just replace `AttachAddon` with `TrzszAddon`.
  ```js
  import { Terminal } from 'xterm';
  import { TrzszAddon } from 'trzsz';

  const terminal = new Terminal();
  const trzszAddon = new TrzszAddon(webSocket);
  terminal.loadAddon(trzszAddon);
  ```


## Examples

* [Browser](https://github.com/trzsz/trzsz.js/blob/main/examples/browser) web shell example.

* [Electron](https://github.com/trzsz/trzsz.js/blob/main/examples/electron) terminal app example.

* [TrzszAddon](https://github.com/trzsz/trzsz.js/blob/main/examples/addon) xterm addon example.


## Screenshot

#### upload and download files in web browser

  ![browser example](https://trzsz.github.io/images/browser.gif)

#### upload and download files in electron app

  ![electron example](https://trzsz.github.io/images/electron.gif)


## Contact

  Feel free to email me <lonnywong@qq.com>.
