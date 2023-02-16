# ⛏ oSnap Module Admin Functions

This section assumes the user has utilized the [deployment tutorial ](osnap-deployment-tutorial.md)to deploy an oSnap module and understands how to use the Zodiac app.&#x20;

In the Zodiac app, the ‘Write Contract’ tab can be used for admin changes. These functions enable the oSnap module parameters to be updated instead of requiring a new contract to be deployed when updates are needed.

As an example, the below walks through how to change the liveness period for the oSnap module. Let's assume the current liveness period is 12 hours (43,200 seconds) and the DAO decides to lower it to 6 hours.&#x20;

Once you are in the Zodiac app, click on the oSnap module you would like to interact with from the ‘Modules and Modifiers’ sidebar:

<figure><img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 5.31.28 PM.png" alt=""><figcaption></figcaption></figure>

Next, click the ‘Write Contract’ tab to make changes to the contract you have selected:

<figure><img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 5.28.04 PM.png" alt=""><figcaption></figcaption></figure>

In the ‘Write Contract’ tab, select `setLiveness` and input the new value in seconds. For 6 hours, the value `21600` would be used. The changes can be finalized by clicking the 'Add this transaction' button and going through the steps to sign and execute the transaction.

<figure><img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 5.31.58 PM.png" alt=""><figcaption></figcaption></figure>

**Admin Functions Overview**

Below are other admin functions that could be useful. This section describes the purpose of each admin function and the circumstances for calling that function.

**setCollateralAndBond**

The `collateral` and `bondAmount` parameters are set on deployment, however, these values can be updated after deployment by calling `setCollateralAndBond`. This method sets the collateral and bond amount for proposals based on the values of the following two arguments:&#x20;

* **\_collateral:** The collateral token used for bonds. This value is input as the contract address on the network the optimistic governor contract has been deployed on.&#x20;
* **\_bondAmount:** The amount of collateral that will be required for proposals. This value is scaled to the number of decimals of the collateral token. Therefore, to use a collateral value of 1 for USDC would be `1000000` since it uses 6 decimals while WETH would be `1000000000000000000` since it uses 18 decimals.

**setRules**

This method sets the rules that will be used to evaluate all future proposals. On deployment, the ‘Snapshot Space URL’, ‘Voting Quorum (%)’, and ‘Voting Period’ parameters use the following template:&#x20;

_Proposals approved on Snapshot, as verified at {_Snapshot Space URL_}, are valid as long as there is a minimum quorum of {_Voting Quorum (%)_} and a minimum voting period of {_Voting Period_} hours and it does not appear that the Snapshot voting system is being exploited. The quorum and voting period are minimum requirements for a proposal to be valid. Quorum and voting period values set for a specific proposal in Snapshot should be used if they are more strict than the rules parameter. The explanation included with the on-chain proposal must be the unique IPFS identifier for the specific Snapshot proposal that was approved or a unique identifier for a proposal in an alternative voting system approved by DAO social consensus if Snapshot is being exploited or is otherwise unavailable._

While the oSnap module enables flexibility in setting custom rules, be aware of the risks when deviating from the rules template. It is recommended to use the oSnap module with the pre-set Snapshot rules template and only use the ‘setRules’ method to update specific Snapshot parameters, like the voting quorum or voting period, if necessary.

**setLiveness**

The liveness period is the number of seconds a proposed transaction can be disputed before it becomes executable.  `setLiveness` can be called to update the liveness period for future proposals.&#x20;

The minimum recommended liveness period is 6 hours (21600 seconds).

**setIdentifier**

The default price identifier when using the Zodiac module is \[Zodiac]\([https://github.com/UMAprotocol/UMIPs/blob/master/UMIPs/umip-152.md](https://github.com/UMAprotocol/UMIPs/blob/master/UMIPs/umip-152.md)). The identifier can be changed by calling `setIdentifier` as the owner. It is not recommended to use a different identifier. Please reach out to the team if considering changes to the Zodiac identifier.

_Note, changing the identifier after a proposal is made but before it is executed will prevent the proposal from being able to be executed._

**sync**

The `sync` method updates the oSnap module with the current Optimistic Oracle contract. This prevents oSnap module contracts from needing to be redeployed when the Optimistic Oracle contract is updated. If a new Optimistic Oracle contract is added and `sync` is called between proposeTransaction and execute, the proposal will be unable to be executed.

**deleteDisputedProposal**

This method gives the ability for anyone to delete a proposal that has been disputed. After a proposal has been deleted it can be resubmitted so that the optimistic governor does not need to wait for resolution on the original proposal to propose and execute a transaction. The proposal hash is used as the argument when calling this method.

**deleteProposal**

This method gives the owner the ability to delete a proposal. This method can be used by the owner to prevent malicious proposals from being executed. The proposal hash is used as the argument when calling this method.

**deleteRejectedProposal**

This method gives the ability for anyone to delete a proposal after it has been rejected, however, a proposal is likely to be deleted before it has been rejected. The proposal hash is used as the argument when calling this method.
