# Getting Started

The [developer quickstart repo](https://github.com/UMAprotocol/dev-quickstart) has been created to enable more efficient development, testing, and contract deployment when working with UMA contracts. The below tutorial walks through setting up your environment to work with the repo and integrating your contracts with the UMA ecosystem.

#### Install dependencies

To install dependencies, you will need to install the long-term support version of nodejs, currently nodejs v14 or v16 and yarn. You can then install dependencies by running yarn with no arguments:

```
yarn
```

#### Compiling your contracts

For testing, the repo is configured to deploy core components of the UMA ecosystem. The [OptimisticDepositBox](https://github.com/UMAprotocol/dev-quickstart/blob/main/contracts/OptimisticDepositBox.sol) contract has been added as an example for testing. For reference, [here](https://github.com/UMAprotocol/protocol/tree/master/packages/core/contracts) is the full list of UMA contracts.

To add new or existing contracts for testing, add files to the [contracts](https://github.com/UMAprotocol/dev-quickstart/tree/main/contracts) folder and confirm your contracts imports are accurate.

Compile your contracts using the below command:

```
yarn hardhat compile
```

When working on changes to your contracts, this command will also recompile contracts so that changes will be implemented.

#### Testing your contracts

The [UmaEcosystem fixture](https://github.com/UMAprotocol/dev-quickstart/blob/main/test/fixtures/UmaEcosystem.Fixture.ts) deploys UMA ecosystem contracts and the [OptimisticDepositBox fixture](https://github.com/UMAprotocol/dev-quickstart/blob/main/test/fixtures/OptimisticDepositBox.Fixture.ts) deploys an OptimisticDepositBox and USDC contract which is whitelisted with the `IdentifierWhitelist` to be used as collateral. Before creating your own tests, confirm that all UMA contracts you plan to work with are included in the UmaEcosystem fixture.

Tests can be composed to validate expected functionality and running through the lifecycle of a contract without needing to consider aspects such as the liveness period for proposals or gas. To create tests, add scripts to the [test folder](https://github.com/UMAprotocol/dev-quickstart/tree/main/test) and run the command:

```
yarn test
```

Below are example unit tests for the OptimisticDepositBox `deposit` method:

```typescript
import { SignerWithAddress, expect, Contract, ethers } from "./utils";
import { umaEcosystemFixture } from "./fixtures/UmaEcosystem.Fixture";
import { optimisticDepositBoxFixture } from "./fixtures/OptimisticDepositBox.Fixture";
import { amountToSeedWallets, amountToDeposit } from "./constants";

let optimisticDepositBox: Contract, usdc: Contract, collateralWhitelist: Contract;
let deployer: SignerWithAddress, depositor: SignerWithAddress;

describe("Optimistic Deposit Box Deposit functions", function () {
  beforeEach(async function () {
    [deployer, depositor] = await ethers.getSigners();
    ({ collateralWhitelist } = await umaEcosystemFixture());
    ({ optimisticDepositBox, usdc } = await optimisticDepositBoxFixture());

    // mint some fresh tokens for the depositor.
    await usdc.connect(deployer).mint(depositor.address, amountToSeedWallets);
    
    // Approve the OptimisticDepositBox to spend tokens
    await usdc.connect(depositor).approve(optimisticDepositBox.address, amountToSeedWallets);
  });
  it("Depositing ERC20 tokens correctly pulls tokens and changes contract state", async function () {
    // Confirms usdc is whitelisted collateral.
    expect(await collateralWhitelist.connect(deployer).isOnWhitelist(usdc.address)).to.equal(true);

    expect(await optimisticDepositBox.totalOptimisticDepositBoxCollateral()).to.equal(0);

    // Deposits ERC20 tokens into the Optimistic Deposit Box contract
    await expect(optimisticDepositBox.connect(depositor).deposit(amountToDeposit))
      .to.emit(optimisticDepositBox, "Deposit")
      .withArgs(depositor.address, amountToDeposit);

    // The collateral should have transferred from depositor to contract.
    expect(await usdc.balanceOf(depositor.address)).to.equal(amountToSeedWallets.sub(amountToDeposit));
    expect(await usdc.balanceOf(optimisticDepositBox.address)).to.equal(amountToDeposit);

    // getCollateral for user should equal deposit amount.
    expect(await optimisticDepositBox.getCollateral(depositor.address)).to.equal(amountToDeposit);
    expect(await optimisticDepositBox.totalOptimisticDepositBoxCollateral()).to.equal(amountToDeposit);
  });
  it("Deposits a denominated collateral amount of 0", async function () {
    // collateral deposit amount should be above 0
    await expect(optimisticDepositBox.connect(depositor).deposit("0")).to.be.revertedWith("Invalid collateral amount");
  });
});
```

#### Deploying your contracts

If you wish to deploy your contract to a testnet for further testing, the hardhat deploy command can be used. The OptimisticDepositBox deploy command for Kovan is shown as an example below:

```
NODE_URL_42=https://kovan.infura.com/xxx yarn hardhat deploy --tags OptimisticDepositBox --network kovan
```

After deploying your contract, your contract can be verified on Etherscan using the following command:

```
ETHERSCAN_API_KEY=XXX yarn hardhat etherscan-verify --network kovan --license AGPL-3.0 --force-license --solc-input
```

You have now compiled your contracts, tested functionality within the UMA ecosystem, and deployed to a testnet. For any questions or feedback you may have, you can find us in the [UMA Discord](https://discord.com/invite/jsb9XQJ).
