# escrow
RIDE smart contract for exchanging assets between two parties without trust

# Interface
@Callable(i)
func escrow(wantAssetId: String, wantAmount: Int, partnerAddress: String)
- Each of two parties call this function. Wanted assetId and amount are passed as arguments, as well as partner's Waves address
- Their offered assetId and amount are passed as payment


@Callable(i)
func cancel()
- If, for some reason, you don't want to finish the deal, you can cancel it and payment (previously locked during calling escrow) will be returned
