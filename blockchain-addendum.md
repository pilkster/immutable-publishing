# Immutable Publishing - Blockchain & Public Good Addendum

## Philosophy Shift

**Original model:** Business selling verified books
**New model:** Public good infrastructure for cultural preservation, sustainably funded

> "The original texts of humanity's literary heritage should be freely available and cryptographically verifiable by anyone, forever."

---

## Blockchain Architecture

### Dual-Layer Verification

```
┌─────────────────────────────────────────────────────────┐
│                    VERIFICATION LAYERS                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Layer 1: TIMESTAMP PROOF (OpenTimestamps)              │
│  ├── Hash anchored to Bitcoin                           │
│  ├── Proves text existed at specific time               │
│  ├── Cost: FREE (batched transactions)                  │
│  └── Verification: ots verify <file>                    │
│                                                          │
│  Layer 2: PERMANENT STORAGE (Arweave)                   │
│  ├── Full text stored on-chain                          │
│  ├── Immutable, censorship-resistant                    │
│  ├── Cost: ~$0.01-0.10 per book                        │
│  └── Access: ar://[transaction-id]                      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Why Two Layers?

| Layer | Purpose | Survives |
|-------|---------|----------|
| OpenTimestamps | Prove hash existed at time T | Website death, company death, server seizure |
| Arweave | Store actual content | Internet censorship, DMCA, company death |

Together: **Cryptographic proof of what the text said, stored forever, verifiable by anyone**

---

## OpenTimestamps Integration

### What It Is
- Free, open-source Bitcoin timestamping
- Batches thousands of hashes into one BTC transaction
- Run by volunteers, no company to fail
- Used by Internet Archive, journalists, legal firms

### How It Works

```bash
# Install
pip install opentimestamps-client

# Timestamp a file (creates .ots proof file)
ots stamp book.txt
# Uploads hash, returns immediately
# Proof confirmed when Bitcoin block is mined (~10 min - 2 hours)

# Verify a timestamp
ots verify book.txt.ots
# Output: "Success! Bitcoin block 871234 attests data existed as of 2026-01-17"
```

### Integration Code

```python
#!/usr/bin/env python3
"""OpenTimestamps integration for book verification"""

import subprocess
import hashlib
from pathlib import Path
from datetime import datetime

def timestamp_book(book_path: Path) -> Path:
    """Create Bitcoin-anchored timestamp for a book"""
    
    # Generate hash
    content = book_path.read_bytes()
    sha256 = hashlib.sha256(content).hexdigest()
    
    # Create timestamp
    result = subprocess.run(
        ['ots', 'stamp', str(book_path)],
        capture_output=True, text=True
    )
    
    ots_path = book_path.with_suffix(book_path.suffix + '.ots')
    
    return {
        'book': str(book_path),
        'sha256': sha256,
        'ots_proof': str(ots_path),
        'timestamp': datetime.utcnow().isoformat(),
        'status': 'pending_confirmation'  # Confirmed after Bitcoin block
    }

def verify_timestamp(book_path: Path) -> dict:
    """Verify a book's timestamp against Bitcoin"""
    
    ots_path = book_path.with_suffix(book_path.suffix + '.ots')
    
    result = subprocess.run(
        ['ots', 'verify', str(ots_path)],
        capture_output=True, text=True
    )
    
    return {
        'verified': 'Success' in result.stdout,
        'output': result.stdout,
        'bitcoin_block': _extract_block(result.stdout)
    }

def _extract_block(output: str) -> str:
    """Extract Bitcoin block number from OTS output"""
    import re
    match = re.search(r'block (\d+)', output)
    return match.group(1) if match else None
```

---

## Arweave Integration

### What It Is
- Permanent, decentralized storage
- Pay once, stored forever (200+ year economic model)
- ~$0.01-0.10 per book (text is small)
- Content-addressed: ar://[txid]

### Setup

```bash
# Install Arweave bundler CLI
npm install -g @bundlr-network/client

