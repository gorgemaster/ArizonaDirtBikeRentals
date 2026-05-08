# WordPress → Astro Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Port the design in `/index.html` (project root) into Astro components, then build each migrated page using that design system — preserving URL slugs and SEO metadata from `crawl.csv`, and integrating WordPress content from `_migration/`.

**Architecture:** Astro static site with Tailwind v4 + CSS custom properties. The CSS design system lives in `global.css` (root vars, base styles, shared utilities). Component-specific layout CSS goes in each `.astro` file's `<style>` block. Images from `index.html` (`/images/`) are already at the project root and need to be copied to `public/images/`. WordPress uploads go in `public/images/YYYY/MM/`.

**Tech Stack:** Astro 6, Tailwind v4 (CSS-native), Google Fonts (Big Shoulders Display + DM Sans), Cloudflare Pages

---

## Design system (from `/index.html`)

### Color tokens
```css
--cream: #FDFCF9      /* page background */
--cream-soft: #FFFFFF  /* card backgrounds */
--cream-deep: #F2EFE7  /* alternate section bg */
--leather: #2A1F16    /* primary text */
--leather-soft: #55443A /* secondary text */
--ink: #17110C        /* header, footer, dark sections */
--rust: #E35A1C       /* CTA buttons, accents, highlights */
--rust-dark: #B8431A  /* rust hover state */
```

### Typography
- **Display**: `Big Shoulders Display` — 900 weight, uppercase, `line-height: 0.88`
- **Body**: `DM Sans` — 400/500/700 weights

### Key design components in `index.html`
1. **Header** — sticky, `var(--ink)` bg, 3px `var(--rust)` border-bottom, text logo + nav + phone CTA
2. **Hero** — fullscreen bg image `images/hero-main.webp` with directional gradient overlay, eyebrow + display h1 + sub + 2 CTAs
3. **Price strip** — `var(--rust)` bg, `$375` large display number, includes list, CTA
4. **Ride cards** — two large cards with image hover zoom, rust tag, display h3, border-top rust
5. **Reviews** — white cards with rust top border, star rating, quote, author
6. **Photo break** — fullscreen `images/hero-action.jpg` with overlay + display h2
7. **Why section** — 2-col grid, portrait image + stats (15+ years, 100+ reviews, $375)
8. **Meet Doug** — 2-col, square portrait, bio copy, CTA
9. **Final CTA** — dark fullscreen, display phone number
10. **Footer** — dark, rust top border, 3-col info

### Images at project root `/images/` (already exist, copy to `public/images/`)
- `hero-main.webp` — hero background
- `hero-action.jpg` — photo break section
- `families-group.jpg` — family ride card
- `advanced-canyon.jpg` — advanced ride card
- `father-son.jpg` — why section
- `doug-portrait.jpg` — Meet Doug
- `kid-rider.jpg`, `rider-forest.jpg`, `rider-ktm.jpg` — available for inner pages
- `tire-tread.png` — decorative

---

## Pages in scope (first 5 + infrastructure)

| Page | Astro file | URL slug | Notes |
|------|-----------|---------|-------|
| Home | `src/pages/index.astro` | `/` | Port from `index.html` |
| Contact | `src/pages/contact-us.astro` | `/contact-us/` | New design, WP content |
| Tours/Reservations | `src/pages/arizona-dirt-bike-tours-reservations.astro` | `/arizona-dirt-bike-tours-reservations/` | New design, WP content |
| Kids School | `src/pages/kids-dirt-bike-school.astro` | `/kids-dirt-bike-school/` | New design, WP content |
| Motorcycles | `src/pages/motorcycles.astro` | `/motorcycles/` | New design, WP content |

---

## SEO data (from `crawl.csv` — use exactly)

| Page | Title | Meta description |
|------|-------|-----------------|
| `/` | Arizona Dirt Bike Rentals - KTM & Husqvarna Motorcycle Tours & Training | Off-road motorcycle tours, training, and moto rentals. Ride the latest Husqvarna and KTM dirt bikes and the best Wickenburg, Phoenix, Arizona moto trails. |
| `/contact-us/` | Contact Us - Arizona Dirt Bike Rentals | Our Location Wickenburg, AZ is a super-cool western town where you can enjoy a true southwest experience. And we're less than an hour from Phoenix. We offer single-track guided moto-tours and motorcycle rentals seven days a week all year. Call Doug at 509-281-0156 if you have any questions about dirt bike rentals, |
| `/arizona-dirt-bike-tours-reservations/` | Arizona Dirt Bike Tours Reservations | Check out our exclusive Arizona dirt bike tours all inclusive rent and ride packages. Includes an experienced trail guide. |
| `/kids-dirt-bike-school/` | Kids Dirt Bike School - Arizona Dirt Bike Rentals | Give your kids an unforgettable experience |
| `/motorcycles/` | Motorcycles - Arizona Dirt Bike Rentals | We rent the latest Husqvarna and KTM dirt bikes in the Wickenburg, Phoenix, Arizona area. We have 2-strokes and 4-stroke motorcycles to choose from. |

---

## File structure

**Modify:**
- `src/styles/global.css` — add `:root` vars, base styles, shared utilities from `index.html`
- `src/layouts/Layout.astro` — add Google Fonts, Nav, Footer
- `src/pages/index.astro` — replace coming-soon with port of `index.html`

**Create:**
- `src/components/Nav.astro`
- `src/components/Footer.astro`
- `src/pages/contact-us.astro`
- `src/pages/arizona-dirt-bike-tours-reservations.astro`
- `src/pages/kids-dirt-bike-school.astro`
- `src/pages/motorcycles.astro`
- `public/_redirects`
- `scripts/copy-images.sh`

---

## Task 1: Copy design images and WordPress uploads to public/

**Files:**
- Create: `scripts/copy-images.sh`
- Populates: `public/images/` (design images + WordPress uploads)

The design images in `/images/` at the project root need to be at `public/images/`.
WordPress uploads in `_migration/uploads/YYYY/MM/` need to be at `public/images/YYYY/MM/`.

- [ ] **Step 1: Write the script**

```bash
#!/bin/bash
# scripts/copy-images.sh
set -e

# Copy design images from project root /images/ to public/images/
echo "Copying design images..."
cp -r images/. public/images/

# Copy WordPress media uploads by year
echo "Copying WordPress uploads..."
for year in 2018 2019 2020 2021 2022 2023 2024 2025 2026; do
  if [ -d "_migration/uploads/$year" ]; then
    rsync -a --include="*/" \
      --include="*.jpg" --include="*.jpeg" \
      --include="*.png" --include="*.webp" --include="*.gif" \
      --exclude="*" \
      "_migration/uploads/$year/" "public/images/$year/"
  fi
done

echo "Done."
```

- [ ] **Step 2: Make executable and run**

```bash
chmod +x scripts/copy-images.sh
mkdir -p public/images
bash scripts/copy-images.sh
```

Expected: "Done." — no errors.

- [ ] **Step 3: Verify key files exist**

```bash
ls public/images/hero-main.webp public/images/doug-portrait.jpg
ls public/images/2022/06/ADBR-Home-Slider1.jpg
```

Expected: both commands show files, no errors.

- [ ] **Step 4: Commit**

```bash
git add scripts/copy-images.sh public/images/
git commit -m "feat: copy design images and WordPress media to public/images"
```

---

## Task 2: Set up global.css with design system

