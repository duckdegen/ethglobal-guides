---
id: 4269a
slug: introduction-to-connext
description: Intro to Connext protocol
---


<Section name="1. What is Connext?" description="What is Connext?">

# What is Connext?

Connext is a decentralized interoperability protocol to empower developers to build fully-expressive, secure cross-domain apps.
With its toolkit, Connext powers fast, secure, trust minimized bridging between blockchains and rollups.

## What is a Bridge?

A bridge (or interoperability network) is a system that relays assets and/or data between blockchains.

## What does the Connext flow look like?

![Connext Diagram](../assets/Connext_quick_overview.png "Title")

In this diagram we can observe two domains, "origin" and "destination", and two paths, "fast" and "slow".

In this case, let’s assume the intention of the user is to bridge tokens from the origin domain to the destination domain, and have access to fast liquidity.
Unlike bridging through an AMB alone, which often comes with several hours of latency for the user, Connext shortcuts this by allowing its routers to front you the capital, thereby taking on the liquidity risk for you while they wait for the tokens to move through the slow path.

It's important to note that if the user is bridging data, or if the call needs to be authenticated by the caller, the bridging operation will always go through the slow path.

## What can you build with Connext?

Some example use cases:

- Execute the outcome of **DAO votes** across chains
- Lock-and-mint or burn-and-mint **token bridging**
- **Aggregate DEX liquidity** across chains in a single seamless transaction
- Crosschain **vault zaps** and **vault strategy management**
- **Lend funds** on one chain and borrow on another
- Bringing **UniV3 TWAPs** to every chain without introducing oracles
- **NFT bridging** and chain-agnostic NFT marketplaces
- **Store data on Arweave/Filecoin** directly from within an Ethereum smart contract

## What's the architecture of the protocol?

The Connext architecture can be seen as a layered system, as follows:

| Layer                   | Protocol/Stakeholders             |
| ----------------------- | --------------------------------- |
| `Application Layer`     | `Crosschain Applications (xApps)` |
| `Liquidity Layer`       | `NXTP`                            |
| `Messaging Layer`       | `AMBs, etc`                       |
| `Transport Layer`       | `Connext Routers`                 |


Let's take a closer look at some of these:

1. xApps are applications built by developers like you!
2. NXTP is a *liquidity* layer and a *developer interface* on top of an optimisitic briding protocol.
3. Our messaging layer is currently based off of multiple AMBs. We plan to migrate to an optimistic bridge mechanism in the near future.
4. Routers watch AMB messages and “short-circuit” them by immediately executing calls/providing liquidity on the destination chain. **This removes the AMB 4 hour minute latency in all user-facing cases.**

A simple mental model is to think of Connext as the liquidity layer to an AMB messaging layer. Connext and AMBs, as well as optimistic bridges soon, together provide the two halves of the “ideal” solution for interoperability.

## How do tokens reach the end-user on the destination chain?

1. AMMs on each chain enable swapping from the chain’s asset to bridge assets → this ensures users give and receive the assets the expect to on each chain.
2. Connext executes AMB interactions on the destination chain and provides a consolidated interface for devs that abstracts away the complexities of building against AMBs directly.

## How do I interact with Connext?

Developers can call `xCall`, and the protocol will then splits actions between a Liquidity Layer (Connext) and a Messaging Layer.
You can interact with xCall in ways described in the `Send your first xCall` section of this document

## How do I interact with the Connext community?

Find our discord at https://discord.gg/pef9AyEhNz and join the `dev-hub` channel

</Section>

<Section name="2. Trust Assumptions" description="Trust assumptions">

# Trust assumptions

## What is a Bridge?

A bridge (or interoperability network) is a system that relays assets and/or data between blockchains.

## The Bridging Landscape

One of the key challenges of blockchains & distributed systems is that they **always have tradeoffs.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99f16e25-731f-461e-8787-57f17d293fd4/Untitled.png)

For bridges, historically this has meant that bridges can only have two of the following three properties:

1. **Trust-minimization:** The system does not introduce new trust/security assumptions beyond those of the underlying chains.
2. **Generalizeability:** The system supports passing around arbitrary data and not just funds.
3. **Extensibility:** The system can be deployed without lots of custom work for a variety of different types of underlying chains.

We can classify all bridges by their properties/tradeoffs:

💡 Learn more: [The Interoperability Trilemma](https://blog.connext.network/the-interoperability-trilemma-657c2cf69f17)

## Optimistic Bridges

Optimistic bridges create another tradeoff, **latency.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cf1cb72f-0d8f-4cd4-aec5-e70b77059616/Untitled.png)

Optimistic bridges, similar to optimistic rollups, use **fraud proofs** to ensure the validity of data relayed across chains. Every message that passes through an optimistic bridge remains in a “pending” state during the dispute window until it is considered valid. During this time, **watchers** can dispute the message if the data is incorrect.