# Fund your bundler account (accepts crypto or fiat)
bundlr fund 0.01 -c arweave

# Upload a file
bundlr upload book.txt -c arweave
# Returns: https://arweave.net/[transaction-id]
```

### Integration Code

```python
#!/usr/bin/env python3
"""Arweave permanent storage for books"""

import subprocess
import json
from pathlib import Path

def upload_to_arweave(book_path: Path, metadata: dict) -> dict:
    """Upload book to Arweave permanent storage"""
    
    # Add metadata tags
    tags = [
        f"-t", "Content-Type", "text/plain",
        f"-t", "App-Name", "ImmutablePublishing",
        f"-t", "Title", metadata.get('title', 'Unknown'),
        f"-t", "Author", metadata.get('author', 'Unknown'),
        f"-t", "SHA256", metadata.get('hash', ''),
        f"-t", "Source", metadata.get('source', 'Project Gutenberg'),
    ]
    
    result = subprocess.run(
        ['bundlr', 'upload', str(book_path), '-c', 'arweave'] + tags,
        capture_output=True, text=True
    )
    
    # Parse transaction ID from output
    # Output format: "Uploaded to https://arweave.net/[txid]"
    txid = result.stdout.split('/')[-1].strip()
    
    return {
        'arweave_txid': txid,
        'arweave_url': f'https://arweave.net/{txid}',
        'ar_protocol': f'ar://{txid}',
        'permanent': True
    }

def verify_arweave(txid: str, expected_hash: str) -> dict:
    """Verify content on Arweave matches expected hash"""
    
    import requests
    import hashlib
    
    url = f'https://arweave.net/{txid}'
    response = requests.get(url)
    
    actual_hash = hashlib.sha256(response.content).hexdigest()
    
    return {
        'matches': actual_hash == expected_hash,
        'expected': expected_hash,
        'actual': actual_hash,
        'url': url
    }
```

---

## Revised Business Model

### Free Public Good Tier

Everything needed to verify and read original texts:

| Asset | Location | Cost to User |
|-------|----------|--------------|
| Full text (TXT) | Arweave | Free |
| PDF (formatted) | Website + IPFS | Free |
| EPUB | Website + IPFS | Free |
| OTS proof file | Website + GitHub | Free |
| Verification tool | Open source | Free |
| Hash registry | Public API | Free |

**Funding:** Donations, grants, institutional sponsors

### Sustainable Revenue Tier

Convenience and physical goods:

| Product | Price | Margin | Notes |
|---------|-------|--------|-------|
| Kindle delivery | $0.99 | 35% | One-click convenience |
| Kindle (premium format) | $2.99 | 70% | Enhanced formatting |
| Paperback | $9.99-14.99 | $2-5 | Print-on-demand |
| Hardcover | $19.99-29.99 | $5-10 | Collectors |
| Library bundle | $99/year | 100% | Institutional access |
| "Patron" tier | $5-50/month | 100% | Supporter donations |

### Funding Mix Target

```
Year 1:
├── Kindle/POD sales: 60%
├── Donations: 30%
└── Grants: 10%

