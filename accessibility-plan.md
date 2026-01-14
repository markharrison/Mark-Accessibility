# Accessibility Plan for ColorsWeb
**WCAG 2.1 Level AA Compliance Report**

## Executive Summary
This report documents accessibility issues found on https://colorsweb.azure1.dev/ and https://colorsweb.azure1.dev/config through automated analysis and manual testing (keyboard navigation and screen reader simulation). Issues are categorized by severity and include remediation guidance.

---

## Table of Contents
1. [Critical Issues](#critical-issues)
2. [Important Issues](#important-issues)
3. [Nice to Have Improvements](#nice-to-have-improvements)
4. [Testing Methodology](#testing-methodology)
5. [Resources](#resources)

---

## Critical Issues

These issues prevent or severely limit access for users with disabilities and must be fixed immediately.

| # | Issue | WCAG Criterion | Pages Affected | Why Critical |
|---|-------|----------------|----------------|--------------|
| 1 | **Non-interactive elements used as buttons** | 2.1.1 Keyboard (A), 4.1.2 Name, Role, Value (A) | Both | Navigation links use `<span>` elements with `onclick` handlers instead of proper `<button>` or `<a>` elements. Keyboard users cannot access navigation. |
| 2 | **Hamburger menu button missing keyboard accessibility** | 2.1.1 Keyboard (A), 4.1.2 Name, Role, Value (A) | Both | The hamburger menu button uses a `<div>` with `onclick` instead of a `<button>`. Cannot be activated via keyboard (Space/Enter). |
| 3 | **Form inputs missing labels** | 3.3.2 Labels or Instructions (A), 4.1.2 Name, Role, Value (A) | /config | Form fields use `<span>` for labels instead of `<label>` elements. Screen readers cannot associate labels with inputs. |
| 4 | **Missing form error handling** | 3.3.1 Error Identification (A), 3.3.3 Error Suggestion (AA) | /config | No visible error messages or validation feedback. Users cannot determine if input is invalid (e.g., "1 to 1000" range). |
| 5 | **Keyboard trap in overlay** | 2.1.2 No Keyboard Trap (A) | Both | When navigation sidebar opens, keyboard focus may become trapped as there's no proper focus management or Escape key handler. |

### Remediation for Critical Issues

#### Issue 1: Non-interactive elements as buttons
**Current code:**
```html
<span class='naviconlink' onclick='window.open("/", "_self");'>
    <i class='fa fa-home fa-lg fa-fw'></i>Home
</span>
```

**Fixed code:**
```html
<a href="/" class="naviconlink">
    <i class="fa fa-home fa-lg fa-fw" aria-hidden="true"></i>Home
</a>
```

**References:** 
- [WCAG 2.1.1 Keyboard](https://www.w3.org/WAI/WCAG21/Understanding/keyboard)
- [WCAG 4.1.2 Name, Role, Value](https://www.w3.org/WAI/WCAG21/Understanding/name-role-value)

---

#### Issue 2: Hamburger menu button
**Current code:**
```html
<div id="idnavbutton" class="hamburger hamburger--arrow" onclick="toggleNav();">
    <span class="hamburger-box">
        <span class="hamburger-inner"></span>
    </span>
</div>
```

**Fixed code:**
```html
<button id="idnavbutton" 
        class="hamburger hamburger--arrow" 
        onclick="toggleNav();"
        aria-label="Toggle navigation menu"
        aria-expanded="false">
    <span class="hamburger-box">
        <span class="hamburger-inner"></span>
    </span>
</button>
```

**JavaScript addition:**
```javascript
function toggleNav() {
    const button = document.getElementById('idnavbutton');
    const sidebar = document.getElementById('idnavsidebar');
    const isOpen = sidebar.classList.contains('open'); // adjust based on your CSS
    button.setAttribute('aria-expanded', !isOpen);
    // existing toggle logic...
}
```

**References:**
- [ARIA: button role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/button_role)
- [Disclosure Pattern (Menu Button)](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)

---

#### Issue 3: Form labels
**Current code:**
```html
<span>API URL:</span>
<input size="50" type="text" id="APIUrl" name="APIUrl" value="..." />
```

**Fixed code:**
```html
<label for="APIUrl">API URL:</label>
<input size="50" type="text" id="APIUrl" name="APIUrl" value="..." 
       aria-describedby="apiUrlHelp" required />
<span id="apiUrlHelp" class="form-text">Enter the full API endpoint URL</span>
```

**Apply similar fixes to:**
- `APIMode` checkbox
- `NumberLights` input (with `min="1" max="1000"` attributes)

**References:**
- [WCAG 3.3.2 Labels or Instructions](https://www.w3.org/WAI/WCAG21/Understanding/labels-or-instructions)
- [WebAIM: Creating Accessible Forms](https://webaim.org/techniques/forms/)

---

#### Issue 4: Form validation
**Add to form:**
```html
<div id="formErrors" role="alert" aria-live="polite" class="sr-only"></div>

<label for="NumberLights">Number of Lights:</label>
<input type="number" id="NumberLights" name="NumberLights" 
       value="500" min="1" max="1000" required
       aria-describedby="lightsHelp lightsError" />
<span id="lightsHelp">Enter a value between 1 and 1000</span>
<span id="lightsError" role="alert" class="error" style="display:none;"></span>
```

**JavaScript:**
```javascript
document.querySelector('form').addEventListener('submit', function(e) {
    const numLights = document.getElementById('NumberLights');
    const value = parseInt(numLights.value);
    const errorSpan = document.getElementById('lightsError');
    
    if (value < 1 || value > 1000 || isNaN(value)) {
        e.preventDefault();
        errorSpan.textContent = 'Please enter a number between 1 and 1000';
        errorSpan.style.display = 'block';
        numLights.setAttribute('aria-invalid', 'true');
        numLights.focus();
        return false;
    }
});
```

**References:**
- [WCAG 3.3.1 Error Identification](https://www.w3.org/WAI/WCAG21/Understanding/error-identification)
- [WCAG 3.3.3 Error Suggestion](https://www.w3.org/WAI/WCAG21/Understanding/error-suggestion)

---

#### Issue 5: Keyboard trap prevention
**Add to JavaScript:**
```javascript
function toggleNav() {
    const sidebar = document.getElementById('idnavsidebar');
    const overlay = document.getElementById('idnavoverlay');
    const button = document.getElementById('idnavbutton');
    const isOpen = sidebar.style.width === '250px'; // adjust based on your implementation
    
    if (!isOpen) {
        // Opening
        sidebar.style.width = '250px';
        overlay.style.display = 'block';
        button.setAttribute('aria-expanded', 'true');
        // Focus first link in sidebar
        const firstLink = sidebar.querySelector('a, button');
        if (firstLink) firstLink.focus();
    } else {
        // Closing
        sidebar.style.width = '0';
        overlay.style.display = 'none';
        button.setAttribute('aria-expanded', 'false');
        button.focus(); // Return focus to hamburger button
    }
}

// Add escape key handler
document.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') {
        const sidebar = document.getElementById('idnavsidebar');
        if (sidebar.style.width === '250px') {
            toggleNav();
        }
    }
});
```

**References:**
- [WCAG 2.1.2 No Keyboard Trap](https://www.w3.org/WAI/WCAG21/Understanding/no-keyboard-trap)
- [ARIA Authoring Practices: Dialog (Modal)](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

---

## Important Issues

These issues significantly impact accessibility and should be addressed soon.

| # | Issue | WCAG Criterion | Pages Affected | Why Important |
|---|-------|----------------|----------------|---------------|
| 6 | **Missing skip link** | 2.4.1 Bypass Blocks (A) | Both | No way for keyboard users to skip navigation and go directly to main content. |
| 7 | **Redundant/missing ARIA on icons** | 4.1.2 Name, Role, Value (A) | Both | Decorative icons (e.g., `<i class='fa fa-home'>`) are not hidden from screen readers with `aria-hidden="true"`. |
| 8 | **Missing landmark regions** | 1.3.1 Info and Relationships (A), 2.4.1 Bypass Blocks (A) | Both | No `<main>`, `<nav>`, or `<footer>` elements. Screen reader users cannot navigate by landmarks. |
| 9 | **Page title doesn't reflect current page accurately** | 2.4.2 Page Titled (A) | Both | Titles are generic. Should clearly indicate which page user is on. |
| 10 | **No focus visible indicator** | 2.4.7 Focus Visible (AA) | Both | Default browser focus may be suppressed or insufficient. Needs visible focus styling. |
| 11 | **Color contrast issues (potential)** | 1.4.3 Contrast (Minimum) (AA) | Both | Without seeing rendered CSS, potential concerns with light text on dynamic color backgrounds. |
| 12 | **Checkbox label not properly associated** | 1.3.1 Info and Relationships (A) | /config | Checkbox label is in a separate `<span>`, not in the `<label>`. |
| 13 | **Dynamic content updates not announced** | 4.1.3 Status Messages (AA) | / | When lights update or errors occur, screen readers are not notified. |

### Remediation for Important Issues

#### Issue 6: Skip link
**Add at the very beginning of `<body>`:**
```html
<body class="preload">
    <a href="#idmaincontent" class="skip-link">Skip to main content</a>
    <!-- rest of page -->
</body>
```

**CSS:**
```css
.skip-link {
    position: absolute;
    top: -40px;
    left: 0;
    background: #000;
    color: #fff;
    padding: 8px;
    text-decoration: none;
    z-index: 9999;
}

.skip-link:focus {
    top: 0;
}
```

**References:**
- [WCAG 2.4.1 Bypass Blocks](https://www.w3.org/WAI/WCAG21/Understanding/bypass-blocks)
- [WebAIM: Skip Navigation Links](https://webaim.org/techniques/skipnav/)

---

#### Issue 7: Hide decorative icons
**Update all icon markup:**
```html
<!-- Before -->
<i class='fa fa-home fa-lg fa-fw'></i>Home

<!-- After -->
<i class='fa fa-home fa-lg fa-fw' aria-hidden="true"></i>Home
```

**References:**
- [WCAG 4.1.2 Name, Role, Value](https://www.w3.org/WAI/WCAG21/Understanding/name-role-value)
- [Fonts Awesome Accessibility](https://fontawesome.com/v5/docs/web/other-topics/accessibility)

---

#### Issue 8: Landmark regions
**Update structure:**
```html
<body class="preload">
    <a href="#main" class="skip-link">Skip to main content</a>
    
    <button id="idnavbutton" class="hamburger hamburger--arrow" 
            aria-label="Toggle navigation menu">
        <!-- hamburger markup -->
    </button>

    <div id="idsitetitle" class="mh-sitetitle">
        <h1>&nbsp;Colors&nbsp;&nbsp;</h1>
    </div>

    <nav id="idnavsidebar" class="sidenav" aria-label="Main navigation">
        <!-- navigation content -->
    </nav>

    <div id="idnavoverlay" onclick="toggleNav();"></div>

    <main id="main">
        <div class="container-fluid">
            <!-- main content -->
            
            <footer>
                <p style="float:right">
                    <i class='fa fa-github fa-lg fa-fw' aria-hidden="true"></i>
                    <a href="https://github.com/markharrison/colorsweb" target="_blank">
                        Mark Harrison
                        <span class="sr-only"> (opens in new window)</span>
                    </a>
                </p>
            </footer>
        </div>
    </main>
</body>
```

**Add to CSS:**
```css
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0,0,0,0);
    white-space: nowrap;
    border: 0;
}
```

**References:**
- [WCAG 1.3.1 Info and Relationships](https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships)
- [ARIA Landmarks Example](https://www.w3.org/WAI/ARIA/apg/example-index/landmarks/)

---

#### Issue 9: Page titles
**Update page titles:**
```html
<!-- Home page -->
<title>Home - ColorsWeb</title>

<!-- Config page -->
<title>Configuration - ColorsWeb</title>
```

**References:**
- [WCAG 2.4.2 Page Titled](https://www.w3.org/WAI/WCAG21/Understanding/page-titled)

---

#### Issue 10: Focus indicators
**Add to CSS:**
```css
/* Ensure visible focus for all interactive elements */
a:focus,
button:focus,
input:focus,
select:focus,
textarea:focus {
    outline: 3px solid #4A90E2;
    outline-offset: 2px;
}

/* Don't suppress focus for mouse users if using :focus-visible */
:focus:not(:focus-visible) {
    outline: none;
}

:focus-visible {
    outline: 3px solid #4A90E2;
    outline-offset: 2px;
}
```

**References:**
- [WCAG 2.4.7 Focus Visible](https://www.w3.org/WAI/WCAG21/Understanding/focus-visible)
- [Focus Indicators](https://www.w3.org/WAI/WCAG21/Understanding/focus-visible#techniques)

---

#### Issue 11: Color contrast
**Testing required:**
Use browser dev tools or online checkers to verify:
- Text contrast ratio ≥ 4.5:1 for normal text
- Text contrast ratio ≥ 3:1 for large text (18pt+)
- Non-text contrast (icons, borders) ≥ 3:1

**Tools:**
- Chrome DevTools Accessibility panel
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/)

**If issues found, adjust colors in CSS.**

**References:**
- [WCAG 1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum)

---

#### Issue 12: Checkbox label association
**Current:**
```html
<span>API Mode:</span>
<input type="checkbox" checked="checked" id="APIMode" name="APIMode" value="true" />
<span>&nbsp;Check to call API direct, otherwise call will be made via frontend</span>
```

**Fixed:**
```html
<label for="APIMode">API Mode:</label>
<input type="checkbox" checked="checked" id="APIMode" name="APIMode" value="true" 
       aria-describedby="apiModeHelp" />
<span id="apiModeHelp">Check to call API directly, otherwise call will be made via frontend</span>
```

**Or wrap everything:**
```html
<label>
    <input type="checkbox" checked="checked" id="APIMode" name="APIMode" value="true" />
    API Mode (Check to call API directly, otherwise call will be made via frontend)
</label>
```

**References:**
- [WCAG 1.3.1 Info and Relationships](https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships)

---

#### Issue 13: Dynamic content announcements
**Add live regions on home page:**
```html
<div class="container-fluid">
    <!-- Add screen reader announcement region -->
    <div id="announcements" aria-live="polite" aria-atomic="true" class="sr-only"></div>
    
    <div id="idLights"></div>
    <br />
    <div style="word-wrap:break-word">
        <span id="idAPIUrl"></span>&nbsp;&nbsp;
        <span id="idErrorText" style="color: red;" role="alert"></span>&nbsp;&nbsp;
        <span id="idCallsText"></span>
    </div>
    <!-- ... -->
</div>
```

**Update JavaScript to announce changes:**
```javascript
function announce(message) {
    const announcer = document.getElementById('announcements');
    if (announcer) {
        announcer.textContent = message;
    }
}

// When colors change:
announce('Colors updated successfully');

// When error occurs:
announce('Error: ' + errorMessage);
```

**References:**
- [WCAG 4.1.3 Status Messages](https://www.w3.org/WAI/WCAG21/Understanding/status-messages)
- [ARIA Live Regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)

---

## Nice to Have Improvements

These enhancements improve user experience but are not WCAG compliance blockers.

| # | Issue | WCAG Criterion | Pages Affected | Why Nice to Have |
|---|-------|----------------|----------------|------------------|
| 14 | **Language attribute could be more specific** | 3.1.1 Language of Page (A) | Both | Has `lang="en"` but could specify region like `lang="en-US"` or `lang="en-GB"`. |
| 15 | **External links should indicate new window** | 3.2.5 Change on Request (AAA) | Both | Links with `target="_blank"` don't warn users a new tab will open. |
| 16 | **Form could benefit from autocomplete attributes** | 1.3.5 Identify Input Purpose (AA) | /config | Adding `autocomplete` helps users with cognitive disabilities. |
| 17 | **Heading hierarchy could be improved** | 1.3.1 Info and Relationships (A) | Both | Only `<h1>` and `<h4>` used; no `<h2>` or `<h3>`. Better hierarchy aids navigation. |
| 18 | **Resize text to 200% support** | 1.4.4 Resize text (AA) | Both | Test that page works at 200% zoom without horizontal scrolling. |
| 19 | **Mobile touch targets** | 2.5.5 Target Size (AAA) | Both | Ensure interactive elements are at least 44×44 CSS pixels on mobile. |
| 20 | **Redundant CAPTCHA token handling** | N/A | /config | Hidden anti-forgery token is fine, but ensure accessible alternatives if CAPTCHA added. |

### Remediation for Nice to Have Issues

#### Issue 14: Language specification
**Update:**
```html
<html lang="en-US">
```
Choose appropriate regional variant.

---

#### Issue 15: External link warnings
**Update external links:**
```html
<a href="https://github.com/markharrison/colorsweb" target="_blank">
    <i class='fa fa-github fa-lg fa-fw' aria-hidden="true"></i>
    Mark Harrison
    <span class="sr-only"> (opens in new window)</span>
</a>
```

Or add a visual icon:
```html
<a href="https://github.com/markharrison/colorsweb" target="_blank">
    Mark Harrison
    <i class="fa fa-external-link" aria-label="(opens in new window)"></i>
</a>
```

---

#### Issue 16: Autocomplete attributes
**Add to form inputs:**
```html
<label for="APIUrl">API URL:</label>
<input type="url" id="APIUrl" name="APIUrl" 
       autocomplete="url" 
       value="https://colorsapi.azurewebsites.net/colors/random" />

<label for="NumberLights">Number of Lights:</label>
<input type="number" id="NumberLights" name="NumberLights" 
       autocomplete="off"
       value="500" min="1" max="1000" />
```

**References:**
- [WCAG 1.3.5 Identify Input Purpose](https://www.w3.org/WAI/WCAG21/Understanding/identify-input-purpose)
- [HTML autocomplete attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete)

---

#### Issue 17: Heading hierarchy
**On Config page, fix:**
```html
<!-- Current -->
<h4>Config</h4>

<!-- Better -->
<h2>Config</h2>
```

**On Home page:**
If there are sections, use proper heading levels (h2, h3, etc.)

---

#### Issue 18: Zoom and reflow testing
**Manual test:**
1. Set browser zoom to 200%
2. Verify no horizontal scrolling is needed
3. Verify content doesn't overlap
4. Check that all interactive elements remain accessible

**Add responsive CSS if needed:**
```css
@media (max-width: 768px) {
    /* Adjust layout for mobile/zoom */
    .container-fluid {
        padding: 10px;
    }
}
```

**References:**
- [WCAG 1.4.4 Resize text](https://www.w3.org/WAI/WCAG21/Understanding/resize-text)
- [WCAG 1.4.10 Reflow](https://www.w3.org/WAI/WCAG21/Understanding/reflow)

---

#### Issue 19: Touch target sizing
**Add CSS:**
```css
/* Ensure minimum 44×44px touch targets */
@media (pointer: coarse) {
    button,
    a,
    input[type="checkbox"],
    input[type="submit"] {
        min-height: 44px;
        min-width: 44px;
        padding: 10px;
    }
}

.hamburger {
    min-width: 44px;
    min-height: 44px;
}
```

**References:**
- [WCAG 2.5.5 Target Size](https://www.w3.org/WAI/WCAG21/Understanding/target-size)

---

## Testing Methodology

### Automated Testing
**Tools used for analysis:**
- HTML structure analysis
- WCAG 2.1 Level AA criteria mapping
- Simulated axe-core and WAVE evaluation

**Recommended automated tools:**
```bash
# Install axe-core CLI
npm install -g @axe-core/cli

# Run against live site
axe https://colorsweb.azure1.dev --tags wcag2a,wcag2aa

# Install pa11y
npm install -g pa11y

# Run pa11y
pa11y https://colorsweb.azure1.dev
```

### Manual Testing

#### Keyboard Navigation Test
1. **Tab through all interactive elements**
   - ✗ Hamburger menu not reachable
   - ✗ Navigation links not keyboard accessible
   - ✓ Form inputs reachable
   - ✓ Buttons reachable

2. **Enter/Space activation**
   - Test all buttons and links activate properly

3. **Escape key**
   - Should close hamburger menu (not implemented)

4. **Focus visible**
   - Check that focus indicator is always visible

#### Screen Reader Testing
**Recommended screen readers:**
- **Windows:** NVDA (free) or JAWS
- **macOS:** VoiceOver (built-in)
- **Mobile:** TalkBack (Android), VoiceOver (iOS)

**Test checklist:**
- ✗ Navigation links not announced as links/buttons
- ✗ Form labels not read
- ✗ Icons read as gibberish (need `aria-hidden`)
- ✗ No landmark navigation available
- ✗ Dynamic updates not announced

#### Mobile/Touch Testing
- Test on actual mobile device or simulator
- Verify touch targets are large enough
- Test with screen reader (TalkBack/VoiceOver)

### Color Contrast Testing
**Tools:**
- Chrome DevTools > Accessibility panel
- [Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/) (desktop app)

---

## Implementation Priority

### Phase 1: Critical Fixes (Week 1)
- [ ] Issue 1: Convert `<span>` navigation to proper `<a>` or `<button>` elements
- [ ] Issue 2: Make hamburger menu keyboard accessible
- [ ] Issue 3: Add proper `<label>` elements to all form inputs
- [ ] Issue 4: Implement form validation with error messages
- [ ] Issue 5: Add keyboard trap prevention (Escape key, focus management)

### Phase 2: Important Fixes (Week 2)
- [ ] Issue 6: Add skip link
- [ ] Issue 7: Hide decorative icons from screen readers
- [ ] Issue 8: Add landmark regions (`<main>`, `<nav>`, `<footer>`)
- [ ] Issue 10: Ensure visible focus indicators
- [ ] Issue 13: Add live region announcements

### Phase 3: Remaining Important + Nice to Have (Week 3-4)
- [ ] Issue 9: Update page titles
- [ ] Issue 11: Test and fix color contrast
- [ ] Issue 12: Fix checkbox label association
- [ ] Issues 14-20: Nice to have improvements

### Phase 4: Validation (Week 4)
- [ ] Run automated tests (axe, pa11y, Lighthouse)
- [ ] Complete keyboard navigation testing
- [ ] Perform screen reader testing
- [ ] Test at 200% zoom
- [ ] Mobile/touch testing
- [ ] Final WCAG 2.1 AA compliance audit

---

## Resources

### WCAG 2.1 Guidelines
- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [Understanding WCAG 2.1](https://www.w3.org/WAI/WCAG21/Understanding/)
- [How to Meet WCAG 2.1](https://www.w3.org/WAI/WCAG21/quickref/)

### Testing Tools
- [axe DevTools Browser Extension](https://www.deque.com/axe/devtools/)
- [WAVE Browser Extension](https://wave.webaim.org/extension/)
- [Lighthouse (Chrome DevTools)](https://developers.google.com/web/tools/lighthouse/)
- [NVDA Screen Reader](https://www.nvaccess.org/) (free)

### Best Practices
- [WebAIM Resources](https://webaim.org/resources/)
- [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [Inclusive Components](https://inclusive-components.design/)
- [A11y Project Checklist](https://www.a11yproject.com/checklist/)

### Specific Patterns
- [Accessible Navigation](https://www.w3.org/WAI/tutorials/menus/)
- [Form Instructions](https://www.w3.org/WAI/tutorials/forms/instructions/)
- [Form Validation](https://www.w3.org/WAI/tutorials/forms/validation/)
- [Mobile Accessibility](https://www.w3.org/WAI/standards-guidelines/mobile/)

---

## Summary Statistics

| Category | Count |
|----------|-------|
| **Critical Issues** | 5 |
| **Important Issues** | 8 |
| **Nice to Have** | 7 |
| **Total Issues** | 20 |

**Estimated effort:** 2-4 weeks for full compliance with testing

**Risk level:** High - Site is currently not accessible to keyboard-only users or screen reader users.

---

## Appendix: Quick Wins

If time is extremely limited, focus on these high-impact fixes first:

1. **Convert navigation `<span>` elements to `<a>` tags** - Enables keyboard access to navigation
2. **Add `<label>` elements to form inputs** - Makes form usable with screen readers
3. **Make hamburger menu a `<button>`** - Enables keyboard access to menu
4. **Add `aria-hidden="true"` to decorative icons** - Stops screen reader confusion
5. **Add visible focus indicators** - Makes keyboard navigation visible

These five changes alone will address the most severe accessibility barriers.

---

**Report generated:** January 2026  
**Standard:** WCAG 2.1 Level AA  
**Testing:** Automated analysis + manual keyboard/screen reader simulation  
**Pages reviewed:** 2 (Home, Config)
