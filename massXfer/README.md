Rewritten and optimized version of this contract: https://github.com/Isgeny/mass-token-transfer <br>
Mainnet address: 3PMXFRwQY3VfZcJdywACvJRSkS9bsLTPWjr <br>
Testnet address: 3NAAQMEFj8Cy37pzLC4gtTeHbuaEiGEX1M3 <br>
<br>
In the following example: <br>
3MxAH1Q1kCsHdMrKioThpNTPCdKdty4ikdx gets 1000_0000 WAVELETS <br>
3N7HtrKeFFLz5oy6PfGv1Lg3GwPk4gCPdGy gets 4000_0000 WAVELETS <br>
3N7HtrKeFFLz5oy6PfGv1Lg3GwPk4gCPdGy gets 2000_0000 of token 8i8WqH4KvWauvEiLmhod4nrt1Xgpn8pc6bzCPR8CozK5 <br>
3MxAH1Q1kCsHdMrKioThpNTPCdKdty4ikdx gets 3000_0000 of token FFtkQvTzC3nfKJuXeWL3ijwchqb3WfMno4piDX7vPvWV <br>

    "dApp": "3NAAQMEFj8Cy37pzLC4gtTeHbuaEiGEX1M3",
    "payment": [
        {
            "amount": 50000000,
            "assetId": null
        },
        {
            "amount": 20000000,
            "assetId": "8i8WqH4KvWauvEiLmhod4nrt1Xgpn8pc6bzCPR8CozK5"
        },
        {
            "amount": 30000000,
            "assetId": "FFtkQvTzC3nfKJuXeWL3ijwchqb3WfMno4piDX7vPvWV"
        }
    ],
    "call": {
        "function": "massXfer",
        "args": [
            {
                "type": "list",
                "value": [
                    {
                        "type": "string",
                        "value": "3MxAH1Q1kCsHdMrKioThpNTPCdKdty4ikdx"
                    },
                    {
                        "type": "string",
                        "value": "3N7HtrKeFFLz5oy6PfGv1Lg3GwPk4gCPdGy"
                    },
                    {
                        "type": "string",
                        "value": "3N7HtrKeFFLz5oy6PfGv1Lg3GwPk4gCPdGy"
                    },
                    {
                        "type": "string",
                        "value": "3MxAH1Q1kCsHdMrKioThpNTPCdKdty4ikdx"
                    }
                ]
            },
            {
                "type": "list",
                "value": [
                    {
                        "type": "integer",
                        "value": 10000000
                    },
                    {
                        "type": "integer",
                        "value": 40000000
                    },
                    {
                        "type": "integer",
                        "value": 20000000
                    },
                    {
                        "type": "integer",
                        "value": 30000000
                    }
                ]
            },
            {
                "type": "list",
                "value": [
                    {
                        "type": "integer",
                        "value": 0
                    },
                    {
                        "type": "integer",
                        "value": 0
                    },
                    {
                        "type": "integer",
                        "value": 1
                    },
                    {
                        "type": "integer",
                        "value": 2
                    }
                ]
            },
            {
                "type": "string",
                "value": "hi there"
            }
        ]
    },