**Files:**
- Modify: `src/styles/global.css`

Extract the `:root` tokens, base body styles, and shared utilities (`.display`, `.wrap`, `.btn`, `.section-label`, `.eyebrow`) from `index.html` into `global.css`. These are used across all pages.

- [ ] **Step 1: Replace global.css**

```css
@import "tailwindcss";

/* ===== Design tokens ===== */
:root {
  --cream: #FDFCF9;
  --cream-soft: #FFFFFF;
  --cream-deep: #F2EFE7;
  --leather: #2A1F16;
  --leather-soft: #55443A;
  --ink: #17110C;
  --ink-2: #241913;
  --rust: #E35A1C;
  --rust-dark: #B8431A;
  --line: rgba(42, 31, 22, 0.16);
  --line-soft: rgba(42, 31, 22, 0.08);
  --line-dark: rgba(253, 252, 249, 0.15);
}

/* ===== Base ===== */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { scroll-behavior: smooth; }

body {
  font-family: 'DM Sans', system-ui, sans-serif;
  background: var(--cream);
  color: var(--leather);
  line-height: 1.55;
  -webkit-font-smoothing: antialiased;
}

img { max-width: 100%; display: block; }
a { color: inherit; }

/* ===== Layout ===== */
.wrap { max-width: 1240px; margin: 0 auto; padding: 0 24px; }

/* ===== Typography ===== */
.display {
  font-family: 'Big Shoulders Display', sans-serif;
  font-weight: 900;
  letter-spacing: -0.01em;
  line-height: 0.88;
  text-transform: uppercase;
}

/* ===== Buttons ===== */
.btn {
  display: inline-block;
  background: var(--rust);
  color: #fff;
  padding: 14px 24px;
  text-decoration: none;
  font-weight: 800;
  font-size: 14px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  border: 2px solid var(--rust);
  transition: all 0.15s ease;
  cursor: pointer;
}
.btn:hover { background: var(--ink); border-color: var(--ink); color: #fff; }
.btn-outline { background: transparent; color: var(--leather); border-color: var(--leather); }
.btn-outline:hover { background: var(--leather); color: var(--cream); }
.btn-outline-light { background: transparent; color: var(--cream); border-color: var(--cream); }
.btn-outline-light:hover { background: var(--cream); color: var(--ink); border-color: var(--cream); }
.btn-big { padding: 18px 34px; font-size: 16px; }

/* ===== Section elements ===== */
.section-label {
  font-size: 13px;
  letter-spacing: 0.24em;
  text-transform: uppercase;
  color: var(--rust);
  font-weight: 800;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 14px;
}
.section-label::before {
  content: '';
  width: 36px;
  height: 3px;
  background: var(--rust);
}

.section-title {
  font-size: clamp(42px, 6.5vw, 84px);
  max-width: 18ch;
  margin-bottom: 18px;
  color: var(--leather);
}

.eyebrow {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 0.24em;
  text-transform: uppercase;
  color: var(--cream);
  margin-bottom: 32px;
}
.eyebrow::before {
  content: '';
  display: block;
  width: 36px;
  height: 3px;
  background: var(--rust);
}
```

- [ ] **Step 2: Commit**

```bash
git add src/styles/global.css
git commit -m "feat: extract design system to global.css"
```

---

## Task 3: Create Nav and Footer + update Layout

**Files:**
- Create: `src/components/Nav.astro`
- Create: `src/components/Footer.astro`
- Modify: `src/layouts/Layout.astro`

Match the exact structure from `index.html`: sticky dark header with text logo, nav links (desktop only), phone + CTA button.

- [ ] **Step 1: Create src/components/Nav.astro**

```astro
---
// src/components/Nav.astro
---
<header>
  <div class="header-inner">
    <a href="/" class="logo">
      Arizona Dirt Bike<br>Rentals
      <span>Wickenburg, AZ · Since 2010</span>
    </a>
    <nav>
      <a href="/dirt-bike-rentals-near-me/">Advanced Riders</a>
      <a href="/kids-dirt-bike-school/">Kids &amp; Family</a>
      <a href="/arizona-dirt-bike-tours-reservations/">Reservations</a>
      <a href="/reviews/">Reviews</a>
      <a href="/motorcycles/">Motorcycles</a>
      <a href="/contact-us/">Contact</a>
    </nav>
    <div class="header-cta">
      <a href="tel:5092810156" class="phone-link">509-281-0156</a>
      <a href="tel:5092810156" class="btn">Call to Book</a>
    </div>
  </div>
</header>

<style>
  header {
    position: sticky;
    top: 0;
    z-index: 100;
    background: var(--ink);
    color: var(--cream);
    border-bottom: 3px solid var(--rust);
  }
  .header-inner {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 14px 24px;
    max-width: 1240px;
    margin: 0 auto;
    gap: 16px;
  }
  .logo {
    font-family: 'Big Shoulders Display', sans-serif;
    font-weight: 900;
    font-size: 22px;
    text-transform: uppercase;
    letter-spacing: -0.01em;
    line-height: 0.95;
    text-decoration: none;
    color: var(--cream);
  }
  .logo span {
    display: block;
    font-size: 11px;
    color: var(--rust);
    letter-spacing: 0.15em;
    font-weight: 700;
  }
  nav {
    display: none;
    gap: 28px;
    font-size: 14px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  nav a { text-decoration: none; color: var(--cream); padding: 4px 0; }
  nav a:hover { color: var(--rust); }
  .header-cta { display: flex; align-items: center; gap: 14px; }
  .phone-link {
    display: none;
    font-weight: 700;
    text-decoration: none;
    font-size: 15px;
    color: var(--cream);
  }
  .phone-link:hover { color: var(--rust); }
  @media (min-width: 900px) {
    nav { display: flex; }
    .phone-link { display: flex; }
  }
</style>
```

- [ ] **Step 2: Create src/components/Footer.astro**

```astro
---
// src/components/Footer.astro
---
<footer>
  <div class="footer-inner">
    <div>
      <strong>Arizona Dirt Bike Rentals</strong><br>
      56550 Rancho Casitas Road, Wickenburg, AZ 85390
    </div>
    <div>
      <a href="tel:5092810156">509-281-0156</a> ·
      <a href="mailto:doug@arizonadirtbikerentals.com">doug@arizonadirtbikerentals.com</a>
    </div>
    <div>© 2011–2025 All Rights Reserved</div>
  </div>
</footer>

<style>
  footer {
    background: var(--ink);
    color: rgba(253, 252, 249, 0.7);
    padding: 40px 24px;
    border-top: 3px solid var(--rust);
    font-size: 13px;
  }
  .footer-inner {
    max-width: 1240px;
    margin: 0 auto;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 20px;
  }
  footer a { color: var(--cream); text-decoration: none; font-weight: 600; }
  footer a:hover { color: var(--rust); }
</style>
```

- [ ] **Step 3: Update src/layouts/Layout.astro**

```astro
---
import '../styles/global.css';
import Nav from '../components/Nav.astro';
import Footer from '../components/Footer.astro';

interface Props {
  title?: string;
  description?: string;
}

const {
  title = 'Arizona Dirt Bike Rentals',
  description = 'Unforgettable guided dirt bike rides in Wickenburg, Arizona.'
} = Astro.props;
---
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="generator" content={Astro.generator} />
    <meta name="description" content={description} />
    <title>{title}</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=Big+Shoulders+Display:wght@700;800;900&family=DM+Sans:wght@400;500;700&display=swap" rel="stylesheet" />
  </head>
  <body>
    <Nav />
    <main>
      <slot />
    </main>
    <Footer />
  </body>
</html>
```

