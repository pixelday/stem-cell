# Stem Cell

**A generic, accessible, standards-first front-end foundation that adapts to any brand through design tokens.**

Stem Cell is a web framework designed and built by and for designers and front-end developers, with a strong emphasis on markup quality. Semantics, accessibility, SEO-friendliness, standards compliance, general best practices, and a best-in-class user experience are all essential.

This document currently describes Stem Cell's design and evaluation process. Usage, installation, and contribution guides will follow.

*Draft document. The library is pre-alpha and under active development. Last updated: May 31, 2026.*

## Philosophy

We're calling the framework "Stem Cell" because, like its biological namesake, it is designed to be generic and fungible and can be quickly instanced and adapted into any custom branded experience with minimal changes through modification of design tokens and various configuration settings.

One of the core tenets of this project is that front-end web code (HTML in particular) is the "tip of the spear" of the web. It is the layer that search crawlers, bots, validators, and audits evaluate. But more importantly, it is what is directly presented to the user, so getting it right is critical.

While HTML and CSS are straightforward technologies for engineering teams to understand, they are easy to underestimate, hard to master, and so often neglected.

We are aiming for the output of this framework to be **WCAG 2.2 Level AAA conformant out of the box**, with one important and honest caveat. W3C/WAI itself notes that AAA is *not* recommended as a blanket requirement for entire sites, because some AAA success criteria simply cannot be satisfied for all types of content. So we treat AAA as a **shared responsibility**:

- **Stem Cell guarantees** the criteria a framework can control through its defaults and components. These include 2.1.3 Keyboard (No Exception), 2.2.3 No Timing, 2.3.2 Three Flashes, and 2.4.10 Section Headings.
- **Stem Cell enables** the remaining, content-dependent criteria so they are achievable by the author rather than fought against. These include 1.4.6 Contrast (Enhanced), 1.4.8 Visual Presentation, 1.4.9 Images of Text, 3.1.5 Reading Level, 2.4.9 Link Purpose (Link Only), the time-based-media criteria, and 3.3.6 Error Prevention (All).

A note on terminology we try to keep straight throughout: WCAG defines **conformance** (levels A, AA, AAA), while **compliance** refers to legal or policy obligations such as the ADA, Section 508, or the European Accessibility Act (EAA, enforceable as of June 2025). The EAA's technical baseline, EN 301 549, currently maps to WCAG **Level AA**, so our default target deliberately exceeds the legal bar. Because of that, adapting Stem Cell *down* to a brand that only needs AA conformance is comparatively easy for developers.

## Design system

Stem Cell's design layer is token-driven and built from the same first principles as its markup: start from a minimal, valid, accessible default, and let everything brand-specific flow from a small set of curated design tokens (CSS custom properties).

Note that this section of content is an early work in progress. While the color system below is currently the most fully specified piece, the remaining areas are planned and will be expanded incrementally as the library is being developed:

- **Token taxonomy beyond color** (planned): type scale, spacing, line length, focus styles, and motion, including the constraints that back specific AAA criteria (for example, the line-length and spacing requirements in 1.4.8 Visual Presentation).
- **Component acceptance criteria** (planned): the bar a component must clear to be considered done, such as a documented keyboard model per the ARIA Authoring Practices Guide, reduced-motion and forced-colors behavior, a no-JavaScript fallback, and AAA contrast across every background family.
- **Decision-making and curation** (planned): how token values are chosen, who signs off, and how trade-offs or exceptions are recorded.

### Color system

Color in Stem Cell is **background-driven**. Rather than hard-coding text colors, every content color is derived from the background color of the section it lives in, and each background/foreground pair is curated to meet AAA contrast minimums (7:1 for normal text, 4.5:1 for large text). We never ship a color in isolation. We only ship *pairs*, and every pair is verified against the WCAG 2.x contrast formula.

Colors are expressed as **design tokens via CSS custom properties**, organized into **families** rather than fixed values. A token like `--color-accent` doesn't resolve to a single hex value. It resolves to whichever member of the accent family is correctly paired with the current section background. Style a `<section>` with a black background and any `accent` content automatically inherits the accent value that maintains contrast against black. Style it white and it inherits the white-paired value instead. The same idea extends to near-neutrals: an off-white or light-gray section (a common choice to complement pure-white sections, especially on marketing pages) shifts the accent slightly, still within the same family and still meeting contrast.

