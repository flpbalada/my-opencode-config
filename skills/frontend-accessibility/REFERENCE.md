# Frontend Accessibility Reference Guide

This document provides comprehensive WCAG 2.1 AA + 2.2 guidelines, detailed implementation patterns, and testing methodology.

## WCAG 2.1 AA Quick Reference

### Perceivable

| Criterion | Requirement | Implementation |
|-----------|-------------|----------------|
| 1.1.1 | Non-text content has text alternative | `alt` attributes, `aria-label` |
| 1.3.1 | Info and relationships programmatically determined | Semantic HTML, proper heading hierarchy |
| 1.3.2 | Meaningful sequence | Logical DOM order, CSS doesn't break reading order |
| 1.4.1 | Use of color not only way to convey info | Add icons, patterns, text labels |
| 1.4.3 | Contrast minimum (4.5:1) | Verify color pairs |
| 1.4.4 | Resize text to 200% | Responsive design, no pixel dependencies |
| 1.4.10 | Reflow (no horizontal scroll at 320px) | CSS containment, responsive containers |
| 1.4.11 | Non-text contrast (3:1) | UI components, graphics |
| 1.4.12 | Text spacing adjustable | Support line-height, letter-spacing changes |

### Operable

| Criterion | Requirement | Implementation |
|-----------|-------------|----------------|
| 2.1.1 | Keyboard accessible | All functionality via keyboard |
| 2.1.2 | No keyboard trap | Focus can always be moved away |
| 2.4.1 | Bypass blocks | Skip links |
| 2.4.2 | Page titled | Descriptive `<title>` elements |
| 2.4.3 | Focus order | Logical tab sequence |
| 2.4.4 | Link purpose (from context) | Descriptive link text |
| 2.4.6 | Headings and labels | Descriptive headings, form labels |
| 2.4.7 | Focus visible | Custom `:focus-visible` styles |
| 2.5.1 | Pointer gestures | Support single-point activation |
| 2.5.3 | Label in name | Accessible name matches visible label |
| 2.5.5 | Target size (44x44px minimum) | Adequate touch targets |

### Understandable

| Criterion | Requirement | Implementation |
|-----------|-------------|----------------|
| 3.1.1 | Language of page | `lang` attribute on `<html>` |
| 3.2.1 | On focus no context change | Don't trigger actions on focus |
| 3.2.2 | On input no context change | No unexpected submissions |
| 3.3.1 | Error identification | Clear error messages |
| 3.3.2 | Labels or instructions | Visible labels for inputs |

### Robust

| Criterion | Requirement | Implementation |
|-----------|-------------|----------------|
| 4.1.1 | Parsing | Valid HTML |
| 4.1.2 | Name, role, value | Proper ARIA usage |

### WCAG 2.2 New Criteria (2023)

| Criterion | Requirement | Implementation |
|-----------|-------------|----------------|
| 2.4.11 | Focus not obscured (minimum) | Focused element fully visible |
| 2.4.12 | Focus not obscured (enhanced) | No content overlaps focused element |
| 3.3.7 | Redundant entry | Don't ask for info already provided |

## Implementation Deep Dive

### Form Accessibility

#### Labels and Associations

```jsx
// ✅ Explicit association via htmlFor
<label htmlFor="email">Email address</label>
<input id="email" type="email" />

// ✅ Implicit association (wrapped)
<label>
  Email address
  <input type="email" />
</label>

// ❌ No association - screen reader can't link them
<label>Email address</label>
<input type="email" />
```

#### Error Handling

Per [ARIA APG naming practices](https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/):

```jsx
export function AccessibleFormField({ 
  label, 
  error, 
  id, 
  ...props 
}) {
  const errorId = `${id}-error`;
  
  return (
    <div className="form-field">
      <label htmlFor={id}>{label}</label>
      <input
        id={id}
        aria-invalid={!!error}
        aria-describedby={error ? errorId : undefined}
        {...props}
      />
      {error && (
        <p id={errorId} role="alert" className="text-red-600">
          {error}
        </p>
      )}
    </div>
  );
}
```