- [ ] **Step 4: Start dev server, open http://localhost:4321, verify header and footer render**

```bash
npm run dev
```

Expect: sticky dark header with text logo + "Call to Book" rust button, coming-soon content, dark footer with rust top border.

- [ ] **Step 5: Commit**

```bash
git add src/components/Nav.astro src/components/Footer.astro src/layouts/Layout.astro
git commit -m "feat: add Nav, Footer, update Layout with design system fonts"
```

---

## Task 4: Create _redirects

**Files:**
- Create: `public/_redirects`

- [ ] **Step 1: Create public/_redirects**

```
/reservations/                    /arizona-dirt-bike-tours-reservations/  301
/home-new/                        /dirt-bike-rentals/                      301
/home/                            /dirt-bike-rentals-2/                    301
/dirt-bike-rentals-2/             /dirt-bike-rentals/                      301
/dirt-bike-rentals-phoenix-az-2/  /dirt-bike-rentals-phoenix-az/           301
```

- [ ] **Step 2: Commit**

```bash
git add public/_redirects
git commit -m "feat: add Cloudflare Pages _redirects"
```

---

## Task 5: Home page (/) — port from index.html

**Files:**
- Modify: `src/pages/index.astro`

This is a direct Astro port of `/index.html`. All CSS goes in a `<style>` block; all HTML maps 1:1. Use the SEO title/description from crawl.csv (not the `index.html` title, which is a draft).

- [ ] **Step 1: Replace src/pages/index.astro**

