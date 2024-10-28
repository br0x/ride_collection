## Puzzle Lend markets Rebalancer
1. Get current daily income on 3 Puzzle Lend markets
2. Find the delta amount on each market that maximizes the daily income

### Current Rate curves
```
(200_0000, 2500_0000, 8000_0000, 1_0000_0000) : main market
(200_0000, 2500_0000, 8000_0000, 1_0000_0000) : defi market
(2000_0000, 1_0000_0000, 6000_0000, 4_0000_0000) : low cap market
```
### Usage example

```curl -X 'POST' \
  'https://nodes.wavesnodes.com/utils/script/evaluate/3PK88nVJM6mXHCNkp5WKkk1Y4wWxu47zWzz' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "expr": "rebalanceREADONLY(\"9wc3LXNA4TEBsXyKtoLE9mrbDD7WMHXvXrCjZvabLAsi\", \"3P64qEVzuGzBJuYfDXYisFtokJChSRa8uja\")"
}'
```