Year 3 (mature):
├── Kindle/POD sales: 40%
├── Donations: 25%
├── Grants: 15%
├── Institutional: 15%
└── Merchandise: 5%
```

---

## Technical Architecture (Updated)

```
┌─────────────────────────────────────────────────────────────────┐
│                     IMMUTABLE PUBLISHING STACK                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   SOURCE    │───▶│  PIPELINE   │───▶│   OUTPUT    │         │
│  │             │    │             │    │             │         │
│  │ • Gutenberg │    │ • Hash      │    │ • Arweave   │         │
│  │ • Std Ebooks│    │ • Format    │    │ • IPFS      │         │
│  │ • IA Scans  │    │ • OTS Stamp │    │ • Website   │         │
│  └─────────────┘    │ • Arweave   │    │ • KDP       │         │
│                     │ • KDP Prep  │    │ • GitHub    │         │
│                     └─────────────┘    └─────────────┘         │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    PUBLIC REGISTRY                       │   │
│  │                                                          │   │
│  │  {                                                       │   │
│  │    "title": "Alice's Adventures in Wonderland",         │   │
│  │    "author": "Lewis Carroll",                           │   │
│  │    "sha256": "a7f8c3d2e1b4...",                        │   │
│  │    "bitcoin_block": 871234,                             │   │
│  │    "bitcoin_timestamp": "2026-01-17T06:00:00Z",        │   │
│  │    "arweave_txid": "Abc123...",                        │   │
│  │    "downloads": {                                       │   │
│  │      "txt": "ar://Abc123...",                          │   │
│  │      "pdf": "https://immutablepub.com/dl/alice.pdf",   │   │
│  │      "epub": "ipfs://Qm...",                           │   │
│  │      "ots": "https://immutablepub.com/proofs/alice.ots"│   │
│  │    }                                                    │   │
│  │  }                                                      │   │
│  │                                                          │   │
│  │  API: GET /api/v1/books/{id}                           │   │
│  │  Bulk: GET /api/v1/registry.json                       │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Updated Pipeline Script

```bash
#!/bin/bash
# publish.sh - Full pipeline with blockchain integration

BOOK_ID=$1
TITLE=$2
AUTHOR=$3

echo "📚 Immutable Publishing Pipeline (Blockchain Edition)"
echo "======================================================"

# 1. Fetch source
echo "1. Fetching from Project Gutenberg..."
python3 fetch_book.py $BOOK_ID

# 2. Generate hash
echo "2. Computing SHA-256..."
HASH=$(sha256sum books/$BOOK_ID/content.txt | cut -d' ' -f1)
echo "   Hash: $HASH"

# 3. Bitcoin timestamp (OpenTimestamps)
echo "3. Creating Bitcoin timestamp..."
ots stamp books/$BOOK_ID/content.txt
echo "   Proof: books/$BOOK_ID/content.txt.ots"
echo "   (Will confirm in ~1-2 hours when Bitcoin block is mined)"

# 4. Permanent storage (Arweave)
echo "4. Uploading to Arweave..."
ARWEAVE_URL=$(bundlr upload books/$BOOK_ID/content.txt -c arweave \
    -t Content-Type text/plain \
    -t App-Name ImmutablePublishing \
    -t Title "$TITLE" \
    -t Author "$AUTHOR" \
    -t SHA256 "$HASH" | grep -o 'https://arweave.net/[^ ]*')
echo "   Permanent URL: $ARWEAVE_URL"

# 5. Generate formats
echo "5. Generating EPUB and PDF..."
python3 generate_epub.py books/$BOOK_ID "$TITLE" "$AUTHOR"
python3 generate_pdf.py books/$BOOK_ID "$TITLE" "$AUTHOR"

# 6. Generate cover
echo "6. Generating cover..."
python3 generate_cover.py "$TITLE" "$AUTHOR" books/$BOOK_ID/cover.jpg

# 7. Upload to IPFS (redundancy)
echo "7. Pinning to IPFS..."
IPFS_CID=$(ipfs add -q books/$BOOK_ID/content.txt)
echo "   IPFS: ipfs://$IPFS_CID"

# 8. Update registry
echo "8. Updating public registry..."
python3 registry.py add \
    --id "$BOOK_ID" \
    --title "$TITLE" \
    --author "$AUTHOR" \
    --hash "$HASH" \
    --arweave "$ARWEAVE_URL" \
    --ipfs "$IPFS_CID" \
    --ots "books/$BOOK_ID/content.txt.ots"

# 9. Generate static site
echo "9. Regenerating website..."
python3 registry.py generate-site

# 10. Deploy
echo "10. Deploying to GitHub Pages..."
cd docs && git add -A && git commit -m "Add $TITLE" && git push

echo ""
echo "✅ PUBLISHED!"
echo ""
echo "Free downloads:"
echo "  Arweave (permanent): $ARWEAVE_URL"
echo "  IPFS: ipfs://$IPFS_CID"
echo "  Website: https://immutablepublishing.com/books/$BOOK_ID"
echo ""
echo "Verification:"
echo "  Hash: $HASH"
echo "  OTS Proof: books/$BOOK_ID/content.txt.ots"
echo "  Bitcoin confirmation: pending (~1-2 hours)"
echo ""
echo "KDP upload (manual): books/$BOOK_ID/*.epub + cover.jpg"
```

