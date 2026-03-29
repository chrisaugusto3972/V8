# V8 Safe UI-Only Patch Pack
## Zero-Logic-Change UI Upgrade Plan

This patch pack is designed to upgrade the V8 interface **without changing scanner, autopilot, execution, risk, broker, or analytics behavior**.

The goal is simple:

- improve appearance
- improve layout
- improve navigation
- improve readability
- preserve all current trading behavior

This is a **safe UI-only patch**, not a product redesign that touches logic.

---

# CORE RULE

## DO NOT CHANGE ANY OF THESE
The following files or folders must remain untouched unless absolutely necessary for imports only:

- `server/`
- `shared/`
- `migrations/`
- broker code
- scanner code
- autopilot logic
- selection engine
- risk engine
- analytics formulas
- API routes
- API request payloads
- API response shapes
- data-fetching hooks behavior
- settings defaults
- any trading workflow timing

This patch must only change:
- page layout
- CSS / Tailwind classes
- presentational wrappers
- visual hierarchy
- labels
- icons
- spacing
- typography
- navigation shell

---

# SAFE PATCH STRATEGY

## Only modify these areas
- `client/src/App.tsx`
- `client/src/index.css`
- `client/src/components/` (new presentational components only)
- `client/src/pages/` (layout and styling only)

## Allowed new files
- `client/src/components/app-shell.tsx`
- `client/src/components/page-hero.tsx`
- `client/src/components/stat-card.tsx`
- `client/src/components/section-card.tsx`
- `client/src/components/status-badge.tsx`

These components must be **pure presentational components** only.

They must:
- accept props
- render UI
- not fetch data
- not mutate state outside their local display state
- not call trading APIs
- not alter business logic
- not change existing handler behavior

---

# SAFETY REQUIREMENTS

## 1. No new API calls
Do not add:
- new fetch calls
- new polling loops
- new websocket subscriptions
- new mutations
- new POST requests
- new settings writes

## 2. No change to existing API calls
Do not change:
- endpoints
- request bodies
- response parsing
- timing
- retry logic
- query keys
- invalidation logic

## 3. No change to page logic
If a page currently:
- fetches data
- toggles autopilot
- submits settings
- displays trades

Then the UI patch may only:
- reorganize the layout
- wrap sections in cards
- improve spacing and labels
- preserve exact event handlers and props

## 4. No form behavior changes
Do not change:
- default values
- checkbox meaning
- button handlers
- submission order
- debounce behavior

## 5. No hidden side effects
No:
- useEffect additions that trigger API actions
- automatic saves
- new timers
- implicit state syncing

---

# VISUAL DESIGN GOAL

Transform current V8 into a cleaner SaaS layout:

- left nav app shell
- clean page headers
- summary cards
- card-based sections
- consistent spacing
- clear hierarchy
- dark-mode-first institutional fintech look

But preserve:
- exact data
- exact actions
- exact control behavior

---

# FILE-BY-FILE PATCH PLAN

## 1) Create `client/src/components/app-shell.tsx`

Purpose:
- provide page chrome only
- left nav
- top bar
- content container

Must not:
- fetch data
- manage business state
- alter routing behavior beyond rendering nav links

Safe implementation:

```tsx
import React from "react";
import { Link, useLocation } from "wouter";

type AppShellProps = {
  title?: string;
  children: React.ReactNode;
};

const navItems = [
  { href: "/", label: "Dashboard" },
  { href: "/scanner", label: "Scanner" },
  { href: "/positions", label: "Positions" },
  { href: "/portfolio-greeks", label: "Risk" },
  { href: "/autopilot", label: "Autopilot" },
];

export function AppShell({ title = "V8", children }: AppShellProps) {
  const [location] = useLocation();

  return (
    <div className="min-h-screen bg-slate-950 text-slate-100">
      <div className="flex min-h-screen">
        <aside className="hidden md:flex w-64 shrink-0 flex-col border-r border-slate-800 bg-slate-925">
          <div className="px-6 py-5 border-b border-slate-800">
            <div className="text-xs uppercase tracking-[0.2em] text-slate-400">V8</div>
            <div className="mt-1 text-lg font-semibold text-white">Institutional Scanner</div>
          </div>

          <nav className="flex-1 px-3 py-4 space-y-1">
            {navItems.map((item) => {
              const active = location === item.href;
              return (
                <Link key={item.href} href={item.href}>
                  <a
                    className={[
                      "block rounded-lg px-3 py-2 text-sm transition-colors",
                      active
                        ? "bg-slate-800 text-white"
                        : "text-slate-300 hover:bg-slate-900 hover:text-white",
                    ].join(" ")}
                  >
                    {item.label}
                  </a>
                </Link>
              );
            })}
          </nav>
        </aside>

        <div className="flex-1 min-w-0">
          <header className="border-b border-slate-800 bg-slate-950/90 backdrop-blur">
            <div className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 py-4">
              <div className="flex items-center justify-between gap-4">
                <div>
                  <div className="text-xs uppercase tracking-[0.2em] text-slate-400">Workspace</div>
                  <h1 className="text-xl font-semibold text-white">{title}</h1>
                </div>
                <div className="rounded-full border border-emerald-700/40 bg-emerald-900/20 px-3 py-1 text-xs text-emerald-300">
                  SaaS UI Mode
                </div>
              </div>
            </div>
          </header>

          <main className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 py-6">
            {children}
          </main>
        </div>
      </div>
    </div>
  );
}
```

