# CBORPC

Concise RPC framework based on [RFC 8949 (CBOR)](https://www.rfc-editor.org/rfc/rfc8949.html).

## Server and Client

```ts
import * as CBOR from 'cborpc/codec';
import * as CBORPC from 'cborpc/server';
import { FetchTransport } from 'cborpc-transport-fetch/node';

const service = new CBORPC.Service({
  codec: new CBOR.Codec({ ...CBOR.all }),
  exports: {
    hello: name => {
      return `Hello, ${name}!`;
    },
  },
});

const handler = new FetchTransport.Handler({
  service,
});

self.addEventListener('fetch', event => {
  return event.respondWith(
    handler.handle(event.request),
  );
});
```

```ts
import * as CBOR from 'cborpc/codec';
import * as CBORPC from 'cborpc/client';
import { FetchTransport } from 'cborpc-transport-fetch/web';

const client = new CBORPC.Client({
  codec: new CBOR.Codec({ ...CBOR.all }),
});

const stub = new FetchTransport.Stub({
  client,
  fetch,
  url: new URL('http://localhost:8080/cborpc'),
});

const result = await stub.call.hello('Tim'); // <- Send CBORPC request via a proxy
console.log(result);
// output: Hello, name!
```

## Transports

```
request = [name, metadata, data]
response = [name, metadata, data]
```

CBORPC is transport neutral so it can be customized to suit the application environment.

Example:
- Fetch Transport
- WebSocket Transport
- gRPC Transport
- WebTransport
- etc

## Using IDL (CIDL)

**CIDL(Concise Interface Definition Language)** is a extension of [RFC 8610 (CDDL)](https://datatracker.ietf.org/doc/html/rfc8610)

Example:

```cddl
name = tstr
message = tstr

;export hello = (name) => message
```

Note that a CIDL definition is valid in CDDL since it is using comment for its own part.

### Details

The IDL has only two roles.
- Provides context to the compiler for static typing.
- Reduces the size of the codec module.

That's it.

Using or not using schema does not affect any protocol behavior, but only on your workflow.

The example may generates subset codec, and type definitions for the language.

```ts
import {
  tstr,
  Codec,
  type MajorTypes,
} from 'cborpc/codec';

type Types = MajorTypes & {
  name: Types['tstr'],
  message: Types['tstr'],
};

type Exports = {
  hello: (name: Types['name']) => Types['message'],
};

const name = tstr;
const message = tstr;

export const codec = new Codec<Exports>({ name, message });
```

```ts
import * as CBORPC from 'cborpc/client';
import { codec } from './__generated__/hello.mjs';

const client = new CBORPC.Client({
  codec,
});
```

The subset can be further optimized (e.g. DCE) via bundlers.