```astro
---
// src/pages/index.astro
import Layout from '../layouts/Layout.astro';

const title = 'Arizona Dirt Bike Rentals - KTM & Husqvarna Motorcycle Tours & Training';
const description = 'Off-road motorcycle tours, training, and moto rentals. Ride the latest Husqvarna and KTM dirt bikes and the best Wickenburg, Phoenix, Arizona moto trails.';

const reviews = [
  { quote: "Probably the best experience we've ever had as a family. We had experienced and novice riders and we were all able to ride, push our limits, and get the adrenaline pumping. We'll definitely make this an annual event.", author: "Greg Corniea" },
  { quote: "This riding experience was perfect and well worth a quick flight from Oregon. Doug had everything dialed for my son and I to have a great couple of days of riding. Looking forward to booking again soon.", author: "Darbey Budd" },
  { quote: "One of the top 3 riding days of my life. Doug is an awesome host, whether it's to put you at ease or give you a riding tip. Well-prepared KTM and Husky with top of the line accessories.", author: "Charles Lapierre" },
  { quote: "I don't have a lot of riding experience, and honestly I'm scared of heights, but Doug was patient and understanding. We've continued to rent because he's a wealth of information and a blast to be around.", author: "Jordan Moyer" },
  { quote: "Doug was legendary. My son had a blast and will definitely be doing it again. Top quality bikes, all the gear we needed, and a great area for beginners and experienced riders.", author: "Brandy McDonell" },
  { quote: "Doug did a great job assessing my skills and picking the right trails. I'll do this again next time I visit Phoenix.", author: "Doug Cooper" },
];
---
<Layout title={title} description={description}>

  <!-- Hero -->
  <section class="hero">
    <div class="hero-bg"></div>
    <div class="hero-content">
      <span class="eyebrow">Wickenburg, Arizona · Since 2010</span>
      <h1 class="display">Give them a day they'll never stop <em>talking about.</em></h1>
      <p class="hero-sub">Unforgettable guided dirt bike rides in the Arizona desert. We bring the bikes, the gear, and 15 years of local knowledge. Perfect for families, kids, and advanced riders.</p>
      <div class="hero-ctas">
        <a href="tel:5092810156" class="btn btn-big">Call 509-281-0156</a>
        <a href="#rides" class="btn btn-big btn-outline-light">See Packages</a>
      </div>
    </div>
  </section>

  <!-- Price strip -->
  <div class="price-strip">
    <div class="price-strip-inner">
      <div class="price-tag">
        $375
        <span>PER RIDER / PER DAY</span>
      </div>
      <ul class="price-includes">
        <li>KTM or Husqvarna bike</li>
        <li>Full riding gear</li>
        <li>Guided trail time</li>
        <li>Fuel &amp; lunch</li>
        <li>Pickup available</li>
        <li>Se habla español</li>
      </ul>
      <a href="tel:5092810156" class="btn btn-big">Call to Book</a>
    </div>
  </div>

  <!-- Ride cards -->
  <section class="rides" id="rides">
    <div class="wrap">
      <div class="rides-header">
        <div>
          <div class="section-label">Choose Your Ride</div>
          <h2 class="display section-title">Two ways to ride. One unforgettable day.</h2>
        </div>
        <p>Whether you're bringing the whole family out for their first ride or you're an experienced rider hunting technical singletrack, we build the day around you.</p>
      </div>
      <div class="ride-grid">
        <article class="ride-card">
          <div class="ride-card-img">
            <span class="ride-tag">Most Popular</span>
            <img src="/images/families-group.jpg" alt="Doug with young riders in gear after a ride" />
          </div>
          <div class="ride-body">
            <h3>Family &amp; Kids Riding</h3>
            <p>What kid doesn't want to ride a dirt bike? We'll give your kids a day they'll talk about for years. We teach the basics, build confidence, and pick trails that match each rider. Bikes and gear for every size. Parents ride too.</p>
            <a href="/kids-dirt-bike-school/" class="btn">Book a Family Day</a>
          </div>
        </article>
        <article class="ride-card">
          <div class="ride-card-img">
            <span class="ride-tag">Advanced</span>
            <img src="/images/advanced-canyon.jpg" alt="Riders navigating technical desert trails" />
          </div>
          <div class="ride-body">
            <h3>Advanced Rider Days</h3>
            <p>You know what you want. Every terrain you can imagine is here. Fresh KTM and Husqvarna bikes set up right. Let us dial in a day of technical singletrack you'll remember for life.</p>
            <a href="/dirt-bike-rentals-near-me/" class="btn">Book an Advanced Day</a>
          </div>
        </article>
      </div>
    </div>
  </section>

  <!-- Reviews -->
  <section class="proof" id="reviews">
    <div class="wrap">
      <div class="proof-header">
        <div class="section-label">What Riders Say</div>
        <h2 class="display section-title" style="margin: 0 auto 20px;">Over 100 five-star reviews.</h2>
        <div class="stars">★★★★★</div>
        <span class="count">Google &amp; Facebook verified</span>
      </div>
      <div class="reviews">
        {reviews.map(r => (
          <div class="review">
            <div class="review-stars">★★★★★</div>
            <p>{r.quote}</p>
            <cite>{r.author}</cite>
          </div>
        ))}
      </div>
    </div>
  </section>

  <!-- Photo break -->
  <section class="photo-break">
    <div class="photo-break-bg"></div>
    <div class="photo-break-content">
      <span class="eyebrow">The Desert Is Calling</span>
      <h2 class="display">Built for <em>dirt bikes.</em></h2>
      <p>Dry trails in January. Technical rock gardens. Flowing singletrack through saguaros. Wide open desert washes. This is where winter riding lives.</p>
    </div>
  </section>

  <!-- Why Wickenburg -->
  <section class="why" id="why">
    <div class="wrap">
      <div class="why-grid">
        <div class="why-image">
          <img src="/images/father-son.jpg" alt="Father and son in full riding gear giving thumbs up" />
        </div>
        <div class="why-text">
          <div class="section-label">Why Wickenburg</div>
          <h2 class="display section-title">15 years in the saddle. Thousands of happy riders.</h2>
          <p>Wickenburg sits just north of Phoenix in some of the best winter riding terrain on the planet. We've been guiding here since 2010. We know which trails work for a nervous first-timer and which ones will push an expert rider to their limit.</p>
          <p>That's why people fly in from Oregon, drive down from Canada, and come back year after year.</p>
          <div class="stats">
            <div>
              <div class="stat-num display">15+</div>
              <div class="stat-label">Years guiding</div>
            </div>
            <div>
              <div class="stat-num display">100+</div>
              <div class="stat-label">5-star reviews</div>
            </div>
            <div>
              <div class="stat-num display">$375</div>
              <div class="stat-label">All-inclusive day</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>

  <!-- Meet Doug -->
  <section class="meet" id="meet">
    <div class="wrap">
      <div class="meet-grid">
        <div class="meet-img">
          <img src="/images/doug-portrait.jpg" alt="Doug McDonald, owner and guide" />
        </div>
        <div>
          <div class="section-label">Your Guide</div>
          <h2 class="display">Meet Doug.</h2>
          <p>Doug McDonald has been riding these trails and running this business since 2010. He's the guy who picks you up, sets up your bike, teaches the skills, and leads the ride. No middlemen. No call centers. You call, he answers.</p>
          <p>He'll meet you where you are as a rider and build a day around what you want out of it. That's why people come back year after year.</p>
          <a href="/contact-us/" class="btn" style="margin-top: 12px;">Call Doug Directly</a>
        </div>
      </div>
    </div>
  </section>

  <!-- Final CTA -->
  <section class="final-cta">
    <div class="final-cta-bg"></div>
    <div class="wrap">
      <div class="section-label">Ready to Ride?</div>
      <h2 class="display">Let's go <em>riding.</em></h2>
      <p class="lede">Booking is simple. Call or text Doug, tell him what you're looking for, and he'll get you on the calendar.</p>
      <a href="tel:5092810156" class="final-phone">509 · 281 · 0156</a>
      <p class="final-note">Phone &amp; text · Se habla español</p>
    </div>
  </section>

</Layout>

<style>
  /* Hero */
  .hero {
    position: relative;
    min-height: 92vh;
    display: flex;
    align-items: center;
    overflow: hidden;
    background: var(--ink);
    color: var(--cream);
  }
  .hero-bg {
    position: absolute;
    inset: 0;
    background: url('/images/hero-main.webp') center/cover no-repeat;
  }
  .hero-bg::after {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(95deg,
      rgba(23,17,12,0.78) 0%,
      rgba(23,17,12,0.55) 35%,
      rgba(23,17,12,0.15) 65%,
      rgba(23,17,12,0) 100%);
  }
  @media (max-width: 800px) {
    .hero-bg::after {
      background: linear-gradient(180deg,
        rgba(23,17,12,0.85) 0%,
        rgba(23,17,12,0.55) 55%,
        rgba(23,17,12,0.35) 100%);
    }
  }
  .hero-content {
    position: relative;
    max-width: 1240px;
    margin: 0 auto;
    padding: 80px 24px 100px;
    width: 100%;
  }
  .hero h1 {
    font-size: clamp(56px, 9.5vw, 132px);
    max-width: 14ch;
    margin-bottom: 26px;
    color: var(--cream);
    text-shadow: 0 2px 30px rgba(0,0,0,0.35);
  }
  .hero h1 em { font-style: normal; color: var(--rust); }
  .hero-sub {
    font-size: clamp(17px, 1.8vw, 21px);
    max-width: 48ch;
    margin-bottom: 40px;
    color: rgba(253,252,249,0.85);
  }
  .hero-ctas { display: flex; flex-wrap: wrap; gap: 14px; align-items: center; }

  /* Price strip */
  .price-strip {
    background: var(--rust);
    color: #fff;
    padding: 40px 24px;
  }
  .price-strip-inner {
    max-width: 1240px;
    margin: 0 auto;
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 40px;
    align-items: center;
  }
  .price-tag {
    font-family: 'Big Shoulders Display', sans-serif;
    font-weight: 900;
    font-size: 84px;
    line-height: 0.82;
    color: #fff;
    padding-right: 40px;
    border-right: 3px solid rgba(255,255,255,0.3);
  }
  .price-tag span {
    display: block;
    font-size: 13px;
    color: #fff;
    letter-spacing: 0.16em;
    margin-top: 12px;
    font-family: 'DM Sans', sans-serif;
    font-weight: 700;
  }
  .price-includes {
    display: flex;
    flex-wrap: wrap;
    gap: 10px 32px;
    font-size: 15px;
    font-weight: 600;
  }
  .price-includes li { list-style: none; position: relative; padding-left: 24px; }
  .price-includes li::before { content: '→'; position: absolute; left: 0; font-weight: 900; }
  .price-strip .btn { background: var(--ink); border-color: var(--ink); white-space: nowrap; }
  .price-strip .btn:hover { background: var(--cream); border-color: var(--cream); color: var(--ink); }
  @media (max-width: 900px) {
    .price-strip-inner { grid-template-columns: 1fr; gap: 24px; }
    .price-tag { border-right: none; border-bottom: 3px solid rgba(255,255,255,0.3); padding: 0 0 24px; font-size: 68px; }
  }

  /* Rides */
  .rides { background: var(--cream); padding: 120px 0; }
  .rides-header { display: grid; gap: 32px; margin-bottom: 64px; align-items: end; }
  @media (min-width: 800px) { .rides-header { grid-template-columns: 1.5fr 1fr; } }
  .rides-header p { font-size: 17px; color: var(--leather-soft); max-width: 42ch; }
  .ride-grid { display: grid; gap: 32px; }
  @media (min-width: 900px) { .ride-grid { grid-template-columns: 1fr 1fr; } }
  .ride-card {
    overflow: hidden;
    background: #fff;
    border: 1px solid var(--line);
    display: flex;
    flex-direction: column;
    min-height: 600px;
    transition: transform 0.3s ease, box-shadow 0.3s ease;
  }
  .ride-card:hover { transform: translateY(-6px); box-shadow: 0 24px 48px rgba(58,36,25,0.15); }
  .ride-card-img { position: relative; height: 360px; overflow: hidden; }
  .ride-card-img img { width: 100%; height: 100%; object-fit: cover; transition: transform 0.6s ease; }
  .ride-card:hover .ride-card-img img { transform: scale(1.06); }
  .ride-tag {
    position: absolute;
    top: 0; left: 0;
    background: var(--rust);
    color: #fff;
    padding: 10px 16px;
    font-size: 11px;
    font-weight: 800;
    letter-spacing: 0.18em;
    text-transform: uppercase;
    z-index: 2;
  }
  .ride-body {
    padding: 40px;
    flex: 1;
    display: flex;
    flex-direction: column;
    border-top: 3px solid var(--rust);
  }
  .ride-body h3 {
    font-family: 'Big Shoulders Display', sans-serif;
    font-weight: 900;
    font-size: 48px;
    text-transform: uppercase;
    line-height: 0.9;
    margin-bottom: 18px;
    color: var(--leather);
  }
  .ride-body p { color: var(--leather-soft); margin-bottom: 28px; flex: 1; font-size: 15.5px; }
  .ride-body .btn { align-self: flex-start; }

  /* Reviews */
  .proof {
    padding: 120px 0;
    background: var(--cream-soft);
    border-top: 1px solid var(--line);
    border-bottom: 1px solid var(--line);
  }
  .proof-header { text-align: center; margin-bottom: 68px; }
  .proof-header .section-label { justify-content: center; }
  .proof-header h2 { margin: 0 auto 20px; }
  .proof-header .stars { font-size: 34px; letter-spacing: 8px; color: var(--rust); }
  .proof-header .count {
    display: block;
    font-size: 13px;
    color: var(--leather-soft);
    letter-spacing: 0.18em;
    text-transform: uppercase;
    margin-top: 12px;
    font-weight: 600;
  }
  .reviews { display: grid; gap: 24px; }
  @media (min-width: 700px) { .reviews { grid-template-columns: repeat(2, 1fr); } }
  @media (min-width: 1000px) { .reviews { grid-template-columns: repeat(3, 1fr); } }
  .review {
    background: #fff;
    padding: 32px;
    border: 1px solid var(--line-soft);
    border-top: 4px solid var(--rust);
  }
  .review-stars { color: var(--rust); font-size: 16px; letter-spacing: 3px; margin-bottom: 16px; }
  .review p { font-size: 15px; line-height: 1.6; color: var(--leather); margin-bottom: 20px; }
  .review cite {
    font-style: normal;
    font-weight: 800;
    font-size: 12px;
    letter-spacing: 0.14em;
    text-transform: uppercase;
    color: var(--rust);
  }

  /* Photo break */
  .photo-break {
    position: relative;
    min-height: 60vh;
    background: var(--ink);
    color: var(--cream);
    display: flex;
    align-items: center;
    overflow: hidden;
  }
  .photo-break-bg {
    position: absolute;
    inset: 0;
    background: url('/images/hero-action.jpg') center/cover no-repeat;
    opacity: 0.75;
  }
  .photo-break-bg::after {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(90deg,
      rgba(23,17,12,0.85) 0%,
      rgba(23,17,12,0.55) 50%,
      rgba(23,17,12,0.35) 100%);
  }
  .photo-break-content { position: relative; max-width: 1240px; margin: 0 auto; padding: 100px 24px; width: 100%; }
  .photo-break h2 { font-size: clamp(40px, 6.5vw, 88px); max-width: 16ch; color: var(--cream); margin-bottom: 20px; }
  .photo-break h2 em { font-style: normal; color: var(--rust); }
  .photo-break p { font-size: 18px; max-width: 50ch; color: rgba(253,252,249,0.85); }

  /* Why */
  .why { background: var(--cream); padding: 120px 0; }
  .why-grid { display: grid; gap: 64px; align-items: center; }
  @media (min-width: 900px) { .why-grid { grid-template-columns: 1fr 1fr; } }
  .why-image { position: relative; aspect-ratio: 4/5; overflow: hidden; border: 1px solid var(--line); }
  .why-image img { width: 100%; height: 100%; object-fit: cover; }
  .why-image::before {
    content: 'SINCE 2010';
    position: absolute;
    top: 0; right: 0;
    background: var(--rust);
    color: #fff;
    padding: 12px 18px;
    font-weight: 800;
    font-size: 12px;
    letter-spacing: 0.2em;
    z-index: 2;
  }
  .why-text p { font-size: 17px; color: var(--leather-soft); margin-bottom: 20px; max-width: 48ch; }
  .stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; margin-top: 48px; padding-top: 44px; border-top: 3px solid var(--rust); }
  .stat-num { font-size: 60px; line-height: 0.9; color: var(--leather); }
  .stat-label { font-size: 12px; letter-spacing: 0.14em; text-transform: uppercase; color: var(--leather-soft); font-weight: 700; margin-top: 8px; }

  /* Meet */
  .meet { padding: 120px 0; background: var(--cream-deep); border-top: 1px solid var(--line); border-bottom: 1px solid var(--line); }
  .meet-grid { display: grid; gap: 48px; align-items: center; }
  @media (min-width: 800px) { .meet-grid { grid-template-columns: 1fr 1.3fr; } }
  .meet-img { aspect-ratio: 1/1; overflow: hidden; border: 1px solid var(--line); }
  .meet-img img { width: 100%; height: 100%; object-fit: cover; object-position: center 30%; }
  .meet h2 { font-size: clamp(40px, 5.5vw, 72px); margin-bottom: 24px; color: var(--leather); }
  .meet p { font-size: 17px; color: var(--leather-soft); margin-bottom: 16px; max-width: 52ch; }

  /* Final CTA */
  .final-cta {
    position: relative;
    background: var(--ink);
    color: var(--cream);
    text-align: center;
    padding: 130px 24px;
    overflow: hidden;
  }
  .final-cta-bg {
    position: absolute;
    inset: 0;
    background: url('/images/hero-main.webp') center/cover no-repeat;
    opacity: 0.35;
  }
  .final-cta-bg::after {
    content: '';
    position: absolute;
    inset: 0;
    background: radial-gradient(ellipse at center, rgba(23,17,12,0.5) 0%, rgba(23,17,12,0.92) 75%);
  }
  .final-cta > * { position: relative; }
  .final-cta .section-label { justify-content: center; color: var(--rust); }
  .final-cta h2 { font-size: clamp(52px, 8.5vw, 120px); margin-bottom: 28px; color: var(--cream); }
  .final-cta h2 em { font-style: normal; color: var(--rust); }
  .final-cta .lede { font-size: 19px; max-width: 46ch; margin: 0 auto 32px; color: rgba(253,252,249,0.8); }
  .final-phone {
    font-family: 'Big Shoulders Display', sans-serif;
    font-weight: 900;
    font-size: clamp(56px, 8vw, 108px);
    color: var(--rust);
    text-decoration: none;
    display: block;
    margin: 24px 0;
    letter-spacing: -0.01em;
  }
  .final-phone:hover { color: var(--cream); }
  .final-note { font-size: 13px; letter-spacing: 0.18em; text-transform: uppercase; color: rgba(253,252,249,0.55); font-weight: 600; }
</style>
```

