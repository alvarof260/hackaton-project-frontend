---
name: heroui
description: >
  HeroUI v3 component library patterns, styling, theming, and composition for React.
  Trigger: When creating or modifying HeroUI components, styling with HeroUI, theming, or importing from @heroui/react.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

Use this skill when:
- Importing or creating HeroUI v3 components
- Styling components with Tailwind CSS or HeroUI variant functions
- Customizing themes (light/dark mode, custom color palettes)
- Using compound component patterns (Card.Header, Alert.Icon, etc.)
- Extending component variants with `tv()` from `@heroui/styles`

---

## Project Setup

This project uses HeroUI v3 with:
- **React 19** + React Compiler (babel-plugin-react-compiler)
- **Tailwind CSS v4** via `@tailwindcss/vite`
- **Vite** as bundler

### CSS Setup (already configured in `src/index.css`)

```css
@import "tailwindcss";
@import "@heroui/styles";
```

Import order matters: `tailwindcss` MUST come before `@heroui/styles`.

---

## Critical Patterns

### Pattern 1: Compound Components (v3 signature pattern)

HeroUI v3 uses compound components. NEVER use v2 syntax (`<Card.Body>`, `<Card.Footer>`).

```tsx
// ✅ v3 compound pattern (recommended)
import { Card, Alert, Modal } from "@heroui/react";

<Card>
  <Card.Header>Title</Card.Header>
  <Card.Description>Description text</Card.Description>
  <Card.Footer>Footer content</Card.Footer>
</Card>

// ✅ Alternative: named exports
import { AlertRoot, AlertIcon, AlertTitle } from "@heroui/react";

<AlertRoot>
  <AlertIcon />
  <AlertTitle>Success</AlertTitle>
</AlertRoot>

// ❌ NEVER v2 syntax
<Card.Body>...</Card.Body>
<Card.Footer>...</Card.Footer>
```

### Pattern 2: Styling with className (primary approach)

All HeroUI components accept `className` with Tailwind utilities:

```tsx
<Button className="bg-purple-500 hover:bg-purple-600">Custom</Button>
<Card className="w-96 rounded-xl shadow-lg">...</Card>
```

### Pattern 3: State-Based Styling via Data Attributes

Components expose state through data attributes:

```css
/* Target component states */
.button[data-hovered="true"], .button:hover {
  background: var(--accent-hover);
}

.button[data-pressed="true"], .button:active {
  transform: scale(0.97);
}

.button[data-focus-visible="true"], .button:focus-visible {
  outline: 2px solid var(--focus);
}
```

### Pattern 4: Render Props for Dynamic Styling

```tsx
// Dynamic classes based on state
<Button
  className={({ isPressed }) =>
    isPressed ? 'bg-blue-600' : 'bg-blue-500'
  }
>
  Press me
</Button>

// Dynamic content based on state
<Button>
  {({ isHovered, isPressed }) => (
    <>
      <Icon
        icon="gravity-ui:heart"
        className={isPressed ? 'text-red-500' : 'text-neutral-400'}
      />
      <span className={isHovered ? 'underline' : ''}>Like</span>
    </>
  )}
</Button>
```

### Pattern 5: BEM Classes (global overrides)

HeroUI uses BEM methodology for CSS class naming:

```css
/* Block */
.button { }
.accordion { }

/* Element */
.accordion__trigger { }
.accordion__panel { }

/* Modifier */
.button--primary { }
.button--lg { }
.accordion--outline { }
```

Override globally in `@layer components`:

```css
@layer components {
  .button {
    @apply font-semibold uppercase;
  }
  .button--primary {
    @apply bg-indigo-600 hover:bg-indigo-700;
  }
}
```

---

## Variant Functions (Custom Components)

Use `tv()` from `@heroui/styles` to extend component variants. This is the HERO WAY to create custom branded components.

