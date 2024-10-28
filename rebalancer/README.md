1. Get current daily income on 3 markets (idx 0, 2, 3)
2. Find the delta amount on each market that maximizes the daily income
3. Actual rebalance by withdraw/supply these deltas

func getOutdatedUr (assetIdStr) = {
    let down = fraction(tryGetInteger(("total_supplied_" + assetIdStr)), tryGetInteger((assetIdStr + "_sRate")), Scale16)
    if ((down == 0))
        then 0
        else fraction(Scale8, fraction(tryGetInteger(("total_borrowed_" + assetIdStr)), tryGetInteger((assetIdStr + "_bRate")), Scale16), down)
    }

func getInterest (assetIdStr) = {
    let ur = getOutdatedUr(assetIdStr)
    let curve = getRateCurve(assetIdStr)
    let rate = (curve._1 + (if ((curve._3 >= ur))
        then fraction(ur, curve._2, curve._3)
        else (curve._2 + fraction((ur - curve._3), curve._4, (100000000 - curve._3)))))
    max([fraction(rate, Scale8, (dayBlocks * 365)), 1])
    }

func tokenRatesRecalc(assetIdStr: String, lastRecalcHeight: Int) = {
    let interest = getInterest(assetIdStr)
    let ur = getOutdatedUr(assetIdStr)
    let lastBRate = max([tryGetInteger(assetIdStr + "_bRate"), Scale16])
    let newBRate = lastBRate + (height - lastRecalcHeight) * interest
    let lastSRate = max([tryGetInteger(assetIdStr + "_sRate"), Scale16])
    let newSRate = lastSRate + (height - lastRecalcHeight) * fraction(interest, ur, Scale8) * (100 - reserveFund) / 100
    [IntegerEntry(assetIdStr + "_sRate", newSRate), IntegerEntry(assetIdStr + "_bRate", newBRate)]
}

func getActualRate(assetIdStr: String, rateType: String) = {
    let lastRecalcHeight = tryGetInteger("lastRateHeight")

    func f(accum: (Int, List[IntegerEntry]), token: String) = {
        let recalc = tokenRatesRecalc(token, lastRecalcHeight)
        (if (token != assetIdStr) then accum._1 else 
            if (rateType == "sRate")
                then recalc[0].value # sRate
                else recalc[1].value, # bRate
            (accum._2 ++ recalc)
        )
    }
    let r = FOLD<12>(getMarketAssets(), (0, []), f)
    (r._1, r._2 :+ IntegerEntry("lastRateHeight", height))
}

marketAddress.supply(assetAmount, assetId):

    let (sRate, ratesRecalcResult) = getActualRate(assetIdStr, "sRate", marketStr)
    let amount = fraction(assetAmount, Scale16, sRate, DOWN)
    [
        # for every token in the market:
        IntegerEntry(assetIdStr + "_sRate", newSRate), 
        IntegerEntry(assetIdStr + "_bRate", newBRate),
        # for market:
        IntegerEntry("lastRateHeight", height),

        IntegerEntry(address + "_supplied_" + assetIdStr, tryGetInteger(address + "_supplied_" + assetIdStr) + amount), 
        IntegerEntry("total_supplied_" + assetIdStr, tryGetInteger("total_supplied_" + assetIdStr) + amount),
    ]


marketAddress.withdraw(assetAmount, assetId):

    let (sRate, ratesRecalcResult) = getActualRate(assetIdStr, "sRate", marketStr)
    let amount = fraction(assetAmount, Scale16, sRate, CEILING)
    [
        # for every token in the market:
        IntegerEntry(assetIdStr + "_sRate", newSRate), 
        IntegerEntry(assetIdStr + "_bRate", newBRate),
        # for market:
        IntegerEntry("lastRateHeight", height),

        IntegerEntry(address + "_supplied_" + assetIdStr, tryGetInteger(address + "_supplied_" + assetIdStr) - amount), 
        IntegerEntry("total_supplied_" + assetIdStr, tryGetInteger("total_supplied_" + assetIdStr) - amount), 
    ]

// (200_0000, 2500_0000, 8000_0000, 1_0000_0000) main+
// (200_0000, 2500_0000, 8000_0000, 1_0000_0000) defi+
// (2000_0000, 1_0000_0000, 6000_0000, 4_0000_0000) low

