# escrow
RIDE smart contract for exchanging assets on Waves blockchain between two parties without a trust

# Interface
@Callable(i)
func escrow(wantAssetId: String, wantAmount: Int, partnerAddress: String)
- Each of two parties call this function. Wanted assetId and amount are passed as arguments, as well as partner's Waves address
- Their offered assetId and amount are passed as payment


@Callable(i)
func cancel()
- If, for some reason, you don't want to finish the deal, you can cancel it and payment (previously locked during calling escrow) will be returned

# Use the dApp
You can run it on MainNet with official Waves dApp interface:
https://waves-dapp.com/3PAimVkP7cqar6gMKNh5bQDNH9x4U6sXE7u