**Note:** Use `aria-describedby` for errors instead of `aria-errormessage` for better browser/screen reader support.

### Modal and Dialog Accessibility

Following [ARIA APG Dialog Modal Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/):

```jsx
import { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom';

export function AccessibleModal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);
  const previousFocusRef = useRef(null);
  
  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement;
      // Focus first element in modal
      const focusable = modalRef.current.querySelector(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      focusable?.focus();
      
      // Trap focus
      const handleKeyDown = (e) => {
        if (e.key === 'Escape') {
          onClose();
          return;
        }
        
        if (e.key === 'Tab') {
          const focusableElements = modalRef.current.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          );
          const firstElement = focusableElements[0];
          const lastElement = focusableElements[focusableElements.length - 1];
          
          if (e.shiftKey && document.activeElement === firstElement) {
            e.preventDefault();
            lastElement.focus();
          } else if (!e.shiftKey && document.activeElement === lastElement) {
            e.preventDefault();
            firstElement.focus();
          }
        }
      };
      
      document.addEventListener('keydown', handleKeyDown);
      return () => {
        document.removeEventListener('keydown', handleKeyDown);
        previousFocusRef.current?.focus();
      };
    }
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return createPortal(
    <div 
      role="dialog" 
      aria-modal="true" 
      aria-labelledby="modal-title"
      aria-describedby="modal-description"
      ref={modalRef}
      className="modal-overlay"
    >
      <h2 id="modal-title" className="sr-only">{title}</h2>
      <p id="modal-description" className="sr-only">{description}</p>
      {children}
      <button onClick={onClose} aria-label="Close modal">
        <CloseIcon />
      </button>
    </div>,
    document.body
  );
}
```

**Key patterns per ARIA APG:**
- Focus the **first meaningful element** inside the dialog, not the dialog itself
- Use `aria-describedby` for additional context beyond the title
- Always restore focus to the previously focused element when closing

### Dynamic Content and Live Regions

