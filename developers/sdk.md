# SDK

The UMA SDK can be used to interact with UMA's Optimistic Oracle using an eventful client designed for a frontend dApp with global state management. To begin, install the SDK using the command:

```
yarn add @uma/sdk
```

**Optimistic Oracle Client Usage**

The below script is an example for getting request events from the Optimistic Oracle:

```typescript
import { ethers } from 'ethers'
import uma from '@uma/sdk'

// assume you have a url injected from env
const provider = new ethers.providers.WebSocketProvider(process.env.CUSTOM_NODE_URL)

// get the contract instance
const contractAddress:string = "0xc43767f4592df265b4a9f1a398b97ff24f38c6a6" // update with optimistic oracle address you want to connect to
const client:uma.clients.optimisticOracle.Instance = uma.clients.optimisticOracle.connect(contractAddress, provider)

// gets all events using ethers query filter api
async function events() {
    const events = await client.queryFilter({})

    // returns EventState, defined in the optimistic oracle client
    const state:uma.clients.optimisticOracle.EventState = uma.clients.optimisticOracle.getEventState(events)
    
    // see all requests given even details
    console.log(state.requests)
}
```

For more information using the SDK, go to the [sdk package](https://github.com/UMAprotocol/protocol/tree/master/packages/sdk). Other useful resources are the following projects that use the UMA SDK:

* [UMAverse](https://github.com/UMAprotocol/umaverse): a user interface for UMA contracts and integrations.
* [Optimistic Oracle dApp](https://github.com/UMAprotocol/optimistic-oracle-dapp): a user interface for interacting with the Optimistic Oracle through price requests, proposals, and disputes.
