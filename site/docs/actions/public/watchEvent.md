---
head:
  - - meta
    - property: og:title
      content: watchEvent
  - - meta
    - name: description
      content: Watches and returns emitted Event Logs.
  - - meta
    - property: og:description
      content: Watches and returns emitted Event Logs.

---

# watchEvent

Watches and returns emitted [Event Logs](/docs/glossary/terms#event-log).

This Action will batch up all the Event Logs found within the [`pollingInterval`](#pollinginterval-optional), and invoke them via [`onLogs`](#onlogs).

`watchEvent` will attempt to create an [Event Filter](https://viem.sh/docs/actions/public/createEventFilter.html) and listen to changes to the Filter per polling interval, however, if the RPC Provider does not support Filters (ie. `eth_newFilter`), then `watchEvent` will fall back to using [`getLogs`](/docs/actions/public/getLogs) instead.

## Usage

By default, you can watch all broadcasted events to the blockchain by just passing `onLogs`.

These events will be batched up into [Event Logs](/docs/glossary/terms#event-log) and sent to `onLogs`:

::: code-group

```ts [example.ts]
import { publicClient } from './client'
import { wagmiAbi } from './abi'

const unwatch = publicClient.watchEvent({
  onLogs: logs => console.log(logs)
})
// > [{ ... }, { ... }, { ... }]
// > [{ ... }, { ... }]
// > [{ ... }, { ... }, { ... }, { ... }]
```

```ts [client.ts]
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

export const publicClient = createPublicClient({
  chain: mainnet,
  transport: http()
})
```

:::

## Scoping

You can also scope `watchEvent` to a set of given attributes (listed below).

### Address

`watchEvent` can be scoped to an **address**:

::: code-group

```ts [example.ts]
import { publicClient } from './client'
import { wagmiAbi } from './abi'

const unwatch = publicClient.watchEvent({
  address: '0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48', // [!code focus]
  onLogs: logs => console.log(logs)
})
// > [{ ... }, { ... }, { ... }]
// > [{ ... }, { ... }]
// > [{ ... }, { ... }, { ... }, { ... }]
```

```ts [client.ts]
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

export const publicClient = createPublicClient({
  chain: mainnet,
  transport: http()
})
```

:::

### Event

`watchEvent` can be scoped to an **event**.

The `event` argument takes in an event in ABI format – we have a [`parseAbiItem` utility](/docs/abi/parseAbiItem) that you can use to convert from a human-readable event signature → ABI.

::: code-group

```ts [example.ts]
import { parseAbiItem } from 'viem' // [!code focus]
import { publicClient } from './client'
import { wagmiAbi } from './abi'

const unwatch = publicClient.watchEvent({
  address: '0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48',
  event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'), // [!code focus]
  onLogs: logs => console.log(logs)
})
// > [{ ... }, { ... }, { ... }]
// > [{ ... }, { ... }]
// > [{ ... }, { ... }, { ... }, { ... }]
```

```ts [client.ts]
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

export const publicClient = createPublicClient({
  chain: mainnet,
  transport: http()
})
```

:::

By default, `event` accepts the [`AbiEvent`](/docs/glossary/types.html#abievent) type:

::: code-group

```ts [example.ts]
import { publicClient } from './client'

const unwatch = publicClient.watchEvent(publicClient, {
  address: '0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48',
  event: { // [!code focus:8]
    name: 'Transfer', 
    inputs: [
      { type: 'address', indexed: true, name: 'from' },
      { type: 'address', indexed: true, name: 'to' },
      { type: 'uint256', indexed: false, name: 'value' }
    ] 
  },
  onLogs: logs => console.log(logs)
})
```

```ts [client.ts]
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

export const publicClient = createPublicClient({
  chain: mainnet,
  transport: http()
})
```

:::

### Arguments

`watchEvent` can be scoped to given **_indexed_ arguments** on the event:

::: code-group

```ts [example.ts]
import { parseAbiItem } from 'viem'
import { publicClient } from './client'

const unwatch = publicClient.watchEvent({
  address: '0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48',
  event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'),
  args: { // [!code focus:4]
    from: '0xd8da6bf26964af9d7eed9e03e53415d37aa96045',
    to: '0xa5cc3c03994db5b0d9a5eedd10cabab0813678ac'
  },
  onLogs: logs => console.log(logs)
})
// > [{ ... }, { ... }, { ... }]
// > [{ ... }, { ... }]
// > [{ ... }, { ... }, { ... }, { ... }]
```

```ts [client.ts]
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

export const publicClient = createPublicClient({
  chain: mainnet,
  transport: http()
})
```

:::

Only indexed arguments in `event` are candidates for `args`.

These arguments can also be an array to indicate that other values can exist in the position:

```ts
const unwatch = publicClient.watchEvent({
  address: '0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48',
  event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'),
  args: { // [!code focus:8]
    // '0xd8da...' OR '0xa5cc...' OR '0xa152...'
    from: [
      '0xd8da6bf26964af9d7eed9e03e53415d37aa96045', 
      '0xa5cc3c03994db5b0d9a5eedd10cabab0813678ac',
      '0xa152f8bb749c55e9943a3a0a3111d18ee2b3f94e',
    ],
  }
  onLogs: logs => console.log(logs)
})
```

### Multiple Events

`watchEvent` can be scoped to **multiple events**:

```ts
const unwatch = publicClient.watchEvent({
  events: parseAbi([ // [!code focus:5]
    'event Approval(address indexed owner, address indexed sender, uint256 value)',
    'event Transfer(address indexed from, address indexed to, uint256 value)',
  ]),
  onLogs: logs => console.log(logs)
})
```

Note: `watchEvent` scoped to multiple events cannot be also scoped with [indexed arguments](#arguments) (`args`).

## Returns

`UnwatchFn`

A function that can be invoked to stop watching for new Event Logs.

## Parameters

### onLogs

- **Type:** `(logs: Log[]) => void`

The new Event Logs.

```ts
const unwatch = publicClient.watchEvent(
  { onLogs: logs => console.log(logs) } // [!code focus:1]
)
```

### address (optional)

- **Type:** `Address | Address[]`

The contract address or a list of addresses from which Logs should originate.

```ts
const unwatch = publicClient.watchEvent(
  { 
    address: '0xfba3912ca04dd458c843e2ee08967fc04f3579c2', // [!code focus]
    onLogs: logs => console.log(logs) 
  }
)
```

### event (optional)

- **Type:** [`AbiEvent`](/docs/glossary/types#abievent)

The event in ABI format.

A [`parseAbiItem` utility](/docs/abi/parseAbiItem) is exported from viem that converts from a human-readable event signature → ABI.

```ts
import { parseAbiItem } from 'viem' // [!code focus]

const unwatch = publicClient.watchEvent(
  { 
    address: '0xfba3912ca04dd458c843e2ee08967fc04f3579c2',
    event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'), // [!code focus]
    onLogs: logs => console.log(logs) 
  }
)
```

### args (optional)

- **Type:** Inferred.

A list of _indexed_ event arguments.

```ts
const unwatch = publicClient.watchEvent(
  { 
    address: '0xfba3912ca04dd458c843e2ee08967fc04f3579c2',
    event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'),
    args: { // [!code focus:4]
     from: '0xd8da6bf26964af9d7eed9e03e53415d37aa96045',
      to: '0xa5cc3c03994db5b0d9a5eedd10cabab0813678ac'
    }
    onLogs: logs => console.log(logs) 
  }
)
```

### batch (optional)

- **Type:** `boolean`
- **Default:** `true`

Whether or not to batch the Event Logs between polling intervals.

```ts
const unwatch = publicClient.watchEvent(
  { 
    batch: false, // [!code focus]
    onLogs: logs => console.log(logs),
  }
)
```

### onError (optional)

- **Type:** `(error: Error) => void`

Error thrown from listening for new Event Logs.

```ts
const unwatch = publicClient.watchEvent(
  { 
    onError: error => console.log(error) // [!code focus:1]
    onLogs: logs => console.log(logs),
  }
)
```

### pollingInterval (optional)

- **Type:** `number`

Polling frequency (in ms). Defaults to the Client's `pollingInterval` config.

```ts
const unwatch = publicClient.watchEvent(
  { 
    pollingInterval: 1_000, // [!code focus]
    onLogs: logs => console.log(logs),
  }
)
```

## Live Example

Check out the usage of `watchEvent` in the live [Event Logs Example](https://stackblitz.com/github/wagmi-dev/viem/tree/main/examples/filters-and-logs/event-logs) below.

<iframe frameborder="0" width="100%" height="500px" src="https://stackblitz.com/github/wagmi-dev/viem/tree/main/examples/filters-and-logs/event-logs?embed=1&file=index.ts&hideNavigation=1&hideDevTools=true&terminalHeight=0&ctl=1"></iframe>

## JSON-RPC Methods

**RPC Provider supports `eth_newFilter`:**

- Calls [`eth_newFilter`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_newfilter) to create a filter (called on initialize).
- On a polling interval, it will call [`eth_getFilterChanges`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getfilterchanges).

**RPC Provider does not support `eth_newFilter`:**

- Calls [`eth_getLogs`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getlogs) for each block between the polling interval.
