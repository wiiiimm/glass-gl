# Glass UI — SCSS

A `glass()` mixin parameterized by the same knobs as the CSS token system.

```scss
// _glass.scss
$glass-tint:      255, 255, 255 !default;   // RGB
$glass-fill:      0.10 !default;
$glass-blur:      22px !default;
$glass-saturate:  1.2 !default;
$glass-radius:    24px !default;

@mixin glass(
  $fill: $glass-fill,
  $blur: $glass-blur,
  $tint: $glass-tint,
  $saturate: $glass-saturate,
  $radius: $glass-radius,
  $border: 0.30,
  $highlight: 0.35
) {
  background: rgba($tint, $fill);
  backdrop-filter: blur($blur) saturate($saturate);
  -webkit-backdrop-filter: blur($blur) saturate($saturate);
  border: 1px solid rgba(255, 255, 255, $border);
  border-radius: $radius;
  box-shadow:
    0 24px 60px rgba(0, 0, 0, 0.18),
    inset 0 1px 0 rgba(255, 255, 255, $highlight);
  color: #fff;

  // Mandatory fallbacks
  @supports not ((backdrop-filter: blur(1px)) or (-webkit-backdrop-filter: blur(1px))) {
    background: rgba(30, 30, 40, 0.85);
  }
  @media (prefers-reduced-transparency: reduce) {
    background: rgba(30, 30, 40, 0.92);
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
  }
}
```

## Usage

```scss
@use "glass" as *;

.card    { @include glass(); padding: 24px; }
.navbar  { @include glass($fill: 0.08, $blur: 14px, $saturate: 1.3, $radius: 0);
           position: sticky; top: 0; }
.button  { @include glass($fill: 0.07, $blur: 12px, $radius: 14px, $border: 0.45);
           cursor: pointer;
           &:hover { background: rgba($glass-tint, 0.16); } }

// Dark glass over a light page
.tooltip { @include glass($tint: (10, 12, 20), $fill: 0.35, $border: 0.12, $highlight: 0.10); }
```

Override defaults globally with SCSS `with`:

```scss
@use "glass" with ($glass-blur: 30px, $glass-saturate: 1.4);
```
