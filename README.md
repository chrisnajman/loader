# Loader

The loader runs until the page content is fully loaded. It is then removed from the DOM.

---

## Table of Contents

- [Description](#description)
- [Accessibility](#accessibility)
  - [Screen Reader Accessibility](#screen-reader-accessibility)
  - [User Preference: Reduced Motion](#user-preference-reduced-motion)
  - [WAVE Web Accessibility Evaluation Tools Report](#wave-web-accessibility-evaluation-tools-report)
- [Testing](#testing)
  - [To Disable Animations (Chrome, Windows 10)](#to-disable-animations-chrome-windows-10)
  - [To re-enable animations](#to-re-enable-animations)
- [Code](#code)
  - [HTML](#html)
  - [CSS](#css)
  - [JavaScript](#javascript)
- [Source](#source)

## Description

- **If user-preferences allow animations**:
  - Display loading animation.
- **Else**:
  - Display static text: "Loading...".

---

## Accessibility

### Screen Reader Accessibility

The HTML contains text only accessible by screen readers:

```HTML
<div id="loader" class="loader">
  <span class="visually-hidden">The page is loading...</span>
</div>

<p class="visually-hidden" aria-hidden="true" id="page-loaded"></p>
```

After the page has loaded (and the loader has been removed from the DOM), `p id="page-loaded"` is rendered as:

```HTML
<p class="visually-hidden" aria-hidden="false" id="page-loaded">The page has loaded.</p>
```

### User Preference: Reduced Motion

All animation/transition CSS is wrapped inside:

```CSS
@media (prefers-reduced-motion: no-preference) {
  ...
}
```

This ensures that if `prefers-reduced-motion` has _**not**_ been set, the animation will run.

If it _**has**_ been set, fallback CSS will display a static message.

### WAVE Web Accessibility Evaluation Tools Report

- [Automated accessibility analysis results](https://wave.webaim.org/report#/https://chrisnajman.github.io/loader/)

---

## Testing

Refresh the page to trigger the loader.

### To Disable Animations (Chrome, Windows 10):

1. Open inspector,
2. `Ctrl+Shift+P`,
3. `Run>`. Type 'reduce',
4. Click 'Rendering' on 'Emulate CSS prefers-reduced-motion <b>reduce</b>',
5. Refresh the page.

### To re-enable animations:

- Follow steps 1-3 then,
- Click 'Rendering' on 'Emulate CSS prefers-reduced-motion',
- Refresh the page.

**Tested on Windows 10 with**:

- Chrome
- Firefox
- Microsoft Edge

The page has been tested in both browser and device views.

---

## Code

### HTML

```HTML
<div id="loader" class="loader">
  <span class="visually-hidden">The page is loading...</span>
</div>

<p class="visually-hidden" aria-hidden="true" id="page-loaded"></p>
```

### CSS

```CSS
:root {
    --clr-lightest: white;
    --clr-green: rgb(4, 167, 167);
    --clr-dark: rgb(30, 39, 39);
}

.loader {
    position: fixed;
    z-index: 500;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: var(--clr-dark);
}

.loader-hidden {
    opacity: 0;
    visibility: hidden;
}

.loader::after {
    content: "Loading...";
    font-size: 5rem;
}

@media (prefers-reduced-motion: no-preference) {
    .loader {
        transition:
            opacity 0.75s,
            visibility 0.75s;
    }

    .loader::after {
        content: "";
        font-size: 0rem;
        width: 10rem;
        height: 10rem;
        border: 2rem solid var(--clr-lightest);
        border-top-color: var(--clr-green);
        border-radius: 50%;
        animation: loading 0.75s ease infinite;
    }
    @keyframes loading {
        from {
            transform: rotate(0turn);
        }
        to {
            transform: rotate(1turn);
        }
    }
}
```

### JavaScript

```JavaScript
window.addEventListener("load", () => {
  const loader = document.getElementById("loader")
  const pageLoaded = document.getElementById("page-loaded")

  loader.classList.add("loader-hidden")

  loader.addEventListener("transitionend", () => {
    loader.remove()

    // For screen readers
    pageLoaded.textContent = "Page has loaded."
    pageLoaded.setAttribute("aria-hidden", "false")
  })
})
```

---

## Source

This is a fork from [CSS Only Page Loader (CodePen)](https://codepen.io/dcode-software/pen/rNYGdeg), with accessibility enhancements of the CSS and JavaScript.

I also made a further change to the JavaScript, replacing

```JavaScript
loader.addEventListener("transitionend", () => {
  document.body.removeChild(loader)
})
```

with

```JavaScript
loader.addEventListener("transitionend", () => {
  loader.remove()
})
```

as the former triggered a console error:

> Uncaught DOMException: Failed to execute 'removeChild' on 'Node': The node to be removed is not a child of this node.