---

## 2) Create `client/src/components/page-hero.tsx`

Purpose:
- visual page header only

```tsx
import React from "react";

type PageHeroProps = {
  eyebrow?: string;
  title: string;
  description?: string;
  actions?: React.ReactNode;
};

export function PageHero({
  eyebrow,
  title,
  description,
  actions,
}: PageHeroProps) {
  return (
    <div className="mb-6 rounded-2xl border border-slate-800 bg-gradient-to-br from-slate-900 to-slate-950 p-6">
      {eyebrow ? (
        <div className="text-xs uppercase tracking-[0.2em] text-slate-400">{eyebrow}</div>
      ) : null}
      <div className="mt-1 flex flex-col gap-4 lg:flex-row lg:items-end lg:justify-between">
        <div>
          <h2 className="text-2xl font-semibold text-white">{title}</h2>
          {description ? (
            <p className="mt-2 max-w-3xl text-sm text-slate-300">{description}</p>
          ) : null}
        </div>
        {actions ? <div className="shrink-0">{actions}</div> : null}
      </div>
    </div>
  );
}
```

---

## 3) Create `client/src/components/stat-card.tsx`

```tsx
import React from "react";

type StatCardProps = {
  label: string;
  value: React.ReactNode;
  hint?: React.ReactNode;
};

export function StatCard({ label, value, hint }: StatCardProps) {
  return (
    <div className="rounded-xl border border-slate-800 bg-slate-900/70 p-4">
      <div className="text-xs uppercase tracking-[0.15em] text-slate-400">{label}</div>
      <div className="mt-2 text-2xl font-semibold text-white">{value}</div>
      {hint ? <div className="mt-1 text-xs text-slate-400">{hint}</div> : null}
    </div>
  );
}
```

---

## 4) Create `client/src/components/section-card.tsx`

```tsx
import React from "react";

type SectionCardProps = {
  title?: string;
  subtitle?: string;
  children: React.ReactNode;
};

export function SectionCard({ title, subtitle, children }: SectionCardProps) {
  return (
    <section className="rounded-2xl border border-slate-800 bg-slate-900/60 p-5">
      {title ? (
        <div className="mb-4">
          <div className="text-base font-semibold text-white">{title}</div>
          {subtitle ? <div className="mt-1 text-sm text-slate-400">{subtitle}</div> : null}
        </div>
      ) : null}
      {children}
    </section>
  );
}
```

---

## 5) Create `client/src/components/status-badge.tsx`

```tsx
import React from "react";

type StatusBadgeProps = {
  tone?: "neutral" | "success" | "warning" | "danger";
  children: React.ReactNode;
};

export function StatusBadge({
  tone = "neutral",
  children,
}: StatusBadgeProps) {
  const toneClass =
    tone === "success"
      ? "border-emerald-700/40 bg-emerald-900/20 text-emerald-300"
      : tone === "warning"
        ? "border-amber-700/40 bg-amber-900/20 text-amber-300"
        : tone === "danger"
          ? "border-rose-700/40 bg-rose-900/20 text-rose-300"
          : "border-slate-700 bg-slate-800 text-slate-300";

  return (
    <span className={`inline-flex items-center rounded-full border px-2.5 py-1 text-xs ${toneClass}`}>
      {children}
    </span>
  );
}
```

---

# SAFE PAGE WRAP PATTERN

