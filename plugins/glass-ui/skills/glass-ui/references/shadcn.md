# Glass UI — shadcn / React

shadcn components are plain Radix + Tailwind + `cn()`. Add a `glass` look two ways.

## Option A — drop-in `GlassCard`

```tsx
// components/ui/glass-card.tsx
import * as React from "react";
import { cn } from "@/lib/utils";

const GlassCard = React.forwardRef<
  HTMLDivElement,
  React.HTMLAttributes<HTMLDivElement>
>(({ className, ...props }, ref) => (
  <div
    ref={ref}
    className={cn(
      "rounded-3xl border border-white/30 bg-white/10 p-6 text-white",
      "backdrop-blur-xl backdrop-saturate-150",
      "shadow-[0_24px_60px_rgba(0,0,0,0.18),inset_0_1px_0_rgba(255,255,255,0.35)]",
      // fallbacks
      "supports-[not_(backdrop-filter:blur(0))]:bg-slate-900/85",
      "motion-reduce:backdrop-blur-none",
      className
    )}
    {...props}
  />
));
GlassCard.displayName = "GlassCard";
export { GlassCard };
```

## Option B — a `glass` variant on the existing shadcn `Card` via `cva`

```tsx
import { cva, type VariantProps } from "class-variance-authority";

const cardVariants = cva("rounded-xl border text-card-foreground shadow", {
  variants: {
    variant: {
      default: "bg-card",
      glass:
        "border-white/30 bg-white/10 text-white backdrop-blur-xl backdrop-saturate-150 " +
        "shadow-[0_24px_60px_rgba(0,0,0,0.18),inset_0_1px_0_rgba(255,255,255,0.35)]",
    },
  },
  defaultVariants: { variant: "default" },
});

export interface CardProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof cardVariants> {}

function Card({ className, variant, ...props }: CardProps) {
  return <div className={cn(cardVariants({ variant }), className)} {...props} />;
}
```
```tsx
<Card variant="glass">…</Card>
```

## Theme-token version (respects dark/light mode)
Instead of hardcoded white, drive it off shadcn CSS variables so it adapts:

```tsx
className={cn(
  "rounded-3xl border bg-background/10 backdrop-blur-xl backdrop-saturate-150",
  "border-foreground/20 text-foreground",
  "shadow-[0_24px_60px_hsl(var(--foreground)/0.18),inset_0_1px_0_hsl(var(--foreground)/0.20)]"
)}
```

## Reminders
- Glass needs a busy backdrop — put a gradient/image on the page or a parent, not a flat `bg-background`.
- Keep `Dialog`/`Popover` overlays usable: a glass modal still needs enough fill alpha to read text. Bump `bg-white/10` → `bg-white/20` for dialogs.