Есть 3 смартконтракта в сети WAVES, реализующие lending протокол (каждый контракт для своего рынка со своими настройками).
В каждый из этих трех рынков я вложил разное количество USDT в качестве supply, и с каждым блоком сети мои балансы USDT увеличиваются на некоторую величину blockIncome.
Расчеты такие:
Из стейта каждого рынка читаются следующие переменные и константы:
c1, c2, c3, c4 - коэффициенты в формуле зависимости rate от utilization rate ur:
rate = F(ur) = c1 + ur * c2 / c3 { если 0 < ur <= c3}
F(ur) = c1 + c2 + (ur - c3) * c4 / (1 - c3) { если ur > c3}
lastRateHeight - время последнего обновления стейта контракта (в единицах высоты блокчейна)
outdatedTotalSupply - полное количество supplied USDT в рынке (условная величина, не взвешенная на supplyRate)
outdatedTotalBorrow - полное количество borrowed USDT в рынке (условная величина, не взвешенная на borrowRate)
outdatedSRate - supplyRate в рынке на момент lastRateHeight
outdatedBRate - borrowRate в рынке на момент lastRateHeight
outdatedWalletSupply - моё количество supplied USDT в рынке (условная величина, не взвешенная на supplyRate)
Далее вычисляются:
outdatedUr = (outdatedTotalBorrow * outdatedBRate) / (outdatedTotalSupply * outdatedSRate)
rate = F(outdatedUr) // годовая доходность, APR
interest = rate / yearBlocks // доходность за блок
sRate = outdatedSRate + (height - lastRateHeight) * interest * outdatedUr * 0.8
bRate = outdatedBRate + (height - lastRateHeight) * interest
supply = outdatedTotalSupply * sRate
borrow = outdatedTotalBorrow * bRate
ur = borrow / supply
walletSupply = outdatedWalletSupply * sRate
blockIncome = 0.8 * rate * ur * walletSupply
Итак, у нас есть walletSupply[i] и blockIncome[i] для каждого из трех рынков.
Я хочу перераспределить мой суммарный walletSupply между тремя рынками, чтобы получить максимальный суммарный blockIncome.
Итак, в этом же блоке (на высоте height) мы вызываем на каждом из рынков функцию supply(assetAmount) либо withdraw(assetAmount), которые работают аналогично и отличаются только знаком amount.
При их вызове происходит следующее:

supplyOrWithdraw(deltaAssetAmount):
Как и выше, из стейта рынка читаются следующие переменные и константы:
c1, c2, c3, c4
lastRateHeight
outdatedTotalSupply
outdatedTotalBorrow
outdatedSRate
outdatedBRate
outdatedWalletSupply

outdatedUr = (outdatedTotalBorrow * outdatedBRate) / (outdatedTotalSupply * outdatedSRate)
rate = F(outdatedUr)
interest = interest = rate / yearBlocks
outdatedUr = (outdatedTotalBorrow * outdatedBRate) / (outdatedTotalSupply * outdatedSRate)
newSRate = outdatedSRate + (height - lastRateHeight) * interest * outdatedUr * 0.8
newBRate = outdatedBRate + (height - lastRateHeight) * interest
amount = deltaAssetAmount / sRate
В стейте контракта обновляются следующие значения:
outdatedSRate := newSRate
outdatedBRate := newBRate
lastRateHeight := height
outdatedTotalSupply := outdatedTotalSupply + amount
outdatedWalletSupply := outdatedWalletSupply + amount

Найди такой набор значений deltaAssetAmount, который позволит максимизировать суммарный blockIncome

curl -X 'POST' \
  'https://nodes.wavesnodes.com/utils/script/evaluate/3PK88nVJM6mXHCNkp5WKkk1Y4wWxu47zWzz' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "expr": "rebalanceREADONLY(\"9wc3LXNA4TEBsXyKtoLE9mrbDD7WMHXvXrCjZvabLAsi\", \"3P64qEVzuGzBJuYfDXYisFtokJChSRa8uja\")"
}'

1537+40+3639=5216
{
    "type": "Tuple",
    "value": {
        "_1": {
            "type": "Int",
            "value": 6068
        },
        "_2": {
            "type": "Int",
            "value": 0
        },
        "_3": {
            "type": "Int",
            "value": -2275747776
        }
    }
},

let d1 = delta12 + delta13 = -2275.747776
let d2 = -delta12 = 0
let d3 = -delta13 = +2275.747776