For each existing page:
- keep all logic exactly as-is
- keep all hooks exactly as-is
- keep all handlers exactly as-is
- only wrap rendered JSX with shell and visual components

Example safe pattern:

```tsx
import { AppShell } from "@/components/app-shell";
import { PageHero } from "@/components/page-hero";
import { SectionCard } from "@/components/section-card";

export default function ExistingPage() {
  // KEEP ALL CURRENT LOGIC EXACTLY AS IT IS

  return (
    <AppShell title="Scanner">
      <PageHero
        eyebrow="Daily engine"
        title="Scanner"
        description="Review the highest-quality setups selected by V8."
      />

      <div className="grid gap-6">
        <SectionCard title="Scanner Results">
          {/* KEEP EXISTING JSX / TABLE / COMPONENTS HERE */}
        </SectionCard>
      </div>
    </AppShell>
  );
}
```

---

# PAGE-SPECIFIC SAFE PATCHES

## A) Dashboard page
Enhance with:
- PageHero
- top summary cards
- section cards

But:
- do not change existing data sources
- do not add derived calculations unless already present

Suggested layout:
- Hero
- 4 stat cards
- Today’s picks card
- Warnings card
- Activity/performance card

---

## B) Scanner page
Enhance with:
- Hero
- filters in one card
- results table in one card

Do not:
- change sorting logic
- change selection logic
- change click handlers

---

## C) Positions / Portfolio page
Enhance with:
- Hero
- positions table in card
- exposure panels in cards if already available

Do not:
- create new calculations in the page
- rely only on existing API values

---

## D) Portfolio Risk page
Enhance with:
- Hero
- summary cards
- stress panel card
- exposure card

Do not:
- modify analytics formulas
- only restyle returned data

---

## E) Autopilot page
Enhance with:
- Hero
- control card
- status card
- activity/log card

Do not:
- change toggle handlers
- change settings payload shape
- change button meanings

---

# APP-LEVEL SAFE PATCH

## Update `client/src/App.tsx`
Only if needed for route layout compatibility.

Allowed:
- wrapping routes in consistent shell if page components do not already do so
- adding visual-only providers if already used in stack

Not allowed:
- changing route paths
- changing route guards
- changing suspense/loading behavior unless only presentational

---

# SAFE CSS PATCH

## Update `client/src/index.css`
Allowed:
- add visual variables
- improve background colors
- improve card styling
- improve typography scale
- improve tables visually
- improve spacing

Not allowed:
- global rules that hide elements used by logic
- pointer-events changes
- disabled-state changes that alter usability
- animations that delay rendering of key controls

Suggested safe additions:

```css
:root {
  color-scheme: dark;
}

body {
  background: #020617;
  color: #e2e8f0;
}

.bg-slate-925 {
  background-color: #0b1220;
}
```

Add only visual utility classes if needed.

---

# DO-NOT-TOUCH CHECKLIST

Before shipping, confirm none of these changed:

- [ ] scanner query logic
- [ ] autopilot toggle behavior
- [ ] candidate processing
- [ ] order submission flow
- [ ] broker settings writes
- [ ] API payloads
- [ ] polling intervals
- [ ] selection engine imports
- [ ] risk engine imports
- [ ] analytics formulas
- [ ] settings defaults

---

# VALIDATION CHECKLIST

After applying patch, compare to pre-patch behavior.

## Functional validation
- [ ] scanner returns same candidate count
- [ ] same top picks appear
- [ ] autopilot ON/OFF still works
- [ ] no API errors in console
- [ ] no new warnings in network tab
- [ ] settings save exactly as before
- [ ] paper mode behavior unchanged

## UI validation
- [ ] all pages render
- [ ] layout is cleaner
- [ ] controls remain clickable
- [ ] no hidden controls
- [ ] tables still scroll correctly
- [ ] mobile/tablet desktop layout still usable

---

# FINAL DELIVERY RULE FOR REPLIT

Tell Replit:

## Instruction
"Apply this as a strict UI-only patch. Do not modify business logic, API calls, hooks, handlers, scanner behavior, autopilot behavior, analytics formulas, or broker interactions. Only change layout, styling, and presentational components."

That instruction matters.

---

# WHAT THIS PATCH ACCOMPLISHES

This gives V8:
- SaaS-style shell
- cleaner navigation
- better perceived professionalism
- better conversion readiness
- zero intended trading-engine impact

This is the safest path before deeper SaaS work.

End of patch pack.
