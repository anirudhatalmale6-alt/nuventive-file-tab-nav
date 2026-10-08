# Where to put the CSS

The snippet is `nuventive-file-tab-nav.css`. It is pure CSS — no JavaScript, no
changes to your menu, no changes to any template file.

Pick **one** of the two places below. Both survive theme and Avada updates.

---

## Option 1 — Additional CSS (quickest, recommended)

1. WordPress admin → **Appearance → Customize → Additional CSS**
2. Paste the whole contents of `nuventive-file-tab-nav.css` at the bottom
3. **Publish**

This lives in the database, so nothing can overwrite it when Avada or the theme
updates.

## Option 2 — Child theme stylesheet

1. **Appearance → Theme File Editor**, with **Avada Child Theme** selected
2. Open `style.css`
3. Paste the whole contents of `nuventive-file-tab-nav.css` at the bottom
4. **Update File**

(If you edit over SFTP instead, the file is
`/wp-content/themes/Avada-Child-Theme/style.css`.)

> Your site runs **WP Rocket**, which minifies and caches CSS. After pasting,
> go to **Settings → WP Rocket → Clear cache**, then hard-refresh the page
> (Ctrl/Cmd + Shift + R) or you will keep seeing the old stylesheet.

---

# Adjusting it later

Everything is driven by the block of variables at the top of the file. You never
need to touch the rules underneath.

```css
--nv-hover-bg:      #09332E;  /* tab fill on hover */
--nv-hover-text:    #FFFFFF;  /* label colour on hover */
--nv-current-bg:    #09332E;  /* tab fill for the page you are on */
--nv-current-text:  #FFFFFF;  /* label colour for that tab */

--nv-radius:        9px;      /* roundness of the tab's top corners */
--nv-flare:         0px;      /* 0 = straight sides (manila folder). 12-16px = flared feet */
--nv-pad-x:         14px;     /* how far the tab reaches past the label */
--nv-inset-top:     16px;     /* white space left above the tab */
--nv-drop:          6px;      /* bottom of the menu row -> bottom of the header.
                                 6px on nuventive.com, 0 on the staging site */
--nv-speed:         260ms;    /* fade speed */
--nv-lift:          none;     /* drop-shadow(...) if you want the tab to cast a shadow */
```

A few examples:

* **Narrower / wider tab** → `--nv-pad-x: 10px;` / `--nv-pad-x: 24px;`
* **Shorter / taller tab** → raise or lower `--nv-inset-top`
* **Dropdown floating below the bar instead of meeting the tab** →
  change `--awb-submenu-space` in section 2 from `0px` to `12px`
* **Flared "feet" instead of straight sides** → `--nv-flare: 16px;`
* **No fade, instant** → `--nv-speed: 0ms;`
* **Give the tab a shadow** → `--nv-lift: drop-shadow(0 -2px 5px rgba(9,51,46,.2));`

Because the variables block is written without an `#id`, you can override any single
value later with a one-line rule and it will win:

```css
ul[id^="menu-main-menu"]{ --nv-hover-text:#CFD200; }
```

---

# What the snippet does, section by section

1. **Dials** — the variables above.
2. **Item height** — makes each menu item as tall as the white header bar, so the
   area you can hover is exactly the tab you can see (no dead pixels at the
   bottom of the tab). The label itself does not move by a single pixel.
   It also pulls Avada's dropdown offset in by the same amount, so the flyout
   opens in exactly the same place it does today.
3. **The tab** — Avada already paints a background layer behind every menu item
   and already fades it in. The snippet only reshapes that layer into a folder
   tab (rounded top, straight sides, sitting flush on the bottom edge of the
   white bar) and recolours it.
4. **Active state** — the tab for the page you are on stays raised. It covers `current-menu-item` (that exact page) and
   `current-menu-ancestor` / `current-menu-parent` (any page inside that
   dropdown), so "Solutions" stays lit while you are on *Program Review*.
5. **Keyboard** — tabbing through the menu with the keyboard raises the same tab.
6. **Reduced motion** — the fade is dropped for visitors who ask their OS for
   reduced motion.
7. **Mobile** — every rule is scoped to `:not(.collapse-enabled)`, so the
   collapsed/mobile menu below 1200px is left exactly as it is today.

---

# One small bonus fix

Today, hovering a menu item makes Avada swap in a 2px "active border", which is
applied as extra padding. That widens the hovered item by 4px and nudges every
other label sideways by up to 1.5px. The snippet zeroes that border (the tab
draws its own edge), so that wobble — which exists on the site right now, before
any of this — goes away.

