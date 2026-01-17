# Immutable Publishing - TODO

## 🔴 High Priority

### Clarify Verification UX
- [ ] **PDF:** Add clear explanation that hash is of `content.txt` (original text), not the PDF itself
- [ ] **PDF:** Include instructions: "To verify: download content.txt, run `shasum -a 256 content.txt`"
- [ ] **Website:** Add "How It Works" section explaining what's hashed vs what's formatted
- [ ] **Website:** Add link to download original text file alongside PDF
- [ ] **Website:** Show verification command example
- [ ] Create a `/verify/{book-id}` page with step-by-step instructions

### Complete Bitcoin Timestamp
- [ ] Wait for 6 confirmations on transaction `baa98222981f16ae...`
- [ ] Run `ots upgrade content.txt.ots` to complete the proof
- [ ] Update metadata.json with confirmed block number
- [ ] Update website to show "✓ Confirmed in Block #XXXXX"

## 🟡 Medium Priority

### Improve Book Publishing Pipeline
- [ ] Add Arweave upload for permanent storage (~$0.01/book)
- [ ] Automate cover generation with better templates
- [ ] Fix EPUB generation (currently broken)
- [ ] Create `publish.sh` one-command script
- [ ] Add IPFS pinning for redundancy

### Website Improvements
- [ ] Add individual book pages (`/books/alice-wonderland/`)
- [ ] Mobile-responsive design improvements
- [ ] Add OpenTimestamps verification widget (client-side .ots verification)
- [ ] SEO meta tags
- [ ] Add favicon + social preview image
- [ ] Analytics (privacy-respecting, e.g., Plausible)

### Content
- [ ] Publish 5 more pilot titles (see business plan for list)
- [ ] Create "About" page with manifesto
- [ ] Create "How to Verify" tutorial page
- [ ] Add FAQ section

## 🟢 Lower Priority / Future

### Business & Legal
- [ ] Decide on entity structure (non-profit vs public benefit corp)
- [ ] Set up donation/support page (Ko-fi, GitHub Sponsors, etc.)
- [ ] Research grant opportunities (NEH, Mellon, Sloan)
- [ ] Write proper Terms of Service / License page

### Technical Enhancements
- [ ] Set up CI/CD for automated deployments
- [ ] Create GitHub Action to auto-verify timestamps
- [ ] Build CLI tool for publishers to submit books
- [ ] API endpoint for programmatic verification
- [ ] Blockchain explorer integration (link to Bitcoin tx)

### Marketing & Launch
- [ ] Write launch blog post / Twitter thread
- [ ] Submit to Hacker News
- [ ] Reach out to r/books, r/literature, r/privacy
- [ ] Contact Internet Archive / Project Gutenberg for potential partnership
- [ ] Press outreach (Ars Technica, Wired, etc.)

### Kindle/Print
- [ ] Upload Alice to KDP as paid convenience option
- [ ] Set up print-on-demand via KDP Print
- [ ] Design premium hardcover editions

---

## ✅ Completed

- [x] Register domain (immutablepublishing.com)
- [x] Create GitHub repo
- [x] Deploy to GitHub Pages
- [x] Configure DNS
- [x] Publish first book (Alice in Wonderland)
- [x] Generate SHA-256 hash
- [x] Submit to OpenTimestamps (Bitcoin timestamp)
- [x] Create verification website
- [x] Create book cover
- [x] Write business plan
- [x] Write blockchain addendum

---

## Notes

**Hash Clarification Text (draft for PDF/website):**

> **What is verified?**
> 
> The SHA-256 hash verifies the original text file (`content.txt`), not this formatted PDF. The PDF includes additional formatting, our verification page, and branding - none of which affect the hash.
>
> **To verify yourself:**
> 1. Download `content.txt` from immutablepublishing.com/books/alice/content.txt
> 2. Run: `shasum -a 256 content.txt`
> 3. Compare the hash to: `4e04ea77acf3b0215cae2089c977bc07526fc834597a1d463598d895354ba41d`
> 4. Verify the Bitcoin timestamp: `ots verify content.txt.ots`
>
> The hash matches? You have the authentic, unaltered original text.

---

*Last updated: 2026-01-17*