- [ ] **Step 2: Open http://localhost:4321 and verify it matches `/index.html` visually**

Open `index.html` in browser alongside dev server. Sections should match: hero, price strip, ride cards, reviews, photo break, why, meet Doug, final CTA.

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: port index.html design to Astro home page"
```

---

## Task 6: Contact page

**Files:**
- Create: `src/pages/contact-us.astro`

Uses the design system (header/footer from layout, brand colors). Page-specific CSS in `<style>` block.

- [ ] **Step 1: Create src/pages/contact-us.astro**

```astro
---
// src/pages/contact-us.astro
import Layout from '../layouts/Layout.astro';
const title = 'Contact Us - Arizona Dirt Bike Rentals';
const description = "Our Location Wickenburg, AZ is a super-cool western town where you can enjoy a true southwest experience. And we're less than an hour from Phoenix. We offer single-track guided moto-tours and motorcycle rentals seven days a week all year. Call Doug at 509-281-0156 if you have any questions about dirt bike rentals,";
---
<Layout title={title} description={description}>
  <div class="page-wrap">

    <div class="page-hero">
      <div class="section-label">Get In Touch</div>
      <h1 class="display">Contact Us.</h1>
    </div>

    <div class="contact-grid">
      <div>
        <h2>Our Location</h2>
        <p>Wickenburg, AZ is a super-cool western town where you can enjoy a true southwest experience — less than an hour from Phoenix.</p>
        <p>We offer single-track guided moto-tours and motorcycle rentals seven days a week, all year. Call Doug at <a href="tel:5092810156">509-281-0156</a> with any questions about dirt bike rentals, training, riding locations, and reservations. (Se Habla Español)</p>
        <p class="note">* If you can't ride on your reservation date we'll roll any deposit over to a future date.</p>

        <div class="contact-info">
          <strong>Doug McDonald</strong>
          <p>56550 Rancho Casitas Road<br>Wickenburg, AZ 85390</p>
          <p><a href="tel:5092810156">Tel: 509-281-0156</a></p>
          <p><a href="mailto:doug@arizonadirtbikerentals.com">doug@arizonadirtbikerentals.com</a></p>
        </div>
      </div>

      <div>
        <div class="contact-photo">
          <img src="/images/doug-portrait.jpg" alt="Doug McDonald — Arizona Dirt Bike Rentals guide" />
        </div>
        <blockquote>
          <p>"Best experience I have ever had with a dirt bike. Loved the trails and the views. Got to see a mine used in World War 2. First time riding in this terrain and he gave me great instruction."</p>
          <cite>— Russel Roodenburg</cite>
        </blockquote>
      </div>
    </div>
  </div>

  <section class="final-cta-simple">
    <div class="section-label">Ready to Ride?</div>
    <a href="tel:5092810156" class="final-phone display">509 · 281 · 0156</a>
    <p class="final-note">Phone &amp; text · Seven days a week · Se habla español</p>
  </section>
