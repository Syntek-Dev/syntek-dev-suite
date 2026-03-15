# Accessibility

**Last Updated:** 15/03/2026
**Version:** 1.8.0
**Maintained By:** Development Team
**Language:** British English (en_GB)
**Timezone:** Europe/London
**Plugin Scope:** syntek-dev-suite (Python/Django, PHP/Laravel, TypeScript/React, React Native)

---

## Table of Contents

- [Overview](#overview)
- [Standards and Compliance](#standards-and-compliance)
- [Semantic HTML](#semantic-html)
- [ARIA — When and How](#aria--when-and-how)
- [Keyboard Navigation](#keyboard-navigation)
- [Focus Management](#focus-management)
- [Forms and Inputs](#forms-and-inputs)
- [Colour and Contrast](#colour-and-contrast)
- [Images, Icons, and Media](#images-icons-and-media)
- [Typography and Readability](#typography-and-readability)
- [Motion and Animation](#motion-and-animation)
- [React Component Patterns](#react-component-patterns)
- [React Native / Mobile Accessibility](#react-native--mobile-accessibility)
- [Server-Rendered Accessibility (Django / Laravel)](#server-rendered-accessibility-django--laravel)
- [Testing Accessibility](#testing-accessibility)
- [Accessibility Checklist](#accessibility-checklist)

---

## Overview

Accessibility is not optional and not a feature — it is a baseline requirement. Every user-facing component, page, and interaction must be usable by people with disabilities, including those using screen readers, keyboard navigation, switch devices, and magnification.

This guide covers practical rules for building accessible interfaces across the syntek-dev-suite stack. It applies to server-rendered pages (Django templates, Laravel Blade), single-page applications (React), and mobile applications (React Native).

---

## Standards and Compliance

All projects target **WCAG 2.2 Level AA** as the minimum standard. This covers the vast majority of accessibility requirements and is the standard referenced by UK public sector regulations (The Public Sector Bodies (Websites and Mobile Applications) Accessibility Regulations 2018) and the Equality Act 2010.

**Level A** requirements are non-negotiable baseline (e.g., text alternatives for images, keyboard operability).

**Level AA** requirements are the target for all projects (e.g., colour contrast ratios, resize support, focus visibility).

**Level AAA** requirements are applied where practical but not mandated across the board (e.g., sign language for video, enhanced contrast).

Reference: [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)

---

## Semantic HTML

Use the correct HTML element for the job. Semantic elements communicate meaning to assistive technology without any additional attributes.

### Rules

- Use `<button>` for actions, `<a>` for navigation. Never use `<div onClick>` or `<span onClick>` as a substitute — they have no keyboard support, no role, and no focus by default.
- Use heading levels (`<h1>` through `<h6>`) in logical order. Do not skip levels. Every page has exactly one `<h1>`.
- Use `<nav>` for navigation regions, `<main>` for the primary content, `<aside>` for supplementary content, `<header>` and `<footer>` for their respective regions.
- Use `<ul>` or `<ol>` for lists. Screen readers announce "list, 5 items" which helps users understand the structure.
- Use `<table>` for tabular data with `<thead>`, `<tbody>`, `<th scope="col">` and `<th scope="row">`. Never use tables for layout.
- Use `<fieldset>` and `<legend>` for related groups of form controls (e.g., a set of radio buttons).
- Use `<dialog>` for modal dialogs where supported. It provides built-in focus trapping and escape key handling.

### Common mistakes

```html
<!-- BAD: div as a button -->
<div class="btn" onclick="handleClick()">Submit</div>

<!-- GOOD: actual button -->
<button type="button" onclick="handleClick()">Submit</button>

<!-- BAD: link used as a button (no navigation) -->
<a href="#" onclick="doSomething()">Delete</a>

<!-- GOOD: button for actions -->
<button type="button" onclick="doSomething()">Delete</button>

<!-- BAD: skipped heading levels -->
<h1>Dashboard</h1>
<h3>Recent Orders</h3>  <!-- skipped h2 -->

<!-- GOOD: logical heading order -->
<h1>Dashboard</h1>
<h2>Recent Orders</h2>
```

---

## ARIA — When and How

ARIA (Accessible Rich Internet Applications) attributes supplement semantic HTML when native elements cannot express the required semantics. **The first rule of ARIA is: don't use ARIA if a native HTML element can do the job.**

### When to use ARIA

- Custom components that have no native HTML equivalent (tabs, tree views, comboboxes, carousels).
- Dynamic content that changes without a page reload (live regions, status messages).
- Relationships between elements that are not expressed by the DOM structure (e.g., `aria-describedby`, `aria-controls`).

### When NOT to use ARIA

- To replicate what a native element already provides. `<button>` already has `role="button"` — adding `role="button"` to a `<button>` is redundant.
- To fix a broken component. If a `<div>` needs ARIA to work like a button, use a `<button>` instead.

### Essential patterns

**Live regions** — for dynamic content updates (notifications, form validation messages, chat messages):

```html
<!-- Polite: announced when the user is idle -->
<div aria-live="polite" aria-atomic="true">
  3 items in your basket
</div>

<!-- Assertive: announced immediately (use sparingly) -->
<div aria-live="assertive" role="alert">
  Payment failed. Please check your card details.
</div>
```

**Described by** — for supplementary descriptions:

```html
<input type="password" id="password" aria-describedby="password-hint" />
<p id="password-hint">Must be at least 12 characters.</p>
```

**Expanded/collapsed** — for disclosure widgets:

```html
<button aria-expanded="false" aria-controls="menu-content">Menu</button>
<div id="menu-content" hidden>
  <!-- menu items -->
</div>
```

**Current page** — for navigation:

```html
<nav aria-label="Main navigation">
  <a href="/" aria-current="page">Home</a>
  <a href="/about">About</a>
</nav>
```

---

## Keyboard Navigation

Every interactive element must be operable with a keyboard alone. Many users with motor impairments, power users, and screen reader users rely entirely on keyboard navigation.

### Rules

- All interactive elements must be reachable with `Tab` and `Shift+Tab`.
- All interactive elements must be activatable with `Enter` or `Space`.
- Custom components must implement the expected keyboard patterns from the [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/patterns/).
- Tab order must follow a logical reading order. Do not use `tabindex` values greater than 0 — they create unpredictable tab order. Use `tabindex="0"` to make a non-interactive element focusable (rare), and `tabindex="-1"` to make an element programmatically focusable but not in the tab order.
- `Escape` must close modal dialogs, popovers, and dropdown menus, returning focus to the triggering element.
- Keyboard shortcuts must not conflict with browser or screen reader shortcuts. Custom shortcuts should use modifier keys (Ctrl, Alt) and be documented.

### Common keyboard patterns

| Component | Keys |
|-----------|------|
| Button | `Enter`, `Space` to activate |
| Link | `Enter` to follow |
| Checkbox | `Space` to toggle |
| Radio group | `Arrow keys` to move between options |
| Tab panel | `Arrow keys` to switch tabs, `Tab` to enter panel content |
| Menu | `Arrow keys` to navigate, `Enter` to select, `Escape` to close |
| Dialog | `Tab` cycles within dialog, `Escape` closes |
| Combobox | `Arrow keys` to navigate options, `Enter` to select, `Escape` to close |

### Skip link

Every page must include a skip link as the first focusable element, allowing keyboard users to bypass repetitive navigation:

```html
<body>
  <a href="#main-content" class="sr-only focus:not-sr-only">
    Skip to main content
  </a>
  <nav><!-- navigation --></nav>
  <main id="main-content">
    <!-- page content -->
  </main>
</body>
```

---

## Focus Management

When content changes dynamically (modals opening, page navigation in an SPA, inline editing), focus must be managed deliberately.

### Rules

- When a modal dialog opens, move focus to the first focusable element inside the dialog (or the dialog itself).
- When a modal closes, return focus to the element that triggered it.
- When content is removed from the page (e.g., deleting an item from a list), move focus to a logical place — the previous item, the next item, or a heading above the list. Do not let focus fall to `<body>`.
- After client-side navigation (React Router, Inertia.js), move focus to the main content area or the page heading. Announce the new page title to screen readers.
- Focus indicators must be visible. Never use `outline: none` without providing a custom focus style. The default browser outline is acceptable; a custom focus ring that meets the 3:1 contrast ratio requirement (WCAG 2.4.11) is better.

### Focus trapping in dialogs

```typescript
// React — focus trap for modal dialogs
function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isOpen || !modalRef.current) return;

    const focusableElements = modalRef.current.querySelectorAll<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const first = focusableElements[0];
    const last = focusableElements[focusableElements.length - 1];

    first?.focus();

    function handleKeyDown(e: KeyboardEvent) {
      if (e.key === "Escape") { onClose(); return; }
      if (e.key !== "Tab") return;

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last?.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first?.focus();
      }
    }

    document.addEventListener("keydown", handleKeyDown);
    return () => document.removeEventListener("keydown", handleKeyDown);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div role="dialog" aria-modal="true" ref={modalRef}>
      {children}
    </div>
  );
}
```

---

## Forms and Inputs

Forms are one of the most common accessibility failure points. Every form control must have a programmatically associated label.

### Rules

- Every `<input>`, `<select>`, and `<textarea>` must have a `<label>` with a matching `for`/`id` attribute, or be wrapped in a `<label>` element.
- Placeholder text is not a label. Placeholders disappear when the user starts typing, making the field's purpose unclear. Use a visible label.
- Error messages must be programmatically associated with the field they describe, using `aria-describedby` or `aria-errormessage`.
- Error messages must not rely solely on colour. Include an icon or text prefix ("Error:") to communicate the error state.
- Required fields must be indicated with `aria-required="true"` or the HTML `required` attribute, plus a visible indicator.
- Group related fields with `<fieldset>` and `<legend>` — especially radio buttons and checkboxes.
- Autocomplete attributes (`autocomplete="email"`, `autocomplete="given-name"`) must be used on fields that collect personal information, per WCAG 1.3.5.

### Accessible form example

```html
<form novalidate>
  <div>
    <label for="email">Email address <span aria-hidden="true">*</span></label>
    <input
      type="email"
      id="email"
      name="email"
      autocomplete="email"
      required
      aria-required="true"
      aria-describedby="email-error"
      aria-invalid="true"
    />
    <p id="email-error" role="alert">
      Error: Please enter a valid email address.
    </p>
  </div>

  <fieldset>
    <legend>Preferred contact method</legend>
    <label><input type="radio" name="contact" value="email" /> Email</label>
    <label><input type="radio" name="contact" value="phone" /> Phone</label>
  </fieldset>

  <button type="submit">Submit</button>
</form>
```

### Inline validation

- Validate on blur (when the user leaves a field), not on every keystroke. Constant error announcements are disruptive to screen reader users.
- When validation errors are corrected, update or remove the error message. Screen readers will announce the change if the error container uses `aria-live="polite"`.

---

## Colour and Contrast

### Rules

- **Normal text** (under 18pt / 24px, or under 14pt / 18.66px bold): minimum contrast ratio of **4.5:1** against the background.
- **Large text** (18pt+ / 24px+, or 14pt+ / 18.66px+ bold): minimum contrast ratio of **3:1**.
- **UI components and graphical objects** (icons, form field borders, focus indicators): minimum contrast ratio of **3:1**.
- **Never use colour alone** to convey information. A red error message must also include an icon or text prefix. A green "success" badge must also include a label.
- Ensure sufficient contrast in both light and dark modes.
- Test with colour blindness simulation tools (e.g., the "Emulate vision deficiencies" feature in Chrome DevTools).

### Checking contrast

Use the [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) or browser DevTools to verify ratios. In Tailwind CSS, check that your colour pairings meet the ratios — `text-gray-500` on a white background often fails at 4.5:1.

---

## Images, Icons, and Media

### Images

- **Informative images** must have descriptive `alt` text that conveys the same information as the image. Describe what the image communicates, not what it looks like.
- **Decorative images** must have `alt=""` (empty alt attribute) so screen readers skip them. Do not omit the `alt` attribute entirely — that causes screen readers to announce the filename.
- **Complex images** (charts, diagrams, infographics) must have a short `alt` text plus a longer description, either adjacent or via `aria-describedby`.

```html
<!-- Informative -->
<img src="chart.png" alt="Revenue increased 23% from Q3 to Q4 2025" />

<!-- Decorative -->
<img src="divider.svg" alt="" />

<!-- Complex with extended description -->
<figure>
  <img src="architecture.png" alt="System architecture diagram" aria-describedby="arch-desc" />
  <figcaption id="arch-desc">
    The system consists of a Next.js frontend, a Django API, a PostgreSQL database,
    and a Redis cache, all connected via a private Docker network.
  </figcaption>
</figure>
```

### Icons

- **Meaningful icons** (a trash icon for delete) must have an accessible label: `aria-label` on the button, or visually hidden text inside it.
- **Decorative icons** alongside visible text should be hidden from screen readers: `aria-hidden="true"`.

```tsx
// Good — icon with accessible label
<button aria-label="Delete item">
  <TrashIcon aria-hidden="true" />
</button>

// Good — icon alongside text (icon is decorative)
<button>
  <TrashIcon aria-hidden="true" />
  <span>Delete</span>
</button>
```

### Video and audio

- All video content must have captions (synchronised text alternatives).
- All audio-only content (podcasts) must have a transcript.
- Auto-playing media must be avoidable — provide a pause/stop control.

---

## Typography and Readability

- Text must be resizable up to 200% without loss of content or functionality (WCAG 1.4.4). Use relative units (`rem`, `em`, `%`) for font sizes, not fixed `px`.
- Line height should be at least 1.5 times the font size for body text.
- Paragraph spacing should be at least 2 times the font size.
- Text blocks should not exceed approximately 80 characters per line for readability.
- Do not justify text (`text-align: justify`) — the uneven word spacing makes text harder to read for users with dyslexia.
- Ensure the page is usable at 400% zoom in a 1280px viewport (WCAG 1.4.10 — Reflow). Content must reflow to a single column without horizontal scrolling.

---

## Motion and Animation

- Provide a way to pause, stop, or hide any content that moves, blinks, or auto-updates for more than 5 seconds (WCAG 2.2.2).
- Respect the user's `prefers-reduced-motion` setting. Disable or reduce non-essential animations when this preference is active.

```css
/* Default: animation enabled */
.fade-in {
  animation: fadeIn 0.3s ease-in;
}

/* Respect user preference */
@media (prefers-reduced-motion: reduce) {
  .fade-in {
    animation: none;
  }
}
```

```typescript
// React — check motion preference
const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
```

- No content should flash more than three times per second (WCAG 2.3.1 — seizure risk).

---

## React Component Patterns

### Accessible component props

Every component that renders interactive elements must accept and forward accessibility props:

```typescript
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "danger";
  isLoading?: boolean;
}

function Button({ children, isLoading, disabled, ...props }: ButtonProps) {
  return (
    <button
      disabled={disabled || isLoading}
      aria-busy={isLoading || undefined}
      {...props}
    >
      {isLoading ? (
        <>
          <Spinner aria-hidden="true" />
          <span className="sr-only">Loading</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}
```

### Route announcements

When navigating between pages in a React SPA, screen readers do not announce the transition by default. Use a live region to announce the new page:

```typescript
function RouteAnnouncer() {
  const location = useLocation();
  const [announcement, setAnnouncement] = useState("");

  useEffect(() => {
    // Set the document title, then announce it
    const title = document.title;
    setAnnouncement(`Navigated to ${title}`);
  }, [location]);

  return (
    <div aria-live="assertive" aria-atomic="true" className="sr-only">
      {announcement}
    </div>
  );
}
```

### Visually hidden utility

For text that should be read by screen readers but not visible:

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

/* Show on focus (for skip links) */
.sr-only:focus,
.sr-only:focus-within {
  position: static;
  width: auto;
  height: auto;
  padding: inherit;
  margin: inherit;
  overflow: visible;
  clip: auto;
  white-space: normal;
}
```

In Tailwind CSS, use the `sr-only` and `not-sr-only` utility classes.

---

## React Native / Mobile Accessibility

React Native provides built-in accessibility props that map to platform-native accessibility APIs (iOS VoiceOver, Android TalkBack).

### Essential props

```tsx
// Accessible button
<Pressable
  accessibilityRole="button"
  accessibilityLabel="Delete booking"
  accessibilityHint="Removes this booking from your list"
  accessibilityState={{ disabled: isLoading }}
  onPress={handleDelete}
>
  <TrashIcon />
</Pressable>

// Accessible image
<Image
  source={require("./avatar.png")}
  accessibilityLabel="Profile photo of Sam Bailey"
/>

// Decorative image — hidden from screen readers
<Image
  source={require("./divider.png")}
  accessibilityElementsHidden={true}  // iOS
  importantForAccessibility="no-hide-descendants"  // Android
/>

// Live region for dynamic updates
<Text accessibilityLiveRegion="polite">
  {itemCount} items in your basket
</Text>
```

### Rules for mobile

- Set `accessibilityRole` on every interactive element.
- Set `accessibilityLabel` when the visual content is insufficient (icon-only buttons, images).
- Use `accessibilityHint` to explain what happens when the element is activated, if it is not obvious from the label.
- Group related elements with `accessible={true}` on a container `<View>` so they are announced as a single unit.
- Test with VoiceOver (iOS) and TalkBack (Android) on real devices. Emulators are insufficient for accessibility testing.
- Ensure touch targets are at least 44x44 points (iOS) / 48x48 dp (Android).

---

## Server-Rendered Accessibility (Django / Laravel)

For server-rendered pages, accessibility is primarily an HTML and template concern.

### Django templates

```html
{# Good — accessible form with error handling #}
<form method="post" novalidate>
  {% csrf_token %}
  {% for field in form %}
    <div>
      <label for="{{ field.id_for_label }}">{{ field.label }}</label>
      {{ field }}
      {% if field.errors %}
        <p id="{{ field.id_for_label }}-error" role="alert">
          {{ field.errors.0 }}
        </p>
      {% endif %}
    </div>
  {% endfor %}
  <button type="submit">Submit</button>
</form>
```

### Laravel Blade templates

```blade
{{-- Good — accessible form with error handling --}}
<form method="POST" action="{{ route('bookings.store') }}" novalidate>
    @csrf
    <div>
        <label for="guest_name">Guest name</label>
        <input
            type="text"
            id="guest_name"
            name="guest_name"
            value="{{ old('guest_name') }}"
            autocomplete="name"
            @if($errors->has('guest_name'))
                aria-invalid="true"
                aria-describedby="guest_name-error"
            @endif
        />
        @error('guest_name')
            <p id="guest_name-error" role="alert">{{ $message }}</p>
        @enderror
    </div>
    <button type="submit">Book</button>
</form>
```

### Rules for server-rendered pages

- Set the `lang` attribute on the `<html>` element (`<html lang="en-GB">`).
- Include a descriptive `<title>` that reflects the current page content.
- Use semantic landmarks (`<nav>`, `<main>`, `<aside>`, `<footer>`).
- Ensure all form fields generated by Django forms or Laravel components have associated labels.
- For Livewire / HTMX / Turbo partial page updates, ensure updated content is announced to screen readers (use `aria-live` regions or focus management).

---

## Testing Accessibility

### Automated testing

Automated tools catch approximately 30-50% of accessibility issues. They are necessary but not sufficient.

- **vitest-axe / jest-axe** — run axe-core on rendered React components in unit tests. See `TESTING.md` for examples.
- **@axe-core/playwright** — run axe in E2E tests against real browser rendering.
- **Lighthouse** — audit accessibility in Chrome DevTools (CI integration available via `lighthouse-ci`).
- **Pa11y** — command-line accessibility testing for server-rendered pages.

### Manual testing

Automated tools cannot catch issues like logical tab order, meaningful alt text, or correct focus management. Manual testing is required.

**Keyboard testing (every PR):**

1. Unplug or disable the mouse/trackpad.
2. Navigate the entire feature using only `Tab`, `Shift+Tab`, `Enter`, `Space`, `Escape`, and arrow keys.
3. Verify that every interactive element is reachable and operable.
4. Verify that focus order is logical.
5. Verify that focus is visible at all times.

**Screen reader testing (every major feature):**

- **macOS:** VoiceOver (built-in, Cmd+F5 to toggle)
- **Windows:** NVDA (free) or JAWS
- **iOS:** VoiceOver (built-in, Settings > Accessibility)
- **Android:** TalkBack (built-in, Settings > Accessibility)

Navigate the feature with the screen reader and verify that all content is announced correctly, interactive elements are labelled, and dynamic updates are communicated.

**Zoom and reflow testing:**

1. Set browser zoom to 200% — verify no content is lost or overlapping.
2. Set browser zoom to 400% at 1280px viewport width — verify content reflows to a single column.

---

## Accessibility Checklist

Before submitting any user-facing change for review:

- [ ] All interactive elements are operable with keyboard alone
- [ ] Tab order follows a logical reading sequence
- [ ] All form fields have programmatically associated labels
- [ ] Error messages are associated with their fields via `aria-describedby`
- [ ] Colour is not the only means of conveying information
- [ ] Text contrast meets 4.5:1 (normal) or 3:1 (large) minimums
- [ ] Images have appropriate `alt` text (informative or empty for decorative)
- [ ] Icon-only buttons have accessible labels
- [ ] Focus is managed correctly when content changes dynamically
- [ ] Modals trap focus and return focus on close
- [ ] `prefers-reduced-motion` is respected for animations
- [ ] Touch targets are at least 44x44 points (mobile)
- [ ] The page has a skip link to main content
- [ ] Heading levels are in logical order with one `<h1>` per page
- [ ] `lang` attribute is set on `<html>`
- [ ] axe-core assertions pass in automated tests
- [ ] Keyboard-only navigation has been manually verified
