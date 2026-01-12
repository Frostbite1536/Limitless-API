# Finding Markets by Name or Characteristics

## Symptoms

- You know the market name (e.g., "1hr BTC") but need the slug identifier
- You want to find all markets matching certain criteria (ticker, timeframe, etc.)
- You're trying to look up a market programmatically without browsing the UI

## Possible Causes

### 1. No direct "lookup by name" endpoint

The API doesn't have a single endpoint that takes "1hr BTC" and returns the slug. Instead, you need to either search or browse and filter.

### 2. Markets are indexed by slug, not title

Market operations use the `slug` identifier (e.g., `btc-1hr-price`), not the human-readable title, so you need to find the slug first.

### 3. Multiple markets may match

There could be multiple markets with similar characteristics (different deadlines, different venues, etc.), so you need a way to filter the results.

## Solutions: Three Approaches

### Method 1: Semantic Search (Fastest for Known Markets)

Use the search endpoint for keyword-based discovery. This is ideal when you know the market name.

```python
import requests

# Search for "1hr BTC" market
response = requests.get(
    "https://api.limitless.exchange/markets/search",
    params={
        "query": "1hr BTC",
        "limit": 10,
        "similarityThreshold": 0.5  # Optional: min similarity score 0-1
    }
)

results = response.json()

for result in results['results']:
    print(f"Slug: {result['slug']}")
    print(f"Title: {result['title']}")
    print(f"Relevance Score: {result['score']:.2f}")
    print("---")
```

**Advantages:**
- Fast - optimized for keyword matching
- Returns relevance scores so you can rank results
- Works with partial queries

**Disadvantages:**
- Requires knowing at least part of the market name
- Threshold filtering available but not strict filtering by category/ticker

**Best for:** Finding a specific market when you know its name or description

---

### Method 2: Get All Slugs and Filter (Best for Bulk Operations)

Fetch all market slugs with metadata, then filter locally. This is best when you need to find multiple markets or want to cache the index.

```python
import requests

# Get all active market slugs with metadata
response = requests.get(
    "https://api.limitless.exchange/markets/active/slugs"
)

markets = response.json()

# Filter for 1hr BTC markets
btc_1hr_markets = [
    m for m in markets
    if "btc" in m['slug'].lower()
    and "1hr" in m['slug'].lower()
]

for market in btc_1hr_markets:
    print(f"Slug: {market['slug']}")
    print(f"Ticker: {market.get('ticker', 'N/A')}")
    print(f"Strike Price: {market.get('strikePrice', 'N/A')}")
    print(f"Deadline: {market['deadline']}")
    print("---")
```

**Response format:**
```json
[
  {
    "slug": "btc-1hr-price-jan",
    "ticker": "BTC",
    "strikePrice": "45000",
    "deadline": "2024-12-31T23:59:59Z"
  },
  {
    "slug": "btc-1hr-direction",
    "ticker": "BTC",
    "strikePrice": null,
    "deadline": "2025-01-31T23:59:59Z"
  }
]
```

**Advantages:**
- Get all markets at once (with metadata)
- Full control over filtering locally
- Can filter by multiple criteria (ticker, deadline, etc.)
- Useful for building market indexes

**Disadvantages:**
- Larger response (all markets)
- Need to handle filtering logic yourself
- Less relevant if you just need one market

**Best for:** Building market indexes, bulk operations, or when you need to filter by multiple criteria

---

### Method 3: Browse by Category (If Category is Known)

If you know which category the market belongs to (e.g., crypto, commodities), browse that category.

```python
import requests

# First, get category counts to find the right ID
categories_response = requests.get(
    "https://api.limitless.exchange/markets/categories/count"
)

categories = categories_response.json()
print("Available categories:", categories['categories'])

# Then browse a specific category (example: category 1 for crypto)
response = requests.get(
    "https://api.limitless.exchange/markets/active/1",
    params={"limit": 100, "sortBy": "volume"}
)

markets = response.json()

# Filter for 1hr BTC in this category
for market in markets['markets']:
    if "1hr" in market['title'].lower() and "btc" in market['title'].lower():
        print(f"Slug: {market['slug']}")
        print(f"Title: {market['title']}")
        print(f"Description: {market['description']}")
        print(f"Volume: {int(market['volume']) / 1e6} USDC")
        print("---")
```

