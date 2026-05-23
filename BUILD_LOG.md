# HoF Site Redesign — Build Log

A step-by-step record of decisions, problems, and solutions. This log will inform the Blueprint client site template.

---

## Step 1 — Scaffold Astro Project

**What we did:**
Initialized a blank Astro project in the HoF Site Redesign folder using the minimal (empty) starter.

```bash
npm create astro@latest .
```

**Settings chosen:**
- Template: Empty
- TypeScript: No
- Install dependencies: Yes

**Result:**
Standard Astro scaffold generated:
- `src/pages/index.astro` — single starting page
- `public/` — static assets folder
- `astro.config.mjs` — config file
- `tsconfig.json`, `package.json`, `package-lock.json`

**Notes:**
No issues. Clean baseline to build from.

---

## Step 2 — Confirm Dev Server

**What we did:**
Ran `npm run dev` and confirmed the site loads at `localhost:4321`.

**Result:**
Blank Astro page renders as expected. Baseline confirmed.

**Notes:**
No issues.

---

## Step 3 — Add Tailwind CSS

**What we did:**
Ran the Astro Tailwind integration:

```bash
npx astro add tailwind
```

**Issues:**
CLI prompted to add the `global.css` import manually rather than doing it automatically.

**Solution:**
Manually added `import '../styles/global.css'` to `Layout.astro`. The CLI had already created `src/styles/global.css` and `src/layouts/Layout.astro`.

**Result:**
Tailwind installed and configured. `tailwind.config.mjs` created and `astro.config.mjs` updated automatically.

---

## Step 4 — Wire Up Layout Component

**What we did:**
Moved the HTML shell out of `index.astro` and into `src/layouts/Layout.astro`. Pages now import and wrap content with `<Layout>`.

- `Layout.astro` — holds `<html>`, `<head>`, `<body>`, `<slot />`, and the global CSS import
- `index.astro` — stripped down to just the layout import and page content

**Result:**
Clean separation between page structure and page content. Every future page gets the shell automatically by using `<Layout>`.

---

## Step 5 — Nav as a Standalone Component

**Decision:**
Built the nav as `src/components/Nav.astro` rather than inline in the layout.

**Why:**
Nav will grow in complexity — mobile menu, active states, conditional elements. Isolating it makes it easier to manage and modify independently.

**Pattern:**
Layout imports Nav and renders it. Pages stay unaware of nav entirely.

---

## Step 6 — Nav Data Config File

**What we did:**
Created `src/data/nav.js` as a config-driven approach to nav links instead of hardcoding them in the component.

**Why:**
Separating data from presentation makes the Nav component reusable across client sites. To set up a new client's nav, you only touch the data file — not the component logic.

**Structure:**
- `navLinks` — array of link objects, supports a `children` array for sub-menus and a `temp` flag for temporary items
- `ctaLink` — separated from navLinks since it renders differently (as a button)
- Sub-menu trigger decided: **hover dropdown** — purely a CSS/JS concern, no impact on data structure

**CTA label decision:**
Chose "Support" over "Serve" — covers both volunteering and giving without favoring either.

**Notes:**
- Adding or removing nav items only requires editing `nav.js`
- Hover vs click can be changed later without touching the data file

---

## Step 7 — Build Nav Component

**What we did:**
Built `src/components/Nav.astro` to consume the nav data config and render the nav links dynamically.

**Result:**
Nav renders in the browser with all links and CTA pulled from `nav.js`. Imported into `Layout.astro` so it appears on every page.

---

## Issue — Dev Server Hanging

**Problem:**
`npm run dev` would hang after `> astro dev` with no output and no local URL.

**Cause:**
Project was located inside iCloud Drive (`~/Library/Mobile Documents/...`). iCloud's file syncing (fileproviderd) was interfering with Astro's dev server and file watching.

**Solution:**
Moved the project to a local folder outside iCloud: `~/LocalBlueprintFiles/HoF Site Redesign/`

**Template note:**
Always store active Astro dev projects in a local folder, not inside iCloud Drive, Google Drive, or any other synced folder. Use version control (git) for backup instead.

---

## Step 8 — Placeholder Pages & Routing

**What we did:**
Created placeholder pages for all nav links:
- `src/pages/about.astro`
- `src/pages/events.astro`
- `src/pages/programs.astro`
- `src/pages/building-project.astro`
- `src/pages/support.astro`

**How Astro routing works:**
Every file in `src/pages/` automatically becomes a route. No router config needed.

**Result:**
All nav links resolve to their pages. No 404s.

---

## Step 9 — Footer Component

**What we did:**
Created `src/data/footer.js` and `src/components/Footer.astro` following the same data-driven pattern as the nav.