Per [ARIA APG Live Regions](https://www.w3.org/WAI/ARIA/apg/practices/live-regions/):

```jsx
// Polite updates (announced when user is idle)
// Always use aria-atomic="true" to announce entire region, not just changed parts
function StatusMessage({ message }) {
  return (
    <div aria-live="polite" aria-atomic="true" className="status">
      {message}
    </div>
  );
}

// Assertive updates (announced immediately - use sparingly)
function ErrorAlert({ error }) {
  return (
    <div role="alert" aria-live="assertive" className="error-banner">
      {error}
    </div>
  );
}

// Loading state
function LoadingIndicator() {
  return (
    <div 
      aria-live="polite" 
      aria-busy="true" 
      aria-label="Loading content"
    >
      <Spinner />
    </div>
  );
}
```

### Reduced Motion Implementation

#### CSS-Only Approach

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

#### JavaScript Detection

```jsx
import { useReducedMotion } from 'framer-motion';

function AnimatedComponent() {
  const shouldReduceMotion = useReducedMotion();
  
  const animation = shouldReduceMotion 
    ? { opacity: 1 }  // Instant, no animation
    : { 
        opacity: [0, 1], 
        scale: [0.8, 1],
        transition: { duration: 0.5 }
      };
  
  return <motion.div animate={animation} />;
}
```

#### Custom Hook

```jsx
import { useState, useEffect } from 'react';

function usePrefersReducedMotion() {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);
  
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);
    
    const handler = (event) => {
      setPrefersReducedMotion(event.matches);
    };
    
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);
  
  return prefersReducedMotion;
}
```

### Color Contrast Tools and Verification

#### Common Contrast Ratios

| Combination | Ratio | AA Normal | AA Large | AAA Normal | AAA Large |
|-------------|-------|-----------|----------|------------|-----------|
| Black on White | 21:1 | ✅ | ✅ | ✅ | ✅ |
| #333 on White | 12.6:1 | ✅ | ✅ | ✅ | ✅ |
| #666 on White | 7.5:1 | ✅ | ✅ | ✅ | ✅ |
| #767676 on White | 4.5:1 | ✅ | ✅ | ❌ | ✅ |
| #767676 on #FFF | 4.48:1 | ❌ | ✅ | ❌ | ✅ |
| #999 on White | 3.0:1 | ❌ | ✅ | ❌ | ❌ |

#### Design System Token Examples

```css
/* tokens.css - Semantic color tokens with guaranteed contrast */
:root {
  /* Text tokens - verified 4.5:1+ on background */
  --text-primary: #1a1a1a;      /* 17.4:1 on white */
  --text-secondary: #4b5563;    /* 7.5:1 on white */
  --text-disabled: #9ca3af;    /* 3.0:1 on white - only for decoration */
  
  /* UI tokens - verified 3:1+ against adjacent */
  --button-primary-bg: #2563eb;
  --button-primary-text: #ffffff;  /* 7.2:1 */
  --button-secondary-bg: #e5e7eb;
  --button-secondary-text: #1f2937; /* 11.9:1 */
  
  /* Focus tokens */
  --focus-ring: #2563eb;
  --focus-ring-offset: #ffffff;
}
```

### Heading Hierarchy

```jsx
// ✅ Correct hierarchy - no skipping levels
function PageStructure() {
  return (
    <article>
      <h1>Main Title</h1>
      <p>Introduction paragraph</p>
      
      <section>
        <h2>Section Title</h2>
        <p>Section content</p>
        
        <section>
          <h3>Subsection Title</h3>
          <p>Subsection content</p>
        </section>
      </section>
    </article>
  );
}

// ❌ Bad hierarchy - skips h2
function BadPageStructure() {
  return (
    <article>
      <h1>Main Title</h1>
      <section>
        <h3>Skipped h2!</h3>  {/* Screen reader users will miss structure */}
      </section>
    </article>
  );
}
```

## Testing Checklist

### Automated Testing

- Run axe-core or Lighthouse accessibility audits
- Verify no critical accessibility errors
- Check color contrast passes

### Keyboard Testing

1. Press `Tab` to move through all interactive elements
2. Verify focus indicator is visible on every element
3. Press `Enter`/`Space` to activate buttons and links
4. Test modal opens and focuses correctly
5. Verify `Escape` closes modals/dropdowns
6. Check no keyboard traps (can always move focus forward/backward)

### Screen Reader Testing

Test with at least one of:
- **VoiceOver** (macOS/iOS) - `Cmd + F5`
- **NVDA** (Windows) - `Insert + Down Arrow`
- **JAWS** (Windows)
- **TalkBack** (Android)

Check:
- All images have alt text
- Form fields are labeled
- Headings form a logical outline
- Links have meaningful text
- Dynamic content is announced

### Touch Target Testing

- Verify all interactive elements are at least 44x44px
- Check spacing between adjacent touch targets
- Test on actual mobile device when possible

### Reduced Motion Testing

1. Enable "Reduce Motion" in OS settings
2. Visit your application
3. Verify:
   - No unnecessary animations play
   - All functionality remains usable
   - No animations that could cause discomfort

## Resources

- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/)
- [ARIA APG - Dialog Modal Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
- [ARIA APG - Names and Descriptions](https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/)
- [ARIA APG - Landmark Regions](https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/)
- [ARIA APG - Live Regions](https://www.w3.org/WAI/ARIA/apg/practices/live-regions/)
- [A11y Project Checklist](https://www.a11yproject.com/checklist/)
- [MDN Accessibility Docs](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Framer Motion Reduced Motion](https://www.framer.com/motion/reduce-motion/)