**Advantages:**
- Filtered by category upfront
- Get full market details (description, volume, liquidity)
- Useful if you know the market category

**Disadvantages:**
- Need to know or discover the category ID first
- Still need to filter locally by name

**Best for:** When you know the market category (crypto, commodities, etc.)

---

## Complete Example: Find and Fetch Market Details

Here's a complete workflow to find a market and get its full details:

```python
import requests

class MarketFinder:
    API_URL = "https://api.limitless.exchange"

    def search_by_name(self, query, limit=10):
        """Search for markets by name/keywords"""
        response = requests.get(
            f"{self.API_URL}/markets/search",
            params={"query": query, "limit": limit}
        )
        return response.json()['results']

    def get_slug_from_search(self, query):
        """Find and return the first matching market slug"""
        results = self.search_by_name(query, limit=1)
        if results:
            return results[0]['slug']
        return None

    def get_market_details(self, slug):
        """Fetch full market details by slug"""
        response = requests.get(
            f"{self.API_URL}/markets/{slug}"
        )
        market = response.json()

        return {
            "slug": market['slug'],
            "title": market['title'],
            "description": market.get('description'),
            "status": market['status'],
            "deadline": market['deadline'],
            "volume": int(market['volume']) / 1e6,
            "liquidity": int(market['liquidity']) / 1e6,
            "current_price": market.get('yesPrice', 'N/A')
        }

    def find_market_details(self, query):
        """One-shot: search for market and get full details"""
        slug = self.get_slug_from_search(query)
        if not slug:
            print(f"No market found for '{query}'")
            return None

        return self.get_market_details(slug)

# Usage
finder = MarketFinder()

# Find "1hr BTC" and get details
market = finder.find_market_details("1hr BTC")

if market:
    print(f"Market: {market['title']}")
    print(f"Slug: {market['slug']}")
    print(f"Status: {market['status']}")
    print(f"Deadline: {market['deadline']}")
    print(f"Volume: ${market['volume']:.2f}")
    print(f"YES Price: {market['current_price']}")
```

---

## Comparison: Which Method to Use?

| Use Case | Method | Why |
|----------|--------|-----|
| Finding a single specific market | Semantic Search | Fast, keyword-based, returns relevance scores |
| Building a market index/cache | Get All Slugs | One API call, all markets with metadata |
| Finding markets in a category | Browse by Category | Filtered results, full market details |
| Filter by multiple criteria (ticker, deadline) | Get All Slugs | Most control over filtering |
| Recurring lookups of same market | Cache slug after first search | Avoid repeated searches |

---

## Tips

1. **Cache the slug** - Once you have the slug, store it. You don't need to search repeatedly:
   ```python
   # Bad: searching every time
   for i in range(100):
       slug = find_slug("1hr BTC")
       trade(slug)

   # Good: search once, then reuse
   slug = find_slug("1hr BTC")
   for i in range(100):
       trade(slug)
   ```

2. **Use similarity threshold in search** - Adjust `similarityThreshold` to be stricter (higher) or more lenient (lower):
   ```python
   # Strict: only very similar results
   requests.get(
       f"{API_URL}/markets/search",
       params={"query": "1hr BTC", "similarityThreshold": 0.8}
   )
   ```

3. **Handle multiple matches** - Markets change/resolve, so there may be multiple active markets with the same name:
   ```python
   results = search_by_name("1hr BTC", limit=10)

   # Filter by deadline if needed
   active_markets = [
       r for r in results
       if "2024-12" in r['deadline']  # Example: recent deadline
   ]
   ```

4. **For recurring hourly markets** - If markets reset every hour (1hr BTC), you may need to find a new slug each hour:
   ```python
   # Refresh market slug every hour
   import schedule

   def update_market_slug():
       global current_btc_1hr_slug
       current_btc_1hr_slug = find_slug("1hr BTC")

   schedule.every(1).hours.do(update_market_slug)
   ```

## Related

- [Markets Endpoints](../endpoints/markets.md) - Full market browsing and search API reference
- [Market Schemas](../schemas/market.md) - Complete market data field definitions
- [Automating Market Search by Category](market-search-automation.md) - Bulk market data fetching for analysis
