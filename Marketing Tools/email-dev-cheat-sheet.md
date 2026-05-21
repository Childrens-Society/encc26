# Email development cheat sheet

This was meant to be a brief "cheat sheet" but it's became unwieldy. 

The structure:
1. Useful resources
2. How far behind is email development?
3. What you cannot use
4. What's required / what works
5. Email client differences
6. Responsiveness
7. Dark mode and light mode
8. Testing
9. Accessibility
10. Upsides of email development
11. Other things (Gmail clipping, preheader text, template versioning, frameworks, engagement)

There's also a quick do/don't list at the end.

> This is a reference guide for developing email content. This is intended for anyone developing email templates in 2026. It may be especially useful for developers coming from a web background. Citations are included so you can dig deeper.


## 1. Useful resources

Articles:
* [DesignModo](https://designmodo.com/html-css-emails/) - "HTML and CSS in Emails: What Works in 2026?"
* [Email on Acid](https://www.emailonacid.com/blog/article/email-development/email-development-best-practices-2/) - "Email Development Best Practices" (from 2023 but still relevant)
* [https://www.htmlemailbuilders.com/blog/responsive-email-design-guide](https://www.htmlemailbuilders.com/blog/responsive-email-design-guide) - "Responsive Email Design: A Technical Guide"

References and tools:
* [Can I Email](https://www.caniemail.com) - to check HTML/CSS feature support across email clients
* [Email on Acid Blog](https://www.emailonacid.com/blog/) - insights and tutorials on email design and development
* [Litmus Blog](https://www.litmus.com/blog/) - news and updates from the email testing and development world
* [Really Good Emails](https://reallygoodemails.com) (for design inspiration) - examples of well-designed emails
* [MJML](https://mjml.io) - a responsive email framework that simplifies email development
* [Google Workspace Gmail Design](https://developers.google.com/workspace/gmail/design/css) - guidelines for designing emails in Gmail -- this may also be covered in "Can I Email" but Google's own documentation is worth checking for Gmail-specific quirks and features.


---

## 2. How Far Behind Is Email Development?

Email development requires **table-based layout** — the same technique the web industry moved away from in the early-to-mid 2000s. CSS-based layout took over on the web during that period and tables have not been used for page layout in professional web development since.

> "With the push toward web standards in the early 2000s, tables were pushed aside as a layout solution in favor of other approaches, most notably the CSS float property."
> — [CSS-Tricks](https://css-tricks.com/in-defense-of-tables-and-floats-in-modern-day-development/)

> "I launched my web development company in September of 2001, and the only time I used tables-for-layout in a client project was for HTML email."
> — [Rachel Andrew](https://rachelandrew.co.uk/archives/2023/09/07/why-are-we-not-still-using-tables-for-layout/) (CSS Working Group member)

That puts the lag at roughly **20 years on layout alone** — and email doesn't use web-standard features such as JavaScript, semantic HTML, flexbox, grid or linked stylesheets.

There **do not seem to be enforced standards for email clients** as there are for browsers. Email clients renders HTML and CSS differently, and none are obligated to follow web standards.

> "There is NO strict adherence to web standards by email clients that display HTML email."
> — [Martech Zone](https://martech.zone/understanding-the-challenges-and-frustrations-of-html-email-design/)


---

## 3. What You Cannot Use

### JavaScript
It's blocked.
> "Interactive elements such as Flash or JavaScript won't work in most email clients."
> — [Brevo](https://help.brevo.com/hc/en-us/articles/6632412983186)

This means form `<button type="submit">` buttons also don't work as expected, because they depend on JS.
> "The submit button requires JavaScript to work properly, but JavaScript is forbidden."
> — [Designmodo](https://designmodo.com/mailchimp-limitations-email/)


### CSS Flexbox & Grid
- `display: flex` has ~85% client support, but its child properties (`flex-wrap`, `align-items`, `justify-content`, etc.) have terrible support — making it unreliable.
- `display: grid` has very poor support across email clients.
> "Grid isn't well supported at all, although display:flex surprisingly works in 84.85% of email clients... However, flex's related CSS properties have terrible support — tables are the better choice."
> — [Designmodo](https://designmodo.com/html-css-emails/)

### Embedded Video & Audio
HTML5 `<video>` and `<audio>` tags are not supported by most clients. Apple Mail is the main exception.
> "As a rule, email clients do not support HTML5 `<video>` and `<audio>` tags."
> — [Designmodo](https://designmodo.com/mailchimp-limitations-email/)

### Semantic HTML Elements
Useful HTML elements that symantically structure a page, such as `<article>`, `<section>`, `<aside>`, `<nav>`, `<header>`, `<footer>`, etc. are not reliably safe to use for layout or structure in email.
> "This rules out the possibility of using semantic HTML elements (e.g. `<article>`) too, which is a shame for readability and accessibility reasons."
> — [Designmodo](https://designmodo.com/html-css-emails/)

### External CSS / Stylesheets
Linked stylesheets (`<link rel="stylesheet">`) are stripped by most email clients.

As a result, you must use inline styles or internal `<style>` blocks. However, some clients (notably Gmail) seem to also strip `<style>` blocks from the `<head>`, so even internal CSS is not universally reliable.

## 4. What's required / What works

### Tables
These seem to be the backbone of email layout, and are the only reliably cross-client method.
> "Emails are coded using tables for layout purposes due to the limitations and inconsistencies of HTML and CSS support in various email clients and platforms."
> — [Email on Acid](https://www.emailonacid.com/blog/article/email-development/email-development-best-practices-2/)

```html
<table role="presentation" cellpadding="0" cellspacing="0" border="0" width="100%">
  <tr>
    <td>
      CONTENT HERE
    </td>
  </tr>
</table>
```

`cellpadding="0"` and `cellspacing="0"` reset default browser spacing. 
`border="0"` removes the default table border. `role="presentation"` tells screen readers to skip the table structure.


![Alt text](https://c.tenor.com/_n_jxpQJ7b4AAAAd/tenor.gif)



### Inline CSS
Inline styles (`style="..."` on every element) are the most reliable approach, especially for layout-critical properties. Email clients that strip `<style>` blocks will fall back to these.
> "Undoubtedly, one of the most annoying aspects of creating HTML email templates is having to declare styles for every individual element within its style attribute."
> — [Designmodo](https://designmodo.com/html-css-emails/)

```html
<td style="padding: 20px; background-color: #ffffff; font-family: Arial, sans-serif; font-size: 16px; color: #333333;">
  CONTENT HERE
</td>
```

Every layout-critical property goes inline. A class does not seem reliable.

### Internal CSS (`<style>` blocks)
Internal CSS works in ~85% of email clients, but with caveats:
- Must be placed in the `<head>`, not inside `<body>` or a `<td>`.
- Always use `!important` for overrides that need to win over inline styles.
- Do not rely on these alone — always provide inline fallbacks.
> "Internal CSS works in 84.85% of today's email clients, but there are a few rules that must be followed."
> — [Designmodo](https://designmodo.com/html-css-emails/)

```html
<head>
  <style type="text/css">
    /* Resets */
    body, table, td { margin: 0; padding: 0; }

    /* Responsive override — needs !important to override inline styles */
    @media only screen and (max-width: 480px) {
      .column {
        display: block !important;
        width: 100% !important;
      }
    }
  </style>
</head>
```

### A Proper DOCTYPE
Always declare a DOCTYPE. Without it, email clients render in "quirks mode" and layouts break unpredictably.
> "A doctype... prevents the browser or email client from rendering the email in quirks mode."
> — [Email on Acid](https://www.emailonacid.com/blog/article/email-development/email-development-best-practices-2/)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="x-apple-disable-message-reformatting">
  <!--[if mso]>
    <style>table, td, div, p { font-family: Arial, sans-serif !important; }</style>
  <![endif]-->
</head>
```

`x-apple-disable-message-reformatting` prevents iOS Mail from resizing small-width emails. The MSO conditional comment forces a safe font stack in Outlook's Word renderer.

NB - More about [https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Quirks_mode_and_standards_mode](quirks mode). Quirks mode is a legacy rendering mode that emulates the behaviour of old browsers. It causes unpredictable layout issues.


### Fonts (font stacks)
Custom web fonts are partially supported (Apple Mail, Outlook for Mac), but always define fallbacks, as with web dev:
```html
<p style="font-family: 'Roboto', 'Arial', sans-serif;">...</p>
```
> "Font stacks allow you to specify multiple fonts in the CSS font-family property, providing a hierarchy of font choices that the email client will attempt to render based on availability."
> — [Email on Acid](https://www.emailonacid.com/blog/article/email-development/email-development-best-practices-2/)

### HTML attributes (not just CSS)
For layout properties, like `width`, `align`, `cellpadding`, `bgcolor`, use HTML attributes alongside CSS. Many email clients (esp. Outlook) respect attributes more reliably than CSS properties alone.

```html
<!-- Width: HTML attribute for Outlook, CSS for others -->
<table width="600" style="max-width: 600px; width: 100%;">

<!-- Background colour: both attribute and CSS -->
<td bgcolor="#ffffff" style="background-color: #ffffff;">

<!-- Alignment: both attribute and CSS -->
<td align="center" style="text-align: center;">

<!-- Images: set width attribute as well as CSS -->
<img src="hero-image.png" width="600" alt="Hero image"
     style="display: block; width: 100%; max-width: 600px; height: auto;">
```

### Plain text alternative
Send a `multipart/alternative` email with both HTML and plain text versions.
EN SUPPORTS THIS
> "It's also important to maintain identical messaging in both versions for consistency."
> — [MailSoar](https://www.mailsoar.com/blog/guide/html-email-best-practices/)

```
Hello,

We have a special offer for you this week.

View the offer: https://example.com/offer

To unsubscribe: https://example.com/unsubscribe
```

All links must be spelled out in full. There are no hyperlinks, no images, no formatting in plain text.


### Alt text for images
This is the same as web dev but images are frequently blocked by default in email clients. Always include descriptive `alt` text. Accessibility tools also require it.

```html
<!-- Descriptive alt text for content images -->
<img src="product.png" width="300" alt="Children's s blue raincoat with hood"
     style="display: block; width: 100%; max-width: 300px; height: auto;">

<!-- Empty alt for decorative images — screen readers skip these -->
<img src="divider.png" width="600" alt=""
     style="display: block; width: 100%; height: 2px;">

<!-- Alt text also renders as styled text when images are blocked in some clients -->
<img src="logo.png" width="200" alt="Acme Co"
     style="display: block; font-family: Arial, sans-serif; font-size: 24px; color: #333333;">
```

### Outlook and conditional comments
Outlook on Windows uses the Microsoft Word rendering engine to display emails. This requires special handling via HTML conditional comments:
> "The desktop versions of Outlook (2007–2019) utilize the Microsoft Word rendering engine to display HTML emails, which drastically limits support for modern web standards... The New Outlook for Windows (launched in 2023) abandons Word in favour of a modern web engine."
> — [CaptainVerify](https://captainverify.com/blog/issues-sending-emails-outlook.html)


Here's an example snippet for a centred, fixed-width container with a fluid fallback for all other clients:

```html
<!--[if mso]>
<table role="presentation" align="center" cellpadding="0" cellspacing="0" width="600">
  <tr>
    <td>
<![endif]-->

<div style="max-width: 600px; margin: 0 auto;">
  ADD CONTENT HERE
</div>

<!--[if mso]>
    </td>
  </tr>
</table>
<![endif]-->
```

In the above snippet:
* Outlook reads the conditional comment table and ignores the `<div>`. 
* Every other client ignores the conditional comments and uses the `<div>` with `max-width`.

---

## 5. Email client differences

### The core problem
Email clinets, such as Gmail, Outlook, Yahoo Mail, Samsung Mail, Apple Mail etc behave differently from one another.
> "Hundreds of email clients render HTML differently across desktops, apps, mobile devices, and webmail clients."
> — [Martech Zone](https://martech.zone/understanding-the-challenges-and-frustrations-of-html-email-design/)

### Key clients to know and notes about them
NOTE: Dark mode behaviour is complex and varies significantly even within the same client across OS versions. Some notes are here but also see the dark mode section for more details -- CONSOLIDATE THIS.


The same email client can behave differently depending on the OS:

- **Outlook on Windows vs. Mac**: Completely different rendering engines (Word vs. WebKit). Treat them as different clients.
- **Gmail on iOS vs. Android vs. Web**: Different levels of media query support, different dark mode behaviour, different colour inversion approaches.
- **Apple Mail on iOS vs. macOS**: Very similar but touch targets and viewport behaviour differ.
- **Outlook mobile (iOS/Android)**: More modern than desktop Outlook, but still has quirks; applies full colour inversion in dark mode.

> "If someone is using a different operating system or device, it can make it harder to use dark mode for email. That's why it's helpful to test how your campaigns will look on different devices and operating systems."
> — [Cakemail](https://www.cakemail.com/blog/post/create-the-perfect-email-to-be-displayed-in-the-complex-dark-mode)

More detail re clients and OSes:

| Client | Platform | Rendering Engine | Notes |
|---|---|---|---|
| **Outlook 2007–2019 / Classic Outlook** | Windows | Microsoft Word | MOST RESTRICTIVE, breaks modern CSS, needs conditional comment workarounds. Microsoft scheduled to end support for Word-based versions Oct 2026 |
| **New Outlook** | Windows (2023+) | Chromium | Essentially identical rendering to Outlook.com, far better CSS support, gradually replacing Classic Outlook |
| **Outlook** | macOS (2019+) | WebKit | Much better CSS support than Windows counterparts, treat as a different client from Windows versions |
| **Outlook** | iOS | WebKit | More modern than desktop Outlook, still idiosyncratic, full colour inversion in dark mode |
| **Outlook** | Android | Chromium (WebView) | More modern than desktop Outlook, still idiosyncratic, partial colour inversion in dark mode |
| **Outlook.com** | Web (all browsers, all OS) | Chromium | Better than Classic Outlook Windows but has irregular behaviours, also serves all Hotmail, Live, and MSN addresses |
| **Windows Mail** | Windows | WebKit | Built-in Windows client being phased into New Outlook, full colour inversion in dark mode, no CSS override support |
| **Gmail** | Web (all browsers, all OS) | WebKit | Strips `<style>` from `<head>`, requires inline CSS, no dark mode UI on web |
| **Gmail** | iOS | WebKit (via WKWebView) | Supports media queries, partial colour inversion in dark mode, no `prefers-color-scheme` support |
| **Gmail** | Android | Chromium (via WebView) | Supports media queries, aggressive full colour inversion in dark mode, no `prefers-color-scheme` support |
| **Gmail — GANGA** | iOS / Android | Reduced support | Gmail app used with a non-Gmail account (eg Outlook, Yahoo), strips `<style>` blocks entirely, treat like old Gmail web |
| **Apple Mail** | iOS | WebKit | Excellent CSS support, full `prefers-color-scheme` support, partial colour inversion in dark mode |
| **Apple Mail** | macOS | WebKit | Excellent CSS support, full `prefers-color-scheme` support, partial colour inversion in dark mode |
| **Yahoo Mail** | Web (all browsers, all OS) | WebKit | Moderate CSS support, strips some styles, no `prefers-color-scheme` support |
| **Yahoo Mail** | iOS | WebKit (via WKWebView) | Similar CSS filtering to web version, supports `<style>` in both `<head>` and `<body>` |
| **Yahoo Mail** | Android | Chromium (via WebView) | Does not support `<style>` in `<head>` — only in `<body>`, making it significantly more restrictive than the iOS and web versions: Use inline CSS only |
| **Samsung Mail** | Android | Chromium | Good modern CSS support, supports `prefers-color-scheme` |


> Use [Can I Email](https://www.caniemail.com) — the email equivalent of Can I Use — to check feature support per client.

### PS: Outlook on Windows: The Worst Offender
Outlook's Word-based renderer is the single biggest pain point in email development. It does not support:
- `max-width`
- `background-image` on `<div>` elements
- Flexbox, Grid, or any modern layout
- `border-radius` on certain elements
- Many padding/margin CSS properties

### PPS, Gmail's CSS Stripping
Gmail strips `<link>` tags and `<style>` blocks from the `<head>`. This historically forced 100% inline CSS. Gmail seems to now have limited internal CSS support, but inline styles remain essential for critical layout rules.


---

## 6. Responsiveness

There are three main approaches.

### A. Responsive (media queries)
Uses `@media` queries to apply styles at breakpoints. This is the standard web approach, but not supported in all clients (notably Gmail web).
> "Responsive email design uses CSS media queries to transform fixed elements on desktops into fluid ones for smaller screens."
> — [Litmus](https://www.litmus.com/blog/understanding-responsive-and-hybrid-email-design)

Typical breakpoint pattern:
```css
@media only screen and (max-width: 480px) {
  .column { display: block !important; width: 100% !important; }
}
```

### B. Fluid / hybrid design — RECOMMENDED
This seems to be the safest and most widely supported approach. It uses percentage-based widths and `max-width` constraints so layouts adapt without needing media queries. 
Outlook gets fixed-width fallbacks via conditional comments; all other clients get fluid layouts.
> [Tutorial from Email on Acid](https://www.emailonacid.com/blog/article/email-development/a-fluid-hybrid-design-primer/)

> "The fluid part refers to the fact that we use lots of percentages and elements that can move and expand to fit the space they are given. The hybrid part is because we also use max-width to constrain these free-flowing elements."
> — [Envato Tuts+](https://webdesign.tutsplus.com/creating-a-future-proof-responsive-email-without-media-queries--cms-23919t)

```html
<table width="100%" style="max-width: 600px;">
```
NB: Outlook ignores `max-width`, so wrap content in conditional comments with a fixed-width table for Outlook. See above for an example.

### C. Fluid images

```html
<img src="..." width="600" alt="..."
     style="display: block; width: 100%; max-width: 600px; height: auto;">
```

NOTES:
* The `width` HTML attribute is used by classic Outlook (which ignores `max-width`).
* The `width: 100%` inline style makes the image fill its container on all other clients.
* The `max-width: 600px` caps it at the intended size on large screens. 
* This scales the image responsively, but does not affect the layout around it — column stacking and layout responsiveness require the fluid/hybrid table approach covered above.

> — [HTML Email Builders](https://www.htmlemailbuilders.com/blog/responsive-email-design-guide)


### Column Stacking
The most common responsive pattern is two columns side-by-side on desktop, stacked on mobile.
> "Reading on small displays tends to work better if it's done in a linear fashion, from top-to-bottom. Using media queries we can switch a multi-column layout to a single-column one."
> — [Mailchimp Design Reference](https://templates.mailchimp.com/development/responsive-email/responsive-column-layouts/)

Touch targets on mobile should be at least **44×44px** (Apple's guideline).

This example is fluid hybrid stacking (see section 6.B above -- it uses no media queries — works everywhere including Gmail)

```html
<!--[if mso]>
<table role="presentation" cellpadding="0" cellspacing="0" border="0" width="600" align="center">
  <tr>
    <td width="300" valign="top">
<![endif]-->

<!-- Left column -->
<div style="display: inline-block; width: 100%; max-width: 300px; vertical-align: top;">
  <table role="presentation" cellpadding="0" cellspacing="0" border="0" width="100%">
    <tr>
      <td style="padding: 20px; font-family: Arial, sans-serif; font-size: 16px;">
        Left column content -- TEXT OR IMAGE
      </td>
    </tr>
  </table>
</div>

<!--[if mso]>
    </td>
    <td width="300" valign="top">
<![endif]-->

<!-- Right column -->
<div style="display: inline-block; width: 100%; max-width: 300px; vertical-align: top;">
  <table role="presentation" cellpadding="0" cellspacing="0" border="0" width="100%">
    <tr>
      <td style="padding: 20px; font-family: Arial, sans-serif; font-size: 16px;">
        Right column content
      </td>
    </tr>
  </table>
</div>

<!--[if mso]>
    </td>
  </tr>
</table>
<![endif]-->
```

The example above uses the fluid hybrid method (Approach B), which works  everywhere including Gmail. 
A media query version is simpler to write but won't stack in Gmail web or GANGA. TO crate this, use Approach A's breakpoint pattern, targeting `.column` `<td>` elements with `display: block !important; width: 100% !important;` at your chosen breakpoint. Both approaches are valid but the fluid hybrid method is more universally compatible.

---

## 7. Dark mode and light mode

### This is in users' control

**Dark mode in email is an unsolved problem. You cannot write CSS that controls dark mode universally.** 
The CSS-based techniques documented online — eg `@media (prefers-color-scheme: dark)`, image swapping, `[data-ogsc]` — do not work universally, only in a subset of clients.
Gmail supports none of them. 
Outlook mobile inverts aggressively and ignores overrides. 
Design emails so they degrade acceptably when clients do whatever they want to your colours.
Layer CSS techniques on top as progressive enhancement for clients that support them, but even this is hard to clarify.

> "While @media (prefers-color-scheme: dark) is widely supported, client-specific behaviors (like Outlook's [data-ogsc] or Gmail web's aggressive auto-inversion) mean you can't always achieve pixel-perfect control everywhere."
> — [Jeffrey Overmeer, "How to code dark mode email: A seamless CSS guide"](https://www.jeffreyovermeer.com/how-to-code-dark-mode-email-seamless-css-guide)

---

### How major clients behave

| Client | Dark mode UI? | What it does to your email | CSS override possible? |
|---|---|---|---|
| **Apple Mail (iOS)** | Yes | Partial inversion | ✅ `prefers-color-scheme` |
| **Apple Mail (macOS)** | Yes | Partial inversion | ✅ `prefers-color-scheme` |
| **Gmail Web** | No dark mode UI | Renders unchanged | ❌ None |
| **Gmail iOS app** | Yes | Full colour inversion | ❌ Not reliably |
| **Gmail Android app** | Yes | Partial inversion (bg only) | ❌ Not reliably |
| **Outlook Windows 2007-19** | Yes | Partial inversion | ⚠️ Limited, client-specific hacks only |
| **Outlook Mac 2019+** | Yes | Partial inversion | ✅ `prefers-color-scheme` |
| **Outlook iOS** | Yes | Full inversion | ❌ None |
| **Outlook Android** | Yes | Partial inversion | ⚠️ `[data-ogsc]` selector hack |
| **Outlook.com** | Yes | Partial inversion | ⚠️ `[data-ogsc]` selector hack |
| **Samsung Mail** | Yes | Partial inversion | ✅ `prefers-color-scheme` |
| **Windows Mail** | Yes | Full inversion | ❌ None |

> — [Enchant Agency](https://www.enchantagency.com/blog/dark-mode-email-design-best-practices-css-guide-2026), [Litmus](https://www.litmus.com/blog/the-ultimate-guide-to-dark-mode-for-email-marketers)

**Key takeaway:** CSS dark mode techniques work for roughly half of users. Use techniques specified below.

---

### Approach 1, bulletproof (works everywhere)

These require no CSS, no media query support, no `<head>` access. They work in every client.

**1. Use transparent PNGs for all logos and icons.**
A solid-white-background image on an inverted dark background shows as an ugly white box in dark mode.
A transparent PNG renders cleanly on any background the client chooses. This is the single most universally effective dark mode technique.

**2. Avoid pure white backgrounds on product images.**
White-background product photos can create a bright halo effect when a client inverts or darkens the surrounding email. Aim for a mid-grey (`#808080` range) or remove backgrounds entirely and export the image as a transparent PNG.

**3. Use mid-range brand colours where possible.**
Very light and very dark colours are the ones most aggressively targeted by auto-inversion algorithms. Mid-range colours (e.g. a medium brand blue or teal) are less likely to be touched, preserving more of your design across clients that auto-invert.

**4. Never put critical content inside images.**
If an image inverts or disappears, any text baked into it becomes unreadable or invisible. **Keep text as real HTML text, even if that constrains your design.** The reason for this is HTML text will be treated more predictably by auto-inversion algorithms, and will still be readable if the image is blocked or fails to load. 
This is a principle of accessible email design, but it also has the extra benefit of improving dark mode compatibility.

**5. Do not use `filter: invert(1)` on images.**
It produces unnatural results on photographs and is not reliably supported across clients.
> — [Jeffrey Overmeer](https://www.jeffreyovermeer.com/how-to-code-dark-mode-email-seamless-css-guide)

---

### Approach 2, CSS Techniques — progressive enhancements only (ie this gets progressively better as the email client's capabilities increase — it shouldn't break for anyone, just improves for some) but TEST THOROUGHLY

These only function when your `<style>` block is in `<head>` and the client honours it.

> "Internal CSS works in 84.85% of today's email clients, but there are a few rules that must be followed."
> — [Designmodo](https://designmodo.com/html-css-emails/)

#### Meta tags (add to `<head>` — prevents full inversion in some clients)
```html
<meta name="color-scheme" content="light dark" />
<meta name="supported-color-schemes" content="light dark" />
```
Also add inside your `<style>` block:
```css
:root { color-scheme: light dark; }
```
This signals to supporting clients (LIST TK) that you are aware of dark mode, which can stop them from applying their own full inversion. It does nothing in Gmail.

#### `@media (prefers-color-scheme: dark)` — Apple Mail, Outlook Mac, Samsung Mail
```css
@media (prefers-color-scheme: dark) {
  .email-body   { background-color: #1a1a1a !important; }
  .text-primary { color: #f0f0f0 !important; }
  .btn { background-color: #ffffff !important; color: #000000 !important; }
}
```
Use `!important` on every rule — you are competing against inline styles.

> ❌ **Not supported in Gmail (web or apps) or Yahoo Mail.**
> — [Can I Email](https://www.caniemail.com/features/css-at-media-prefers-color-scheme/)

#### `[data-ogsc]` — Outlook.com and Outlook Android only
Outlook.com injects a `data-ogsc` attribute into the DOM. You can target it as an attribute selector:
```css
[data-ogsc] .email-body   { background-color: #1a1a1a !important; }
[data-ogsc] .text-primary { color: #f0f0f0 !important; }
```
Mirror your `prefers-color-scheme` rules here. This seems to be a hack, not a standard — it may break without notice if Outlook changes their implementation.
> — [htmlemail.io](https://htmlemail.io/blog/dark-mode-email-styles)

#### DO NOT USE THIS EXCEPT IN VERY SPECIFIC CASES: CSS image swapping — Apple Mail, Outlook Mac only
**In Gmail and Outlook mobile, both images will be visible or both inverted — the swap will not fire.** Use transparent PNGs (Approach 1, above) instead, with image swapping as an enhancement for clients that support it (atm only Apple Mail and Outlook Mac).

Works only where `prefers-color-scheme` is supported. Ignored in Gmail and Outlook mobile.

```html
<!-- Default: visible in light mode, hidden in dark mode -->
<img class="img-light" src="logo-light.png" alt="Company logo"
     style="display:block;">

<!-- Hidden by default: visible only in dark mode -->
<img class="img-dark" src="logo-dark.png" alt="Company logo"
     style="display:none; max-height:0; overflow:hidden;">
```
```css
@media (prefers-color-scheme: dark) {
  .img-light { display: none   !important; max-height: 0  !important; overflow: hidden !important; }
  .img-dark  { display: block  !important; max-height: none !important; overflow: visible !important; }
}
```
Note the `max-height:0` / `overflow:hidden` on the hidden image — `display:none` alone is sometimes ignored in email clients; this belt-and-braces approach is more reliable.

---

### Colour design principles that work across both dark and light modes

These are design decisions, not CSS — they apply regardless of client support.

- **Avoid pure `#000000` backgrounds.** Use near-black (`#1a1a1a`, `#121212`). Pure black reads as harsh and is more aggressively targeted by inversion algorithms.
- **Avoid pure `#ffffff` text.** Use near-white (`#f0f0f0`, `#e8e8e8`). Easier on the eyes and less likely to be inverted to pure black.
- **Maintain WCAG 4.5:1 contrast ratio in both modes.** Verify with a contrast checker in both light and dark configurations.
- **Test your email on a black background before sending.** Place every image on a `#000000` background in a photo editor. Any background fringing, white boxes, or edge artefacts will show up immediately.
- **Use decorative images sparingly, and in ways that are not fundamental to the message or page structure.** 

---

### Dark Mode Decision Hierarchy

```
1. Transparent PNG? → Works everywhere. Do this.
2. Mid-range brand colours? → Reduces inversion damage everywhere. Do this.
3. Color-scheme meta tags? → Prevents worst-case full inversion in some clients. Low effort. Do this.
4. prefers-color-scheme CSS? → Custom dark theme for Apple Mail, Outlook Mac, Samsung. Worth doing.
5. [data-ogsc] CSS? → Extends #4 to Outlook.com and Outlook Android. Mirror your rules.
6. Image swap CSS? → Polish for Apple Mail / Outlook Mac only. Not a fix for Gmail. **NOT WORTH THE EFFORT UNLESS YOU KNOW A SIGNIFICANT PORTION OF YOUR AUDIENCE USES THESE CLIENTS.**
```


---

## 8. Testing

Be sure to test.
What looks perfect in your email client may be broken elsewhere.

> "What you see in your inbox may be totally different than what I see in my inbox. That's why rendering tools like Email on Acid or Litmus are a must."
> — [Martech Zone](https://martech.zone/understanding-the-challenges-and-frustrations-of-html-email-design/)

### Testing Tools

**[Email on Acid](https://www.emailonacid.com)** — More affordable alternative. Previews across 90+ clients. Includes Campaign Precheck (links, images, spam, accessibility). Unlimited tests on all plans.
> — [Email on Acid](https://www.emailonacid.com/email-on-acid-vs-litmus/)

**[Litmus](https://www.litmus.com)** — Industry standard. Previews across 70+ real clients (not emulators). Includes accessibility checks, spam testing, link validation, dark mode testing. 
> — [Litmus](https://www.litmus.com/litmus-vs-email-on-acid)

**[PutsMail](https://putsmail.com)** — Free. Send test emails to up to 10 inboxes. No screenshots.

**[Can I Email](https://www.caniemail.com)** — This isn't a testing platform but it offers a CSS/HTML feature support table by email client.


### What to Test
- Rendering across key clients (see table above, check your analytics for top clients in your audience)
- Rendering with images **off** (many corporate clients block images by default)
- Dark mode rendering on key platforms and clients (see table above)
- Mobile rendering and touch target sizes
- Plain text version
- All links (broken link check)
- Spam filter score
- Accessibility (alt text, colour contrast, screen reader behaviour)
- Internal seed list review before wide send
> — [Martech Zone](https://martech.zone/understanding-the-challenges-and-frustrations-of-html-email-design/), [Email Mavlers](https://www.emailmavlers.com/blog/email-template-testing-workflow/)

---

## 9. Accessibility

> "The truth is, there’s a disconnect between intention and action. Only 47% of companies use even basic accessibility considerations like alt text in their emails."
> — [Litmus](https://www.litmus.com/blog/accessible-email-made-easy-introducing-accessibility-checks-in-litmus)

Key practices:
- **Alt text** on all images (meaningful descriptions, not filenames)
- **Colour contrast**: minimum 4.5:1 ratio for body text (WCAG AA)
- **Logical heading hierarchy** (`<h1>`, `<h2>`, etc.)
- **`role="presentation"`** on layout tables so screen readers skip over them
- **Minimum touch target sizes** on mobile (44×44px)
- **Screen reader testing**: Tools such as NVDA (Windows) or VoiceOver (macOS/iOS) can simulate how your email will be read aloud.

> "2.2 billion people live with some form of visual impairment and many rely on screen readers."
> — [Litmus](https://www.litmus.com/blog/accessible-email-made-easy-introducing-accessibility-checks-in-litmus)

---


## 10. Upsides of Email Development

Despite the constraints, there are advantages to email.

**Direct channel with no algorithm.** Unlike social media, you own your list.

**High ROI.** Email consistently outperforms other marketing channels in return on investment. There are lots of stats in the links below, presumably for commercial u but it seems very promising for all users:
> "Email marketing returns [well]."
> — [Litmus](https://www.litmus.com/blog/why-email-deliverability-matters)

**Forced simplicity.** The constraints prevent scope creep. You can't build something complicated, which often produces cleaner, more focused experiences.

**Constraints sharpen fundamentals.** 

**Progressive enhancement still applies.** Some modern features may be availablefor clients that support them, while maintaining a safe fallback for the rest.

**Highly measurable.** Opens, clicks, conversions, A/B tests

**Templates are reusable.** Unlike web pages, a well-built email template can be reused for years with minimal change.

---

## 11. Other things

### Email size limits (Gmail clipping)
Gmail clips emails larger than 1002kb of HTML, showing a "View entire message" link. Keep HTML lean. Use a minifier and avoid inline base64 images. Source: [Litmus, 2024](https://www.litmus.com/blog/how-to-keep-gmail-from-clipping-your-emails)

### Use preheader text
EN provides this via the editor's UI. The preview text shown in the inbox after the subject line. It should be set explicitly in the UI, otherwise email clients grab content unpredictably from your email.
```html
<span style="display:none; max-height:0; overflow:hidden;">
  ADD PREHEADER TEXT
</span>
```

### Template Versioning
Marketing Tools allows for multiple folders and easy copying of blocks and templates, so make a test version of anything you're working on, as it's easy to accidentally overwrite the original/functional code.
> "Don't change your layouts or designs without working on a new version of your template that can be designed, properly tested, and deployed."
> — [Martech Zone](https://martech.zone/understanding-the-challenges-and-frustrations-of-html-email-design/)

### Email frameworks / tooling -- these need to be used outside of EN
They require a build step to compile down to the kind of HTML that is useful. They can speed up development and handle quirks, but add complexity and may not be worth it for simple updates.
A big benefit of these is that they allow you to write more semantic, modern HTML/CSS, and then compile it down to the table-based, inline style-heavy code that email clients require. ALSO they should handle Outlook quirks and responsive patterns automatically.
- **[MJML](https://mjml.io)** — Markup language that compiles to responsive email HTML. Works with EN: write in MJML, compile to HTML, paste the output into EN's HTML editor. MJML itself does not run inside EN.
- **[Foundation for Emails](https://get.foundation/emails.html)** (Zurb) — CSS framework for email. Same workflow as MJML: compile first, paste HTML into EN.
- **[Maizzle](https://maizzle.com)** — Tailwind CSS-based email framework. Same workflow as above.

### Over/highly formatted emails may reduce engagement
> (Research from 2025)[https://blog.hubspot.com/marketing/plain-text-vs-html-emails-data]
> "Research by leading marketing companies, like Hubspot, have also found that highly formatted emails with lots of HTML, color, and images reduce engagement by 25% on average and click through rates of 51%."
> — [Finalsite](https://schooladmin.zendesk.com/hc/en-us/articles/6219140568973)