The framework supports the `prefers-color-scheme` media query (and other `prefers-*` queries, covered in the testing notes below), so the light and dark families switch automatically with the user's stated preference.

Each default pairing below is anchored to its background color and meets the AAA contrast target for its role:

| Mode | Background color | Role | Text color | Text contrast | Accent color | Accent contrast | AAA target |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Light | `#FFFFFF` | Normal text | `#595959` | 7.00:1 | `#0037FF` | 7.02:1 | ≥ 7:1 |
| Light | `#FFFFFF` | Large text | `#767676` | 4.54:1 | `#006DFF` | 4.53:1 | ≥ 4.5:1 |
| Dark | `#000000` | Normal text | `#959595` | 7.01:1 | `#0099FF` | 7.00:1 | ≥ 7:1 |
| Dark | `#000000` | Large text | `#757575` | 4.55:1 | `#006AFF` | 4.50:1 | ≥ 4.5:1 |

Large-text values may be lighter than normal-text values because AAA requires only 4.5:1 at large sizes, versus 7:1 for normal text. The 7:1 values are always safe to use at any size.

The above color values are a sample of our work-in-progress token set and are subject to change. Each family carries additional members for the intermediate neutral backgrounds (light gray, dark gray, etc.). Every value in the default palette is curated and validated by a human designer.

*Contrast ratios are shown as reported by the WebAIM Contrast Checker, which truncates to two decimal places rather than rounding, so a displayed value never overstates the true ratio.*

**Inheritance and background resets:** because content colors are driven by the *nearest* background, any DOM node that resets its sub-tree's background also resets the content colors beneath it. For example, a `<section>` with a dark background that contains a `<div>` with a light background will give that div's descendants the colors paired with a *light* background (darker text and accents), not the light-on-dark colors used elsewhere in the section. This keeps every leaf node's text correctly paired with whatever background is actually behind it, regardless of nesting depth.

## Technology & scope

Stem Cell is not built on any existing frameworks and does not require a build system. The current target is raw HTML, CSS, and (as minimal as possible) vanilla JavaScript. While we may expand in the future to an officially supported TSX/React implementation, that is out of scope for this repository.

## Built from first principles

This project started as a "hello world" page. It had the absolute bare minimum markup to validate across all tools tested. Starting from the first principles of "minimum code to remain valid and compliant and always shippable," every incremental change is thoroughly tested. This discipline builds toward the release gate described in the When we ship section below.

## Auditing tools

These tools are all run manually today, with no continuous-integration (CI) testing yet. The list below is what we currently use:

- [W3C Nu Html Checker](https://validator.w3.org/nu/): the live HTML5/CSS/SVG validator (the legacy [Markup Validation Service](https://validator.w3.org/) is effectively frozen)
- [W3C CSS Validation Service (Jigsaw)](https://jigsaw.w3.org/css-validator/)
- [WAVE](https://wave.webaim.org/extension/) (browser extension)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [axe DevTools](https://www.deque.com/axe/devtools/extension/chrome/) (browser extension, powered by the axe-core engine)
- [Lighthouse](https://developer.chrome.com/docs/lighthouse) (in Chrome DevTools, with accessibility audits powered by axe-core)
- [Microsoft Accessibility Insights](https://accessibilityinsights.io/) (automated checks plus a guided manual assessment)
- [Markuplint Playground](https://playground.markuplint.dev/)
- [Issues tab](https://developer.chrome.com/blog/issues-tab) in Chrome DevTools
- [Google Schema Markup Validator](https://validator.schema.org/)
- [Google Rich Results Test](https://search.google.com/test/rich-results)
- [Ahrefs Site Audit](https://ahrefs.com/site-audit)

As of this writing, the output shows zero errors, zero issues, and zero warnings across the board, and scores 100% in all Lighthouse categories.

It's worth stating plainly what that does and doesn't prove: a perfect automated score is our **baseline, not evidence of full conformance**. Automated tooling detects only part of WCAG: per Deque, roughly 57% of issues by volume and only about 30% of success criteria are machine-detectable, so the manual and assistive-technology testing described below is what we actually rely on to substantiate an AAA claim.

**Planned CI additions.** Likely additions include [HTML Validate](https://html-validate.org/) for markup linting and (for the future TSX/React path) [eslint-plugin-jsx-a11y](https://github.com/jsx-eslint/eslint-plugin-jsx-a11y), plus a headless accessibility engine such as [Pa11y](https://pa11y.org/) (`pa11y-ci`), [axe-core](https://github.com/dequelabs/axe-core) driven through Playwright or Cypress, and [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci). All of these can run against the built static HTML, so the absence of a build system isn't a blocker.

Side note: We have not yet hit a case where two authoritative tools give conflicting recommendations, and we expect it to be rare. If it happens, accessibility wins, and W3C-developed tools carry more authority.

## Browser testing

The table is organized by rendering engine rather than browser. On iOS, third-party browsers (Chrome, Firefox, Brave, etc.) are in practice still WebKit under the hood, so testing in Mobile Safari effectively covers them all.

One caveat worth recording: the EU Digital Markets Act forced Apple to *permit* alternative engines (Gecko, Blink) on iOS as of iOS 17.4 (March 2024), and Japan and the UK are moving the same direction. But as of mid-2026 no vendor has actually shipped a non-WebKit browser on iOS, so the "iOS = WebKit" assumption still holds for our purposes. We'll revisit if that changes.

We definitely test the bolded browsers listed on each platform, but are unlikely to test Chrome, Brave, and Opera on the same system, since they are unlikely to present meaningful rendering differences.

| Engine | Device type | Operating System | Browsers |
| --- | --- | --- | --- |
| Chromium | desktop | macOS | **Chrome**, Brave, Opera, etc. |
| Chromium | desktop | Windows 11 | **Edge** or Chrome, Brave, Opera, etc. |
| Chromium | desktop | Linux | **Chrome** or Brave, Opera, etc. |
| Chromium | mobile | Android | **Chrome Mobile**, Brave Mobile, Opera Mobile, etc. |
| Chromium | tablet | Android | **Chrome Mobile**, Brave Mobile, Opera Mobile, etc. |
| WebKit | desktop | macOS | **Safari** |
| WebKit | mobile | iOS | **Mobile Safari** |
| WebKit | tablet | iPad OS | **Mobile Safari** |
| Gecko | desktop | macOS | **Firefox** |
| Gecko | desktop | Windows 11 | **Firefox** |
| Gecko | desktop | Linux | **Firefox** |
| Gecko | mobile | Android | **Firefox Mobile** |
| Gecko | tablet | Android | **Firefox Mobile** |

Note that before shipping, we may use a service like BrowserStack or Playwright to test any long-tail browser rendering differences.

## Manual testing checklists

Beyond automated testing, which in our minds merely gets us to a baseline, we work through established external checklists for the things automated audits don't cover:

- [WebAIM Evaluation Quick Reference](https://webaim.org/resources/evalquickref)
- [WebAIM Checklist](https://webaim.org/standards/wcag/checklist/)
- [A11y Project Checklist](https://www.a11yproject.com/checklist/)

In addition, we run the following manual checks that automated tools largely cannot catch, each mapped to the relevant WCAG criteria:

- **Zoom and reflow:** text resize to 200% (1.4.4) and reflow at 320 CSS px / 400% zoom with no loss of content and no horizontal scrolling (1.4.10).
- **Text spacing:** apply the standard text-spacing overrides (line height 1.5×, paragraph spacing 2×, letter spacing 0.12×, word spacing 0.16×) with no clipping or overlap (1.4.12).
- **Non-text contrast:** UI components, focus indicators, and meaningful graphics meet 3:1 (1.4.11).
- **User preferences and forced colors:** correct behavior for `prefers-reduced-motion` (2.3.3) and `prefers-color-scheme`, plus Windows High Contrast / forced-colors mode (the `forced-colors` media query). The Chrome DevTools Rendering panel can emulate all three.
- **Progressive enhancement:** the page remains usable and legible with JavaScript disabled, with CSS disabled, and with images off, a natural fit for a minimal-JS framework and the clearest proof that meaning lives in the markup.

## Assistive technology testing

We don't currently have a process for procuring and testing hardware input devices (switches, head pointers, sip-and-puff controls, etc.), but taking UX research and usability testing to the next level is on our roadmap.

In the meantime, keyboard-only navigation is a hard gate: nothing ships unless the entire page can be operated without ever touching the mouse. (Keyboard access isn't a stand-in for "motor impairment" specifically. It underpins the experience for screen-reader users, switch and voice-control users, and keyboard power users alike.) Our keystroke coverage: Tab, Shift+Tab, Enter, Space, Escape, the arrow keys (Up, Down, Left, Right), Home, End, Page Up, and Page Down. The arrow keys plus Home/End matter specifically for composite ARIA widgets (menus, tabs, listboxes, grids, etc.), as described in the [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/).

Beyond the keyboard, several software-based assistive technologies are testable on commodity hardware and are on our list to fold in: Voice Control (macOS/iOS) and Voice Access (Windows), OS-level screen magnification, and switch-control emulation (built into macOS, iOS, and Android).

We test everything with a screen reader. Current pairings:

- [VoiceOver](https://support.apple.com/guide/voiceover/welcome/mac) + Safari (macOS)
- [VoiceOver](https://support.apple.com/guide/iphone/use-voiceover-in-apps-iphe4ee74be8/ios) + Safari (iOS): our primary mobile pairing
- [NVDA](https://www.nvaccess.org/about-nvda/) + Firefox/Chrome (Windows 11)

Known gaps we intend to close, in priority order:

- **Android TalkBack:** we already test Android Chrome and Firefox, so testing their rendering without TalkBack leaves the actual Android assistive-technology experience unverified. This is our highest-priority addition.
- **JAWS + Chrome:** per WebAIM's Screen Reader User Survey, this is the single most common real-world desktop pairing. We're currently deferring it only because JAWS is paid. NVDA is a reasonable (though not equivalent) free proxy in the meantime.
- **Windows Narrator** and **Orca (Linux):** both untested and lower-priority given their usage share, but noted for completeness.

## Performance

Lighthouse gives us lab performance scores. For field data we use [PageSpeed Insights](https://pagespeed.web.dev/), which runs Lighthouse on Google's servers and overlays real-user Core Web Vitals from the Chrome UX Report: Largest Contentful Paint, Interaction to Next Paint (INP, which replaced First Input Delay in 2024), and Cumulative Layout Shift. We cross-check with [WebPageTest](https://www.webpagetest.org/). A minimal-JS, no-build framework should have a natural advantage here, and we treat that as a claim to verify rather than assume.

## On the horizon: WCAG 3.0

Our conformance target is WCAG 2.2, the current W3C Recommendation (now also ISO/IEC 40500:2025). WCAG 3.0 (retitled the "W3C Accessibility Guidelines") is worth tracking but is still an early Working Draft (a fresh draft landed in March 2026) and is explicitly not citable as anything other than a work in progress. It proposes a different conformance model (graded outcomes with Bronze/Silver/Gold tiers rather than binary A/AA/AAA) and a broader scope beyond the web. Realistic projections put a Candidate Recommendation no earlier than roughly 2027, with a full Recommendation later still. Either way, 3.0 will coexist with 2.2 for years rather than replacing it overnight.

One specific thing to watch: the Accessible Perceptual Contrast Algorithm (APCA) is often described as 3.0's replacement for the 2.x contrast-ratio math. It's promising, but it is *not* in the current normative draft, and the 3.0 contrast method is officially undecided, so we continue to design and verify against the 2.x ratios (which is what our color families are built on) and treat APCA as research to monitor.

## When we ship

Only when every automated check is clean *and* every manual and assistive-technology check passes (navigable, clear, and working as intended) do we ship. The automated green light is necessary but not sufficient. The manual and assistive-technology passes are what we actually trust.