---

## Estimated Blockchain Costs

### Per Book

| Service | Cost | Notes |
|---------|------|-------|
| OpenTimestamps | $0.00 | Free forever |
| Arweave (~50KB text) | $0.01 | One-time, permanent |
| IPFS pinning | $0.00 | Free via Pinata free tier |
| **Total** | **~$0.01** | Per book, forever |

### At Scale (1000 books)

| Item | Cost |
|------|------|
| Arweave storage | $10 |
| Domain | $12/year |
| Hosting | $0 (GitHub Pages) |
| **Total Year 1** | **~$22** |

---

## Governance & Legal Structure

### Recommended: Non-Profit (501c3) or Public Benefit Corp

**Why:**
- Aligns with public good mission
- Eligible for grants (NEH, Mellon, Sloan)
- Tax-deductible donations
- Credibility with institutions

**Board composition:**
- Librarians / archivists
- Digital humanities scholars
- Open source advocates
- Legal (copyright/public domain experts)

### Alternative: DAO

For full decentralization:
- Treasury controlled by token holders
- Proposals for new books to publish
- Community-driven curation

**Pros:** Censorship-resistant, community-owned
**Cons:** Regulatory uncertainty, harder to get grants

---

## Partnerships

### Natural Allies

| Organization | Collaboration |
|--------------|---------------|
| Internet Archive | Source texts, cross-promotion |
| Project Gutenberg | Official verification partner |
| Standard Ebooks | Share formatting, co-brand |
| EFF | Advocacy, legal support |
| Library of Congress | Credibility, grants |
| Universities | Research partnerships |

### Potential Funders

- **NEH (National Endowment for Humanities)** - Digital humanities grants
- **Mellon Foundation** - Library/archive funding
- **Sloan Foundation** - Tech + public interest
- **Knight Foundation** - Media/information integrity
- **Protocol Labs** - IPFS/Filecoin ecosystem grants
- **Arweave** - Permanent storage grants

---

## Manifesto Draft

> ### The Immutable Publishing Manifesto
>
> The great works of human literature belong to everyone.
>
> When a publisher silently changes Roald Dahl, when Amazon deletes 1984 from your Kindle, when sensitivity readers erase the past—we lose more than words. We lose our ability to understand where we came from.
>
> We believe:
>
> 1. **Original texts should be freely available.** If it's in the public domain, it should be free to read.
>
> 2. **Verification should be trustless.** You shouldn't have to trust us, or anyone. Verify against Bitcoin. Verify against Arweave. The math doesn't lie.
>
> 3. **Preservation is a public good.** We exist to serve readers, scholars, and future generations—not shareholders.
>
> 4. **Technology should serve culture.** Blockchains aren't just for speculation. They're tools for permanent, uncensorable truth.
>
> We publish books that cannot be unpublished.
> We preserve texts that cannot be altered.
> We verify truth that cannot be denied.
>
> **Immutable Publishing**
> *Verified. Permanent. Free.*

---

## Summary

Adding blockchain transforms this from "business with a trust problem" to "public infrastructure for cultural preservation."

**Cost increase:** ~$0.01 per book
**Credibility increase:** Massive
**Mission alignment:** Perfect

The paid tiers (Kindle, print) sustain the free public good. The blockchain ensures it survives regardless of what happens to the company.

This is how you build something that matters.

🐱
