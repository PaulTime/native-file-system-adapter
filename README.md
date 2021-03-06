# Native File System adapter (ponyfill)

> This is an in-browser file system that follows [native-file-system](https://wicg.github.io/native-file-system/) and supports storing and retrieving files from various backends.

### Adapters

This polyfill/ponyfill ships with 5 filesystem backends:

* `native`: Stores files the `Native Sandboxed` file storage
* `Sandbox`: Stores files into the Blinks `Sandboxed FileSystem` API.
* `IndexedDB`: Stores files into the browser's `IndexedDB` object database.
* `Memory`: Stores files in-memory. Thus, it is a temporary file store that clears when the user navigates away.
* `Cache storage`: Stores files in cache storage like a request/response a-like.

The api is designed in such a way that it can work with or without the ponyfill if you choose to remove or add this.<br>
It's not trying to interfear with the changing spec by using other properties that may conflict with the feature changes to the spec. A few none spec options are prefixed with a `_`

( The current minium supported browser I have choosen to support is the ones that can handle import/export )<br>
( Some parts are lazy loaded when needed )

### Using

```js
import { showOpenFilePicker, getOriginPrivateDirectory }
from 'https://cdn.jsdelivr.net/gh/jimmywarting/native-file-system-adapter/src/es6.js'

export {
  showDirectoryPicker,
  showOpenFilePicker,
  showSaveFilePicker,
  FileSystemDirectoryHandle,
  FileSystemFileHandle,
  FileSystemHandle,
  FileSystemWritableFileStream
}


// pick a file
const fileHandle = await showOpenFilePicker({
  types: [], // default
  multiple: false, // default
  excludeAcceptAllOption: false, // default
  _preferPolyfill: false // default
})

const file = await fileHandle.getFile()

// store a file
const folderHandle = await getOriginPrivateDirectory()
const fileHandle = await folderHandle.getFileHandle(file.name, { create: true })
await fileHandle.write(file)

// save/download a file
const fileHandle = await showSaveFilePicker({
  _preferPolyfill: false,
  _name: 'Untitled.png', // the name being used when preferPolyfill is true or native is unavalible
  types: {},
  excludeAcceptAllOption: false,
})

const extensionChosen = fileHandle.name.split('.').pop()

const blob = {
  jpg: generateCanvas({ type: 'blob', format: 'jpg' }),
  png: generateCanvas({ type: 'blob', format: 'png' }),
  webp: generateCanvas({ type: 'blob', format: 'webp' })
}[extensionChosen]

await blob.stream().pipeTo(fileHandle.getWriter())
```

PS: storing a file handle in IndexedDB or sharing it with postMessage isn't currently possible unless you use native.


### A note when downloading with the polyfilled version

Saving/downloading a file borrowing some of ideas from [StreamSaver.js](https://github.com/jimmywarting/StreamSaver.js).
The difference is:
 - Using service worker is optional choice with this adapter. 🤷‍
 - It dose not rely on some man-in-the-middle or external hosted service worker.
 - If you want to stream large data to the disk directly instead of taking up much RAM you need to set up a service worker yourself.<br>(note that it requires https - but again worker is optional)
 - You don't have to handle web-streams-polyfills it's lazy loaded when needed when you need that writable stream. 😴

to set up a service worker you have to basically copy [the example](https://github.com/jimmywarting/native-file-system-adapter/tree/master/example/sw.js) and register it:

```js
navigator.serviceWorker.register('sw.js')
```

Without service worker you are going to write all data to the memory and download it once it closes.

Seeking and truncating won't do anything. You should be writing all data in sequental order when using the polyfilled version.

-----

If you have chosen to `open-directory` when the polyfilled version is in use (`input[webkitdirectory]`)
than you can't get any write access to it. So unless you are using chanary with experimental flags or enabled the [Origin Trials](https://github.com/GoogleChrome/OriginTrials/blob/gh-pages/developer-guide.md) for beta testing on your origin, then you better use `save-file` instead to be safe. It's also possible to query/request permission.

### Testing

start up a server and open `/examples/test.html` in your browser.


### Resources

I recommend to follow up on this links for you to learn more about the API and how it works

- https://web.dev/native-file-system/
- https://wicg.github.io/native-file-system/
- https://github.com/wicg/native-file-system

### Alternatives
- [browser-nativefs](https://github.com/GoogleChromeLabs/browser-nativefs) by [@tomayac](https://github.com/tomayac): A similar, more like a shim (without `getSystemDirectory`).
- [StreamSaver](https://github.com/jimmywarting/StreamSaver.js) by [@jimmywarting](https://github.com/jimmywarting): A way to save large data to the disk directly with a writable stream <br><small>(same technique can be achieved if service worker are setup properly)</small>
- [FileSaver](https://github.com/eligrey/FileSaver.js) by [@eligrey](https://github.com/eligrey): One among the first libs to save blobs to the disk

### License

native-file-system-adapter is licensed under the MIT License. See `LICENSE` for details.
