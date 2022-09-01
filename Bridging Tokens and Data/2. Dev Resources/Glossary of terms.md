Glossary of Terms

- `Domain`: Essentially the ‘nomadic’ term for chain.
- `Origin`: Sending chain/domain
- `Destination`: Receiving chain/domain
- `Source`: Contract that an interaction starts from (and calls into Connext)
- `Target`: Contract that an interaction happens to (is called from Connext)
- `Canonical Asset`:  The token has been issued on its specified home chain (as opposed to being a wrapped token). The asset that is locked on the bridge to mint on another chain.
- `Representation Asset`: Token originally deployed on some other chain (a synthetic IOU issued by the protocol). This is the asset that you receive from the bridge, it is a pointer that represents some locked funds for this asset on the canonical domain (i.e. mad-USDC).
- `Local Asset`: A token originally deployed on the local chain. The asset that the bridge will deliver to you - this will be the canonical asset on the canonical domain, otherwise it’s the representation.
- `Adopted Asset`: The asset that has traction (user adoption) on the domain (i.e. POS-USDC on polygon).
- `Transacting Asset` The asset the user sends into `xcall` or gets from `execute`, depending on the context. Can be the local or adopted asset.