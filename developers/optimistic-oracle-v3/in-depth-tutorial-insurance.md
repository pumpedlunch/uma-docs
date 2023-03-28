---
description: Using the Optimistic Oracle V3 to allow for verification of insurance claims.
---

# 👨🏫 Insurance

This section covers the [Insurance contract](https://github.com/UMAprotocol/dev-quickstart-oov3/blob/master/src/Insurance.sol), which is available in the Optimistic Oracle V3 [quick-start repo](https://github.com/UMAprotocol/dev-quickstart-oov3). This tutorial shows an example of how insurance claims can be resolved and settled through the [Optimistic Oracle V3](https://github.com/UMAprotocol/protocol/blob/master/packages/core/contracts/optimistic-oracle-v3/implementation/OptimisticOracleV3.sol) contract.

### Insurance Contract

This smart contract allows insurers to issue insurance policies by depositing the insured amount, designating the insured beneficiary, and describing the insured event.

Anyone can request payout to the insured beneficiary at any time. The Insurance contract resolves the claim through the Optimistic Oracle V3 by asserting that the insured event has occurred as of request time using the default identifier `ASSERT_TRUTH` specified in [UMIP-170](https://github.com/UMAprotocol/UMIPs/blob/master/UMIPs/umip-170.md).

If the claim is confirmed and settled through the Optimistic Oracle V3, this contract automatically pays out insurance coverage to the beneficiary. If the claim is rejected, the policy continues to be active and ready for subsequent claim attempts.

There is no limit to the number of payout requests that can be made of the same policy, however, only the first truthfully resolved request will settle the insurance payment, whereas the Optimistic Oracle V3 will settle bonds for all requests.

### Development environment and tests

#### Clone repository and Install dependencies&#x20;

Clone the UMA [Optimistic Oracle V3 quick-start repository](https://github.com/UMAprotocol/dev-quickstart-oov3) and install the dependencies. To install dependencies, you will need to install the long-term support version of nodejs and yarn. You can then install package dependencies by running `yarn` with no arguments:

<pre class="language-bash"><code class="lang-bash"><strong>git clone https://github.com/UMAprotocol/dev-quickstart-oov3.git
</strong>cd dev-quickstart-oov3
yarn
</code></pre>

This project uses [forge](https://github.com/foundry-rs/foundry/tree/master/forge) as the Ethereum testing framework. You will also need to install Foundry, refer to [Foundry installation documentation](https://book.getfoundry.sh/getting-started/installation) if you don’t have it already.

Instructions below for interacting with the contracts also assumes `bash` shell and `jq` tool is installed in order to parse transaction outputs.

### Contract implementation

The contract discussed in this tutorial can be found at `dev-quickstart-oov3/src/Insurance.sol` ([here](https://github.com/UMAprotocol/dev-quickstart-oov3/blob/master/src/Insurance.sol)) within the repo.

#### Contract creation and initialization

`_defaultCurrency` parameter in the constructor identifies the token used for settlement of insurance claims, as well as the bond currency for assertions and disputes. This token should be approved as whitelisted UMA collateral. Please check [Approved Collateral Types](../../resources/approved-collateral-types.md) for production networks or call `getWhitelist()` on the [Address Whitelist](https://github.com/UMAprotocol/protocol/blob/master/packages/core/contracts/common/implementation/AddressWhitelist.sol) contract for any of the test networks. Note that the deployed Optimistic Oracle V3 instance already has its `defaultCurrency` added to the whitelist, so it can also be used by the Insurance contract. Alternatively, you can approve a new token address with `addToWhitelist` method in the Address Whitelist contract if working in a sandboxed UMA environment.

`_optimisticOracleV3` is used to locate the address of UMA Optimistic Oracle V3. Address of `OptimisticOracleV3` contact can be fetched from the relevant [networks](https://github.com/UMAprotocol/protocol/tree/master/packages/core/networks) file, if you are on a live network, or you can provide your own contract instance if deploying UMA Oracle contracts in your own sandboxed testing environment.

<pre class="language-solidity"><code class="lang-solidity"><strong>    constructor(address _defaultCurrency, address _optimisticOracleV3) {
</strong>        defaultCurrency = IERC20(_defaultCurrency);
        oo = OptimisticOracleV3Interface(_optimisticOracleV3);
        defaultIdentifier = oo.defaultIdentifier();
    }
</code></pre>

#### Issuing insurance

`issueInsurance` method allows any insurer to deposit `insuranceAmount` of `defaultCurrency` tokens by designating an insurance beneficiary (`payoutAddress`) and defining the insured event (`insuredEvent`). Before calling this method, the insurer should have approved this contract to spend the required amount of `defaultCurrency` tokens.

```solidity
    function issueInsurance(
        uint256 insuranceAmount,
        address payoutAddress,
        bytes memory insuredEvent
    ) public returns (bytes32 policyId) ...
```

Internally, the issued policy is stored in the `policies` mapping using the calculated `policyId` key that is generated by hashing the insured event and beneficiary address.

After pulling `insuranceAmount` from the caller in the `issueInsurance` method, the contract emits a `InsuranceIssued` event including the `policyId` parameter that should be used when claiming insurance.

#### Submitting insurance claim

Anyone can submit an insurance claim on the issued policy by calling the `requestPayout` method with the relevant `policyId` parameter. This method will make an assertion with the Optimistic Oracle V3. An assertion bond is required, hence the caller should have approved this contract to spend the required minimum amount of `defaultCurrency` tokens for the proposal bond (call `getMinimumBond` method on the Optimistic Oracle V3).

```solidity
    function requestPayout(bytes32 policyId) public returns (bytes32 assertionId) ...
```

After checking that the `policyId` represents a valid insurance policy, the contract gets the current `timestamp` and composes claim that the insured event has occurred as of request time, that is passed to the Optimistic Oracle V3 when making the assertion with the `assertTruth` method:

```solidity
        assertionId = oo.assertTruth(
            abi.encodePacked(
                "Insurance contract is claiming that insurance event ",
                policies[policyId].insuredEvent,
                " had occurred as of ",
                ClaimData.toUtf8BytesUint(block.timestamp),
                "."
            ),
            msg.sender, // asserter
            address(this), // callbackRecipient (the contract)
            address(0), // No sovereign security.
            assertionLiveness,
            defaultCurrency,
            bond,
            defaultIdentifier,
            bytes32(0) // No domain.
        );
        assertedPolicies[assertionId] = policyId;
```

Optimistic Oracle V3 pulls the required bond and returns `assertionId` that is used as a key when storing the linked `policyId` in the `assertedPolicies` mapping. This information will be required when receiving a callback from the Optimistic Oracle V3.

#### Disputing insurance claim

For the sake of simplicity this contract does not implement a dispute method, but the disputer can dispute the submitted claim directly through Optimistic Oracle V3 before the liveness passes by calling its `disputeAssertion` method:

```solidity
    function disputeAssertion(bytes32 assertionId, address disputer) ...
```

The disputer should pass the `assertionId` from the request above, as well as the address for receiving back bond and rewards if the disputer was right.

If the claim is disputed, the request is escalated to the UMA DVM and it can be settled only after UMA voters have resolved it. To learn more about the DVM, see the docs section on the DVM: [how does UMA's Oracle work](../../protocol-overview/how-does-umas-oracle-work.md#umas-data-verification-mechanism).

#### Settling insurance claim

Similar to disputes, claim settlement should be initiated through the Optimistic Oracle V3 contract by calling its `settleAssertion` method with the same `assertionId` parameter:

```solidity
    function settleAssertion(bytes32 assertionId) ...
```

In case the liveness has expired or a dispute has been resolved by the UMA DVM, this call would initiate a `assertionResolvedCallback` callback in the Insurance contract:

```solidity
    function assertionResolvedCallback(bytes32 assertionId, bool assertedTruthfully) ...
```

Importantly, all callbacks should be restricted to accept calls only from the Optimistic Oracle V3 to avoid someone spoofing a resolved answer:

```solidity
        require(msg.sender == address(oo));
```

Depending on the resolved answer received in the `assertedTruthfully` callback parameter, this contract would either pay out the insured beneficiary if this was the first successful claim (in case of `true` representing that insurance claim was valid) or reject the payout:

```solidity
        if (assertedTruthfully) _settlePayout(assertionId);
...
    function _settlePayout(bytes32 assertionId) internal {
        bytes32 policyId = assertedPolicies[assertionId];
        Policy storage policy = policies[policyId];
        if (policy.settled) return;
        policy.settled = true;
        defaultCurrency.safeTransfer(policy.payoutAddress, policy.insuranceAmount);
        emit InsurancePayoutSettled(policyId, assertionId);
    }
```

### Tests and deployment

All the unit tests covering the functionality described above are available [here](https://github.com/UMAprotocol/dev-quickstart-oov3/blob/master/test/Insurance.t.sol). To execute all of them, run:

```bash
forge test --match-path test/Insurance.t.sol
```



#### Deployment

Before deploying and interacting with the contracts export the required environment variables:

* `ETHERSCAN_API_KEY`: your secret API key used for contract verification on Etherscan if deploying on a public network
* `ETH_RPC_URL`: your RPC node used to interact with the selected network
* `MNEMONIC`: your passphrase used to derive private keys of deployer (index 0) and any other addresses interacting with the contracts
* `FINDER_ADDRESS`: address of the [Finder](https://github.com/UMAprotocol/protocol/blob/master/packages/core/contracts/data-verification-mechanism/implementation/Finder.sol) contract used to locate other UMA ecosystem contracts (in order to resolve disputes you would need to use the one from a sandboxed environment)

Use `cast` command from Foundry to locate the address of Optimistic Oracle V3 and we will also use its default bonding token for settling insurance claims:

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>export OOV3_ADDRESS=$(cast call $FINDER_ADDRESS "getImplementationAddress(bytes32)(address)" $(cast --format-bytes32-string "OptimisticOracleV3"))
</strong><strong>export DEFAULT_CURRENCY_ADDRESS=$(cast call $OOV3_ADDRESS "defaultCurrency()(address)")
</strong></code></pre>

To deploy the Insurance contract, run `forge create` command.&#x20;

{% code overflow="wrap" %}
```bash
export INSURANCE_ADDRESS=$(forge create src/Insurance.sol:Insurance --mnemonic "$MNEMONIC" --json --constructor-args $DEFAULT_CURRENCY_ADDRESS $OOV3_ADDRESS | jq -r .deployedTo)
```
{% endcode %}

Finally, we can verify the deployed (if deployed to a public network) contract with `forge verify-contract`:

```
forge verify-contract --chain-id $(cast chain-id) --constructor-args $(cast abi-encode "constructor(address,address)" $DEFAULT_CURRENCY_ADDRESS $OOV3_ADDRESS) $INSURANCE_ADDRESS Insurance
```

### Interacting with deployed contract

The following section provides instructions on how to interact with the deployed contract from the foundry `cast` tool, though one can also use it for guidance for interacting through another interface (e.g. Remix or Etherscan).

#### Initial setup

Export required user addresses and their derivation indices:

{% code overflow="wrap" %}
```bash
export INSURER_ID=1
export INSURED_ID=2
export INSURER_ADDRESS=$(cast wallet address --mnemonic "$MNEMONIC" --mnemonic-index $INSURER_ID)
export INSURED_ADDRESS=$(cast wallet address --mnemonic "$MNEMONIC" --mnemonic-index $INSURED_ID)
```
{% endcode %}

Make sure the user addresses above have sufficient funding for the gas to execute the transactions.

#### Issue insurance

Assuming [TestnetERC20](https://github.com/UMAprotocol/protocol/blob/master/packages/core/contracts/common/implementation/TestnetERC20.sol) was used as `defaultCurrency` when deploying the Insurance contract, mint the required insurance amount (e.g. 10,000 TEST tokens) to the insurer and approve the Insurance contract to pull them:

{% code overflow="wrap" %}
```bash
export INSURANCE_AMOUNT=$(cast --to-wei 10000)
cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURER_ID $DEFAULT_CURRENCY_ADDRESS "allocateTo(address,uint256)" $INSURER_ADDRESS $INSURANCE_AMOUNT
cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURER_ID $DEFAULT_CURRENCY_ADDRESS "approve(address,uint256)" $INSURANCE_ADDRESS $INSURANCE_AMOUNT
```
{% endcode %}

Issue the insurance policy and grab the resulting `policyId` from the emitted `InsuranceIssued` event (it should be the last emitted event in the transaction and indexed `policyId` is at topic index `1`):

{% code overflow="wrap" %}
```bash
export ISSUE_TX=$(cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURER_ID --json $INSURANCE_ADDRESS "issueInsurance(uint256,address,bytes)" $INSURANCE_AMOUNT $INSURED_ADDRESS $(cast --from-utf8 "Bad things have happened") | jq -r .transactionHash)
export POLICY_ID=$(cast receipt --json $ISSUE_TX | jq -r .logs[-1].topics[1])
```
{% endcode %}

If in doubt on the parsing of transaction receipt you can manually view full logs from tracing:

```bash
cast run $ISSUE_TX
```

#### Submit insurance claim

Get the expected assertion bond and provide funding/approval for the insured claimant:

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash">export BOND_AMOUNT=$(cast call $OOV3_ADDRESS "getMinimumBond(address)(uint256)" $DEFAULT_CURRENCY_ADDRESS)
cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURED_ID $DEFAULT_CURRENCY_ADDRESS "allocateTo(address,uint256)" $INSURED_ADDRESS $BOND_AMOUNT
<strong>cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURED_ID $DEFAULT_CURRENCY_ADDRESS "approve(address,uint256)" $INSURANCE_ADDRESS $BOND_AMOUNT
</strong></code></pre>

Now initiate the insurance claim and grab the resulting `assertionId` from the emitted `InsurancePayoutRequested` event (it should be the last emitted event and indexed `assertionId`  is at topic index `2`):

{% code overflow="wrap" %}
```bash
export ASSERTION_TX=$(cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURED_ID --json $INSURANCE_ADDRESS "requestPayout(bytes32)" $POLICY_ID | jq -r .transactionHash)
export ASSERTION_ID=$(cast receipt --json $ASSERTION_TX | jq -r .logs[-1].topics[2])
```
{% endcode %}

If in doubt on the parsing of transaction receipt you can manually view full logs from tracing:

```bash
cast run $ASSERTION_TX
```

#### Dispute insurance claim

Before liveness passes, the insurer can dispute the claim through the Optimistic Oracle V3. First, they must fund and approve with the same bonding amount:

{% code overflow="wrap" %}
```bash
cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURER_ID  $DEFAULT_CURRENCY_ADDRESS "allocateTo(address,uint256)" $INSURER_ADDRESS $BOND_AMOUNT
cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURER_ID $DEFAULT_CURRENCY_ADDRESS "approve(address,uint256)" $OOV3_ADDRESS $BOND_AMOUNT
```
{% endcode %}

Now initiate the dispute and export related transaction hash that we will need to collect additional request parameters for resolving the dispute:

{% code overflow="wrap" %}
```bash
export DISPUTE_TX=$(cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURER_ID --json $OOV3_ADDRESS "disputeAssertion(bytes32,address)" $ASSERTION_ID $INSURER_ADDRESS | jq -r .transactionHash)
```
{% endcode %}

#### Settle insurance claim

Resolving disputes in production environment involves UMA token holders to vote on the request. Thus, testing is possible only in a sandboxed UMA ecosystem environment where the [Mock Oracle](https://github.com/UMAprotocol/protocol/blob/master/packages/core/contracts/data-verification-mechanism/test/MockOracleAncillary.sol) is used to resolve requests:

{% code overflow="wrap" %}
```bash
export MOCK_ORACLE_ADDRESS=$(cast call $FINDER_ADDRESS "getImplementationAddress(bytes32)(address)" $(cast --format-bytes32-string "Oracle"))
```
{% endcode %}

In order to resolve request Mock Oracle expects the following parameters:

* `identifier`: identifier that references instructions to resolve the request (Optimistic Oracle V3 and Insurance contract uses default identifier `ASSERT_TRUTH` specified in [UMIP-170](https://github.com/UMAprotocol/UMIPs/blob/master/UMIPs/umip-170.md).)
* `time`: timestamp when the disputed assertion was made
* `ancillaryData`: bytes encoded additional data required to resolve the request (Optimistic Oracle V3 includes `assertionId` and the address of the claiming asserter)
* `price`: numerical value to decide the outcome of the request (Optimistic Oracle V3 requires this to be `1e18` in order to resolve assertion as truthful)

The commands below export the required parameters and resolves the request at the Mock Oracle:

{% code overflow="wrap" %}
```bash
export IDENTIFIER=$(cast call $INSURANCE_ADDRESS "defaultIdentifier()(bytes32)")
export ASSERTION_TIME=$(cast block $(cast tx --json $ASSERTION_TX | jq -r .blockNumber) timestamp)
export ANCILLARY_DATA=$(cast --from-utf8 $(echo assertionId:$(echo $ASSERTION_ID | sed 's/0x//' | tr [:upper:] [:lower:]),ooAsserter:$(echo $INSURED_ADDRESS | sed 's/0x//' | tr [:upper:] [:lower:])))
export PRICE=$(cast --to-wei 1)
cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURED_ID $MOCK_ORACLE_ADDRESS "pushPrice(bytes32,uint256,bytes,int256)" $IDENTIFIER $ASSERTION_TIME $ANCILLARY_DATA $PRICE
```
{% endcode %}

Note that the syntax above for getting ancillary data involves stripping `0x` and transforming `assertionId` and `ooAsserter` to lowercase to replicate the logic how Optimistic Oracle V3 contract is composing the ancillary data. If in doubt, you can always check the emitted `PriceRequestAdded` event parameters from the Mock Oracle contract in the dispute transaction traces:

```bash
cast run $DISPUTE_TX
```

Now we can settle the request through the Optimistic Oracle V3 and observe the emitted `InsurancePayoutSettled` from our Insurance contract:

{% code overflow="wrap" %}
```bash
export SETTLE_TX=$(cast send --mnemonic "$MNEMONIC" --mnemonic-index $INSURED_ID --json $OOV3_ADDRESS "settleAssertion(bytes32)" $ASSERTION_ID | jq -r .transactionHash)
cast run $SETTLE_TX
```
{% endcode %}

The above settlement transaction should also transfer `INSURANCE_AMOUNT` tokens to the insured beneficiary as well as return the assertion bond plus half of disputer's bond to the claim initiator.

Alternatively, if `0` value was resolved in `PRICE` (representing the asserted claim was not true), the settlement transaction should only return the bond plus half of claim initiator's bond to the disputer.