💡 Learn more: [Optimistic Bridges](https://blog.connext.network/optimistic-bridges-fb800dc7b0e0)

</Section>

<Section name="3. Security" description="Breaking Down Bridge Security Models">

# Breaking Down Bridge Security Models
## Types of security

- Economic ->  trust minimization / custody model of the bridge 
- Implementation ->  defensive building and planning
- Environmental -> 51% attacks on the underlying chains

## Types of bridges

- Locally verified (atomic swaps & fast liquidity systems)
- Externally verified (multisig, mpc, threshold, PoS, and validator bridges)
- Natively verified (light client header relays, rollup bridges)

In each case, the verification mechanism led to a tradeoff of at least ONE of three highly desireable properties:

- Trust-minimization: adding no economic security assumptions beyond those of the underlying chain.
- Generalizeability: supporting passing arbitrary data across chains.
- Extensibility: deployable to many heterogenous chains with minimal custom work.

## Economic security at Connext

We can define economic security as how much it would cost to currupt the system.

Connext uses AMB Bridges to pass messages around. AMBs, often called canonical bridges, are the current standard to bring tokens/data from one chain to a L2 or another L1. Examples are the Arbitrum-Ethereum bridge (totally trustless), or the Polygon PoS-Ethereum bridge (that incurs in some trust assumptions).  Using AMBs, the canonical bridges to move tokens to a chain, means we rely on the same security and trust minimization assumptions that the receiving chains do.

As we are using the native AMBs, there will be no added trust to the messaging of these systems in most cases. In a few, however, there are concessions:

- *Optimistic rollup delay bypassing*. Optimistic rollups rely on a delay window to ensure there is sufficient time to prove fraud on the rollup; this delay is parameterized per-system but is usually around 1 week. Instead of waiting for the duration of the delay, our contracts may use a shorter timeout in favor of capital efficiency. Inclusion of the message in the committed root will still be required, but there is a chance the rollup could rollback in the event of fraud. To address this, routers are encouraged to validate the underlying rollup state before bidding to ensure no fraud was committed.
- *Handling non-AMB connected domains*. In some cases, there is not a suitable AMB from mainnet to the given domain. This means a separate message-passing layer must be used, which will introduce the trust assumptions of that protocol. This is the case for the Binance Smart Chain, for example.

We will migrate our messaging layer to an optimistic brige design in the near future. Optimistic bridge designs rely on a set of watchers to watch the chain and report fraud.
The watcher + fraud proof pattern uses a single honest verifier assumption. In other words, optimistic bridges only require 1 of n parties in the system to correctly verify updates. The economical cost to attack the network is thus to corrupt every single verifier in the system.

## Implementation security at Connext

We can define implementation security as the complexity of implementing a system, and what our team security hygiene look like.

Implementation security is much riskier in bridges than in other applications because of contagion.

Our principles on implementation are as follows:

- General secure development: guarded launch, bounties, audits, and other security reviews (formal verification, fuzzing, etc.)
- Rate limits: bridges should automatically be halting or slowing down transactions if there is a large outflow of funds happening in a short period of time
- Time is on your side: delay + watcher model means you check offchain no fraud has occurred and taking on the risk. You can also do equivalency checks across chains, which isn't possible in a synchronous bridging pattern
- Reducing implementation scope and complexity — KISS
    1. Code that you run doesn't run the same on each chain
    2. Exists in light clients, ZK bridge, etc.
    3. Key takeaway — dependencies trickle up! if you're building in an already heterogenous environment, you have to make sure your implementation is as simple as possible

## Environmental security at Connext

We can define environmental securit as the ability for a bridge to withstand attacks on the underlying domains.

- Because we are able to disconnect messages from specific domains, we can maintain the security of high information integrity zones (Ethereum; not susceptible to 51% attacks) while supporting zones with growing information integrity security (new domains; smaller validator set; more susceptible to 51% attacks)
- Watchers help us maintain the security of ^^
- Using the same settlement layer is better because you're sharing economic security
- Introducing a delay can combat this effectively because you can check this easily offchain and halt the contagion

</Section>

<Section name="4. Supported chains" description="Supported Chains">

# Supported Chains

The Connext testnet contains various pieces of infrastructure and smart contracts.

## Deployed Contract Addresses

A full reference to deployed contracts can be found in the [deployments.json](https://github.com/connext/nxtp/blob/main/packages/deployments/contracts/deployments.json) file.

## Domain IDs

Domain IDs for testnet can be found below:

<details>

  <summary>Production Testnet Contracts</summary>

  ### Goerli

  <table>
    <tr>
      <th>Chain ID</th>
      <th>Domain ID</th>
    </tr>
    <tr>
      <td>5</td>
      <td>1735353714</td>
    </tr>
  </table>

  <table>
    <tbody>
      <tr>
        <th>Contract</th>
        <th>Address</th>
      </tr>
      <tr>
        <td>Test Token (TEST ERC20)</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1">
            0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1
          </a>
        </td>
      </tr>
      <tr>
        <td>Wrapped Ether ERC20 (WETH) [canonical]</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0xB4FBF271143F4FBf7B91A5ded31805e42b2208d6">
            0xB4FBF271143F4FBf7B91A5ded31805e42b2208d6
          </a>
        </td>
      </tr>
      <tr>
        <td>ConnextHandler</td>
        <td>
          <a href="https://louper.dev/diamond/0xB4C1340434920d70aD774309C75f9a4B679d801e?network=goerli">
            0xB4C1340434920d70aD774309C75f9a4B679d801e
          </a>
        </td>
      </tr>
      <tr>
        <td>TokenRegistry</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0x3f95CEF37566D0B101b8F9349586757c5D1F2504">
            0x3f95CEF37566D0B101b8F9349586757c5D1F2504
          </a>
        </td>
      </tr>
      <tr>
        <td>PromiseRouter</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0xD25575eD38fa0F168c9Ba4E61d887B6b3433F350">
            0xD25575eD38fa0F168c9Ba4E61d887B6b3433F350
          </a>
        </td>
      </tr>
    </tbody>
  </table>

  ### Optimism-Goerli

  <table>
    <tr>
      <th>Chain ID</th>
      <th>Domain ID</th>
    </tr>
    <tr>
      <td>420</td>
      <td>1735356532</td>
    </tr>
  </table>
  
  <table>
    <tbody>
      <tr>
        <th>Contract</th>
        <th>Address</th>
      </tr>
      <tr>
        <td>Test Token (TEST ERC20)</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF">
            0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF
          </a>
        </td>
      </tr>
      <tr>
        <td>Wrapped Ether ERC20 (madWETH) [representation]</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x39B061B7e41DE8B721f9aEcEB6b3f17ECB7ba63E">
            0x39B061B7e41DE8B721f9aEcEB6b3f17ECB7ba63E
          </a>
        </td>
      </tr>
      <tr>
        <td>Wrapped Ether ERC20 (WETH) [adopted]</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x4E283927E35b7118eA546Ef58Ea60bfF59E857DB">
            0x4E283927E35b7118eA546Ef58Ea60bfF59E857DB
          </a>
        </td>
      </tr>
      <tr>
        <td>ConnextHandler</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0xe37f1f55eab648dA87047A03CB03DeE3d3fe7eC7">
            0xe37f1f55eab648dA87047A03CB03DeE3d3fe7eC7
          </a>
        </td>
      </tr>
      <tr>
        <td>TokenRegistry</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x67fE7B3a2f14c6AC690329D433578eEFE59954C8">
            0x67fE7B3a2f14c6AC690329D433578eEFE59954C8
          </a>
        </td>
      </tr>
      <tr>
        <td>PromiseRouter</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x7aA60f0D8E234EdCbcB119d0e569376E93431Ee2">
            0x7aA60f0D8E234EdCbcB119d0e569376E93431Ee2
          </a>
        </td>
      </tr>
    </tbody>
  </table>

</details>

<details>

  <summary>Staging Testnet Contracts</summary>

  ### Goerli

  <table>
    <tr>
      <th>Chain ID</th>
      <th>Domain ID</th>
    </tr>
    <tr>
      <td>5</td>
      <td>1735353714</td>
    </tr>
  </table>

  <table>
    <tbody>
      <tr>
        <th>Contract</th>
        <th>Address</th>
      </tr>
      <tr>
        <td>Test Token (TEST ERC20)</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1">
            0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1
          </a>
        </td>
      </tr>
      <tr>
        <td>Wrapped Ether ERC20 (WETH) [canonical]</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0xB4FBF271143F4FBf7B91A5ded31805e42b2208d6">
            0xB4FBF271143F4FBf7B91A5ded31805e42b2208d6
          </a>
        </td>
      </tr>
      <tr>
        <td>ConnextHandler</td>
        <td>
          <a href="https://louper.dev/diamond/0x8664bE4C5C12c718838b5dCd8748B66F3A0f6A18?network=goerli">
            0x8664bE4C5C12c718838b5dCd8748B66F3A0f6A18
          </a>
        </td>
      </tr>
      <tr>
        <td>TokenRegistry</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0x458a2AE80fbe7e043ec18b62515423e63Ee5cBed">
            0x458a2AE80fbe7e043ec18b62515423e63Ee5cBed
          </a>
        </td>
      </tr>
      <tr>
        <td>PromiseRouter</td>
        <td>
          <a href="https://goerli.etherscan.io/address/0x3E3d48C7636A446C59423C95A89F1dE40f3a1F22">
            0x3E3d48C7636A446C59423C95A89F1dE40f3a1F22
          </a>
        </td>
      </tr>
    </tbody>
  </table>

  ### Optimism-Goerli 

  <table>
    <tr>
      <th>Chain ID</th>
      <th>Domain ID</th>
    </tr>
    <tr>
      <td>420</td>
      <td>1735356532</td>
    </tr>
  </table>

  <table>
    <tbody>
      <tr>
        <th>Contract</th>
        <th>Address</th>
      </tr>
      <tr>
        <td>Test Token (TEST ERC20)</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF">
            0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF
          </a>
        </td>
      </tr>
      <tr>
        <td>Wrapped Ether ERC20 (madWETH) [representation]</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x39B061B7e41DE8B721f9aEcEB6b3f17ECB7ba63E">
            0x39B061B7e41DE8B721f9aEcEB6b3f17ECB7ba63E
          </a>
        </td>
      </tr>
      <tr>
        <td>Wrapped Ether ERC20 (WETH) [adopted]</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x4E283927E35b7118eA546Ef58Ea60bfF59E857DB">
            0x4E283927E35b7118eA546Ef58Ea60bfF59E857DB
          </a>
        </td>
      </tr>
      <tr>
        <td>ConnextHandler</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0xB7CF5324641bD9F82903504c56c9DE2193B4822F">
            0xB7CF5324641bD9F82903504c56c9DE2193B4822F
          </a>
        </td>
      </tr>
      <tr>
        <td>TokenRegistry</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0x35d3a7C14de030dC9a1375009620c99369827a5E">
            0x35d3a7C14de030dC9a1375009620c99369827a5E
          </a>
        </td>
      </tr>
      <tr>
        <td>PromiseRouter</td>
        <td>
          <a href="https://blockscout.com/optimism/goerli/address/0xdd247dc5C3f446825FB00eA5bA074B6BAE8E2cae">
            0xdd247dc5C3f446825FB00eA5bA074B6BAE8E2cae
          </a>
        </td>
      </tr>
    </tbody>
  </table>

</details>

## Test Token (TEST)

The Test ERC20 Token has an open mint function with the signature `mint(address account, uint256 amount)`. These test ERC20 tokens can be freely minted by anyone and they are collateralized by routers on the test network to enable swaps between them on the different chains.

Please ping the team to request for more testnet assets and swaps added!

## Sequencer

URL: `https://sequencer.testnet.connext.ninja`

## Testnet Bridge

Note this new Bridge UI is still under development. Here you can mint TEST tokens via the faucet and also send tokens. 

URL: `https://amarok-testnet.coinhippo.io/`

## Testnet Connextscan

This is the testnet scanner site where you can track the status of transfers by `transferId`. 

URL: `https://testnet.amarok.connextscan.io/`


</Section>

<Section name="Developer Resources" description="Developer Resources">

# Developer Resources

- Check out our [hacker kit](https://www.notion.so/connext/Connext-Hacker-Kit-febb25e70b6c4596a57a1e05b6df4b19) that we put together for hackathon participants
- Our [technical documentation](https://docs.connext.network) is a great starting point to read more into technical details
- Our [Connext Academy](https://connext.academy) is a community driven information hub to read more coverage
- Join our [Discord](https://discord.gg/pef9AyEhNz) to ask the team for help or to meet the community

## Existing implementations

- With the [SDK Quickstart](https://docs.connext.network/developers/sdk/sdk-quickstart)
- With the [Solidity Quickstart](https://docs.connext.network/developers/contracts/contracts-quickstart)
- Using any language with a local [http-server](https://github.com/connext/nxtp/tree/main/packages/examples/sdk-server)
- A [bridge frontend implementation](https://github.com/connext/nxtp/tree/main/packages/examples/bridge-reference/) shows you how to quickly build a minimal bridge UI

</Section>
<Section name="Glossary of Terms" description="Glossary of Terms">

# Glossary of Terms

- `Domain Id`: Domain Ids are unique identifiers for each chain. Similar to chain ids, but applicable to chains that don't have that concept (non-evm chains)! They refer to the Domain IDs which are a custom mapping of ID to specific execution environment (not always equivalent to "chain", hence we have Domain IDs). The Domain ID is a hashed value of the chain name.
- `Origin`: Sending chain/domain
- `Destination`: Receiving chain/domain
- `Source`: Contract that an interaction starts from (and calls into Connext)
- `Target`: Contract that an interaction happens to (is called from Connext)
- `Canonical Asset`:  The token has been issued on its specified home chain (as opposed to being a wrapped token). The asset that is locked on the bridge to mint on another chain.
- `Representation Asset`: Token originally deployed on some other chain (a synthetic IOU issued by the protocol). This is the asset that you receive from the bridge. It is a pointer that represents some locked funds for this asset on the canonical domain (i.e. mad-USDC).
- `Local Asset`: A token originally deployed on the local chain. The asset that the bridge will deliver to you - this will be the canonical asset on the canonical domain, otherwise it’s the representation.
- `Adopted Asset`: The asset that has traction (user adoption) on the domain (i.e. POS-USDC on polygon).
- `Transacting Asset` The asset the user sends into `xcall` or gets from `execute`, depending on the context. Can be the local or adopted asset.

</Section>
<Section name="1. Frontend Integrations" description="Frontend Integrations">

# Frontend Integrations

## SDK Quickstart

The Connext SDK allows developers to interact with the Connext protocol in standard Node.js or web environments. This quickstart will go through how to build on top of Connext using the TypeScript SDK. 

These examples (and others) can be found in our xApp Starter Kit, under `src/sdk-interactions`.

[xApp Starter Kit](https://github.com/connext/xapp-starter/)

## Cross-Chain Transfer

In this quickstart, we'll demonstrate how to execute an `xcall` to transfer funds from a wallet on Goerli to a destination address on Optimism-Goerli.

### 1. Setup project

If you have an existing project, you can skip to [Install dependencies](./sdk-quickstart#2-install-dependencies).

Create the project folder and initialize the package.

```bash
mkdir node-examples && cd node-examples
yarn init
```

We'll use TypeScript / Node.js in this example.

```bash
yarn add @types/node typescript 
yarn add -D @types/chai
yarn tsc --init
```

We want to use top-level await so we'll set the compiler options accordingly.

```json title="tsconfig.json"
{
  "compilerOptions": {
    "outDir": "./dist",
    "target": "es2017",
    "module": "esnext",
    "moduleResolution": "node"
  },
  "exclude": ["node_modules"]
}
```

And add the following to `package.json`:

```json title="package.json"
"type": "module",
"scripts": {
  "build": "tsc && node dist/xtransfer.js",
  "xtransfer": "node dist/xtransfer.js"
}
```

Create `xtransfer.ts` in project directory, where we will write all the code in this example.

```bash
mkdir src && touch src/xtransfer.ts
```

### 2. Install dependencies

Install the SDK. Use the latest `0.2.0-beta.*` version for Amarok (see versions [here](https://www.npmjs.com/package/@connext/nxtp-sdk))

```bash
yarn add @connext/nxtp-sdk
```

Also, install `ethers`.

```bash
yarn add ethers
```

### 3. Pull in imports

We only need a few imports for this example.

```ts title="src/xtransfer.ts"
import { create, NxtpSdkConfig } from "@connext/nxtp-sdk";
import { ethers } from "ethers";
```

The rest of this guide will be working with this file.

### 4. Create a Signer

Use a wallet (i.e. MetaMask) as a [Signer](https://docs.ethers.io/v5/api/signer/).

```ts
const privateKey = "<PRIVATE_KEY>";
let signer = new ethers.Wallet(privateKey);
```

And connect it to a [Provider](https://docs.ethers.io/v5/api/providers/) on the sending chain ([Infura](https://infura.io/), [Alchemy](https://www.alchemy.com/), etc).

```ts
const provider = new ethers.providers.JsonRpcProvider("<GOERLI_RPC_URL>");
signer = signer.connect(provider);
const signerAddress = await signer.getAddress();
```

### 5. Construct the `NxtpSdkConfig`

Fill in the placeholders with the appropriate provider URLs.

```ts
const nxtpConfig: NxtpSdkConfig = {
  logLevel: "info",
  signerAddress: signerAddress,
  chains: {
    "1735353714": {
      providers: ["<GOERLI_RPC_URL>"],
      assets: [
        {
          name: "TEST",
          symbol: "TEST",
          address: "0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1",
        },
      ],
    },
    "1735356532": {
      providers: ["<OPTIMISM_GOERLI_RPC_URL>"],
      assets: [
        {
          name: "TEST",
          symbol: "TEST",
          address: "0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF",
        },
      ],
    },
  },
};
```

> Not sure where those IDs came from? They refer to the Domain IDs which are a custom mapping of ID to specific execution environment (not always equivalent to "chain", hence we have Domain IDs). The chain id is a hashed value of the chain name.

### 6. Create the SDK

Simply call `create()` with the config from above. We'll also fetch the signer address and set the amount to send.

```ts
const {nxtpSdkBase} = await create(nxtpConfig);

const signerAddress = await signer.getAddress();

const amount = 1000000000000000000; // amount to send (1 TEST)
```

### 7. Construct the `xCallArgs`

Now, we construct the arguments that will be passed into the `xcall`.

```ts
const callParams = {
  to: signerAddress, // the address that should receive the funds
  callData: "0x", // empty calldata for a simple transfer
  originDomain: "1735353714", // send from Goerli
  destinationDomain: "1735356532", // to Optimism-Goerli
  agent: signerAddress, // address allowed to execute transaction on destination side in addition to relayers
  recovery: signerAddress, // fallback address to send funds to if execution fails on destination side
  forceSlow: false, // option to force slow path instead of paying 0.05% fee on fast liquidity transfers
  receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
  callback: ethers.constants.AddressZero, // zero address because we don't expect a callback
  callbackFee: "0", // fee paid to relayers; relayers don't take any fees on testnet
  relayerFee: "0", // fee paid to relayers; relayers don't take any fees on testnet
  destinationMinOut: (amount * 0.97).toString(), // the minimum amount that the user will accept due to slippage from the StableSwap pool (3% here)
};

const xCallArgs = {
  params: callParams,
  transactingAsset: "0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1", // the Goerli Test Token
  transactingAmount: amount.toString(), 
  originMinOut: (amount * 0.97).toString() // the minimum amount that the user will accept due to slippage from the StableSwap pool (3% here)
};
```

A detailed reference of all the `xcall` arguments can be found [here](../xcall-params.md).

### 8. Approve the asset transfer

This is necessary because funds will first be sent to the ConnextHandler contract before being bridged.

`approveIfNeeded()` is a helper function that finds the right contract address and executes the standard "increase allowance" flow for an asset.

```ts
const approveTxReq = await nxtpSdkBase.approveIfNeeded(
  xCallArgs.params.originDomain,
  xCallArgs.transactingAsset,
  xCallArgs.transactingAmount
)
const approveTxReceipt = await signer.sendTransaction(approveTxReq);
await approveTxReceipt.wait();
```

### 9. Send it!

Send the `xcall`.

```ts
const xcallTxReq = await nxtpSdkBase.xcall(xCallArgs);
xcallTxReq.gasLimit = ethers.BigNumber.from("20000000"); 
const xcallTxReceipt = await signer.sendTransaction(xcallTxReq);
console.log(xcallTxReceipt);
const xcallResult = await xcallTxReceipt.wait();
```

Finally, run the following to fire off the cross-chain transfer!

```shell
yarn xtransfer
```

### 10. Track the `xcall`

We can use the transaction hash from the transaction receipt we logged above to track the status of the `xcall`, following instructions here.

[Tracking an xcall](../xcall-status)

After the DestinationTransfer shows up on the Optimism-Goerli side, the freshly transferred tokens should show up in the destination wallet.

--- 

## Cross-Chain Mint (unauthenticated)

We can also send arbitrary `calldata`, along with the `xcall`, to be executed on the destination domain.

In this example, we're going to construct some `calldata` targeting an existing contract function to avoid having to deploy a new contract. We'll aim for the `mint` function of the [Test ERC20 Token (TEST) contract](https://blockscout.com/optimism/goerli/address/0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF/write-contract) to demonstrate this. 

> Minting usually requires authentication but the Test Token has a public `mint` function (callable by anyone!) that we can leverage for this example. Hence, this is an "unauthenticated" `xcall` with calldata - nothing extra needs to be done on the destination side.

### 7. Encode the `calldata`

After creating the SDK (steps 1-6 above), we have to create and encode the `calldata`.

To do this, we'll just grab the Test Token contract's ABI (we only care about the `mint` function here) and encode the `calldata` with the correct arguments.

```js
const contractABI = [
  "function mint(address account, uint256 amount)"
];
const iface = new ethers.utils.Interface(contractABI);

const calldata = iface.encodeFunctionData(
  "mint", 
  [
    await signer.getAddress(), // the address that should receive the minted funds
    ethers.BigNumber.from("100000000000000000000") // amount to mint (100 TEST)
  ]
)
```

### 8. Construct the `xCallArgs`

Now with the `calldata` ready, we supply it to the `xCallArgs`.

```js
const callParams = {
  to: "0x68Db1c8d85C09d546097C65ec7DCBFF4D6497CbF", // Opt-Goerli Test Token contract - the target
  callData: calldata, 
  originDomain: "1735353714", // send from Goerli
  destinationDomain: "1735356532", // to Optimism-Goerli
  agent: signerAddress, // address allowed to transaction on destination side in addition to relayers
  recovery: await signer.getAddress(), // fallback address to send funds to if execution fails on destination side
  forceSlow: false, // option to force AMB slow path (~30 mins) instead of paying 0.05% fee
  receiveLocal: false, // option to receive the local AMB-flavored asset instead of the adopted asset
  callback: ethers.constants.AddressZero, // no callback so use the zero address
  callbackFee: "0", // fee paid to relayers for the callback; no fees on testnet
  relayerFee: "0", // fee paid to relayers for the forward call; no fees on testnet
  destinationMinOut: "0", // not sending funds so minimum can be 0
};

const xCallArgs = {
  params: callParams,
  transactingAsset: "0x7ea6eA49B0b0Ae9c5db7907d139D9Cd3439862a1", // not sending funds so use any registered asset
  transactingAmount: "0", // not sending funds with this calldata-only xcall
  originMinOut: "0" // not sending funds so minimum can be 0
};
```

### 9. Send it!

Notice that we specified the zero address for `transactingAssetId` and `amount: "0"` above since we're not sending any funds with this `xcall`. Therefore, we can skip the approval dance and just send the transaction.

```ts title="*same code*"
const xcallTxReq = await nxtpSdkBase.xcall(xCallArgs);
xcallTxReq.gasLimit = ethers.BigNumber.from("20000000"); 
const xcallTxReceipt = await signer.sendTransaction(xcallTxReq);
console.log(xcallTxReceipt);
const xcallResult = await xcallTxReceipt.wait();
```

Add a new script to `package.json`:

```json title="package.json"
"scripts": {
  "xmint": "tsc && node dist/xmint.js"
}
```

Finally, run the following to fire off the cross-chain mint!

```shell
yarn xmint
```
</Section>
<Section name="2. Onchain Integrations" description="Onchain Integrations">

# Onchain Integrations

# Contracts Quickstart

This quickstart will go through how to write smart contracts in Solidity that interact with Connext's deployed contracts.

These examples (and others) can be found in our xApp Starter Kit, under `src/contract-to-contract-interactions`.

[xApp Starter Kit](https://github.com/connext/xapp-starter/)

---

## Introduction

An `xcall` can be initiated from a smart contract to send funds and/or conduct arbitrary execution _across domains_. This allows Connext to be used as a base cross-chain layer that can be integrated into dApps, turning them into **xApps**.

For example, here are interesting use cases that the protocol enables:

- Hold a governance vote on one chain and execute the outcome of it on another (plus other DAO operations)
- Lock-and-mint or burn-and-mint token bridging
- Perform a swap and transfer the received tokens across chains
- Connecting DEX liquidity across chains in a single seamless transaction
- Crosschain vault zaps and vault strategy management
- Critical protocol operations such as replicating/syncing global constants (e.g. PCV) across chains
- Bringing UniV3 TWAPs to every chain without introducing oracles
- Chain-agnostic veToken governance
- Metaverse-to-metaverse interoperability

---

## Transfer

Let's start with a simple example of sending funds across domains. We can write a smart contract to demonstrate the interaction with Connext's `xcall`.

### Contract

First, we need to import the `IConnextHandler` interface along with `CallParams` and `XCallArgs`. We'll also use solmate's `ERC20` implementation to handle the token we intend to transfer.

```solidity title="Transfer.sol"
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {CallParams, XCallArgs} from "nxtp/core/connext/libraries/LibConnextStorage.sol";
import {ERC20} from "@solmate/tokens/ERC20.sol";
```

The contract will take the address of a deployed `ConnextHandler.sol` as a constructor argument.

```solidity
contract Transfer {
  IConnextHandler public immutable connext;

  constructor(IConnextHandler _connext) {
    connext = _connext;
  }
```

The `transfer` function will take some arguments to use in the `xcall`.

```solidity
  function transfer(
    address to, // the destination address (e.g. a wallet)
    address asset, // address of token on source domain 
    uint32 originDomain, // e.g. from Goerli (1735353714)
    uint32 destinationDomain, // to Optimism-Goerli (1735356532)
    uint256 amount // amount to transfer
  ) external {
```

Before interacting with this contract, the user must approve the amount to send. The `require` clause checks for this. The tokens will be transferred to this contract and then this contract itself must approve a transfer to `ConnextHandler.sol`.

So tokens will be sent from `User's wallet` -> `Transfer.sol` -> `ConnextHandler.sol` -> `to`.

```solidity
    ERC20 token = ERC20(asset);
    require(
      token.allowance(msg.sender, address(this)) >= amount,
      "User must approve amount"
    );

    token.transferFrom(msg.sender, address(this), amount);
    token.approve(address(connext), amount);
```

Finally, we construct the `XCallArgs` and call `xcall` on the Connext contract.

```solidity
    CallParams memory callParams = CallParams({
      to: to,
      callData: "", // empty here because we're only sending funds
      originDomain: originDomain,
      destinationDomain: destinationDomain,
      agent: msg.sender, // address allowed to execute transaction on destination side in addition to relayers
      recovery: msg.sender, // fallback address to send funds to if execution fails on destination side
      forceSlow: false, // option to force slow path instead of paying 0.05% fee on fast liquidity transfers
      receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
      callback: address(0), // zero address because we don't expect a callback
      callbackFee: 0, // fee paid to relayers; relayers don't take any fees on testnet
      relayerFee: 0, // fee paid to relayers; relayers don't take any fees on testnet
      destinationMinOut: (amount / 100) * 97 // the minimum amount that the user will accept due to slippage from the StableSwap pool
    });

    XCallArgs memory xcallArgs = XCallArgs({
      params: callParams,
      transactingAsset: asset,
      transactingAmount: amount,
      originMinOut: (amount / 100) * 97 // the minimum amount that the user will accept due to slippage from the StableSwap pool
    });

    connext.xcall(xcallArgs);
  }
}
```

A detailed reference of all the `xcall` arguments can be found [here](https://docs.connext.network/developers/xcall-params).

---

## Unauthenticated

While simple transfers are technically unauthenticated, we'll treat cross-domain calls that send `calldata` a bit differently because they involve a target contract on the destination side.

### Target Contract

Suppose we have a target contract on the destination domain as follows.

```solidity title="Target.sol"
contract Target {
  uint256 public value;

  function updateValue(uint256 newValue) external {
    value = newValue;
  }
}
```

Our goal is to call the `updateValue` function, which is unauthenticated (i.e. callable by anyone), from a different domain.

### Source Contract

The source contract initiates the cross-chain interaction with Connext. There isn't much difference between this contract and the one from the transfer example above. The only major differences are:

- we aren't sending funds with the `xcall` so the entire approval dance isn't necessary
- we are sending `calldata` so we need to construct it

We have the same imports and constructor.

```solidity title="Source.sol"
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {CallParams, XCallArgs} from "nxtp/core/connext/libraries/LibConnextStorage.sol";

contract Source {
  IConnextHandler public immutable connext;

  constructor(IConnextHandler _connext) {
    connext = _connext;
  }
```

Then we define this source-side contract's `xChainUpdate` function, which requires a set of arguments necessary for the `xcall` later.

```solidity
  function xChainUpdate(
    address to, // the address of the target contract
    address asset, // address of token on source domain  
    uint32 originDomain, // e.g. from Goerli (1735353714)
    uint32 destinationDomain, // to Optimism-Goerli (1735356532)
    uint256 newValue // value to update to
  ) external payable {
```

We create the `calldata` by encoding the target contract's `updateValue` function with the correct arguments. Recall that the target function's signature is `updateValue(uint256 newValue)`.

```solidity
    bytes4 selector = bytes4(keccak256("updateValue(uint256)"));
    bytes memory callData = abi.encodeWithSelector(selector, newValue);
```

As before, we construct the `XCallArgs` and call `xcall` on the Connext contract.

```solidity
    CallParams memory callParams = CallParams({
      to: to,
      callData: callData,
      originDomain: originDomain,
      destinationDomain: destinationDomain,
      agent: msg.sender, // address allowed to execute transaction on destination side in addition to relayers
      recovery: msg.sender, // fallback address to send funds to if execution fails on destination side
      forceSlow: false, // option to force slow path instead of paying 0.05% fee on fast liquidity transfers
      receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
      callback: address(0), // zero address because we don't expect a callback
      callbackFee: 0, // fee paid to relayers for the callback; no fees on testnet
      relayerFee: 0, // fee paid to relayers for the forward call; no fees on testnet
      destinationMinOut: 0 // not sending funds so minimum can be 0
    });

    XCallArgs memory xcallArgs = XCallArgs({
      params: callParams,
      transactingAsset: asset, // even though this isn't used, we need a registered asset here
      transactingAmount: 0, // not sending funds with this calldata-only xcall
      originMinOut: 0 // not sending funds so minimum can be 0
    });

    connext.xcall(xcallArgs);
  }
}
```

Once these contracts are deployed, anyone can call `xChainUpdate` on the source contract of the origin domain to change the value stored in the target contract of the destination domain.

---

## Authenticated

The most interesting cross-chain use cases can only be accomplished through authenticated calls on the destination domain. With authentication requirements, a xapp developer must carefully implement authentication checks. We'll see how this is done in the following example.

Let's say our target contract contains a function that **_only our source contract should be able to call_**.

```solidity
  function updateValue(uint256 newValue) external onlyExecutor {
    value = newValue;
  }
```

You'll notice we have a custom modifier `onlyExecutor` on this function. This signals some kind of authentication requirement - we'll dig into that in a bit. For the authenticated flow, it's actually easier to consider the source contract first.

### Source Contract

This is the exact same contract as the source contract for the unauthenticated example above except that `forceSlow: true`. See the notes about this parameter [here](../xcall-params.md).

To recap: inside the `xChainUpdate` function, we simply create `calldata` to match the target function signature, construct the `XCallArgs`, and call `xcall` with it.

```solidity title="Source.sol"
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {CallParams, XCallArgs} from "nxtp/core/connext/libraries/LibConnextStorage.sol";

contract Source {
  IConnextHandler public immutable connext;

  constructor(IConnextHandler _connext) {
    connext = _connext;
  }

  function xChainUpdate(
    address to,
    address asset,
    uint32 originDomain,
    uint32 destinationDomain,
    uint256 newValue
  ) external payable {

    bytes4 selector = bytes4(keccak256("updateValue(uint256)"));
    bytes memory callData = abi.encodeWithSelector(selector, newValue);

    CallParams memory callParams = CallParams({
      to: to,
      callData: callData,
      originDomain: originDomain,
      destinationDomain: destinationDomain,
      agent: msg.sender, // address allowed to execute transaction on destination side in addition to relayers
      recovery: msg.sender, // fallback address to send funds to if execution fails on destination side
      //highlight-next-line
      forceSlow: true, // this must be true for authenticated calls
      receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
      callback: address(0), // zero address because we don't expect a callback
      callbackFee: 0, // fee paid to relayers for the callback; no fees on testnet
      relayerFee: 0, // fee paid to relayers for the forward call; no fees on testnet
      destinationMinOut: 0 // not sending funds so minimum can be 0
    });

    XCallArgs memory xcallArgs = XCallArgs({
      params: callParams,
      transactingAsset: asset, // even though this isn't used, we need a registered asset here
      transactingAmount: 0, // not sending funds with this calldata-only xcall
      originMinOut: 0 // not sending funds so minimum can be 0
    });

    connext.xcall(xcallArgs);
  }
}
```

### Target Contract

The target contract now has to be written carefully given our authentication requirements. Remember that **_only our source contract should be able to call_** the target function.

Import the `IConnextHandler` and `IExecutor` interfaces. We also need the `LibCrossDomainProperty` library which will allow us to check the origin domain and sender contract.

```solidity title="Target.sol"
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {IExecutor} from "nxtp/core/connext/interfaces/IExecutor.sol";
import {LibCrossDomainProperty} from "nxtp/core/connext/libraries/LibCrossDomainProperty.sol";
```

In the constructor, we pass the address of the origin-side contract and the Domain ID of the origin domain. We also pass in the address of the Connext Executor contract which should be the only allowed caller of the target function. These are crucial pieces of information that we will check against to uphold our authentication requirement.

```solidity
contract Target {
  uint256 public value; // the value we want to update from origin
  address public originContract; // the address of Source.sol
  uint32 public originDomain; // the origin Domain ID
  address public executor; // the address of Executor.sol

  constructor(
    address _originContract,
    uint32 _originDomain,
    address payable _connext
  ) {
    originContract = _originContract;
    originDomain = _originDomain;
    executor = ConnextHandler(_connext).getExecutor();
  }
```

Here's the `onlyExecutor` modifier we saw earlier. In it, we use the `LibCrossDomainProperty` library functions `originSender()` and `origin()` to check that the originating call came from the expected origin contract and domain. We also need to check that the `msg.sender` is the Connext Executor contract - otherwise, any calling contract could just spoof the origin contract and domain that we're expecting.

```solidity
  modifier onlyExecutor() {
    require(
      LibCrossDomainProperty.originSender(msg.data) == originContract &&
        LibCrossDomainProperty.origin(msg.data) == originDomain &&
        msg.sender == address(executor),
      "Expected origin contract on origin domain called by Executor"
    );
    _;
  }
```

With the `onlyExecutor` modifier in place, our authenticated function is secured.

```solidity
  function updateValue(uint256 newValue) external onlyExecutor {
    value = newValue;
  }
}
```

Now the target contract can only be updated by a cross-chain call from the source contract!

---

## Callbacks

One awesome feature we've introduced is the ability to use JS-style callbacks to respond to results of calls from the destination domain on the origin domain. You can read the [detailed spec here](https://github.com/connext/nxtp/discussions/883).

Let's see how this works by building on the Authenticated example.

### Source Contract

To enable callback handling, some contract on the origin domain must implement the `ICallback` interface. This could be a separate contract or the Source contract itself. 

We'll have our Source contract handle the callback. To do so, Source should import the `ICallback` interface and change the `callback` param to the address of the contract implementing this interface. It will also need a reference to the Connext PromiseRouter contract so we add that to the constructor.

```solidity title="Source.sol"
import {IConnextHandler} from "nxtp/core/connext/interfaces/IConnextHandler.sol";
import {CallParams, XCallArgs} from "nxtp/core/connext/libraries/LibConnextStorage.sol";
//highlight-next-line
import {ICallback} from "nxtp/core/promise/interfaces/ICallback.sol";

contract Source {
  IConnextHandler public immutable connext;
  address public immutable promiseRouter;

  constructor(
    IConnextHandler _connext, 
    //highlight-next-line
    address _promiseRouter
  ) {
    connext = _connext;
    //highlight-next-line
    promiseRouter = _promiseRouter;
  }

  function xChainUpdate(
    address to,
    address asset,
    uint32 originDomain,
    uint32 destinationDomain,
    uint256 newValue
  ) external payable {

    bytes4 selector = bytes4(keccak256("updateValue(uint256)"));
    bytes memory callData = abi.encodeWithSelector(selector, newValue);

    CallParams memory callParams = CallParams({
      to: to,
      callData: callData,
      originDomain: originDomain,
      destinationDomain: destinationDomain,
      agent: msg.sender, // address allowed to execute transaction on destination side in addition to relayers
      recovery: msg.sender, // fallback address to send funds to if execution fails on destination side
      forceSlow: true, // this must be true for authenticated calls
      receiveLocal: false, // option to receive the local bridge-flavored asset instead of the adopted asset
      //highlight-next-line
      callback: address(this), // this contract implements the callback
      callbackFee: 0, // fee paid to relayers for the callback; no fees on testnet
      relayerFee: 0, // fee paid to relayers for the forward call; no fees on testnet
      destinationMinOut: 0 // not sending funds so minimum can be 0
    });

    XCallArgs memory xcallArgs = XCallArgs({
      params: callParams,
      transactingAsset: asset, // even though this isn't used, we need a registered asset here
      transactingAmount: 0, // not sending funds with this calldata-only xcall
      originMinOut: 0 // not sending funds so minimum can be 0
    });

    connext.xcall(xcallArgs);
  }
```

The callback-handling contract needs a reference to the Connext PromiseRouter similarly to how the Target contract needs a reference to the Connext Executor. The contract implementing `callback` should have a modifier that only allows the Connext PromiseRouter to execute the callback.

```solidity
  modifier onlyPromiseRouter () {
    require(
      msg.sender == address(promiseRouter),
      "Expected PromiseRouter"
    );
    _;
  }
```

Now we'll implement the `callback` function. 

The return data from the destination domain is sent with the callback so we can do whatever we want with it. In this case, we'll simply emit an event with the `newValue` we sent so that the callback can be used on the origin domain to verify a successful update on the destination domain.

```solidity
  event CallbackCalled(bytes32 transferId, bool success, uint256 newValue);

  function callback(
    bytes32 transferId,
    bool success,
    bytes memory data
  ) external onlyPromiseRouter {
    uint256 newValue = abi.decode(data, (uint256));
    emit CallbackCalled(transferId, success, newValue);
  }
}
```

### Target Contract

On the Target side, the function just needs to return some data.

```solidity title="Target.sol"
...
  function updateValue(uint256 newValue)
    external onlyExecutor
    //highlight-next-line
    returns (uint256)
  {
    value = newValue;
    //highlight-next-line
    return newValue;
  }
}
```

That's it! Connext will now send the callback execution back to the origin domain to be processed by relayers.

**Note**: Origin-side relayers have not been set up to process callbacks yet. This will be added shortly!

## Deploy and Experiment

To deploy these contracts and try out the `xcalls` yourself, clone the [xApp Starter Kit](https://github.com/connext/xapp-starter/) and see the README for full instructions.

</Section>
<Section name="3. Backend Integrations" description="Backend Integrations">

# Backend Integrations

# SDK Quickstart

The Connext SDK allows developers to interact with the Connext protocol in standard Node.js or web environments. This quickstart will go through how to build on top of Connext using the TypeScript SDK. 

The NXTP repo is required for this project. It can be found in our NXTP repo, under `packages/examples/sdk-server`.

[NXTP repo](https://github.com/connext/nxtp/)

--- 
# SDK Server

This package provides an HTTP server that allows direct interaction with our supported testnets and exposes our SDK via RESTful APIs.

Using Postman or a VScode extension like humao.rest-client you can use the `index.http` file located in the `examples` folder in this package to access the endpoints and perform xCalls

## Configuration

Create `config.json` or copy `config.json.example` in the root of this package (`packages/sdk-server`)

```json
{
  "network": "testnet",
  "chainConfig": {
    "4": {
      "provider": ["https://rinkeby.infura.io/v3/..."]
    },
    "5": {
      "provider": ["https://goerli.infura.io/v3/..."]
    }
  },
  "mnemonic": "<add mnemonic here>"
}
```

**_OR_**

Create a `.env` in the root of this package (`packages/sdk-server`) with the following keys:

- `NXTP_MNEMONIC`: \_required\* string mnemonic of the wallet you wish to connect with.

- `NXTP_CHAIN_CONFIG`: _required_ Stringified JSON object mapping `chainId` to an array of provider URLs. Example: `'{"4":{"provider":["https://rinkeby.infura.io/v3/"]},"5": {"provider":["https://goerli.infura.io/v3/"]}}'`

- `NXTP_NETWORK`: _optional_ Override default network mainnet to any existing network(local, testnet, mainnet).
- `NXTP_NATS_URL`: _optional_ Override default NATS URL. Example: `ws://localhost:4221`
- `NXTP_AUTH_URL`: _optional_ Override default auth URL. Example: `http://localhost:5040`
- `NXTP_LOG_LEVEL`: _optional_ Override default LogLevel info. Example: `debug`
- `NXTP_MESSAGING_MNEMONIC`: _optional_ Override Random messaging key.
- `NXTP_SKIP_POLLING`: _optional_ skip auction waiting for testing.

## First time setup

First of all, make sure you are on the latest yarn version:

- `yarn set version berry`

You must install the dependencies of the connext workspace.
To do so, from the root folder of the git repo, run:

- `yarn`

If you have issues, try deleting `node_modules` and `yarn.lock`. After deleting `yarn.lock` run `touch yarn.lock` since it does not like if there is no lock file.

We must now build all packages:

- `yarn build:all`

If you have issues, try running `yarn clean:all`

### Running the Server

From the root of the repo, run:

`yarn workspace @connext/nxtp-sdk-server dev`

### Parameters for an xCall

You can find the parameter shape for an xCall here: https://docs.connext.network/developers/xcall-params

</Section>
