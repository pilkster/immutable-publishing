# Immutable Publishing - Automated Pipeline v2.0

## Vision
A fully automated system that continuously publishes high-quality, Bitcoin-timestamped editions of public domain books with:
- Professional AI-generated covers
- Rich metadata (author bio, historical context)
- Multiple formats (PDF, EPUB, Kindle, Audiobook)
- Permanent blockchain verification

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     IMMUTABLE PUBLISHING VPS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │   INGESTION  │───▶│  ENRICHMENT  │───▶│  PRODUCTION  │       │
│  └──────────────┘    └──────────────┘    └──────────────┘       │
│         │                   │                   │                │
│         ▼                   ▼                   ▼                │
│  • Gutenberg API     • Author Bio        • Cover Gen            │
│  • Internet Archive  • Book Context      • PDF/EPUB/MOBI        │
│  • HathiTrust        • Genre Tags        • Audiobook TTS        │
│  • Manual Queue      • Era Research      • Bitcoin Timestamp    │
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │ DISTRIBUTION │◀───│   REGISTRY   │◀───│  VERIFICATION│       │
│  └──────────────┘    └──────────────┘    └──────────────┘       │
│         │                   │                   │                │
│         ▼                   ▼                   ▼                │
│  • GitHub Pages      • SQLite DB         • OTS Upgrade          │
│  • Amazon KDP        • JSON Catalog      • Block Confirm        │
│  • Audible/Direct    • Search Index      • Hash Verify          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Ingestion Pipeline

### Sources
| Source | API | Rate Limit | Quality |
|--------|-----|------------|---------|
| Project Gutenberg | gutenberg.org/ebooks/N.txt.utf-8 | None | High (curated) |
| Internet Archive | archive.org/metadata/ID | 100/min | Variable |
| Standard Ebooks | standardebooks.org | None | Excellent |
| HathiTrust | babel.hathitrust.org | Auth req | Academic |

### New Release Monitor
```python
# Daily cron: Check for newly available public domain works
# Each Jan 1: Bulk import works from (current_year - 95)

def get_new_pd_works(year):
    """Fetch works entering public domain from given year"""
    # Check Duke Law PD Day list
    # Cross-reference with Gutenberg catalog
    # Queue for processing
```

### Intake Queue Schema
```json
{
  "id": "pride-prejudice",
  "title": "Pride and Prejudice",
  "author": "Jane Austen",
  "author_birth": 1775,
  "author_death": 1817,
  "publication_year": 1813,
  "pd_year": 1909,  // When it entered US public domain
  "genre": ["romance", "classic", "regency"],
  "source": "gutenberg",
  "source_id": 1342,
  "language": "en",
  "priority": 1,
  "status": "pending"
}
```

---

## 2. Enrichment Pipeline

### Author Research Agent
Uses Claude/GPT to generate:
- **Short bio** (100 words) - for back cover
- **Long bio** (500 words) - for "About the Author" section
- **Historical context** - era, literary movement, influences
- **Notable facts** - interesting tidbits for marketing

### Book Research Agent
Generates:
- **Synopsis** (150 words) - no spoilers
- **Historical significance** - why this book matters
- **Publication history** - original publication, reception
- **Reading level** - estimated grade level
- **Content warnings** - if applicable (dated language, etc.)

### Genre Classification
Auto-tag with consistent taxonomy:
```
Primary: Fiction | Non-Fiction
Secondary: Romance, Mystery, Horror, Adventure, Philosophy, Science, etc.
Era: Ancient, Medieval, Renaissance, Enlightenment, Victorian, Modern
Mood: Dark, Light, Adventurous, Romantic, Philosophical, Humorous
```

---

## 3. Cover Generation System

### Design Philosophy
- **Consistent brand identity** across all books
- **Genre-appropriate aesthetics** (gothic for horror, elegant for classics)
- **Era-accurate visual language** (Regency, Victorian, etc.)
- **High-quality AI art** that looks professional, not "AI-generated"

### Cover Template System
```
┌─────────────────────────────┐
│  [Genre-specific border]    │
│                             │
│   ┌───────────────────┐     │
│   │                   │     │
│   │    AI ART PANEL   │     │
│   │   (scene/symbol)  │     │
│   │                   │     │
│   └───────────────────┘     │
│                             │
│   ════════════════════      │
│         TITLE               │
│   ════════════════════      │
│        Author Name          │
│                             │
│   [Immutable Publishing]    │
│   [Bitcoin Verified Badge]  │
└─────────────────────────────┘
```

