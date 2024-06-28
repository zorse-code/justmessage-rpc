# justmessage-rpc

> This fork:
> 
> - removes `JSON.stringify/parse` from the core library.
> - removes `IMessageEvent` and all other DOM assumptions.
> - renames `readMessages` to `recvMessage` for parity with `postMessage`.
> - adds `defaultPostable` so that the library can be used as-is in iframes.
> - removes `origin` from the core library due to the existence of `serviceId`s.
> 
> Install with `npm install --legacy-peer-deps`

This is a library for making RPC calls (asynchronous method calls) between browser windows or iframes. It builds upon the browser `postMessage` API, which lacks some features that complex applications may depend upon:

 - The ability to have transactional request/response calls between windows
 - Easy scoping between multiple applications; `postMessage` events are global on the window, so multiple postMessage targets must be disambiguated.
 - `postMessage` does not guarantee ordering, which can lead to surprising and difficult to diagnose bugs.

This RPC layer resolves those issues.

### Example Usage

The RPC class is symmetrical and should be created on both windows you want to talk between. Say you want to embed `iframe.html` inside the `parent.html`, and have the parent provide a function that adds things the child gives it. That might look like:

**parent.js**

```js
import { RPC } from 'justmessage-rpc';

const rpc = new RPC({
  // The window you want to talk to:
  target: myIframe.contentWindow,
  // This should be unique for each of your producer<->consumer pairs:
  serviceId: 'my-awesome-service',

  // Optionally, allowlist the origin you want to talk to:
  // origin: 'example.com',
});

rpc.expose('add', (data) => data.a + data.b);
```

**iframe.js**

```js
import { RPC } from 'justmessage-rpc';

const rpc = new RPC({
  target: window.parent,
  serviceId: 'my-awesome-service',
});

rpc.call('add', { a: 3, b: 5 }).then(result => console.log('3 + 5 is', result));
```
