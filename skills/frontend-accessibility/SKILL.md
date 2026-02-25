---
name: frontend-accessibility
description: Enforce WCAG 2.1 AA compliance in visual UI code. Use when generating React/Next.js components, implementing animations, or reviewing designs for accessibility barriers.
---

# Frontend Accessibility (WCAG 2.1 AA + 2.2)

## Core Principle

**Accessibility is not optional. Every user deserves equal access to your product.**

## When to Use

This skill should be invoked when:

- Generating new React/Next.js components
- Implementing animations, transitions, or motion effects
- Creating interactive elements (buttons, forms, modals)
- Reviewing existing code for accessibility barriers

## Motion Safety Rules

### Respect User Motion Preferences

Always implement `prefers-reduced-motion` to honor system-level accessibility settings.

```jsx
import { motion, useReducedMotion } from 'framer-motion';

export function AnimatedCard({ children }) {
  const shouldReduceMotion = useReducedMotion();
  
  return (
    <motion.div
      initial={shouldReduceMotion ? false : { opacity: 0, y: 20 }}
      animate={shouldReduceMotion ? false : { opacity: 1, y: 0 }}
      transition={shouldReduceMotion ? { duration: 0 } : { duration: 0.3 }}
    >
      {children}
    </motion.div>
  );
}
```

### Safe Animation Patterns

- **Duration**: Keep animations under 300ms for micro-interactions
- **Flashing**: Never create content that flashes more than 3 times per second
- **Scroll**: Disable parallax effects when reduced motion is preferred

## Keyboard & Focus Management

### Skip Links

Every page must have a skip link as the first focusable element:

```jsx
export function SkipLink() {
  return (
    <a 
      href="#main-content"
      className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 focus:z-50 focus:px-4 focus:py-2 focus:bg-white focus:text-black focus:ring-2"
    >
      Skip to main content
    </a>
  );
}
```

**Pure CSS alternative (no Tailwind required):**

```css
.skip-link {
  position: absolute;
  left: -9999px;
  z-index: 9999;
  padding: 1rem;
  background: white;
  color: black;
  text-decoration: none;
}

.skip-link:focus {
  left: 0;
  top: 0;
}
```

```jsx
<a href="#main-content" className="skip-link">
  Skip to main content
</a>
```

### Focus Indicators

Never remove focus indicators. Customize them instead:

```jsx
/* ✅ Custom focus styles */
:focus-visible {
  outline: 2px solid #2563eb;
  outline-offset: 2px;
}

/* ❌ Never do this */
:focus {
  outline: none;
}
```

### Focus Traps

Use focus traps in modals and dialogs. Following [ARIA APG dialog pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/):

```jsx
import { useRef, useEffect, useCallback } from 'react';

export function useFocusTrap(isActive, onEscape) {
  const containerRef = useRef(null);
  const previousFocusRef = useRef(null);
  const isActiveRef = useRef(isActive);
  
  useEffect(() => {
    isActiveRef.current = isActive;
  }, [isActive]);
  
  const handleKeyDown = useCallback((e) => {
    if (e.key === 'Escape') {
      onEscape?.();
      return;
    }
    
    if (e.key !== 'Tab') return;
    if (!containerRef.current) return;
    
    const focusableElements = containerRef.current.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    if (focusableElements.length === 0) return;
    
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];
    
    if (e.shiftKey) {
      if (document.activeElement === firstElement || document.activeElement === document.body) {
        e.preventDefault();
        lastElement.focus();
      }
    } else {
      if (document.activeElement === lastElement || document.activeElement === document.body) {
        e.preventDefault();
        firstElement.focus();
      }
    }
  }, [onEscape]);
  
  useEffect(() => {
    if (!isActive || !containerRef.current) return;
    
    // Store previous focus to restore later
    previousFocusRef.current = document.activeElement;
    
    const focusableElements = containerRef.current.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    const firstElement = focusableElements[0];
    // Focus first element - ensure it's visible before focusing
    firstElement?.focus();
    
    document.addEventListener('keydown', handleKeyDown);
    
    return () => {
      if (!isActiveRef.current) return;
      document.removeEventListener('keydown', handleKeyDown);
      // Restore focus when dialog closes
      previousFocusRef.current?.focus();
    };
  }, [isActive, handleKeyDown]);
  
  return containerRef;
}
```

