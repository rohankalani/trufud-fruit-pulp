# CLAUDE.md - Trufud Fruit Pulp Website

This file documents the learnings, patterns, and best practices discovered while building the Trufud Frozen Fruit Pulp website.

---

## Project Overview

| Field | Value |
|-------|-------|
| **Project** | Trufud Frozen Fruit Pulp Website |
| **Live URL** | https://trufud-fruit-pulp.pages.dev |
| **GitHub** | https://github.com/rohankalani/trufud-fruit-pulp |
| **Stack** | HTML, CSS, JavaScript (Vanilla) |
| **Hosting** | Cloudflare Pages |

---

## File Structure

```
Fruit Pulp/
├── index.html                    # Main website (single-page)
├── CLAUDE.md                     # This file - project documentation
├── .gitignore                    # Git ignore rules
│
├── Product Images/               # Compressed product images (260-350KB each)
│   ├── ALPHONSO MANGO FRUIT PULP.png
│   ├── avocado.png
│   ├── STRAWBERRY FRUIT PULP.png
│   ├── PINEAPPLE FRUIT PULP.png
│   ├── ORANGE PULP.png
│   ├── GRAPE FRUIT PULP.png
│   ├── PASSION FRUIT PULP.png
│   ├── SAPOTA PULP.png
│   └── TENDER COCONUT FRUIT PULP.png
│
├── Recipe Images/                # Generated recipe images (64-126KB each)
│   ├── *_anime.png              # Anime-style cocktail images
│   └── *.png                    # Realistic food photography
│
└── original_images/             # Backup of uncompressed originals (not in git)
```

---

## Key Learnings

### 1. Image Optimization

**Problem:** Original product images were 2-7MB each, causing slow page loads.

**Solution:**
```bash
# Resize images to 500px max dimension using macOS sips
sips -Z 500 "original.png" --out "compressed.png"
```

**Results:**
| Before | After | Reduction |
|--------|-------|-----------|
| 2-7MB per image | 260-350KB per image | ~90% |
| 24MB total | ~2.8MB total | ~88% |

**Best Practice:** Always compress images before deployment. Target 200-400KB for hero/product images.

---

### 2. Mobile 3D Card Flip

**Problem:** CSS 3D flip cards didn't work correctly on mobile Safari/iOS - both sides visible simultaneously.

**Root Cause:**
- `backface-visibility: hidden` needs `-webkit-` prefix for Safari
- Hover-based flip doesn't work on touch devices
- `transform-style: preserve-3d` needs `-webkit-` prefix

**Solution:**
```css
/* Add webkit prefixes */
.card-inner {
    transform-style: preserve-3d;
    -webkit-transform-style: preserve-3d;
}

.card-front, .card-back {
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
}

/* Disable hover on mobile, use click */
@media (max-width: 768px) {
    .card:hover .card-inner {
        transform: none; /* Disable hover flip */
    }

    .card.flipped .card-inner {
        transform: rotateY(180deg); /* Use class-based flip */
    }
}
```

```javascript
// JavaScript for tap-to-flip on mobile
const isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;

if (isTouchDevice || window.innerWidth <= 768) {
    cards.forEach(card => {
        card.addEventListener('click', () => {
            card.classList.toggle('flipped');
        });
    });
}
```

---

### 3. Infinite Auto-Scroll Carousel

**Problem:** Create a smooth, infinite horizontal scroll for product showcase.

**Solution:**
```css
/* Container with overflow hidden */
.products-carousel {
    overflow: hidden;
    width: 100%;
    position: relative;
}

/* Fade edges for smooth visual */
.products-carousel::before,
.products-carousel::after {
    content: '';
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100px;
    z-index: 10;
}

.products-carousel::before {
    left: 0;
    background: linear-gradient(to right, var(--bg) 0%, transparent 100%);
}

.products-carousel::after {
    right: 0;
    background: linear-gradient(to left, var(--bg) 0%, transparent 100%);
}

/* Animated track */
.products-scroll {
    display: flex;
    gap: 30px;
    animation: scrollProducts 30s linear infinite;
    width: max-content;
}

.products-scroll:hover {
    animation-play-state: paused; /* Pause on hover */
}

@keyframes scrollProducts {
    0% { transform: translateX(0); }
    100% { transform: translateX(-50%); } /* Move 50% because cards are duplicated */
}
```

```javascript
// Duplicate cards for seamless loop
const scroll = document.querySelector('.products-scroll');
const cards = scroll.querySelectorAll('.product-card');
cards.forEach(card => {
    const clone = card.cloneNode(true);
    scroll.appendChild(clone);
});
```

**Key:** The animation moves 50% because we duplicate all cards, so when it reaches 50%, it loops back seamlessly.

---

### 4. HuggingFace Image Generation

**Problem:** Generate AI images using HuggingFace API.

**API Endpoint (2025+):**
```
https://router.huggingface.co/hf-inference/models/MODEL_NAME
```

**Working Model:** `black-forest-labs/FLUX.1-schnell`

