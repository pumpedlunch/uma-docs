# ⚡ Snapshot Tutorial

**Configuring oSnap with Snapshot**

If you do not have a Snapshot space created, follow [these instructions](https://docs.snapshot.org/spaces/create) to get started. Otherwise, click the 'Settings' option on the sidebar for the Snapshot space you want to add your oSnap module to.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Scroll down until you get to the Plugins section and click 'Add plugin'. In the modal that opens, click the 'Gnosis SafeSnap' option.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Another modal will open that is used to configure the SafeSnap plugin that matches the below:

```
{
  "safes": [
    {
      "network": "CHAIN_ID",
      "realityAddress": "0xSWITCH_WITH_REALITY_MODULE_ADDRESS",
      "umaAddress": "0xSWITCH_WITH_UMA_MODULE_ADDRESS"
    }
  ]
}
```

To configure an oSnap module,  you need to update the `network` and `umaAddress`. For `umaAddress`, input your oSnap module contract address as `umaAddress`. The `network` parameter is the chain ID that your Safe and oSnap module contract are deployed on. You can use the table below to find the chain ID or use [https://chainlist.org/](https://chainlist.org/).

| Network Name | Chain ID |
| ------------ | -------- |
| Goerli       | 5        |
| Mainnet      | 1        |
| Polygon      | 137      |
| Optimism     | 10       |
| Arbitrum     | 42161    |
| Avalanche    | 43114    |

This configuration uses the oSnap module contract `0x0696a608DEB38ec0494165E5CDE6e0C518c63044` on Goerli:

```json
{
  "safes": [
    {
      "network": "5",
      "realityAddress": "0xSWITCH_WITH_REALITY_MODULE_ADDRESS",
      "umaAddress": "0x0696a608DEB38ec0494165E5CDE6e0C518c63044"
    }
  ]
}
```

After clicking the 'Add' button, your changes will not be made until you click 'Save' and sign the transaction.

<figure><img src="../../.gitbook/assets/image (1) (3).png" alt=""><figcaption></figcaption></figure>

**Adding Transaction Data to Proposal**

Now that the oSnap module has been configured with Snapshot, you can use the transaction builder when creating a Snapshot proposal. The example proposal below includes an ETH transfer of 0.000005.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

**Proposing Transaction**

After the voting period has ended, if the proposal passes and meets the criteria set in the rules of the oSnap module, the 0.000005 ETH transfer can be proposed. Anyone can propose the transactions by clicking the 'Request execution' button.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

After transactions have been proposed, a bond is taken from the Proposer and a challenge period starts where anyone can dispute a proposal. The Snapshot proposal will alert the users when the liveness period is complete which is when the transaction can be executed and the bond will be returned to the Proposer.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

**Executing Transaction**

After the challenge period has been completed, the Snapshot proposal gives the user the option to 'Execute transaction batch'. Signing this transaction will execute the proposal and return the bond to the proposer.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

After executing our example proposal, the below shows the 0.000005 ETH transfer being executed.

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