**Footer data structure:**
- `footerLinks` — mirrors nav links in vertical format
- `socialLinks` — placeholder social links using `href: '#'` until real URLs are available
- `copyrightText` — single string, easy to update per client

**Issue:**
Dev server threw `ctaLink is not defined` — Footer.astro had a leftover `<a href={ctaLink.href}>` line copied from Nav.astro. `ctaLink` is not exported from `footer.js`.

**Solution:**
Removed the ctaLink reference from Footer.astro.

**Template note:**
When copying component patterns, double check imports match what the data file actually exports.

---

## Step 10 — Active Link State

**What we did:**
Added current page detection to the nav using `Astro.url.pathname`. Compared against each link's `href` and conditionally applied an `active` class.

**How it works:**
```js
const currentPath = Astro.url.pathname;
```
Then in the map, the `active` class is added when `link.href === currentPath`.

**CSS:**
Added `.active a` and `a.active` selectors to `global.css` to style active links.

**Notes:**
- CTA left without active state for now — will be converted to a button component later
- Template literals allow mixing plain strings and expressions: `` `base-class ${condition ? 'a' : 'b'}` ``

---

## Step 11 — Footer Pinning

**What we did:**
Made the footer stick to the bottom of the viewport even on short pages.

**Pattern:**
- `<body>` gets `flex flex-col min-h-screen`
- `<main>` wrapping the `<slot />` gets `flex-grow`
- Footer naturally sits at the bottom

**Result:**
Footer is pinned on all pages including placeholder pages with minimal content.

---

## Step 12 — Page Layout Structure & Title Props

**What we did:**
- Updated `Layout.astro` to accept a `title` prop passed to the `<title>` tag in `<head>`
- Created `src/layouts/PageLayout.astro` — a nested layout with a hero section and content area
- Updated all placeholder pages to use `PageLayout` with a `title` prop

**How nested layouts work:**
`PageLayout` imports and wraps `Layout`, passing the title prop up the chain. Pages only need to import `PageLayout` — `Layout` is handled internally.

**Pattern:**
- `Layout.astro` — shell (nav, footer, head). Never changes between pages.
- `PageLayout.astro` — inner page structure (hero + content). Used by standard pages.
- Future layouts (PostLayout, EventLayout, etc.) follow the same pattern.

**Notes:**
- Homepage stays on `Layout` directly — it will have a unique structure
- `title` prop drives both the browser tab and the hero `<h1>` automatically

---

## Step 13 — Homepage Structure

**What we did:**
Built out the homepage sections in `index.astro` using `BaseLayout` directly (not `PageLayout`) since the homepage has a unique structure.

**Sections:**
- Hero — heading and subtext
- Mission — mission statement placeholder
- Events — upcoming events placeholder
- Learn More — links to About and Programs

**Notes:**
- Use hyphens not underscores for CSS class names (`learn-more` not `learn_more`)
- Homepage stays on `BaseLayout` — `PageLayout` is for standard inner pages only
- Content is hardcoded for now, content collections can be added later for dynamic sections like events

---

## Step 14 — Mobile Nav (Hamburger Menu)

**What we did:**
Added a mobile hamburger menu to `Nav.astro` with a dropdown that shows/hides the nav links.

**Approach:**
- Hamburger button uses a Unicode character (`☰`) — simple, no dependencies
- `md:hidden` on the button — visible mobile only
- `hidden md:flex` on the nav links — hidden mobile by default, flex on desktop
- CTA hidden on mobile for now
- Client-side `<script>` tag toggles the `hidden` class on click

**First client-side JS in the project:**
Unlike frontmatter JS which runs on the server, the `<script>` tag runs in the browser. Required for interactive elements like toggles.

**Notes:**
- Dropdown works but unstyled — visual polish deferred
- Can be swapped for full screen overlay or side drawer later without changing the data structure

---

## Next Steps
1. Push `AstroTemplate` to GitHub as a reusable starter template
2. Copy into a new project folder for the Hart of Folsom site build

---

## Using This Template — Setup Steps

When starting a new project from this template:

1. **Pull the template using degit** (repo must be public):
```bash
npx degit Blueprint-Story-Co/AstroSiteTemplate YourProjectName
```

2. **Install dependencies** — `node_modules` is not included in the repo:
```bash
cd YourProjectName
npm install
```

3. **Start the dev server:**
```bash
npm run dev
```

**Important notes:**
- Always run `npm install` first — every project needs its own `node_modules`
- Store projects in a local folder, not inside iCloud or Google Drive
- `node_modules` is in `.gitignore` by design — never commit it