</Layout>

<style>
  .page-wrap { max-width: 1240px; margin: 0 auto; padding: 80px 24px 120px; }
  .page-hero { margin-bottom: 64px; }
  .page-hero h1 { font-size: clamp(52px, 8vw, 104px); color: var(--leather); }
  .contact-grid { display: grid; gap: 64px; }
  @media (min-width: 800px) { .contact-grid { grid-template-columns: 1fr 1fr; } }
  .contact-grid h2 { font-size: 26px; font-weight: 700; margin-bottom: 20px; color: var(--leather); }
  .contact-grid p { font-size: 17px; color: var(--leather-soft); margin-bottom: 16px; max-width: 52ch; }
  .contact-grid a { color: var(--rust); font-weight: 700; text-decoration: none; }
  .contact-grid a:hover { text-decoration: underline; }
  .note { font-size: 14px; color: var(--leather-soft); opacity: 0.7; }
  .contact-info { margin-top: 36px; padding-top: 36px; border-top: 3px solid var(--rust); }
  .contact-info strong { font-size: 18px; display: block; margin-bottom: 12px; color: var(--leather); }
  .contact-photo { aspect-ratio: 1/1; overflow: hidden; border: 1px solid var(--line); margin-bottom: 32px; }
  .contact-photo img { width: 100%; height: 100%; object-fit: cover; object-position: center 30%; }
  blockquote { border-left: 4px solid var(--rust); padding-left: 20px; }
  blockquote p { font-size: 16px; color: var(--leather); font-style: italic; margin-bottom: 12px; }
  blockquote cite { font-style: normal; font-size: 12px; font-weight: 800; letter-spacing: 0.14em; text-transform: uppercase; color: var(--rust); }
  .final-cta-simple { background: var(--ink); color: var(--cream); text-align: center; padding: 80px 24px; border-top: 3px solid var(--rust); }
  .final-phone { font-size: clamp(48px, 7vw, 96px); color: var(--rust); text-decoration: none; display: block; margin: 16px 0; letter-spacing: -0.01em; }
  .final-phone:hover { color: var(--cream); }
  .final-note { font-size: 13px; letter-spacing: 0.18em; text-transform: uppercase; color: rgba(253,252,249,0.55); font-weight: 600; }
</style>
```

- [ ] **Step 2: Check http://localhost:4321/contact-us/**

- [ ] **Step 3: Commit**

```bash
git add src/pages/contact-us.astro
git commit -m "feat: add contact page"
```

---

## Task 7: Tours & Reservations page

**Files:**
- Create: `src/pages/arizona-dirt-bike-tours-reservations.astro`

- [ ] **Step 1: Create the file**

```astro
---
// src/pages/arizona-dirt-bike-tours-reservations.astro
import Layout from '../layouts/Layout.astro';
const title = 'Arizona Dirt Bike Tours Reservations';
const description = 'Check out our exclusive Arizona dirt bike tours all inclusive rent and ride packages. Includes an experienced trail guide.';
---
<Layout title={title} description={description}>

  <div class="hero-banner">
    <img src="/images/2021/10/Arizona-Dirt-Bike-Rentals-Share-Feature-Image-1200-628-60.jpg"
         alt="Arizona dirt bike tour on desert trails" />
    <div class="hero-overlay">
      <span class="eyebrow">All-Inclusive</span>
      <h1 class="display">Arizona Dirt Bike Tours.</h1>
    </div>
  </div>

  <div class="page-wrap">

    <div class="price-block">
      <div class="price-num display">$375</div>
      <div class="price-meta">
        <p class="price-per">Per Rider / Per Day</p>
        <p class="price-sub">All-inclusive rent &amp; ride packages</p>
      </div>
    </div>

    <h2 class="includes-title">This all-inclusive price includes:</h2>
    <ul class="includes-list">
      <li>An experienced trail guide to get you to the best trails for the conditions, your riding level, and desired terrain</li>
      <li>All riding gear (bring your own hydration pack)</li>
      <li>Gas</li>
      <li>Lunch</li>
      <li>Bikes prepped and transported to and from the trails</li>
    </ul>

    <p class="addl"><strong>$250 per rider</strong> for any additional rider who brings their own bike, gas, and gear and joins an existing rental group.</p>
    <p class="vip">Tours and training groups are limited to your group only for VIP service. Show up and ride — we'll take care of everything else.</p>

    <div class="ohv-notice">
      <strong>OHV Safety Certificate Required</strong>
      <p>Starting January 1, 2025, anyone operating an Off-Highway Vehicle must take a safety course and obtain an OHV Safety Certificate. <a href="https://arizonagame.org/ohv/" target="_blank" rel="noopener">Click here to obtain your certificate →</a></p>
    </div>

    <div class="bikes-link">
      <a href="/motorcycles/" class="btn">View Our Motorcycles</a>
    </div>

  </div>

  <section class="call-cta">
    <div class="section-label">Reserve Your Day</div>
    <a href="tel:5092810156" class="final-phone display">509 · 281 · 0156</a>
    <p class="final-note">Call Doug · Seven days a week · All year</p>
  </section>

</Layout>