**Example:**
```bash
curl -X POST "https://router.huggingface.co/hf-inference/models/black-forest-labs/FLUX.1-schnell" \
  -H "Authorization: Bearer $HF_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"inputs": "anime style cocktail with mango, dark background"}' \
  --output image.png
```

**Note:** Old endpoint `api-inference.huggingface.co` is deprecated (410 error). Use `router.huggingface.co`.

---

### 5. Cloudflare Pages Deployment

**Deploy Command:**
```bash
npx wrangler pages deploy . --project-name=trufud-fruit-pulp
```

**First-time setup:**
```bash
# Login to Cloudflare
npx wrangler login

# Deploy
npx wrangler pages deploy . --project-name=PROJECT_NAME
```

**GitHub Integration:**
- Push to GitHub: `git push origin main`
- Cloudflare auto-deploys from GitHub (if connected)
- Or manually deploy with wrangler command above

---

### 6. CSS Color Variables

**Dark Theme Color Palette:**
```css
:root {
    /* Background */
    --bg-primary: #0a0a0a;
    --bg-card: #141414;
    --bg-elevated: #1a1a1a;

    /* Text */
    --text-primary: #ffffff;
    --text-secondary: #a3a3a3;
    --text-muted: #666666;

    /* Accents */
    --accent-gold: #ffd700;
    --accent-green: #00D67A;

    /* Fruit Colors */
    --mango: #FFB347;
    --strawberry: #FF6B6B;
    --avocado: #9ACD32;
    --pineapple: #FFD700;
    --orange: #FF8C00;
    --grape: #8B008B;
    --passion: #FF69B4;
    --sapota: #D2691E;
    --coconut: #00CED1;
}
```

---

### 7. Recipe Card Structure

**HTML Structure:**
```html
<div class="recipe-card" data-category="smoothies">
    <div class="recipe-card-inner">
        <!-- Front -->
        <div class="recipe-front">
            <div class="recipe-image has-photo">
                <img src="image.png" class="recipe-photo" alt="Recipe">
            </div>
            <div class="recipe-content">
                <span class="recipe-category-tag">SMOOTHIE</span>
                <h3 class="recipe-title">Recipe Name</h3>
                <p class="recipe-description">Description here</p>
                <div class="recipe-meta">
                    <span>🕐 5 min</span>
                    <span>📊 Easy</span>
                </div>
            </div>
            <div class="flip-hint">↻ Flip for recipe</div>
        </div>

        <!-- Back -->
        <div class="recipe-back">
            <h4>🧊 Ingredients</h4>
            <ul class="ingredients-list">
                <li><span class="trufud-cubes">4x Mango</span> Trufud Cubes</li>
                <li>1 cup milk</li>
            </ul>
            <div class="recipe-instructions">
                <strong>Blend:</strong> Instructions here
            </div>
        </div>
    </div>
</div>
```

**Category Colors:**
```css
[data-category="smoothies"] { --tag-color: #FF6B6B; }
[data-category="cocktails"] { --tag-color: #8B5CF6; }
[data-category="mocktails"] { --tag-color: #06B6D4; }
[data-category="juices"] { --tag-color: #F59E0B; }
[data-category="desserts"] { --tag-color: #EC4899; }
[data-category="breakfast"] { --tag-color: #10B981; }
[data-category="healthy"] { --tag-color: #22C55E; }
```

---

### 8. Performance Optimizations

| Optimization | Implementation |
|--------------|----------------|
| **Image Compression** | Resize to 500px max, ~300KB per image |
| **Lazy Loading** | Add `loading="lazy"` to images below fold |
| **CSS Animations** | Use `transform` and `opacity` (GPU accelerated) |
| **Minimal JS** | Vanilla JS only, no frameworks |
| **Single HTML File** | All CSS/JS inline for fewer HTTP requests |

---

## Deployment Checklist

- [ ] Compress all images (< 400KB each)
- [ ] Test on mobile device (iOS Safari, Android Chrome)
- [ ] Test 3D card flip on mobile
- [ ] Verify auto-scroll works smoothly
- [ ] Check all recipe images load
- [ ] Test category filtering
- [ ] Commit to git
- [ ] Push to GitHub
- [ ] Deploy to Cloudflare

---

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| 3D flip shows both sides on iOS | Add `-webkit-backface-visibility: hidden` |
| Images load slowly | Compress to < 400KB using `sips -Z 500` |
| Auto-scroll jumps | Duplicate cards and animate to `-50%` |
| HuggingFace 410 error | Use `router.huggingface.co` endpoint |
| Cloudflare deploy fails | Run `npx wrangler login` first |

---

## Quick Commands

```bash
# Compress image
sips -Z 500 "large-image.png" --out "compressed.png"

# Deploy to Cloudflare
npx wrangler pages deploy . --project-name=trufud-fruit-pulp

# Push to GitHub
git add -A && git commit -m "Message" && git push origin main

# Generate HuggingFace image
curl -X POST "https://router.huggingface.co/hf-inference/models/black-forest-labs/FLUX.1-schnell" \
  -H "Authorization: Bearer $HF_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"inputs": "prompt here"}' --output output.png
```

---

*Last Updated: January 2026*
