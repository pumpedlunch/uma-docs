# 🎯 oSnap Deployment Tutorial

**Creating a Safe:**

Visit [https://app.safe.global/](https://app.safe.global/) to deploy and access your Safe. If you do not have a safe created, select the ‘Create new Safe’ button and follow [these](https://help.gnosis-safe.io/en/articles/3876461-creating-a-safe-on-a-web-browser) instructions to deploy a safe. Select ‘Add existing Safe’ if you would like to use an existing safe with your oSnap module.

_Note: Confirm your wallet is set to the appropriate network if the safe isn’t displayed._

<figure><img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 3.56.45 PM.png" alt=""><figcaption></figcaption></figure>

**Accessing the Zodiac app:**

Once connected to your safe, take the following steps to get to the Zodiac apps page:

* Step 1: Click ‘Apps’ displayed on the left sidebar.&#x20;
* Step 2: In the search bar, input ‘Zodiac’.
* Step 3: Select ‘Zodiac’ from the search results.

<figure><img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 4.00.17 PM.png" alt=""><figcaption></figcaption></figure>

**Deploying an oSnap module:**

To deploy a new oSnap module, click the option with the title ‘UMA Optimistic Governor Module’:

<figure><img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 4.21.04 PM (1).png" alt=""><figcaption></figcaption></figure>

A modal will open with deployment parameters. The first three parameters below are associated with the oSnap module and can be updated after deployment. Refer [here](../setting-custom-bond-and-liveness-parameters.md) to better understand how to appropriately set these parameters:

* **Collateral:** The ERC20 token used for the bond. The options in the Zodiac app are USDC and WETH.
* **Bond:** The bond value is the amount proposers (and disputers) are required to post in the collateral currency. The minimum recommended bond values are 1,500 USDC and 1 WETH.
* **Liveness:** The length of time a proposal can be disputed. The minimum recommended liveness period is 6 hours to allow Disputers to verify a proposal follows the rules.

The three parameters below are associated with the ‘rules’ parameter of the oSnap module and used by Disputers and Voters to resolve potential disputes.&#x20;

* **Snapshot Space URL:** The Snapshot space that is assigned to the oSnap module. To learn more about Snapshot spaces, visit the Snapshot [documentation](https://docs.snapshot.org/spaces/what-is-a-space).
* **Voting Quorum:** The minimum number of vote participation required for a valid proposal. For example, the proposal should not be considered valid if the voting quorum parameter is 5% and the proposal only had 3% voter participation.
* **Voting Period:** The minimum voting period that should be associated with a proposal. Any voting period set for a proposal that is less than the parameter set in the rules will not be considered a valid proposal.

<img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 4.22.09 PM.png" alt="" data-size="original">

After setting the deployment parameters, click ‘Add Module’. Use the ‘Simulate’ button to simulate the transaction using Tenderly. If successful, click ‘Submit’ and follow the steps to sign the transaction.

![](https://lh4.googleusercontent.com/moROUhN7GTyFBvnMMeOtFeyoe\_Ab2\_XOMLKDdQHTFeFnbe3U14eHlbGdLpt1kvz8i\_1XKu5\_NLIhJASVf8sVqz3BgwZAifHFIf2Bt2bkF\_W8AgSKD4DjCMLQssixXRTBEddgGBQJzGCKHqOkGOuJIg)

\
Once deployed, select the contract on the left sidebar of the Zodiac App. The ‘Read Contract’ tab displays the parameters of your contract, such as the collateral and required bond amount. The ‘Write Contract’ tab enables the owner to perform admin functions described in more detail in the [admin functions section](osnap-module-admin-functions.md).

<figure><img src="../../.gitbook/assets/Screen Shot 2023-02-15 at 5.35.00 PM.png" alt=""><figcaption></figcaption></figure>

**Next Steps:**

The next tutorial will demonstrate how to use the deployed oSnap module to execute Snapshot transaction data.