<style>
  .hero-banner { position: relative; height: 400px; overflow: hidden; }
  .hero-banner img { width: 100%; height: 100%; object-fit: cover; }
  .hero-overlay {
    position: absolute;
    inset: 0;
    background: linear-gradient(90deg, rgba(23,17,12,0.8) 0%, rgba(23,17,12,0.3) 100%);
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    padding: 48px 40px;
    color: var(--cream);
  }
  .hero-overlay h1 { font-size: clamp(40px, 7vw, 88px); color: var(--cream); }
  .page-wrap { max-width: 800px; margin: 0 auto; padding: 80px 24px 40px; }
  .price-block { display: flex; align-items: center; gap: 32px; margin-bottom: 48px; padding-bottom: 48px; border-bottom: 3px solid var(--rust); }
  .price-num { font-size: 96px; color: var(--rust); line-height: 0.9; }
  .price-per { font-size: 20px; font-weight: 700; color: var(--leather); margin-bottom: 4px; }
  .price-sub { font-size: 14px; letter-spacing: 0.12em; text-transform: uppercase; color: var(--leather-soft); font-weight: 600; }
  .includes-title { font-size: 20px; font-weight: 700; margin-bottom: 20px; color: var(--leather); }
  .includes-list { margin-bottom: 36px; padding-left: 0; }
  .includes-list li {
    list-style: none;
    padding: 12px 0 12px 32px;
    position: relative;
    border-bottom: 1px solid var(--line-soft);
    font-size: 17px;
    color: var(--leather-soft);
  }
  .includes-list li::before { content: '→'; position: absolute; left: 0; color: var(--rust); font-weight: 900; }
  .addl { font-size: 17px; color: var(--leather-soft); margin-bottom: 12px; }
  .addl strong { color: var(--leather); }
  .vip { font-size: 17px; color: var(--leather-soft); margin-bottom: 48px; }
  .ohv-notice {
    background: var(--cream-deep);
    border: 1px solid var(--line);
    border-left: 4px solid var(--rust);
    padding: 20px 24px;
    margin-bottom: 40px;
    font-size: 15px;
  }
  .ohv-notice strong { display: block; margin-bottom: 8px; color: var(--leather); }
  .ohv-notice p { color: var(--leather-soft); }
  .ohv-notice a { color: var(--rust); font-weight: 700; text-decoration: none; }
  .ohv-notice a:hover { text-decoration: underline; }
  .bikes-link { margin-bottom: 48px; }
  .call-cta { background: var(--ink); text-align: center; padding: 80px 24px; border-top: 3px solid var(--rust); color: var(--cream); }
  .final-phone { font-size: clamp(48px, 7vw, 96px); color: var(--rust); text-decoration: none; display: block; margin: 16px 0; letter-spacing: -0.01em; }
  .final-phone:hover { color: var(--cream); }
  .final-note { font-size: 13px; letter-spacing: 0.18em; text-transform: uppercase; color: rgba(253,252,249,0.55); font-weight: 600; }
</style>
```

- [ ] **Step 2: Check http://localhost:4321/arizona-dirt-bike-tours-reservations/**

- [ ] **Step 3: Commit**

```bash
git add src/pages/arizona-dirt-bike-tours-reservations.astro
git commit -m "feat: add tours and reservations page"
```

---

## Task 8: Kids Dirt Bike School page

**Files:**
- Create: `src/pages/kids-dirt-bike-school.astro`

Images from WordPress uploads: `ADBR-Family-600x600.jpg`, `IMG_20200107_120230-1200px.jpg`, `ADBR-Kid-Protection-copy.jpg`

- [ ] **Step 1: Create src/pages/kids-dirt-bike-school.astro**

```astro
---
// src/pages/kids-dirt-bike-school.astro
import Layout from '../layouts/Layout.astro';
const title = 'Kids Dirt Bike School - Arizona Dirt Bike Rentals';
const description = 'Give your kids an unforgettable experience';
---
<Layout title={title} description={description}>

  <div class="page-hero-text">
    <div class="wrap">
      <div class="section-label">For Ages 8 &amp; Up</div>
      <h1 class="display">Give your kids an unforgettable experience!</h1>
      <p class="hero-sub">We've got an awesome day planned for you and your kids. "That was insane!" ~ Toby. "OMG! I'm freakin' out!" ~ Marshall.</p>
    </div>
  </div>

  <section class="kids-content">
    <div class="wrap">
      <div class="kids-grid">
        <div class="kids-photo">
          <img src="/images/2022/06/ADBR-Family-600x600.jpg" alt="Family riding dirt bikes in Arizona" />
        </div>
        <div class="kids-text">
          <h2>We have everything you need:</h2>
          <ul class="gear-list">
            <li>All safety and riding gear</li>
            <li>Helmet</li>
            <li>Goggles</li>
            <li>Knee and elbow pads</li>
            <li>Jersey and pants</li>
            <li>Gloves and boots</li>
            <li>Appropriate-size dirt bike</li>
            <li>Patient, expert instruction</li>
          </ul>
        </div>
      </div>

      <div class="kids-photos-row">
        <img src="/images/2022/06/IMG_20200107_120230-1200px.jpg" alt="Kids learning to ride dirt bikes" />
        <img src="/images/2022/06/ADBR-Kid-Protection-copy.jpg" alt="Kids dirt bike safety gear" />
      </div>
    </div>
  </section>

  <section class="call-cta">
    <div class="section-label">Book Your Kid's Day</div>
    <a href="tel:5092810156" class="final-phone display">509 · 281 · 0156</a>
    <p class="final-note">Call Doug · Seven days a week · All year</p>
  </section>

</Layout>

<style>
  .page-hero-text { background: var(--cream-deep); border-bottom: 1px solid var(--line); padding: 80px 0; }
  .page-hero-text h1 { font-size: clamp(44px, 7vw, 96px); color: var(--leather); max-width: 18ch; margin-bottom: 20px; }
  .hero-sub { font-size: 18px; color: var(--leather-soft); max-width: 54ch; }
  .kids-content { padding: 100px 0; background: var(--cream); }
  .kids-grid { display: grid; gap: 64px; align-items: center; margin-bottom: 80px; }
  @media (min-width: 800px) { .kids-grid { grid-template-columns: 1fr 1fr; } }
  .kids-photo { overflow: hidden; border: 1px solid var(--line); aspect-ratio: 1/1; }
  .kids-photo img { width: 100%; height: 100%; object-fit: cover; }
  .kids-text h2 { font-size: 26px; font-weight: 700; margin-bottom: 24px; color: var(--leather); }
  .gear-list { padding-left: 0; }
  .gear-list li {
    list-style: none;
    padding: 10px 0 10px 28px;
    position: relative;
    border-bottom: 1px solid var(--line-soft);
    font-size: 17px;
    color: var(--leather-soft);
  }
  .gear-list li::before { content: '→'; position: absolute; left: 0; color: var(--rust); font-weight: 900; }
  .kids-photos-row { display: grid; gap: 24px; }
  @media (min-width: 700px) { .kids-photos-row { grid-template-columns: 1fr 1fr; } }
  .kids-photos-row img { width: 100%; height: 360px; object-fit: cover; border: 1px solid var(--line); }
  .call-cta { background: var(--ink); text-align: center; padding: 80px 24px; border-top: 3px solid var(--rust); color: var(--cream); }
  .final-phone { font-size: clamp(48px, 7vw, 96px); color: var(--rust); text-decoration: none; display: block; margin: 16px 0; letter-spacing: -0.01em; }
  .final-phone:hover { color: var(--cream); }
  .final-note { font-size: 13px; letter-spacing: 0.18em; text-transform: uppercase; color: rgba(253,252,249,0.55); font-weight: 600; }
