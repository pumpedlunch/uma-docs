# Network Addresses

| Network                                        | UMA Contract Addresses                                                                                                                                                                              | Fully Permissionless     |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| [Ethereum](https://ethereum.org/)              | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/1.json), [Göerli](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/5.json)           | Yes                      |
| [Polygon](https://polygon.technology/)         | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/137.json), [Mumbai](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/80001.json)     | Yes                      |
| [Boba Network](https://boba.network/)          | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/288.json)                                                                                                      | Yes                      |
| [Optimism](https://www.optimism.io/)           | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/10.json)                                                                                                       | Yes                      |
| [Arbitrum](https://arbitrum.io/)               | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/42161.json), [Rinkeby](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/421611.json) | Yes                      |
| [Gnosis Chain](https://www.gnosischain.com/)   | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/100.json)                                                                                                      | Multi-sig for data relay |
| [Avalanche C-Chain](https://www.avax.network/) | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/43114.json)                                                                                                    | Multi-sig for data relay |
| [Evmos](https://evmos.org/)                    | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/9001.json)                                                                                                     | Multi-sig for data relay |
| [SX](https://sx.technology/)                   | [Mainnet](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/416.json)                                                                                                      | Multi-sig for data relay |
| _Deprecated_                                   |  [Kovan](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/42.json), [Rinkeby](https://github.com/UMAprotocol/protocol/blob/master/packages/core/networks/4.json)          | n/a                      |

{% hint style="info" %}
UMA's data verification mechanism (DVM), which is used to resolve disputes, is on Ethereum mainnet. Where possible, UMA uses a chain's native messaging bridge to relay dispute and governance information between that chain and Ethereum.

On chains where no such native messaging bridge exists (marked with an asterisk), UMA uses a multi-sig controlled by UMA core engineers at Risk Labs to relay disputes to the DVM, to return data from the DVM to requesters on that chain, and to execute governance actions (for instance, adding new identifiers).

UMA is researching decentralized cross-chain messaging systems to make these chains fully permissionless.
{% endhint %}
