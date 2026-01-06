# Smart Wallet Signer Mismatch Error

## Error Message

```
API Error 400: {"message":"Signer does not match with correct address - you should use embedded address for smart wallet"}
```

## Cause

Your account is registered as a smart wallet, but you're signing orders with a regular EOA private key. The `maker`/`signer` address in your order doesn't match what the API expects.

## Solutions

### 1. Use EOA authentication (most common fix)

Make sure you login with `client: "eoa"`:

```python
response = requests.post(
    f"{API_URL}/auth/login",
    headers=headers,
    json={"client": "eoa"}  # NOT "smart_wallet"
)
```

### 2. Check your user data after authentication

```python
session, user_data = authenticate(private_key)
print(user_data)

# Look for these fields:
# - 'account': Your address (use this for maker/signer)
# - 'smartWallet': If present, your account is linked to a smart wallet
# - 'client': Should be "eoa" for regular wallets
```

### 3. Use the correct address for orders

```python
# Always use the checksummed address from user_data
maker_address = user_data['account']  # Already checksummed

order = {
    "maker": maker_address,
    "signer": maker_address,  # Same as maker for EOA
    # ... rest of order
}
```

### 4. If your account is permanently linked to a smart wallet

- Create a new account with a fresh wallet address
- Or contact Limitless support to unlink the smart wallet

## Prevention

When first setting up, always use `client: "eoa"` unless you specifically need smart wallet functionality.

## Related

- [Authentication Guide](../guides/authentication.md)
- [Placing Orders Guide](../guides/placing-orders.md)
