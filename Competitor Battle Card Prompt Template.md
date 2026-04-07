# Competitor Design Battle Card — Reusable Claude Prompt Template

> **How to use this prompt:** Copy the entire prompt below (everything inside the `---` fences) into a new Claude conversation. Replace the `[PLACEHOLDER]` fields with the competitor's information. Claude will produce a standalone HTML Design Battle Card matching the exact format used for the Abridge card.

> **Requirements:** Claude in Chrome extension must be connected for live CSS extraction. If Chrome is not available, Claude will use web search and document what could not be verified live.

> **Aggregation:** Each card is saved as `[CompetitorName] Design Battle Card.html`. Once all competitors are complete, a separate aggregation prompt (included at the bottom of this document) can combine them into a single competitive overview.

---

## THE PROMPT — COPY BELOW THIS LINE

---

```
You are acting as a Creative Director, Art Director, and Senior Designer conducting a competitive brand audit for a B2B SaaS Healthcare AI rebrand. Your task is to build a **Design Battle Card** as a standalone HTML file for the competitor described below.

## COMPETITOR DETAILS
- **Competitor Name:** [COMPETITOR NAME]
- **Website URL:** [COMPETITOR URL, e.g. https://www.example.com]
- **Brief Description (optional):** [One-liner about what they do, e.g. "AI-powered clinical documentation platform"]

## WHAT YOU MUST PRODUCE

A single self-contained HTML file named **"[CompetitorName] Design Battle Card.html"** saved to the workspace folder. It must follow the exact structure, visual format, and section order defined below. No deviation.

## PROCESS — FOLLOW THESE STEPS IN ORDER

### Step 1: Research the Competitor
- Use **web search** to find: company description, tagline, mission/vision, design agency (if known), brand guide or identity case study links, LinkedIn, app store listings, press coverage of any rebrand.
- Collect all URLs you find — they go in Section 6.

### Step 2: Extract Live Brand Data via Chrome
- Navigate to the competitor's website using the Chrome extension.
- Run the following JavaScript extraction via the Chrome javascript_tool:

```javascript
(() => {
  const cs = getComputedStyle(document.documentElement);
  const allVars = {};
  for (const sheet of document.styleSheets) {
    try {
      for (const rule of sheet.cssRules) {
        if (rule.selectorText === ':root' || rule.selectorText === 'html') {
          for (const prop of rule.style) {
            if (prop.startsWith('--')) {
              allVars[prop] = rule.style.getPropertyValue(prop).trim();
            }
          }
        }
      }
    } catch(e) {}
  }

  const body = getComputedStyle(document.body);
  const h1 = document.querySelector('h1');
  const h1Style = h1 ? getComputedStyle(h1) : null;
  const navLink = document.querySelector('nav a');
  const navStyle = navLink ? getComputedStyle(navLink) : null;
  const p = document.querySelector('p');
  const pStyle = p ? getComputedStyle(p) : null;

  return JSON.stringify({
    cssVariables: allVars,
    bodyFont: body.fontFamily,
    bodyColor: body.color,
    bodyBg: body.backgroundColor,
    h1Font: h1Style?.fontFamily,
    h1Size: h1Style?.fontSize,
    h1Weight: h1Style?.fontWeight,
    h1Color: h1Style?.color,
    h1Transform: h1Style?.textTransform,
    h1Text: h1?.textContent?.trim()?.substring(0, 120),
    navFont: navStyle?.fontFamily,
    pFont: pStyle?.fontFamily,
    pSize: pStyle?.fontSize,
    pColor: pStyle?.color
  }, null, 2);
})()
```

- Then extract the logo:

```javascript
(() => {
  const logos = [];
  // Check header area
  const headerImgs = document.querySelectorAll('header img, nav img, [class*="logo"] img, a[href="/"] img');
  headerImgs.forEach(img => {
    logos.push({ src: img.src, alt: img.alt, width: img.naturalWidth, height: img.naturalHeight });
  });
  // Check SVG logos
  const svgLogos = document.querySelectorAll('header svg, nav svg, [class*="logo"] svg');
  svgLogos.forEach((svg, i) => {
    logos.push({ type: 'svg', index: i, viewBox: svg.getAttribute('viewBox'), class: svg.className?.baseVal });
  });
  return JSON.stringify(logos, null, 2);
})()
```

- If the logo is an image URL (not inline SVG), record the full CDN/asset URL.
- If it's an inline SVG, note that and describe the logo visually.

### Step 3: Determine Brand Colors
From the CSS variables and computed styles, identify and categorize:
- **Primary accent color(s)** — the most prominent non-neutral color (used in CTAs, links, brand marks)
- **Dark colors** — blacks, near-blacks, dark navies used for text and backgrounds
- **Light colors** — whites, off-whites, creams used for backgrounds
- **Mid-tones** — grays or warm neutrals used for borders, secondary text
- **Any additional brand colors** — secondary accents, gradients, etc.

Record hex codes, RGB values, and CSS variable names for each.

### Step 4: Determine Typography
Identify all font stacks used:
- **Display/Headline font** — what renders on H1s
- **Body/UI font** — what renders on paragraphs, nav, buttons
- **Any fallback or specialty stacks**
- Note if any font is custom/proprietary (loaded from the site's own domain, not Google Fonts or Adobe)
- Record the CSS `font-family` declaration, `font-weight`, `font-size`, and `text-transform` for each

#### Step 4b: Proprietary / Licensed Font Handling (REQUIRED when custom fonts are detected)

If ANY font in the competitor's stack is proprietary, licensed, or not freely available (i.e., not on Google Fonts, Adobe Fonts free tier, or standard system defaults), you MUST perform ALL of the following steps. This is not optional — every battle card must accurately represent how typography actually looks on the competitor's site.

**4b-1. Screenshot the actual font in use.**
Navigate to the competitor's live site and take TWO targeted screenshots using the Chrome extension:
- **Screenshot A — Headline:** Capture the hero H1 or a prominent display headline clearly showing the proprietary typeface at large size. Crop or frame the screenshot so the text is the focal point.
- **Screenshot B — Body text:** Capture a paragraph of body copy that shows the proprietary typeface at reading size.
- Save both screenshots as base64-encoded images (use `canvas.toDataURL()` via JavaScript, or take Chrome screenshots and embed them). These will be embedded directly in the HTML battle card.

**4b-2. Identify the closest freely available approximation.**
For each proprietary font detected, research and select the best visual match from Google Fonts or other free/open-source type libraries. Evaluate based on: letterform proportions, x-height, stroke contrast, terminal style (round vs. flat vs. angled), counter openness, and overall brand personality.

Common proprietary-to-approximation mappings (use as starting points, not gospel):
| Proprietary Font | Closest Free Approximation(s) | Key Differences |
|---|---|---|
| General Grotesque | DM Sans, Work Sans | DM Sans has tighter apertures; Work Sans is slightly more geometric |
| ABC Monument Grotesk | Inter, Archivo | Inter is more neutral; Archivo has more personality in italics |
| Founders Grotesk | Space Grotesk, Outfit | Space Grotesk is more geometric; Outfit is slightly softer |
| GT Walsheim | Nunito Sans, Rubik | Both are rounder than GT Walsheim; Rubik has squarer terminals |
| Graphik | Be Vietnam Pro, Public Sans | Public Sans is narrower; Be Vietnam Pro matches weight distribution better |
| Neue Haas Grotesk | Helvetica Neue (system), Inter | Inter is wider; system Helvetica is close but varies by OS |
| Garnett | Sora, Outfit | Sora matches the geometric feel; both lack Garnett's ink traps |
| Söhne | Inter, DM Sans | DM Sans is closest in warmth; Inter is more neutral |
| Reckless Neue | Lora, Source Serif 4 | Both are more traditional; lack Reckless's contemporary sharpness |

If the competitor's font isn't in this table, search for "[FontName] alternative free" or "[FontName] similar Google Fonts" and evaluate the results. Document your reasoning.

**4b-3. Add the approximation font to the Google Fonts import.**
Update the `@import url(...)` at the top of the battle card's `<style>` to include any additional Google Fonts families needed for the approximation. For example, if you're adding Work Sans:
```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=DM+Sans:wght@400;500;700&family=Space+Grotesk:wght@400;500;600;700&family=Work+Sans:wght@400;500;600;700&display=swap');
```

**4b-4. Build a visual comparison block in Section 3.**
For EACH proprietary font, include a **Font Comparison Panel** in the Typography System section using this exact HTML structure:

```html
<div class="font-comparison">
  <div class="font-comparison-header">
    <h4>Visual Comparison: [Proprietary Font Name]</h4>
    <span class="badge-approximation">⚠ Approximation Required</span>
  </div>
  <div class="font-comparison-grid">
    <div class="font-compare-col actual">
      <div class="compare-label">ACTUAL — from live site</div>
      <img src="data:image/jpeg;base64,[BASE64_SCREENSHOT]" alt="[FontName] as rendered on [competitor] website" class="font-screenshot" />
      <div class="compare-meta">[ProprietaryFontName] · Licensed from [Foundry if known]</div>
    </div>
    <div class="font-compare-col approx">
      <div class="compare-label">APPROXIMATION — [ApproxFontName] (Google Fonts)</div>
      <div class="approx-render headline" style="font-family: '[ApproxFontName]', sans-serif; font-size: 42px; font-weight: 600;">
        The quick brown fox jumps over the lazy dog
      </div>
      <div class="approx-render body" style="font-family: '[ApproxFontName]', sans-serif; font-size: 16px;">
        The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs. How vexingly quick daft zebras jump.
      </div>
      <div class="compare-meta">[ApproxFontName] · Free via Google Fonts</div>
    </div>
  </div>
  <div class="comparison-note">
    <strong>Why this approximation:</strong> [1-2 sentences explaining why this is the closest match and where it differs — e.g., "DM Sans shares General Grotesque's wide apertures and friendly warmth, but has slightly more geometric 'a' and 'g' letterforms and lacks the distinctive ink-trap details at joints."]
  </div>