Measured on the live Contact page, hovering *Upcoming Events*:

| | shift of the other labels |
|---|---|
| site as it is today | −1.5px … +1.5px |
| with the snippet | 0px |

---

# Optional — tab only on the page you are on

`nuventive-file-tab-nav.css` keeps the tab following the mouse, the way it has
behaved all along. If you want the tab to appear **only** on the page you are
actually on, paste `optional-current-page-only.css` after it. Leave that file
out and nothing changes.

Two things to be aware of in this mode:

1. **Pages that are not in the main menu show no tab at all.** The homepage,
   blog posts and event pages are not top-level menu items, so on those pages
   the menu is completely flat with no highlight anywhere. Verified on
   `/` (nothing lit), `/contact/` (Contact lit) and
   `/solutions/program-review/` (Solutions lit, via the dropdown ancestor).
2. **There is no mouse feedback on the menu at all.** If that feels too dead,
   uncomment the optional rule at the bottom of `optional-current-page-only.css`
   - it tints the label on hover without drawing a tab.

---

# Conflict with the child theme - please read

`Avada-Child-Theme/style.css` currently contains a **second, different
file-folder tab implementation** (variables named `--nv-tab-bg`,
`--nv-tab-rise`, `--nv-tab-drop`, selectors starting
`.awb-menu_desktop[aria-label="Main Menu"]`). It is live right now alongside
this file, and the two overlap.

The important one is its last rule:

```css
.awb-menu_desktop[aria-label="Main Menu"] > .awb-menu__main-ul:has(> li:is(:hover, :focus-within)) > li:not(:hover, :focus-within) > .awb-menu__main-background-active {
	opacity: 0;
}
```

That says *"while any item is hovered, hide the highlight on all the others"* -
which cancels current-page-only mode outright: the tab would vanish the moment
the mouse touched the menu. `optional-current-page-only.css` out-specifies it, so
it behaves correctly either way, but **the clean fix is to delete that child-theme block**
so only one implementation is live. Two sets of rules fighting over the same
element will cause confusing results the next time either is changed.

---

# Why --nv-drop is 6px on nuventive.com but was 0 on staging

The two sites do not have the same header. On **nuventive.com** there is an extra
row inside the header underneath the menu (`fusion-builder-row-3`, 6px tall,
5px of padding, no visible content). So:

* bottom of the menu's own row = 118
* bottom of `.fusion-tb-header` = 124

The tab is positioned from the menu row, so it needs 6px to reach the real
bottom edge of the header. On staging that extra row is hidden on desktop and
the two edges are the same, hence 0.

**Derive it, never guess it:** bottom of `.fusion-tb-header` minus bottom of the
menu's own `.fusion-fullwidth` row. Too small and a pale sliver shows under the
tab; too big and the tab hangs over the page.

If you would rather not carry the offset at all, removing the 5px padding from
that empty row makes both edges line up and `--nv-drop` goes back to 0 - but
that nudges everything below the header up by 6px on every page, so it is your
call.

---

# The tab must never reach past the header bar

`--nv-drop` is how far the tab's foot extends below the menu row. Your header
band has **no padding under the menu row**, so the row's bottom edge already *is*
the bottom of the white bar — any drop above `0` hangs over the page content
underneath. That is what section 5c guards:

```css
.fusion-container-stuck ... { --nv-drop: 0px; bottom: 0; }
```

Avada adds `.fusion-container-stuck` to the header when you scroll and it
shrinks. The rule pins the drop to zero while stuck no matter what the dial says,
so the scrolled header can never overhang even if `--nv-drop` is raised later.

Measured on `/solutions/` and `/contact/`, at the top of the page and scrolled:
tab foot and header bottom edge on the same pixel, overhang 0 in all four cases.

---

# Tested on

* `nuventive3dev.wpenginepowered.com` — home, `/contact/` (current-page tab),
  and `/solutions/program-review/` (dropdown-ancestor tab), measured against
  the live page with the child theme's conflicting rules in place
* Desktop 1440px, the sticky/shrunk header after scrolling, and mobile 390px
* Chromium / Chrome, and the same CSS features are supported by Firefox, Safari
  15.4+ and Edge. The only modern selector used is `:has()`, and it is only used
  as a belt-and-braces duplicate of a rule that is also written with a plain
  attribute selector — so nothing breaks if a browser does not support it.
