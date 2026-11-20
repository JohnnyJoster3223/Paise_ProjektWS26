# Research Report: UI Component Libraries for React + TypeScript MVP

- **Topic:** Evaluation of UI Component Libraries for the Smart Task Manager MVP
- **Issue Reference:** [GitHub Issue #7: Research UI component libraries for React and TypeScript](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/7)
- **Target Platform:** Client-side Single-Page Application (SPA) built with React 18/19, TypeScript, Vite, and local persistence (`localStorage`)
- **Author/Investigator:** AI Agent (Antigravity)
- **Date:** November 2026

---

## Executive Summary

This research investigates the architectural and practical trade-offs among three candidate UI approaches for a student-led, client-side Single-Page Application (SPA) task management tool:
1. **Bootstrap 5 / React-Bootstrap (`react-bootstrap`)** — The traditional, battle-tested component wrapper around Bootstrap CSS.
2. **Tailwind CSS + DaisyUI** — A utility-first CSS foundation paired with a zero-JS semantic component plugin.
3. **shadcn/ui (Tailwind CSS + Radix UI Primitives)** — A modern "copy-paste" architecture combining unstyled, accessible primitives with utility styling and full source code ownership.

The evaluation specifically addresses four key criteria defined for the project:
- **Development Velocity:** Time-to-scaffolding, component completeness for task cards, modals, badges, and time inputs.
- **Bundle Size & Runtime Performance:** Minified + gzipped footprint, CSS purging, and tree-shaking efficacy in a modern Vite/Rollup pipeline.
- **Developer Ergonomics & Learning Curve:** Fit for a 3-person student team where 2/3 know TypeScript and have basic/intermediate React and Angular experience.
- **Support for NFR-02 & Accessibility:** Distinguishing task states (Scheduled, Unscheduled, Completed) without relying on color alone (WCAG 2.1 Success Criterion 1.4.1), plus modal dialog focus management.

### Key Finding & Crisp Recommendation

**Primary Recommendation:** **`shadcn/ui` (with `lucide-react`)** is the recommended choice for the team. 
- It delivers first-class TypeScript DX with strongly-typed props (`variant`, `size`) that match the mental model of developers coming from Angular/React.
- It provides uncompromised WAI-ARIA accessibility out of the box via Radix UI primitives (solving keyboard navigation and focus trapping for the FR-TM-03 confirmation dialog).
- It ships bundled with Lucide Icons, making multi-attribute state differentiation for **NFR-02** (icon + text badge + strikethrough/border cues) effortless and visually cohesive.
- Its modular tree-shaking maintains a lightweight bundle (~15–22 kB gzipped total JS/CSS overhead for typical MVP usage).

**Pragmatic Fallback:** If the team wants to minimize configuration steps (avoiding Vite path aliases and local component file management), **Tailwind CSS + DaisyUI** serves as an excellent runner-up with 0 KB JavaScript overhead. However, the team must implement custom accessible modal logic for the deletion confirmation prompt (FR-TM-03).

---

## Comparative Matrix

| Evaluation Dimension | Option 1: Bootstrap / React-Bootstrap | Option 2: Tailwind CSS + DaisyUI | Option 3: shadcn/ui (Radix UI + Tailwind) |
| :--- | :--- | :--- | :--- |
| **Architectural Model** | External npm dependency; pre-compiled React wrappers around Bootstrap 5 | Utility-first CSS engine + pure-CSS semantic component class plugin | Copy-paste source code ownership; headless Radix primitives + Tailwind |
| **JS Bundle Impact (Gzip)** | **~37.6 kB** (with `@restart/ui` & Popper.js) | **0 kB** (Pure CSS, no JS runtime) | **~12–18 kB** (Granular; ~12.9 kB for Dialog, ~1–2 kB for Button/Badge) |
| **CSS Bundle Impact (Gzip)** | **~24 kB** (Full `bootstrap.min.css` unless custom PurgeCSS build is set up) | **~5–10 kB** (Tailwind JIT purges unused DaisyUI classes) | **~4–8 kB** (Tailwind JIT purges unused utility classes) |
| **Total Production Payload** | **~60+ kB** gzipped | **~5–10 kB** gzipped | **~16–26 kB** gzipped |
| **TypeScript Ergonomics** | Good; `@types/react-bootstrap` bundled; typed props on components | Fair; classes are raw strings (`className="btn"`); no prop typing unless self-wrapped | **Excellent**; native TS definitions, strongly-typed `cva` variants, full VS Code intellisense |
| **Team Learning Curve** *(2/3 TS, React/Angular)* | **Very Low**; familiar paradigm from introductory web courses | **Low**; semantic classes (`btn`, `card`, `badge`) feel familiar without class clutter | **Moderate**; requires understanding path aliases (`@/`), CLI tooling, and Radix slot patterns |
| **Development Velocity** | High initially; slows down when customizing layout or custom themes | **Very High**; instant semantic classes + 20+ built-in themes + rapid layout utilities | **High**; fast once initialized; copy-paste components ready for full customization |
| **Accessibility & NFR-02** | Manual; badges require explicit visually-hidden text; no bundled icons; basic dialog a11y | Good visual flexibility; easy badge + icon composition; dialog requires native `<dialog>` handling | **Best-in-class**; full WAI-ARIA compliance, automated focus traps, first-class Lucide icons |
| **Customization & Theming** | Difficult; fights Bootstrap Sass variables and specificity overrides | Very easy; built-in theme generator, CSS custom properties (`data-theme`) | **Total ownership**; direct code modification in `src/components/ui/`, custom Tailwind tokens |

---

## Candidate Deep Dives

### 1. Bootstrap 5 / React-Bootstrap (`react-bootstrap`)

#### Architecture & Paradigm
`react-bootstrap` is the mature, standard React implementation of Bootstrap 5. It replaces Bootstrap’s native JavaScript (`bootstrap.js`) with idiomatic React components (`<Button>`, `<Modal>`, `<Badge>`, `<Form>`), delegating lower-level overlay and dropdown positioning to `@restart/ui` and Popper.js.

```tsx
import Button from 'react-bootstrap/Button';
import Modal from 'react-bootstrap/Modal';
import Badge from 'react-bootstrap/Badge';
import 'bootstrap/dist/css/bootstrap.min.css';
```

#### Development Velocity
- **Strengths:** Nearly zero setup time. Installing `react-bootstrap` and `bootstrap` and importing the CSS stylesheet gets the app running immediately. Standard form controls (`Form.Group`, `Form.Control`, `Form.Select`) and 12-column grid (`Container`, `Row`, `Col`) are familiar to almost every developer.
- **Weaknesses:** Customizing the visual identity beyond the standard "Bootstrap look" is notoriously cumbersome. In an MVP requiring a focused, anti-procrastination UI (such as prominent "Next Up" task banners and timeline schedule blocks), developers often find themselves writing brittle CSS overrides (`!important`) or wrestling with Bootstrap's Sass variable build pipeline.

#### Bundle Size & Performance Impact
- **JavaScript:** According to Bundlephobia metrics, `react-bootstrap` has a minified size of ~114 kB and a **gzipped transfer size of ~37.6 kB**. If developers inadvertently use barrel imports (`import { Button, Modal } from 'react-bootstrap'`), tree-shaking may falter, pulling in unneeded modules.
- **CSS:** The standard `bootstrap.min.css` stylesheet weighs **~160–225 kB raw minified and ~24 kB gzipped**. Because it is imported globally, the entire stylesheet is downloaded by the browser regardless of whether only 3 components are utilized, unless a custom PurgeCSS build step is integrated into Vite.
- **Combined Impact:** ~60+ kB gzipped base payload before application code.

#### Developer Ergonomics & Learning Curve
- For students with an Angular or traditional backend background, React-Bootstrap feels natural: component configuration happens through props (`variant="primary"`, `size="lg"`, `show={isOpen}`, `onHide={handleClose}`).
- TypeScript typings are included out of the box, offering decent auto-completion for component props.

#### Support for NFR-02 & Accessibility
- **State Badges:** Provides `<Badge bg="primary">`, but Bootstrap badges are strictly colored span elements. Bootstrap’s official documentation explicitly warns:
  > *"Using color to add meaning only provides a visual indication, which will not be conveyed to users of assistive technologies – such as screen readers. Ensure that information denoted by the color is either obvious from the content itself (the visible text), or is included through alternative means, such as additional text hidden with the `.visually-hidden` class."*
- **Icons:** Bootstrap does **not** include icons out of the box. The team must install and configure a separate library such as `bootstrap-icons`.
- **Modals:** Includes basic focus trapping and escape-key handling via `@restart/ui`, but managing advanced keyboard focus transitions can feel rigid.

---

### 2. Tailwind CSS + DaisyUI

#### Architecture & Paradigm
DaisyUI is a pure-CSS component plugin for Tailwind CSS. Unlike React-Bootstrap, DaisyUI does not ship React component code or JavaScript runtimes. Instead, it adds semantic component classes to Tailwind (such as `.btn`, `.card`, `.badge`, `.modal`, `.alert`, `.input`) styled using CSS custom properties.

```tsx
// Using DaisyUI semantic classes within React TSX
<div className="card bg-base-100 shadow-md border border-base-200">
  <div className="card-body">
    <div className="flex justify-between items-center">
      <h2 className="card-title text-base">Write Architecture Report</h2>
      <span className="badge badge-primary gap-1">Scheduled</span>
    </div>
  </div>
</div>
```

#### Development Velocity
- **Strengths:** Resolves the biggest complaint about raw Tailwind CSS: the "class soup" where a single button requires 12 utility classes (`px-4 py-2 bg-blue-600 text-white rounded-md font-medium hover:bg-blue-700 focus:outline-none focus:ring-2 ...`). With DaisyUI, writing `<button className="btn btn-primary">` produces a polished, accessible button.
- **Rapid Theming:** Includes 20+ built-in themes (e.g., `light`, `dark`, `cupcake`, `nord`, `corporate`) switchable via a single `data-theme="dark"` attribute on the root HTML tag.
- **Utility Flexibility:** Whenever custom margins, flex alignments, or borders are needed (e.g., `gap-4 p-2 mt-auto`), developers use standard Tailwind utility classes directly without leaving the markup.

#### Bundle Size & Performance Impact
- **JavaScript:** **0 KB**. DaisyUI adds zero JavaScript to the bundle.
- **CSS:** While the raw package stylesheet is large in node_modules, Tailwind's Just-In-Time (JIT) compiler scans all `.tsx` and `.html` files in the repository and removes every unused DaisyUI and Tailwind rule. A typical production build with DaisyUI results in an incremental stylesheet addition of only **5–10 kB gzipped**.
- **Combined Impact:** **~5–10 kB gzipped total** (only CSS). This represents the lowest performance overhead of any evaluated candidate.

#### Developer Ergonomics & Learning Curve
- **Low Barrier to Entry:** For students with partial React or Angular experience, semantic classes (`btn`, `card`, `modal`, `badge`) feel just like Bootstrap or Angular Material classes.
- **TypeScript Trade-off:** Because DaisyUI operates strictly in CSS class strings, there is **no compile-time prop type-checking** out of the box. Typing errors like `className="btn btn-pramary"` fail silently at compile time. However, a student team can easily author small, typed wrapper components:
  ```tsx
  type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
    variant?: 'primary' | 'secondary' | 'error' | 'outline';
    size?: 'sm' | 'md' | 'lg';
  };
  ```
- **Interactive Component State:** Modals and dropdowns in DaisyUI rely on either HTML checkbox hacks (`<input type="checkbox" className="modal-toggle" />`) or native HTML5 `<dialog>` elements. While modern browsers support `<dialog>`, managing it in React requires `useRef<HTMLDialogElement>` and imperative calls (`ref.current?.showModal()`), which introduces state synchronization overhead for junior developers.

#### Support for NFR-02 & Accessibility
- **State Badges:** Very straightforward to compose semantic badges with explicit text and icons:
  ```tsx
  <div className="badge badge-info gap-1.5 font-medium">
    <Clock className="w-3.5 h-3.5" aria-hidden="true" />
    <span>Scheduled (09:00 - 10:30)</span>
  </div>
  ```
- **Visual Styling Cues:** Tailwind utilities allow trivial implementation of non-color indicators: strikethrough text (`line-through opacity-60` for completed tasks), dashed borders (`border-dashed border-2` for unscheduled tasks), and high-contrast left accent bars (`border-l-4 border-l-primary`).
- **Icons:** Requires choosing an icon library (e.g. `lucide-react`).
- **Dialog a11y Caveats:** DaisyUI's CSS-only modal patterns do not automatically handle roving tab focus or backdrop focus lock without native `<dialog>` or custom JS logic.

---

### 3. Component Kits: `shadcn/ui` (Radix UI + Tailwind CSS)

#### Architecture & Paradigm
`shadcn/ui` represents a modern architectural paradigm in frontend development. Rather than distributing compiled components as an opaque npm dependency (like MUI or React-Bootstrap), `shadcn/ui` provides a CLI that scaffolds component source code directly into the application repository (typically in `src/components/ui/`).

It builds upon three core architectural pillars:
1. **Headless Behavior Engine:** Unstyled accessible primitives from **Radix UI** (`@radix-ui/react-dialog`, `@radix-ui/react-dropdown-menu`, `@radix-ui/react-tooltip`, `@radix-ui/react-slot`). Radix manages keyboard interactions, ARIA attributes, focus trapping, and screen reader announcements.
2. **Utility Styling:** Styled with **Tailwind CSS**, allowing arbitrary theme tweaking and styling customization.
3. **Type-Safe Variant Management:** Uses `class-variance-authority` (`cva`) and `tailwind-merge` (`cn` utility) for strongly-typed prop variants (`variant="outline"`, `size="sm"`).

```tsx
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from "@/components/ui/dialog";
```

#### Development Velocity
- **Initial Setup:** Takes approximately 5–10 minutes: configuring Vite path aliases (`@/` mapping to `./src/`), running `npx shadcn@latest init`, and installing desired components with `npx shadcn@latest add button dialog badge card input label`.
- **Ongoing Velocity:** Once initialized, development speed is extremely fast. Pre-composed accessible components with modern typography and spacing allow the team to assemble complex views (task lists, scheduling tables, modal forms) without designing styles from scratch.
- **Zero Lock-In:** Because the component source files reside in `src/components/ui/`, developers can freely modify markup, add custom props, or tweak animation durations without fighting an external package maintainer.

#### Bundle Size & Performance Impact
- **JavaScript:** Highly modular. Unlike monolithic libraries, each component only pulls in its specific Radix primitive.
  - Dialog primitive (`@radix-ui/react-dialog`): ~12.9 kB gzipped.
  - Button, Badge, Card, Label: ~1–2 kB gzipped each.
  - Total JS runtime addition for the full task manager component set is **~12–18 kB gzipped**.
- **CSS:** Tailwind JIT compiles only the utility classes actually present in the code, adding **~4–8 kB gzipped**.
- **Combined Impact:** **~16–26 kB gzipped total**. Substantially lighter than React-Bootstrap (~60+ kB), with tree-shaking fully guaranteed by ES module bundling.

#### Developer Ergonomics & Learning Curve
- **First-Class TypeScript Experience:** Because components are written natively in TypeScript with `cva`, developers get full IDE auto-completion for variants, sizes, and ARIA attributes:
  ```tsx
  // Full TypeScript intellisense for all variants and HTML attributes
  <Badge variant="secondary" className="border-dashed">Unscheduled</Badge>
  ```
  This is especially beneficial for team members familiar with Angular's typed component inputs or strict TypeScript practices.
- **Learning Curve Considerations:**
  - The team must understand the Radix `asChild` composition pattern (using `@radix-ui/react-slot` to forward props to custom children).
  - The project tree will contain ~8–12 new files in `src/components/ui/`. Team members must understand that these are internal project files, not read-only node_modules.

#### Support for NFR-02 & Accessibility
- **WAI-ARIA Compliance:** Gold standard. Radix UI primitives are built specifically to satisfy WAI-ARIA authoring practices.
- **Dialog Focus Trapping (FR-TM-03):** The `Dialog` component automatically traps keyboard focus inside the modal, sets `aria-modal="true"`, dismisses on Escape or backdrop click, and restores focus to the triggering element upon close.
- **Native Icon Integration:** `shadcn/ui` comes pre-configured with **`lucide-react`** as its official icon system, providing clean, consistent SVGs for task status indicators (`Clock`, `CalendarCheck`, `CheckCircle2`, `Inbox`, `AlertTriangle`).
- **NFR-02 Implementation:** Creating multi-attribute status indicators with icons, distinct badge variants, text labels, and assistive screen reader text is native to the ecosystem.

---

## Detailed Evaluation Across Key Criteria

### 1. Development Velocity & Component Fit for MVP Requirements

The Smart Task Manager MVP requires specific interface elements:
- **Task Creation & Editing Form / Modal (FR-TM-01, FR-TM-02, FR-TM-04, FR-TM-05):** Inputs for Title, Description, positive Duration, Deadline, and Importance (Low/Med/High).
- **Delete Confirmation Prompt (FR-TM-03):** An accessible confirmation dialog that prevents accidental loss of tasks.
- **Availability Window Manager (FR-DP-01):** Form rows to add/remove daily time windows (`startTime` < `endTime`).
- **Daily Plan & Next Up Highlight (FR-DR-02, NFR-01):** Chronological task list with a glanceable "Next Up" task banner.
- **Status Indicators (NFR-02):** Clear differentiation among Scheduled, Unscheduled, and Completed tasks.

| Requirement Component | Bootstrap / React-Bootstrap | Tailwind + DaisyUI | shadcn/ui |
| :--- | :--- | :--- | :--- |
| **Delete Confirmation Dialog (FR-TM-03)** | `<Modal show={...} onHide={...}>` works well, but keyboard return focus must be manually verified. | Uses native `<dialog>` or checkbox toggle; requires `useRef` and manual modal management in React. | `<AlertDialog>` or `<Dialog>` provides out-of-the-box accessible focus trap, ESC closing, and return focus. |
| **Task Cards & Lists (FR-DR-01, 02)** | `<Card>` and `<ListGroup>` are quick, but styling responsive layouts and custom badges requires extra CSS classes. | `<div className="card bg-base-100 shadow">` is rapid, expressive, and easily customized with Tailwind utilities. | `<Card>`, `<CardHeader>`, `<CardContent>` provide clean, modern semantic structure with zero extra styling work. |
| **Availability Windows (FR-DP-01)** | Standard `<Form.Control type="time">` and `<Row>` / `<Col>` grid. Quick to implement. | Input with `.input .input-bordered`; simple flexbox rows with `flex gap-2`. Very fast. | `<Input type="time">` and `<Label>` with accessible `htmlFor` bindings. Type-safe and clean. |
| **"Next Up" Banner (NFR-01)** | `<Alert variant="info">` works, but looks like a generic system warning rather than an anti-procrastination hero card. | `.alert .alert-info` or custom styled card with badge. Great visual polish in seconds. | Composable `<Card>` with `border-primary` and prominent typography; looks modern and purposeful. |

**Verdict on Velocity:**
- **Tailwind + DaisyUI** provides the fastest "time-to-first-screen" due to zero setup overhead and rich pre-made semantic classes.
- **shadcn/ui** is only marginally behind on day 1 (due to the 5-minute CLI setup) but matches or exceeds DaisyUI's velocity on days 2–14 because complex components like accessible dialogs and dropdown menus are completely finished and require no custom ref handling.
- **React-Bootstrap** is fast for basic forms, but customizing modern dashboards leads to friction.

---

### 2. Bundle Size & Production Performance

In a client-side Single-Page Application persisting to `localStorage` (NFR-03), minimizing initial bundle transfer time directly improves Time-to-Interactive (TTI) and First Contentful Paint (FCP).

#### Size Comparison Summary (Production Build, Minified + Gzipped)

```
[Bootstrap + React-Bootstrap]  ============================== (~61 kB)
  - react-bootstrap JS: ~37.6 kB
  - bootstrap.min.css:  ~24.0 kB

[shadcn/ui + Radix + Tailwind] =========== (~22 kB)
  - Radix Dialog & Primitives: ~14.0 kB
  - lucide-react (purged icons): ~2.0 kB
  - Tailwind JIT CSS:           ~6.0 kB

[Tailwind CSS + DaisyUI]       ==== (~8 kB)
  - DaisyUI JS:                 0.0 kB
  - Tailwind + DaisyUI JIT CSS: ~8.0 kB
```

*Data Sources: Bundlephobia, Bootstrap CDN distribution stats, Tailwind JIT compiler benchmarks.*

**Verdict on Performance:**
- **Tailwind + DaisyUI** wins on raw size with 0 KB JavaScript overhead.
- **shadcn/ui** is very lean (~20–25 kB total gzipped payload), ensuring negligible load impact on any modern browser.
- **React-Bootstrap** is roughly 3x larger than shadcn/ui and 7x larger than DaisyUI, largely due to unpurged global CSS and Popper.js dependencies.

---

### 3. Developer Ergonomics & Team Learning Curve

#### Team Profile Context
- Team size: 3 students.
- Experience: 2 members know TypeScript; members have partial familiarity with React and Angular.
- Challenges: Angular developers are accustomed to strong type checking, template encapsulation, and declarative component inputs/outputs. One member is less experienced with the modern React ecosystem.

#### Comparative Ergonomic Analysis

1. **Bootstrap / React-Bootstrap:**
   - *Pros:* Universally recognized. Easiest for a student with minimal frontend experience to grasp immediately.
   - *Cons:* Prop types are somewhat generic. Heavy reliance on global classes hinders learning modern CSS architecture (flex/grid utilities, tokens).

2. **Tailwind CSS + DaisyUI:**
   - *Pros:* Removes the intimidation of memorizing hundreds of Tailwind utilities by providing obvious class names (`btn`, `btn-primary`, `badge`, `modal`).
   - *Cons:* Because classes are plain strings, TypeScript cannot catch typos in class names. Angular developers who expect typed component contracts may miss prop-driven interfaces unless the team manually wraps DaisyUI classes in custom React components.
   - React state management around native `<dialog>` elements (calling `.showModal()` via `ref`) can lead to subtle bugs when components re-render or unmount.

3. **shadcn/ui:**
   - *Pros:* Bridges the gap between React and Angular component paradigms. Components behave exactly like typed Angular or React components with props like `variant="destructive"` and `size="sm"`. VS Code provides instant autocompletion and prevents invalid prop passing at compile time.
   - *Cons:* The copy-paste architecture creates local files in `src/components/ui/`, which can initially confuse students who expect all dependencies to live invisibly in `node_modules`. Concepts like `@radix-ui/react-slot` (`asChild`) require a brief 10-minute explanation.

**Verdict on Ergonomics:**
- **shadcn/ui** provides the best long-term developer experience and type safety for a team where 2/3 know TypeScript.
- **DaisyUI** is the easiest to write without reading component documentation, but offers the weakest TypeScript integration.

---

### 4. Accessibility & Support for NFR-02 (Non-Color State Differentiation)

#### Requirement Definition (NFR-02)
> *"Scheduled, unscheduled, and completed states shall be distinguishable without relying on colour alone."*  
> *(Traceability: WCAG 2.1 / 2.2 Success Criterion 1.4.1 — Use of Color)*

To strictly satisfy NFR-02, the UI must provide **multi-modal cues** for every task state:
1. **Explicit Text Labels:** The text itself must identify the state (e.g., "Scheduled [09:00 - 10:30]", "Unscheduled", "Completed").
2. **Distinct Semantic Icons:** Visually distinguishable shapes (e.g., Clock/Calendar for Scheduled, Inbox/Dashed-Box for Unscheduled, Checkmark-Circle for Completed).
3. **Non-Color Visual Treatments:** Distinct typographical treatments (e.g., strikethrough for completed titles, dashed border for unscheduled, solid accent border for scheduled) and checkbox toggle states.
4. **Assistive Technology (Screen Readers):** Semantic ARIA attributes (`aria-checked`, `aria-label`, or `<span className="sr-only">`) so non-sighted users perceive status differences.

#### Implementation Comparison for NFR-02

##### Candidate 1: Bootstrap / React-Bootstrap
```tsx
// React-Bootstrap implementation of NFR-02
import Badge from 'react-bootstrap/Badge';
import { CalendarCheck, Inbox, CheckCircle2 } from 'lucide-react'; // Requires external install

function TaskStatusBadge({ status, startTime, endTime }: TaskStatusProps) {
  if (status === 'scheduled') {
    return (
      <Badge bg="primary" className="d-inline-flex align-items-center gap-1">
        <CalendarCheck size={14} aria-hidden="true" />
        <span>Scheduled ({startTime} – {endTime})</span>
        <span className="visually-hidden">Task is scheduled</span>
      </Badge>
    );
  }
  if (status === 'unscheduled') {
    return (
      <Badge bg="secondary" className="d-inline-flex align-items-center gap-1 border border-secondary">
        <Inbox size={14} aria-hidden="true" />
        <span>Unscheduled</span>
        <span className="visually-hidden">Task is unscheduled</span>
      </Badge>
    );
  }
  return (
    <Badge bg="success" className="d-inline-flex align-items-center gap-1">
      <CheckCircle2 size={14} aria-hidden="true" />
      <span>Completed</span>
      <span className="visually-hidden">Task is completed</span>
    </Badge>
  );
}
```

##### Candidate 2: Tailwind CSS + DaisyUI
```tsx
// Tailwind + DaisyUI implementation of NFR-02
import { CalendarCheck, Inbox, CheckCircle2 } from 'lucide-react'; // Requires external install

function TaskStatusBadge({ status, startTime, endTime }: TaskStatusProps) {
  if (status === 'scheduled') {
    return (
      <span className="badge badge-primary gap-1 font-medium border border-primary">
        <CalendarCheck className="w-3.5 h-3.5" aria-hidden="true" />
        <span>Scheduled ({startTime} – {endTime})</span>
        <span className="sr-only">State: Scheduled</span>
      </span>
    );
  }
  if (status === 'unscheduled') {
    return (
      <span className="badge badge-outline gap-1 font-medium border-dashed border-base-content/40 text-base-content/70">
        <Inbox className="w-3.5 h-3.5" aria-hidden="true" />
        <span>Unscheduled</span>
        <span className="sr-only">State: Unscheduled</span>
      </span>
    );
  }
  return (
    <span className="badge badge-success gap-1 font-medium">
      <CheckCircle2 className="w-3.5 h-3.5" aria-hidden="true" />
      <span>Completed</span>
      <span className="sr-only">State: Completed</span>
    </span>
  );
}
```

##### Candidate 3: `shadcn/ui` (Recommended)
```tsx
// shadcn/ui implementation of NFR-02
import { Badge } from "@/components/ui/badge";
import { CalendarCheck, Inbox, CheckCircle2 } from "lucide-react"; // Pre-bundled in shadcn ecosystem

export function TaskStatusBadge({ status, startTime, endTime }: TaskStatusProps) {
  switch (status) {
    case 'scheduled':
      return (
        <Badge variant="default" className="flex items-center gap-1.5 font-medium shadow-sm">
          <CalendarCheck className="h-3.5 w-3.5" aria-hidden="true" />
          <span>Scheduled ({startTime} – {endTime})</span>
          <span className="sr-only">Status: Scheduled</span>
        </Badge>
      );
    case 'unscheduled':
      return (
        <Badge 
          variant="outline" 
          className="flex items-center gap-1.5 font-medium border-dashed border-muted-foreground/50 text-muted-foreground"
        >
          <Inbox className="h-3.5 w-3.5" aria-hidden="true" />
          <span>Unscheduled</span>
          <span className="sr-only">Status: Unscheduled</span>
        </Badge>
      );
    case 'completed':
      return (
        <Badge variant="secondary" className="flex items-center gap-1.5 font-medium bg-muted text-muted-foreground">
          <CheckCircle2 className="h-3.5 w-3.5 text-foreground" aria-hidden="true" />
          <span>Completed</span>
          <span className="sr-only">Status: Completed</span>
        </Badge>
      );
  }
}
```

#### Task Card Complete Representation under NFR-02
Beyond badges, the task card itself must reflect the state through non-color styling:

```tsx
// Example of multi-sensory state rendering in shadcn/ui:
export function TaskItemCard({ task }: { task: Task }) {
  const isCompleted = task.status === 'completed';
  const isScheduled = task.status === 'scheduled';

  return (
    <div 
      className={cn(
        "flex items-center justify-between p-4 rounded-lg border transition-all",
        // Distinct borders and background treatments
        isCompleted && "bg-muted/30 border-muted opacity-75",
        isScheduled && "bg-card border-l-4 border-l-primary shadow-sm",
        task.status === 'unscheduled' && "bg-card border-dashed border-border"
      )}
    >
      <div className="flex items-center gap-3">
        {/* Semantic native accessible checkbox */}
        <input
          type="checkbox"
          checked={isCompleted}
          onChange={() => toggleComplete(task.id)}
          aria-label={`Mark "${task.title}" as ${isCompleted ? 'incomplete' : 'complete'}`}
          className="h-4 w-4 rounded border-primary"
        />
        
        <div>
          {/* Strikethrough non-color indicator */}
          <h4 className={cn("text-sm font-semibold", isCompleted && "line-through text-muted-foreground")}>
            {task.title}
          </h4>
          {task.description && (
            <p className={cn("text-xs text-muted-foreground", isCompleted && "line-through")}>
              {task.description}
            </p>
          )}
        </div>
      </div>

      <div className="flex items-center gap-3">
        <TaskStatusBadge 
          status={task.status} 
          startTime={task.scheduledStart} 
          endTime={task.scheduledEnd} 
        />
      </div>
    </div>
  );
}
```

**Verdict on Accessibility & NFR-02:**
- **shadcn/ui** is the winner. It bundles Lucide icons, features strongly-typed badge variants, and pairs seamlessly with Radix UI primitives that handle focus traps and ARIA states for the entire application.
- **DaisyUI** makes styling badges easy, but requires third-party icon setup and lacks built-in keyboard focus management for modals.
- **React-Bootstrap** requires external icon setup and verbose helper classes for screen readers.

---

## Synthesis & Trade-Off Analysis

### Trade-Off 1: Developer Experience vs. Architecture Ownership
- **React-Bootstrap** offers the lowest cognitive barrier at initialization, but acts as a "black box" where styling changes require fighting CSS specificity.
- **DaisyUI** abstracts Tailwind into convenient classes, making prototyping extremely fast, but keeps component logic in raw HTML strings without TypeScript safety.
- **shadcn/ui** requires a one-time 5-minute initialization hurdle, but gives the team 100% source code ownership, complete customization freedom, and total TypeScript prop safety.

### Trade-Off 2: Accessible Modals (FR-TM-03)
- The Delete Confirmation prompt (FR-TM-03) is a core requirement.
- In **shadcn/ui**, the `<AlertDialog>` component provides a pre-wired accessible modal with autofocus on the Cancel button, Escape dismissal, backdrop click prevention, and focus restoration to the trigger button.
- In **DaisyUI**, building an accessible confirmation modal in React requires manual hook wiring around the HTML5 `<dialog>` API.
- In **React-Bootstrap**, `<Modal>` handles basic dialog state, but customization is constrained.

### Trade-Off 3: Long-Term Educational Value for Master's Students
- This is a Master's-level software engineering project. Using **shadcn/ui** with **Radix UI** and **Tailwind CSS** aligns directly with current modern industry standards (2024–2026), providing team members with highly relevant portfolio skills in headless UI design and TypeScript component architectures.

---

## Team Recommendation

### Recommendation: Adopt `shadcn/ui` with `lucide-react`

We recommend that the team select **`shadcn/ui`** as the primary component library for the Smart Task Manager MVP.

#### Why This Choice Best Fits the Team & Project:
1. **TypeScript Alignment:** With 2 out of 3 team members knowing TypeScript and coming from React/Angular backgrounds, strongly-typed components with prop autocompletion (`<Button variant="outline">`, `<Badge variant="secondary">`) provide an intuitive, error-free developer experience.
2. **NFR-02 Compliance:** Seamless integration with `lucide-react` makes combining icons, text labels, and structural styling (borders, strikethrough) natural, satisfying WCAG 2.1 SC 1.4.1 without boilerplate.
3. **Accessibility Without Effort:** Radix UI primitives handle keyboard focus trapping, screen reader announcements, and modal closing for the FR-TM-03 delete confirmation dialog out of the box.
4. **Lightweight Bundle:** A total gzipped payload of ~20 kB preserves snappy SPA load times.

---

## Actionable Implementation Roadmap (Vite + React + TS)

To set up the recommended stack in under 10 minutes:

### Step 1: Install Tailwind CSS & Configure Vite Path Aliases
In `vite.config.ts`:
```typescript
import path from 'path';
import react from '@vitejs/plugin-react';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

In `tsconfig.json` (add inside `compilerOptions`):
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Step 2: Initialize shadcn/ui
Run in the terminal:
```bash
npx shadcn@latest init
```
*Prompts: Select TypeScript, Default style, Slate or Neutral base color, and confirm `@/components` path.*

### Step 3: Add Essential MVP Components
```bash
npx shadcn@latest add button badge card dialog alert-dialog input label
```

### Step 4: Install Lucide Icons
```bash
npm install lucide-react
```

With these 4 steps completed, the team has a fully typed, accessible, and high-performance component foundation ready for building the task list, daily schedule view, and modal dialogs.

---

## Primary Sources & References

1. **W3C Web Content Accessibility Guidelines (WCAG) 2.1 / 2.2**
   - *Success Criterion 1.4.1: Use of Color* — [W3C WCAG 2.1 Guideline 1.4.1](https://www.w3.org/WAI/WCAG21/Understanding/use-of-color.html).
   - *Success Criterion 2.1.1: Keyboard Navigation* — [W3C WCAG 2.1 Guideline 2.1.1](https://www.w3.org/WAI/WCAG21/Understanding/keyboard.html).
   - *Success Criterion 2.4.3: Focus Order* — [W3C WCAG 2.1 Guideline 2.4.3](https://www.w3.org/WAI/WCAG21/Understanding/focus-order.html).
2. **shadcn/ui Documentation & Architecture**
   - Official Documentation & Philosophy: [ui.shadcn.com](https://ui.shadcn.com/docs).
   - Radix UI Primitives & Accessibility Standards: [radix-ui.com/primitives](https://www.radix-ui.com/primitives).
3. **Tailwind CSS & DaisyUI**
   - DaisyUI Official Component Documentation: [daisyui.com](https://daisyui.com/docs/use/).
   - Tailwind CSS JIT Optimization & Purging: [tailwindcss.com/docs/optimizing-for-production](https://tailwindcss.com/docs/optimizing-for-production).
4. **React-Bootstrap Documentation & Bundle Metrics**
   - React-Bootstrap Official Docs: [react-bootstrap.netlify.app](https://react-bootstrap.netlify.app/).
   - Bundlephobia Package Analysis (`react-bootstrap` v2.10): [bundlephobia.com/package/react-bootstrap](https://bundlephobia.com/package/react-bootstrap).
   - Bootstrap 5 Distribution CDN Stats: [jsdelivr.com/package/npm/bootstrap](https://www.jsdelivr.com/package/npm/bootstrap).
5. **Project Specifications**
   - Requirements Document: [requirements.md](../requirements.md) (FR-TM-01–05, FR-DP-01–05, FR-DR-01–04, NFR-01, NFR-02).
   - MVP Scope Specification: [mvp.md](../mvp.md).