</style>
```

- [ ] **Step 2: Check http://localhost:4321/kids-dirt-bike-school/**

- [ ] **Step 3: Commit**

```bash
git add src/pages/kids-dirt-bike-school.astro
git commit -m "feat: add kids dirt bike school page"
```

---

## Task 9: Motorcycles page

**Files:**
- Create: `src/pages/motorcycles.astro`

- [ ] **Step 1: Create src/pages/motorcycles.astro**

```astro
---
// src/pages/motorcycles.astro
import Layout from '../layouts/Layout.astro';
const title = 'Motorcycles - Arizona Dirt Bike Rentals';
const description = 'We rent the latest Husqvarna and KTM dirt bikes in the Wickenburg, Phoenix, Arizona area. We have 2-strokes and 4-stroke motorcycles to choose from.';

const bikes = [
  { name: '2023 KTM 300 XC', img: '/images/2023/07/Arizona-Dirt-Bike-Rentals-KTM-300-XC-2023.jpg' },
  { name: '2023 Husqvarna 300 TE', img: '/images/2023/07/Arizona-Dirt-Bike-Rentals-Husqvarna-300-TE-2023.jpg' },
  { name: 'Husqvarna 450 FX', img: '/images/2023/07/Arizona-Dirt-Bike-Rentals-Husqvarna-450-FX-2022.jpg' },
  { name: 'KTM 150 XC-W', img: '/images/2021/12/Arizona-Dirt-Bike-Rentals-KTM-150-XC-W-2022.jpg' },
  { name: 'Husqvarna TE 300i', img: '/images/2021/12/Arizona-Dirt-Bike-Rentals-Husqvarna-TE-300i-2022.jpg' },
  { name: 'Kawasaki KLX 140 RL', img: '/images/2022/05/Arizona-Dirt-Bike-Rentals-Kawasaki-KLX-140-RL-2022.jpg' },
];
---
<Layout title={title} description={description}>

  <div class="page-hero-text">
    <div class="wrap">
      <div class="section-label">Rent &amp; Ride</div>
      <h1 class="display">Motorcycles to Rent.</h1>
      <p class="hero-sub">KTM, Husqvarna, and Kawasaki — 150cc to 300cc 2-strokes, 140cc to 450cc 4-strokes.</p>
    </div>
  </div>

  <div class="price-strip">
    <div class="price-strip-inner">
      <div class="price-tag">$375<span>PER RIDER / PER DAY</span></div>
      <ul class="price-includes">
        <li>Bike of your choice</li>
        <li>All riding gear</li>
        <li>Guided trail time</li>
        <li>Gas &amp; lunch</li>
      </ul>
      <a href="tel:5092810156" class="btn btn-big">Call to Book</a>
    </div>
  </div>

  <section class="bikes-section">
    <div class="wrap">
      <div class="bike-grid">
        {bikes.map(bike => (
          <div class="bike-card">
            <div class="bike-img">
              <img src={bike.img} alt={bike.name} />
            </div>
            <div class="bike-name">{bike.name}</div>
          </div>
        ))}
      </div>
      <p class="guide-note">We do not rent bikes without the accompaniment of our in-person trail guide.</p>
    </div>
  </section>

  <section class="call-cta">
    <div class="section-label">Reserve Your Ride</div>
    <a href="tel:5092810156" class="final-phone display">509 · 281 · 0156</a>
    <p class="final-note">Seven days a week · All year · Se habla español</p>
  </section>

</Layout>

<style>
  .page-hero-text { background: var(--cream-deep); border-bottom: 1px solid var(--line); padding: 80px 0; }
  .page-hero-text h1 { font-size: clamp(44px, 7vw, 96px); color: var(--leather); margin-bottom: 20px; }
  .hero-sub { font-size: 18px; color: var(--leather-soft); max-width: 54ch; }
  .price-strip { background: var(--rust); color: #fff; padding: 40px 24px; }
  .price-strip-inner { max-width: 1240px; margin: 0 auto; display: grid; grid-template-columns: auto 1fr auto; gap: 40px; align-items: center; }
  .price-tag { font-family: 'Big Shoulders Display', sans-serif; font-weight: 900; font-size: 84px; line-height: 0.82; color: #fff; padding-right: 40px; border-right: 3px solid rgba(255,255,255,0.3); }
  .price-tag span { display: block; font-size: 13px; letter-spacing: 0.16em; margin-top: 12px; font-family: 'DM Sans', sans-serif; font-weight: 700; }
  .price-includes { display: flex; flex-wrap: wrap; gap: 10px 32px; font-size: 15px; font-weight: 600; }
  .price-includes li { list-style: none; position: relative; padding-left: 24px; }
  .price-includes li::before { content: '→'; position: absolute; left: 0; font-weight: 900; }
  .price-strip .btn { background: var(--ink); border-color: var(--ink); white-space: nowrap; }
  .price-strip .btn:hover { background: var(--cream); border-color: var(--cream); color: var(--ink); }
  @media (max-width: 900px) {
    .price-strip-inner { grid-template-columns: 1fr; gap: 24px; }
    .price-tag { border-right: none; border-bottom: 3px solid rgba(255,255,255,0.3); padding: 0 0 24px; font-size: 68px; }
  }
  .bikes-section { padding: 100px 0; background: var(--cream); }
  .bike-grid { display: grid; gap: 32px; margin-bottom: 48px; }
  @media (min-width: 600px) { .bike-grid { grid-template-columns: repeat(2, 1fr); } }
  @media (min-width: 900px) { .bike-grid { grid-template-columns: repeat(3, 1fr); } }
  .bike-card { border: 1px solid var(--line); background: #fff; overflow: hidden; transition: transform 0.3s ease, box-shadow 0.3s ease; }
  .bike-card:hover { transform: translateY(-4px); box-shadow: 0 16px 40px rgba(58,36,25,0.12); }
  .bike-img { height: 240px; overflow: hidden; }
  .bike-img img { width: 100%; height: 100%; object-fit: cover; transition: transform 0.5s ease; }
  .bike-card:hover .bike-img img { transform: scale(1.05); }
  .bike-name { padding: 16px 20px; font-weight: 700; font-size: 15px; color: var(--leather); border-top: 3px solid var(--rust); }
  .guide-note { font-size: 15px; color: var(--leather-soft); font-style: italic; }
  .call-cta { background: var(--ink); text-align: center; padding: 80px 24px; border-top: 3px solid var(--rust); color: var(--cream); }
  .final-phone { font-size: clamp(48px, 7vw, 96px); color: var(--rust); text-decoration: none; display: block; margin: 16px 0; letter-spacing: -0.01em; }
  .final-phone:hover { color: var(--cream); }
  .final-note { font-size: 13px; letter-spacing: 0.18em; text-transform: uppercase; color: rgba(253,252,249,0.55); font-weight: 600; }
</style>
```

- [ ] **Step 2: Check http://localhost:4321/motorcycles/**

- [ ] **Step 3: Commit**

```bash
git add src/pages/motorcycles.astro
git commit -m "feat: add motorcycles page"
```

---

## Implementation order

1. Task 1 — copy images first
2. Task 2 — global.css design tokens
3. Task 3 — Nav/Footer/Layout (start dev server after)
4. Task 4 — _redirects
5. Tasks 5–9 — pages in order (home first to verify design system)

## Phase 2 (not in this plan)

- `/gallery/` — image grid
- `/reviews/` — long review content
- `/blog/` + 6 blog posts
- `/dirt-bike-rentals-near-me/` — duplicate of home, needs canonical tag
- `/category/uncategorized/` → redirect to `/blog/`