### Genre Cover Styles

| Genre | Color Palette | Art Style | Typography |
|-------|---------------|-----------|------------|
| Romance | Blush, cream, gold | Soft, painterly | Elegant serif |
| Horror/Gothic | Deep purple, black, red | Dark, atmospheric | Victorian gothic |
| Mystery | Navy, gold, burgundy | Noir, shadowy | Bold serif |
| Adventure | Earth tones, blue | Dynamic, scenic | Strong sans |
| Philosophy | Cream, forest green | Minimal, symbolic | Classical |
| Sci-Fi | Silver, electric blue | Futuristic | Modern sans |
| Children's | Bright, warm | Whimsical, illustrated | Playful |

### Cover Generation Prompt Template
```
Create a book cover illustration for "{title}" by {author}.

Genre: {genre}
Era: {era}
Mood: {mood}

Visual elements to include:
- {key_scene_or_symbol}
- {era_appropriate_details}
- {color_palette}

Style: {art_style}, professional book cover quality
Composition: Vertical, with space for title text at bottom third
Do NOT include any text in the image.
```

### Cover Assembly Pipeline
1. Generate AI art panel (Nano Banana Pro / DALL-E / Midjourney)
2. Apply genre-specific border/frame template
3. Composite with typography (Pillow/ImageMagick)
4. Add "Immutable Publishing" branding
5. Add "Bitcoin Verified" badge
6. Generate spine art for print versions
7. Output: front cover, full wrap (for print), thumbnail

---

## 4. Production Pipeline

### Text Processing
```python
def process_book(raw_text):
    # 1. Strip Gutenberg header/footer
    # 2. Normalize whitespace and encoding
    # 3. Detect and mark chapters
    # 4. Generate table of contents
    # 5. Calculate reading stats (word count, reading time)
    # 6. Generate SHA-256 hash of clean text
    return processed_text, metadata
```

### Output Formats

#### PDF (Primary distribution)
- Professional typesetting (LaTeX or WeasyPrint)
- Embedded fonts (libre fonts: EB Garamond, Crimson Pro)
- Chapter headers with decorative elements
- Front matter: title page, copyright, about author, historical context
- Back matter: verification page, about Immutable Publishing

#### EPUB (E-readers)
- Valid EPUB 3.0
- Embedded cover
- Table of contents
- Metadata (author, date, ISBN-equivalent)
- Reflowable text

#### MOBI/KPF (Kindle)
- Converted from EPUB via Calibre/KindleGen
- Optimized for Kindle rendering
- Cover meets KDP requirements (2560x1600 min)

#### HTML (Website)
- Readable web version
- SEO-optimized
- Integrated verification widget

### Audiobook Production

#### TTS Options Analysis
| Service | Quality | Cost per Book* | Voices | Best For |
|---------|---------|----------------|--------|----------|
| ElevenLabs | Excellent | ~$75-150 | Cloned/Natural | Premium editions |
| OpenAI TTS | Very Good | ~$7-15 | 6 voices | Bulk production |
| Google Cloud TTS | Good | ~$4-8 | WaveNet | Cost-effective |
| Edge TTS | Fair | Free | Limited | Testing |
| Tortoise TTS | Variable | Free (compute) | Unlimited | Self-hosted |

*Estimated for 100k word book (~500k characters)

#### Audiobook Pipeline
```python
def generate_audiobook(text, voice_config):
    # 1. Split into chapters
    # 2. Add intro/outro narration
    # 3. Generate TTS for each chapter
    # 4. Apply audio processing (normalize, compress)
    # 5. Add chapter markers
    # 6. Generate MP3 (distribution) + M4B (Apple Books)
    # 7. Create audiobook cover (square format)
```

#### Voice Selection by Genre
| Genre | Voice Characteristics | Example Voice |
|-------|----------------------|---------------|
| Classic Literature | Warm, refined British | ElevenLabs "Adam" |
| Horror/Gothic | Deep, atmospheric | OpenAI "Onyx" |
| Romance | Warm, expressive | ElevenLabs "Rachel" |
| Adventure | Dynamic, energetic | OpenAI "Echo" |
| Philosophy | Calm, measured | OpenAI "Nova" |
| Children's | Bright, animated | Multiple for characters |

