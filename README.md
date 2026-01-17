# 📚 Immutable Publishing

**Verified. Permanent. Free.**

We publish cryptographically verified editions of public domain books. Every text is hashed and timestamped on the Bitcoin blockchain via OpenTimestamps.

## Why?

Publishers silently revise books. We provide proof of original text that anyone can verify.

## Verify a Book

1. Get the SHA-256 hash from your book's verification page
2. Visit [immutablepublishing.com](https://immutablepublishing.com)
3. Paste the hash to verify

Or verify the Bitcoin timestamp yourself:
```bash
pip install opentimestamps-client
ots verify <book>.ots
```

## Published Books

| Title | Author | Year | Status |
|-------|--------|------|--------|
| Alice's Adventures in Wonderland | Lewis Carroll | 1865 | ✅ Verified |

## Technology

- **SHA-256** - Cryptographic hash of original text
- **OpenTimestamps** - Bitcoin blockchain anchoring (free)
- **Arweave** - Permanent storage (coming soon)

## License

All published texts are public domain. Code is MIT licensed.

---

*Preserving humanity's literary heritage with cryptographic verification.*
