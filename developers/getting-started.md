# Getting Started

The primary integration point into the UMA ecosystem is the Optimistic Oracle (OO). It is an _oracle for arbitrary off-chain data_ which leverages an interactive escalation game between requesters, proposers and disputers and is secured by economic incentives.

This getting started tutorial will show you how to go from 0 to 1 with the Optimistic Oracle by executing the simplest possible request flow. Later in the docs you can find more information on _how_ the Optimistic Oracle works and dig deeper into its mechanism and more sophisticated code examples.

### **A minimum viable Optimistic Oracle integration**

You will be working through a simple smart contract that asks the oracle the question: `Q:Did the temperature on the 25th of July 2022 in New York City exceed 90 degrees Fahrenheit? A:1e18 for yes. 0 for no.`  After submitting the request, you will propose a solution using the UMA Optimistic Oracle UI.

Once through liveness, you will fetch the price from the smart contract. This shows the full lifecycle of an Optimistic Oracle data request on the Görli testnet, interacting with an actual Optimistic Oracle contract deployment, without needing to write any code or clone any repos. It should give you the basic intuition as to how the Optimistic Oracle works without too much overhead and is a great starting point before digging deeper.  Let's get started!

### Requesting data

1. Go to [the example contract](https://remix.ethereum.org/#version=soljson-v0.8.16+commit.07a7930e.js\&optimize=false\&runs=200\&gist=6daf7b504087e447edabcf078bfef4fa) on Remix. This gives you the minimum data request and retrieval flow possible. We'll work through the code in the sections that follow.
2. Click on "gist-6daf..." to see the files in the gist, and click `OO_GettingStarted.sol` to open to Solidity file.
3. In the far left hand menu, click the link to deploy and run transactions (which looks like the Ethereum logo and a right arrow).
4. In the "Environment" dropdown, choose "Injected Provider," and connect to your wallet. **Make sure you are connected to Görli!** You don't want to deploy to a real network and spend real gas, and the Görli Optimistic Oracle address hardcoded into the contract will not work on other networks.
5. Under the "Contract" dropdown, select `OO_GettingStarted`.
6. Click "Deploy" and confirm the transaction with your wallet.
7. You should now see the `OO_GettingStarted` contract under "Deployed Contracts". Clicking the dropdown carrot will reveal buttons for each of the functions in the contract.
8. Click `requestPrice` to request the data specified in the contract's ancillary data, asking about the temperature in Manhattan on July 25th, 2022. Confirm the transaction in your wallet. This will submit a data request to the Optimistic Oracle.

### Proposing data

1. Go to the [Görli Optimistic Oracle UI](https://testnet.oracle.umaproject.org/) to see your request, which should be a `YES_OR_NO_QUERY` at the top of the index page. Click on the request to go to the details page.
2. Click "Connect Wallet" and make sure you are connected to the Görli testnet so that you can make a proposal.
3. Submit your proposal (either `1` or `0`). The actual value you submit is not super important in this case since this is just for testing purposes, but the [correct response](https://www.wunderground.com/history/daily/us/ny/williston-park/KJFK/date/2022-7-25) is `0`.

### Settling the final answer

1. After the transaction finalizes, wait 30 seconds (the challenge window) and then click `settleRequest` on the `OO_GettingStarted` contract on Remix. Confirm the transaction in your wallet. Because the proposal was not disputed within the challenge window, it can now be settled.
2. Click `getSettledPrice` to get the settled value. It should look like `int256: 0` under `getSettledPrice` if you proposed an answer of `0`. You can also see details in the console logs.