</div>
```

Required CSS for the comparison block (add to the card's `<style>`):
```css
.font-comparison {
  border: 2px solid #F59E0B;
  border-radius: 12px;
  overflow: hidden;
  margin-bottom: 24px;
}
.font-comparison-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 24px;
  background: #FFFBEB;
  border-bottom: 1px solid #FDE68A;
}
.font-comparison-header h4 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 16px;
  font-weight: 700;
  color: #92400E;
  margin: 0;
}
.badge-approximation {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 5px 14px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: 600;
  background: #FEF3C7;
  color: #92400E;
}
.font-comparison-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0;
}
.font-compare-col {
  padding: 24px;
}
.font-compare-col.actual {
  background: #F9FAFB;
  border-right: 1px solid #E5E7EB;
}
.font-compare-col.approx {
  background: #FFFFFF;
}
.compare-label {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-bottom: 16px;
  padding: 4px 10px;
  border-radius: 4px;
  display: inline-block;
}
.actual .compare-label {
  background: #D1FAE5;
  color: #065F46;
}
.approx .compare-label {
  background: #FEF3C7;
  color: #92400E;
}
.font-screenshot {
  width: 100%;
  border-radius: 8px;
  border: 1px solid #E5E7EB;
  margin-bottom: 12px;
}
.approx-render.headline {
  line-height: 1.2;
  color: #111827;
  margin-bottom: 16px;
  letter-spacing: -0.02em;
}
.approx-render.body {
  color: #4B5563;
  line-height: 1.7;
  margin-bottom: 12px;
}
.compare-meta {
  font-size: 11px;
  color: #9CA3AF;
  font-style: italic;
}
.comparison-note {
  padding: 16px 24px;
  background: #FFFBEB;
  border-top: 1px solid #FDE68A;
  font-size: 13px;
  color: #92400E;
  line-height: 1.7;
}
@media (max-width: 768px) {
  .font-comparison-grid { grid-template-columns: 1fr; }
  .font-compare-col.actual { border-right: none; border-bottom: 1px solid #E5E7EB; }
}
```

**4b-5. Flag every approximated type specimen.**
On EACH type specimen card in Section 3 that uses a substitute font (i.e., rendering a Google Font instead of the actual proprietary one), include a visible amber badge directly below the font name:
```html
<div class="badge-approximation" style="margin-top:6px; margin-bottom:12px;">
  ⚠ Approximation — actual font is [ProprietaryFontName] ([FoundryName])
</div>
```
This ensures the reader always knows: "What I'm seeing here is NOT the competitor's real font — it's the nearest free equivalent."

**4b-6. If Chrome screenshots are not possible,** fall back to describing the font visually in the comparison panel's left column instead of embedding a screenshot. Use specific typographic vocabulary: terminal style, aperture width, stroke modulation, x-height, ascender/descender proportions, and overall texture. Flag the comparison as "Description-based — no screenshot available" using a warning-note box.

### Step 5: Build the HTML Battle Card

Use the EXACT structure below. The CSS framework is standardized — only the `:root` color variables, content, and competitor-specific data change between cards.

---

## REQUIRED HTML STRUCTURE (6 Sections, exact order)

### CSS Variables Block
At the top of the `<style>`, define competitor-specific color variables following this naming pattern:
```css
:root {
  /* ═══ VERIFIED [CompetitorName] Brand Colors (from live CSS, [Month Year]) ═══ */
  --comp-primary: #[HEX];        /* Primary accent — describe usage */
  --comp-primary-alt: #[HEX];    /* Variant if exists */
  --comp-black: #[HEX];          /* Pure/near black */
  --comp-dark: #[HEX];           /* Secondary dark */
  --comp-body-text: #[HEX];      /* Body text color */
  --comp-white: #[HEX];          /* Primary background */
  --comp-light: #[HEX];          /* Off-white/cream */
  --comp-light-gray: #[HEX];     /* Light surface */
  --comp-mid-gray: #[HEX];       /* Mid-tone */
  /* Add more as discovered */

  /* Battle Card UI — NEVER CHANGE THESE */
  --card-bg: #FAFAFA;
  --card-border: #E5E7EB;
  --section-bg: #FFFFFF;
  --badge-verify: #FEF3C7;
  --badge-verify-text: #92400E;
  --badge-confirmed: #D1FAE5;
  --badge-confirmed-text: #065F46;
}
```

### HEADER
- Badge: "Design Battle Card"
- Date: "[Month] [Year] • Confidential"
- Company name in large display type
- Tagline/descriptor
- Link to website

### SECTION 1: Logo & Logomark
- 2x2 grid of logo previews on 4 backgrounds:
  1. White background (primary)
  2. Dark background (inverted if needed)
  3. Brand light/canvas background
  4. Brand primary accent color background
- Use the actual logo image URL from the CDN extraction. If SVG-only, embed or describe.
- Include a description block covering: logo type (wordmark, logomark, combination), design characteristics, design agency if known, asset URL source tag.
- Badge: "Verified from Live Site" (green) or "Needs Verification" (amber)

### SECTION 2: Brand Color Palette
- Grid of color swatch cards (one per color). Each card has:
  - Color preview block (100px tall) with hex overlay
  - Color name
  - Role/usage description
  - Hex, RGB, and CSS variable name
- Include a verified-note or warning-note at the bottom explaining extraction method and any key observations about the palette strategy.
- Badge: "Extracted from Live CSS" (green) or "Research-Based" (amber)

### SECTION 3: Typography System
- If a custom/proprietary font is detected, lead with a highlighted callout box (orange-bordered) explaining what it is and that it's proprietary.
- **If proprietary fonts are present:** Include the Font Comparison Panel(s) built in Step 4b-4, showing actual screenshots vs. approximation side by side for EACH proprietary font. This goes immediately after the callout box and before the individual type specimen cards.
- One type specimen card per font stack, containing:
  - Font name and role (e.g., "Body Copy — Navigation — UI")
  - **If the font is proprietary:** Amber approximation badge (from Step 4b-5) immediately below the font name, clearly stating which font is being approximated
  - Large display sample showing the font (use the approximation font identified in Step 4b-2)
  - Body-size sample paragraph
  - Full alphabet + numerals preview
  - CSS declaration details (showing the ACTUAL CSS from the competitor's site, not the approximation)
- Badge: "Verified from Live CSS" or "Research-Based"
- **If all fonts are freely available (Google Fonts, system fonts, etc.):** Skip Step 4b entirely. No comparison panels or approximation badges needed.

### SECTION 4: Brand Positioning & Messaging
- 2-column grid of positioning cards:
  - Hero Headline (from live site H1)
  - Mission (if found)
  - Vision (if found)
  - Category Descriptor
  - Design Agency
  - Key Trust Signal / Social Proof
- Fill in what you find. Leave unfound items as "Not publicly stated" rather than guessing.

### SECTION 5: Design Language & Visual System
- 3-column grid of attribute cards covering:
  - Primary Theme (light-first, dark-first, mixed)
  - Palette Strategy (monochrome + accent, multicolor, gradient-heavy, etc.)
  - Accent Color approach
  - Photography style
  - Graphic System / illustration approach
  - Identity type (static, generative, modular, etc.)
- **Strategic Callout Box** — this is the most important part. Include 5-7 bullet points analyzing:
  - What territory this competitor owns visually
  - What's distinctive about their approach
  - Where they follow vs. break healthcare conventions
  - Potential vulnerabilities or white space they leave open
  - Implications for your rebrand (what to avoid, what opportunity exists)

### SECTION 6: Brand References & Resources
- 2-column grid of reference link cards with icons:
  - Brand guide or design case study (if found)
  - Company website
  - About page
  - LinkedIn
  - App store listing (if applicable)
  - Any press about brand/rebrand
- Each card: icon, title, short description

### FOOTER
- "[CompetitorName] Design Battle Card • Prepared [Date] • Confidential — For Internal Rebrand Use Only"
- Smaller note: "Colors, fonts, and logo verified from [source] via [method]"

---

## VISUAL DESIGN RULES (DO NOT DEVIATE)

1. **Card UI font:** Inter (imported from Google Fonts) for all card chrome/UI text
2. **Display headings in the card UI:** Space Grotesk (imported from Google Fonts)
3. **Code/monospace:** DM Sans or system monospace
4. **Section numbers:** Colored squares using the competitor's primary accent color
5. **All colors in the card chrome** use the standardized `--card-*` variables, NOT the competitor's colors (except where showing the competitor's colors as specimens)
6. **Responsive grid:** Use CSS Grid with `auto-fit` and `minmax` for color swatches
7. **Hover states:** Subtle translateY(-2px) on swatch cards, border-color change on ref cards
8. **Border radius:** 12px for cards, 16px for main container sections, 8px for section number badges
9. **Google Fonts import:** `@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=DM+Sans:wght@400;500;700&family=Space+Grotesk:wght@400;500;600;700&display=swap');` — extend this import with any additional approximation fonts needed per Step 4b-3.

## DATA INTEGRITY RULES

1. **Mark everything with verification status.** If extracted from live CSS → green "Verified" badge. If from web research only → amber "Needs Verification" badge.
2. **Never guess colors.** If you can't extract from live CSS, say so explicitly and use the amber badge.
3. **Never fabricate font names.** If the font-family string says "CustomFont", report it as custom/proprietary and note the CSS declaration.
4. **Logo must be the actual asset** from the site if at all possible. If you can only get an SVG, embed it. If you truly cannot get the logo, use a placeholder and flag it clearly.
5. **Strategic callouts must be original analysis** based on what you observed, not generic statements. Reference specific colors, fonts, and design choices by name.
6. **Proprietary fonts must ALWAYS be flagged.** Never render an approximation font without a visible amber "Approximation" badge. The reader must always know when they are seeing a substitute vs. the real thing. Include the Font Comparison Panel for visual reference.

## OUTPUT

Save the completed HTML file as **"[CompetitorName] Design Battle Card.html"** in the workspace folder. The file must be fully self-contained (all CSS inline in `<style>`, no external stylesheets except Google Fonts import). It should render correctly when opened in any modern browser.
```

---

## AGGREGATION PROMPT (USE AFTER ALL COMPETITORS ARE COMPLETE)

> Copy this into a new Claude conversation once all individual battle cards are built.

---

```
You have access to a workspace folder containing multiple competitor Design Battle Card HTML files. Each file follows an identical 6-section structure with verified brand data.

Your task:
1. Read every file matching the pattern "*Design Battle Card.html" in the workspace folder.
2. Create a single **"Competitive Brand Landscape — Master Overview.html"** file that aggregates all competitors into one document with:

   **Section A: Color Landscape Matrix**
   - A visual grid showing every competitor's full color palette side by side
   - Each row = one competitor, each column = color role (primary accent, secondary, dark, light, etc.)
   - Show actual color swatches with hex codes

   **Section B: Typography Comparison**
   - Table: Competitor | Display Font | Body Font | Custom Y/N | Font Source | Approximation Used
   - Visual specimens of each competitor's headline treatment
   - Flag which specimens are approximations vs. actual fonts

   **Section C: Positioning Map**
   - Grid showing each competitor's hero headline, mission, and category descriptor side by side

   **Section D: Visual Identity Spectrum**
   - Show where each competitor falls on key axes:
     - Light <-> Dark
     - Minimal <-> Expressive
     - Conventional Healthcare <-> Tech-Forward
     - Single Accent <-> Multi-Color

   **Section E: White Space Analysis**
   - Synthesize all strategic callouts from individual cards
   - Identify visual territories that are crowded vs. open
   - Recommend 3-5 differentiation opportunities for the rebrand

   **Section F: Individual Card Links**
   - Link to each individual battle card HTML file for deep-dive reference

Save as **"Competitive Brand Landscape — Master Overview.html"** in the workspace folder. Same visual design language as the individual cards (Inter + Space Grotesk, clean card-based layout, 12px border radius).
```

---

## COMPETITOR CHECKLIST

Use this to track which competitors have been completed. Copy into your project tracker:

| # | Competitor Name | Website | Battle Card Built? | Verified via Chrome? |
|---|----------------|---------|-------------------|---------------------|
| 1 | Abridge | abridge.com | ✅ Complete | ✅ Yes |
| 2 | Suki | suki.ai | ✅ Complete | ✅ Yes |
| 3 | [Name] | [URL] | ☐ | ☐ |
| 4 | [Name] | [URL] | ☐ | ☐ |
| 5 | [Name] | [URL] | ☐ | ☐ |
| 6 | [Name] | [URL] | ☐ | ☐ |
| 7 | [Name] | [URL] | ☐ | ☐ |
| 8 | [Name] | [URL] | ☐ | ☐ |
| 9 | [Name] | [URL] | ☐ | ☐ |
| 10 | [Name] | [URL] | ☐ | ☐ |
| 11 | [Name] | [URL] | ☐ | ☐ |
| 12 | [Name] | [URL] | ☐ | ☐ |
| 13 | [Name] | [URL] | ☐ | ☐ |

---

## QUICK-START EXAMPLE

To build the next card, paste the main prompt above into a new Claude conversation and fill in like this:

```
## COMPETITOR DETAILS
- **Competitor Name:** Nuance (DAX Copilot)
- **Website URL:** https://www.nuance.com/healthcare/dragon-ambient-experience.html
- **Brief Description:** Microsoft-backed AI clinical documentation via DAX Copilot
```

Claude will handle the rest — research, extraction, and HTML generation — following the standardized template.