**Usage:**
```jsx
function Modal({ isOpen, onClose, title, description, children }) {
  const modalRef = useFocusTrap(isOpen, onClose);
  
  if (!isOpen) return null;
  
  return (
    <div 
      role="dialog" 
      aria-modal="true" 
      aria-labelledby="modal-title"
      aria-describedby="modal-description"
      ref={modalRef}
    >
      <h2 id="modal-title" className="sr-only">{title}</h2>
      <p id="modal-description" className="sr-only">{description}</p>
      {children}
      <button onClick={onClose} aria-label="Close modal">×</button>
    </div>
  );
}
```

**Note:** Per [ARIA APG](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/), avoid making the dialog element itself focusable (`tabindex="-1"` on the dialog role). Focus the first meaningful element inside instead.

### Accordion

Use proper ARIA attributes for collapsible sections:

```jsx
function Accordion({ items }) {
  return (
    <div role="region" aria-label="Accordion">
      {items.map((item, index) => (
        <AccordionItem key={index} item={item} />
      ))}
    </div>
  );
}

function AccordionItem({ item }) {
  const [isOpen, setIsOpen] = useState(false);
  const buttonId = `accordion-header-${item.id}`;
  const panelId = `accordion-panel-${item.id}`;
  
  return (
    <div>
      <h3>
        <button
          id={buttonId}
          aria-expanded={isOpen}
          aria-controls={panelId}
          onClick={() => setIsOpen(!isOpen)}
        >
          {item.title}
          <span aria-hidden="true">{isOpen ? '−' : '+'}</span>
        </button>
      </h3>
      <div
        id={panelId}
        role="region"
        aria-labelledby={buttonId}
        hidden={!isOpen}
      >
        {item.content}
      </div>
    </div>
  );
}
```

### Tabs

Implement accessible tabbed interfaces:

```jsx
function Tabs({ tabs }) {
  const [activeIndex, setActiveIndex] = useState(0);
  
  return (
    <div role="tablist" aria-label="Content tabs">
      <div role="presentation">
        {tabs.map((tab, index) => (
          <button
            key={index}
            role="tab"
            aria-selected={index === activeIndex}
            aria-controls={`tabpanel-${index}`}
            id={`tab-${index}`}
            onClick={() => setActiveIndex(index)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      {tabs.map((tab, index) => (
        <div
          key={index}
          id={`tabpanel-${index}`}
          role="tabpanel"
          aria-labelledby={`tab-${index}`}
          tabIndex={0}
          hidden={index !== activeIndex}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

**Keyboard navigation for tabs:**
- Arrow keys to move between tabs
- Home/End to jump to first/last tab
- Tab to move to the active tab panel

## Screen Reader Support

### Semantic HTML

Use the right element for the right purpose:

```jsx
// ✅ Semantic - screen readers understand the structure
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>

// ❌ Non-semantic - loses meaning
<div>
  <div>
    <div><a href="/">Home</a></div>
    <div><a href="/about">About</a></div>
  </div>
</div>
```

### Landmark Regions

Use semantic elements and ARIA landmarks to help screen reader users navigate. Per [ARIA APG landmark regions](https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/):

```jsx
{/* ✅ Use native HTML5 elements (recommended) */}
<header>
  <nav aria-label="Main">
    {/* nav content */}
  </nav>
</header>

<main id="main-content">
  {/* primary content */}
</main>

<aside aria-label="Related articles">
  {/* sidebar content */}
</aside>

<footer>
  {/* footer content */}
</footer>

{/* ✅ Multiple nav elements need unique labels */}
<nav aria-label="Primary">
  {/* primary navigation */}
</nav>
<nav aria-label="Breadcrumb">
  {/* breadcrumb navigation */}
</nav>
<nav aria-label="Footer">
  {/* footer navigation */}
</nav>
```

**Note:** The `<main>` element should have an ID for skip links to target (`id="main-content"`).

### ARIA Labels (When Needed)

Only use ARIA when semantic HTML isn't sufficient:

```jsx
// ✅ Icon button needs aria-label
<button aria-label="Close menu">
  <XIcon />
</button>

// ✅ Input needs aria-describedby for error context
<input 
  type="email" 
  aria-describedby="email-error"
  aria-invalid={hasError}
/>
<p id="email-error" role="alert">
  Please enter a valid email address
