# ✅ oSnap Proposal Verification

**Verifying Proposals**

After a transaction has been proposed, the liveness parameter of the oSnap module contract determines the length of time a proposal can be disputed. Proposal details can be seen in the [Optimistic Oracle dApp](https://oracle.umaproject.org/) which lists all proposals made to the Optimistic Oracle. oSnap proposals, unless manually changed, use the `ZODIAC` identifier and proposals requesting execution will show a proposed value of 1:&#x20;

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

The proposal we made in the Snapshot Tutorial section included a transfer for 0.000005 ETH. The heading of the proposal displays details such as the proposed value, the bond, and the time left in the liveness period.

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

The Input Data section includes the rules of the oSnap module. The rules give the minimum quorum and voting period for the Snapshot proposal. Any proposals that do not meet this criteria should be disputed.

The explanation includes an IPFS hash that is a unique identifier for the Snapshot proposal and can be used to retrieve details of the proposal. For example, [https://ipfs.io/ipfs/bafkreie2hujdeenwz7dcbqcvwq37vjgu6g3alvkk5ysxbm4yqpeahm37xa](https://ipfs.io/ipfs/bafkreie2hujdeenwz7dcbqcvwq37vjgu6g3alvkk5ysxbm4yqpeahm37xa).

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

**Disputing Proposal**

If the Snapshot proposal does not meet the criteria of the rules, it can be disputed. This [section](../../using-uma/disputing-oracle-data.md) outlines the steps to dispute an invalid oSnap proposal, or any other invalid assertions to UMA's Optimistic Oracle.