---

## 5. Verification & Timestamping

### Bitcoin Timestamp Flow
```
1. Generate SHA-256 of content.txt
2. Submit to OpenTimestamps calendars
3. Store pending .ots proof
4. CRON: Check for Bitcoin confirmations
5. After 6 confirmations: upgrade .ots proof
6. Update metadata with block number
7. Update website verification badge
```

### Verification Page (per book)
```
═══════════════════════════════════════════════
        CRYPTOGRAPHIC VERIFICATION
═══════════════════════════════════════════════

This text was permanently timestamped on the
Bitcoin blockchain, proving it existed unchanged
since [DATE].

SHA-256 Hash:
4e04ea77acf3b0215cae2089c977bc07...

Bitcoin Block: #[BLOCK_NUMBER]
Transaction: [TX_ID]
Timestamp: [BLOCK_TIMESTAMP]

To verify yourself:
1. Download content.txt
2. Run: shasum -a 256 content.txt
3. Compare hash above
4. Verify: ots verify content.txt.ots

═══════════════════════════════════════════════
```

---

## 6. Distribution Pipeline

### GitHub Pages (Free tier)
- Static site with all books
- Download links for all formats
- Search/browse interface
- Verification tools

### Amazon KDP (Kindle Store)
```python
def publish_to_kdp(book):
    # 1. Format EPUB to KDP specs
    # 2. Prepare metadata (title, description, keywords)
    # 3. Set price: $0.99 (convenience fee) or FREE with KDP Select
    # 4. Upload via KDP API or manual
    # 5. Track ASIN for linking
```

**KDP Strategy:**
- Price: $0.99 (minimum) or free during promotions
- All proceeds → operational costs or donated
- Link to free version on our site
- Clear description: "Free at immutablepublishing.com"

### Audiobook Distribution

#### ACX/Audible
- Requires exclusive or non-exclusive agreement
- Revenue share or pay-per-production
- Higher visibility but exclusivity concerns

#### Direct Distribution (Recommended)
- Host on own site + Patreon/Gumroad
- Distribute via podcast feeds (free)
- List on Libro.fm, Google Play Books
- No exclusivity requirements

---

## 7. Registry & Catalog

### SQLite Database Schema
```sql
CREATE TABLE books (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    publication_year INTEGER,
    genre TEXT,
    sha256 TEXT NOT NULL,
    bitcoin_block INTEGER,
    bitcoin_tx TEXT,
    ots_status TEXT,  -- pending, confirmed
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE formats (
    book_id TEXT,
    format TEXT,  -- pdf, epub, mobi, mp3, m4b
    file_path TEXT,
    file_size INTEGER,
    created_at TIMESTAMP
);

CREATE TABLE authors (
    id TEXT PRIMARY KEY,
    name TEXT,
    birth_year INTEGER,
    death_year INTEGER,
    bio_short TEXT,
    bio_long TEXT,
    wikipedia_url TEXT
);
```

### Public JSON Catalog
```json
{
  "catalog_version": "2.0",
  "last_updated": "2026-01-17T10:00:00Z",
  "total_books": 150,
  "books": [
    {
      "id": "pride-prejudice",
      "title": "Pride and Prejudice",
      "author": "Jane Austen",
      "year": 1813,
      "sha256": "b49a14027e84800c...",
      "bitcoin_block": 879234,
      "formats": ["pdf", "epub", "mobi", "mp3"],
      "download_url": "https://immutablepublishing.com/books/pride-prejudice/"
    }
  ]
}
```

---

## 8. VPS Infrastructure

### Recommended Stack
- **VPS**: DigitalOcean/Vultr 2GB RAM, 50GB SSD (~$12/mo)
- **OS**: Ubuntu 24.04 LTS
- **Runtime**: Python 3.12 + Node.js 20
- **Database**: SQLite (simple) or PostgreSQL (scalable)
- **Queue**: Redis + Celery for job processing
- **Web**: Nginx + static site generator

