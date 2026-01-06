# Invalid Token ID Error

## Error Message

Variations include:
- `"Invalid token ID"`
- `"Position not found"`
- `"Token ID does not exist"`

## Cause

You're using the wrong `tokenId` for the market. Token IDs are:
- Unique per market
- Large integers (must be strings to avoid precision loss)
- Different for YES vs NO outcomes

## Solution

**Always fetch token IDs fresh from market data:**

```python
market = get_market(slug)

# positionIds[0] = YES token
# positionIds[1] = NO token
yes_token_id = market['positionIds'][0]
no_token_id = market['positionIds'][1]

print(f"YES token: {yes_token_id}")
print(f"NO token: {no_token_id}")
```

## Common Mistakes

### 1. Hardcoding token IDs

```python
# WRONG - hardcoded IDs
yes_token = "19633204485790857949828516737993423758628930235371629943999544859324645414627"

# CORRECT - fetch from API
market = get_market("btc-100k-2024")
yes_token = market['positionIds'][0]
```

### 2. Using token ID from wrong market

Each market has different token IDs. Make sure you're fetching from the correct market slug.

### 3. Confusing YES and NO indices

```python
# positionIds[0] = YES (index 0)
# positionIds[1] = NO  (index 1)

token_type = "YES"
token_id = market['positionIds'][0 if token_type == "YES" else 1]
```

### 4. Losing precision with numbers

Token IDs are very large integers. Always use strings:

```javascript
// WRONG - loses precision in JavaScript
const tokenId = 19633204485790857949828516737993423758628930235371629943999544859324645414627;

// CORRECT - use string
const tokenId = "19633204485790857949828516737993423758628930235371629943999544859324645414627";
```

## Debugging

Print the market data to see the actual token IDs:

```python
import json

market = get_market(slug)
print(json.dumps(market, indent=2))

# Look for:
# {
#   "positionIds": [
#     "123...",  // YES
#     "456..."   // NO
#   ]
# }
```

## Related

- [Market Schema](../schemas/market.md)
- [Placing Orders Guide](../guides/placing-orders.md)