</p>
```

### Live Regions

Announce dynamic content changes. Per [ARIA APG practices](https://www.w3.org/WAI/ARIA/apg/practices/live-regions/):

```jsx
// ✅ Status updates are announced (polite = waits for idle)
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>

// ✅ Errors are announced immediately (assertive = interrupts)
<form aria-describedby="form-errors">
  <div id="form-errors" role="alert" aria-live="assertive" />
</form>

// ✅ Loading states
<div aria-live="polite" aria-busy="true" aria-label="Loading content">
  <Spinner />
</div>
```

**Note:** Always add `aria-atomic="true"` to ensure the entire region is announced, not just the changed portion.

## Visual Accessibility

### Color Contrast (4.5:1 Minimum)

Ensure sufficient contrast for text:

- **Normal text**: 4.5:1 ratio (AA standard)
- **Large text** (18px+ or 14px bold): 3:1 ratio (still criterion 1.4.3)
- **UI components**: 3:1 ratio against adjacent colors

```jsx
// Use semantic color tokens from your design system
// that automatically meet contrast requirements
<button className="bg-blue-600 text-white">
  Submit
</button>
```

### Touch Targets

Minimum 44x44 pixels for interactive elements:

```jsx
// ✅ Large enough touch target
<button className="min-h-[44px] min-w-[44px]">
  <Icon className="w-5 h-5" />
</button>

// ❌ Too small - difficult to tap
<button className="w-4 h-4">
  <Icon />
</button>
```

### Form Labels

Associate labels with inputs for screen readers. Per [ARIA APG naming practices](https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/):

```jsx
// ✅ Explicit association (preferred)
<label htmlFor="email">Email address</label>
<input id="email" type="email" />

// ✅ Implicit association (wrapped)
<label>
  Email address
  <input type="email" />
</label>

// ✅ Complex fields with aria-labelledby
<input aria-labelledby="name-label-hint" />
<span id="name-label-hint">First and last name</span>

// ✅ Form errors - use aria-describedby (better support than aria-errormessage)
<input 
  id="email"
  type="email" 
  aria-describedby="email-error"
  aria-invalid={hasError}
/>
{hasError && (
  <span id="email-error" role="alert" className="text-red-600">
    Please enter a valid email address
  </span>
)}
```

### Color Independence

Never convey information by color alone:

```jsx
// ✅ Color + icon/text
<span className="text-red-600">
  <ErrorIcon /> Required field
</span>

// ❌ Color only - not accessible
<span className="text-red-600">
  Required field
</span>
```

## Definition of Done Checklist

Before marking accessibility work as complete, verify:

- [ ] All interactive elements are keyboard accessible
- [ ] Focus indicators are visible on all focusable elements
- [ ] Skip link present and functional
- [ ] Color contrast meets 4.5:1 (text) or 3:1 (large text/components)
- [ ] Touch targets are minimum 44x44px
- [ ] No information conveyed by color alone
- [ ] `prefers-reduced-motion` respected in all animations
- [ ] ARIA labels used correctly (not overused)
- [ ] Semantic HTML used throughout
- [ ] Form inputs have associated labels
- [ ] Error messages are announced to screen readers
- [ ] Focus trapped in modals/dialogs
- [ ] No content flashes more than 3 times per second

## Quick Reference

| Requirement | WCAG Criterion | Minimum |
|-------------|----------------|---------|
| Color contrast (text) | 1.4.3 | 4.5:1 |
| Color contrast (large text) | 1.4.3 | 3:1 |
| Focus visible | 2.4.7 | Visible indicator |
| Skip link | 2.4.1 | Bypass blocks |
| Touch target | 2.5.5 | 44x44px |
| Motion | 2.3.3 | Respect preference |
| Error identification | 3.3.1 | Announced |

### WCAG 2.2 New Additions

| Requirement | WCAG Criterion | Description |
|-------------|----------------|-------------|
| Focus not obscured | 2.4.11 | Focused element fully visible |
| Redundant entry | 3.3.7 | Don't re-enter info already provided |

## See Also

- [REFERENCE.md](./REFERENCE.md) - Detailed WCAG checklists and deep-dive documentation

## Sources

- [ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/) - Official W3C patterns for accessible widgets
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/) - Accessibility guidelines
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) - Contrast verification tool
- [A11y Project Checklist](https://www.a11yproject.com/checklist/) - Practical accessibility checklist