### Directory Structure
```
/opt/immutable-publishing/
├── bin/                    # CLI tools
│   ├── ingest.py
│   ├── enrich.py
│   ├── produce.py
│   └── publish.py
├── config/
│   ├── settings.yaml
│   └── secrets.env
├── data/
│   ├── queue/              # Pending books
│   ├── processing/         # In-progress
│   └── published/          # Complete
├── output/
│   ├── site/               # Generated website
│   └── kindle/             # KDP uploads
├── templates/
│   ├── covers/             # Cover templates
│   ├── pdf/                # PDF templates
│   └── epub/               # EPUB templates
└── logs/
```

### Cron Schedule
```cron
# Check for new public domain works (daily)
0 6 * * * /opt/immutable-publishing/bin/check-new-releases.py

# Process queue (every 2 hours)
0 */2 * * * /opt/immutable-publishing/bin/process-queue.py

# Upgrade OTS proofs (hourly)
0 * * * * /opt/immutable-publishing/bin/upgrade-timestamps.py

# Generate site & deploy (after processing)
30 */2 * * * /opt/immutable-publishing/bin/deploy-site.sh

# KDP upload check (weekly)
0 9 * * 1 /opt/immutable-publishing/bin/kdp-sync.py
```

---

## 9. Cost Analysis

### Monthly Operating Costs
| Item | Cost | Notes |
|------|------|-------|
| VPS (2GB) | $12 | DigitalOcean/Vultr |
| Domain | $1 | Already owned |
| AI Covers | $10-20 | ~500 images @ $0.02-0.04 |
| Audiobook TTS | $50-150 | ~10-20 books @ $7-15 |
| OpenTimestamps | $0 | Free |
| GitHub Pages | $0 | Free |
| **Total** | **~$75-185/mo** | |

### Per-Book Costs
| Item | Cost |
|------|------|
| Cover generation | $0.02-0.10 |
| PDF/EPUB generation | $0 (compute) |
| Audiobook (OpenAI TTS) | $7-15 |
| Bitcoin timestamp | $0 |
| **Total (with audio)** | **~$8-16** |
| **Total (no audio)** | **~$0.05-0.15** |

### Revenue Potential (Optional)
- KDP sales @ $0.99: ~$0.35/sale royalty
- Donations/Patreon
- Print-on-demand premium editions
- Grants (NEH, Mellon Foundation, etc.)

---

## 10. Implementation Phases

### Phase 1: Core Pipeline (Week 1-2)
- [ ] Set up VPS with basic stack
- [ ] Implement ingestion from Gutenberg
- [ ] Build text processing pipeline
- [ ] Integrate Nano Banana Pro for covers
- [ ] Generate PDF/EPUB outputs
- [ ] Deploy to GitHub Pages

### Phase 2: Enrichment (Week 3-4)
- [ ] Build author research agent
- [ ] Build book context agent
- [ ] Implement genre classification
- [ ] Create cover template system
- [ ] Add consistent typography/branding

### Phase 3: Distribution (Week 5-6)
- [ ] KDP integration
- [ ] Audiobook pipeline (OpenAI TTS)
- [ ] Public catalog API
- [ ] Search functionality
- [ ] Mobile-friendly site

### Phase 4: Automation (Week 7-8)
- [ ] New release monitoring
- [ ] Full cron automation
- [ ] OTS upgrade automation
- [ ] Alerting & monitoring
- [ ] Documentation

---

## Appendix: Sample Book Package

```
/books/pride-prejudice/
├── content.txt              # Original text (hashed)
├── content.sha256           # SHA-256 hash
├── content.txt.ots          # OpenTimestamps proof
├── metadata.json            # Full metadata
├── cover-front.jpg          # Front cover (300 DPI)
├── cover-full.jpg           # Full wrap for print
├── cover-thumb.jpg          # Thumbnail
├── pride-prejudice.pdf      # PDF edition
├── pride-prejudice.epub     # EPUB edition
├── pride-prejudice.mobi     # Kindle edition
├── pride-prejudice.mp3      # Audiobook (single file)
├── pride-prejudice.m4b      # Audiobook (chapters)
├── audiobook/
│   ├── 00-intro.mp3
│   ├── 01-chapter-1.mp3
│   └── ...
└── web/
    ├── index.html           # Book page
    ├── read.html            # Online reader
    └── verify.html          # Verification page
```

---

*Spec Version: 2.0*
*Last Updated: 2026-01-17*
*Author: Aineko 🐱*