```tsx
import type { ButtonRootProps } from "@heroui/react";
import type { VariantProps } from "tailwind-variants";
import { Button } from "@heroui/react";
import { buttonVariants, tv } from "@heroui/styles";

const myButtonVariants = tv({
  extend: buttonVariants,
  base: "font-semibold shadow-md",
  variants: {
    intent: {
      primary: "bg-blue-500 hover:bg-blue-600 text-white",
      secondary: "bg-gray-200 hover:bg-gray-300",
      danger: "bg-red-500 hover:bg-red-600 text-white",
    },
    size: {
      sm: "text-sm px-2 py-1",
      md: "text-base px-4 py-2",
      lg: "text-lg px-6 py-3",
    },
  },
  defaultVariants: {
    intent: "primary",
    size: "md",
  },
});

type MyButtonVariants = VariantProps<typeof myButtonVariants>;
type MyButtonProps = Omit<ButtonRootProps, "className"> &
  MyButtonVariants & { className?: string };

function CustomButton({ intent, size, className, ...props }: MyButtonProps) {
  return (
    <Button
      className={myButtonVariants({ intent, size, className })}
      {...props}
    />
  );
}
```

### Polymorphic Styling (framework-agnostic)

Variant functions work on ANY element, not just HeroUI components:

```tsx
import { buttonVariants } from "@heroui/styles";

// Style a Next.js Link as a button
<Link className={buttonVariants({ variant: "primary" })} href="/about">
  About
</Link>

// Style a native anchor
<a className="button button--primary" href="/dashboard">
  Go to Dashboard
</a>
```

---

## Theming

### Apply a theme

```html
<html class="light" data-theme="light">
  <body class="bg-background text-foreground">
    <!-- App -->
  </body>
</html>
```

### Theme switching

```html
<!-- Light -->
<html class="light" data-theme="light">
<!-- Dark -->
<html class="dark" data-theme="dark">
```

### Override colors (CSS variables)

```css
:root {
  --accent: oklch(0.7 0.25 260);
  --success: oklch(0.65 0.15 155);
}
```

### Add custom semantic colors

```css
:root, [data-theme="light"] {
  --info: oklch(0.6 0.15 210);
  --info-foreground: oklch(0.98 0 0);
}

.dark, [data-theme="dark"] {
  --info: oklch(0.7 0.12 210);
  --info-foreground: oklch(0.15 0 0);
}

@theme inline {
  --color-info: var(--info);
  --color-info-foreground: var(--info-foreground);
}
```

### CSS Variable Naming Convention

- Colors WITHOUT suffix → backgrounds (`--accent`)
- Colors WITH `-foreground` suffix → text on that background (`--accent-foreground`)

---

## Custom DOM Element (render prop)

Replace the rendered element while preserving accessibility:

```tsx
import { Button } from "@heroui/react";
import { motion } from "motion/react";

<Button
  render={(domProps, { isPressed }) => (
    <motion.button
      {...domProps}
      animate={{ scale: isPressed ? 0.9 : 1 }}
    />
  )}
>
  Press me
</Button>
```

---

## Import Strategies

Full import (recommended for this project):

```css
@import "tailwindcss";
@import "@heroui/styles";
```

Selective import (when optimizing bundle):

```css
@layer theme, base, components, utilities;
@import "tailwindcss";
@import "@heroui/styles/base" layer(base);
@import "@heroui/styles/themes/default" layer(theme);
@import "@heroui/styles/components" layer(components);
```

---

## Decision Tree

```
Need to customize component appearance?
├─ Simple Tailwind classes → className prop
├─ Dynamic state-based → render props or data attributes
├─ Global override → BEM classes in @layer components
├─ Custom branded component → tv() extending variants
└─ Different DOM element → render prop

Need to compose components?
├─ Simple → compound pattern (Card.Header)
├─ Custom root → manual className with variant functions
└─ Framework component → BEM classes or variant functions
```

---

## Available Variant Functions

| Component | Function | Import |
|-----------|----------|--------|
| Button | `buttonVariants` | `@heroui/styles` |
| Link | `linkVariants` | `@heroui/styles` |
| Chip | `chipVariants` | `@heroui/styles` |
| Spinner | `spinnerVariants` | `@heroui/styles` |

---

## Commands

```bash
# Install HeroUI (already installed in this project)
bun add @heroui/styles @heroui/react

# Dev server
bun dev
```

---

## Resources

- **Documentation**: https://www.heroui.com/docs/react/getting-started
- **Components**: https://www.heroui.com/docs/react/components
- **Styling**: https://www.heroui.com/docs/react/getting-started/styling
- **Theming**: https://www.heroui.com/docs/react/getting-started/theming
- **Composition**: https://www.heroui.com/docs/react/getting-started/composition
- **Theme Builder**: https://www.heroui.com/docs/react/getting-started/theming#theme-builder
