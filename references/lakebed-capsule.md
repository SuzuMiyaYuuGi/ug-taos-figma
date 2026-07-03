# Lakebed capsule reference

## Porting plain HTML screens into a capsule
Mode B first builds one plain HTML+CSS file per screen from the pipeline's analysis (design tokens as CSS variables, the user's chosen width as a fixed centered container, `<a href>` navigation). To port that HTML into the capsule: each screen's body becomes a Preact component; `class` → `className`; every CSS rule becomes Tailwind arbitrary-value classes on the element (`background:#1e293b` → `bg-[#1e293b]`, `font-size:13px` → `text-[13px]`) because capsules have **no CSS files**; `<a href="dashboard.html">` → `<Link to="/dashboard">` and `index.html` → route `/`; `:hover`/`:focus` and media queries port as Tailwind variants (`hover:bg-[#...]`, `md:w-[...]`); Google Fonts links are dropped (no `<head>` control) — closest system stack instead, flag the gap.

Captured from https://docs.lakebed.dev/llms-full.txt. Enough to scaffold, build, and deploy a static multi-screen prototype without re-fetching. For anything beyond this, fetch `/llms-full.txt`.

## File structure
```
my-app/
├── server/index.ts      # required
├── client/index.tsx     # required
├── shared/              # optional: pure TS helpers/types (no DOM, no Node, no runtime imports)
├── .env.lakebed.server  # optional: server-only secrets
└── lakebed.json         # created after first deploy
```
Constraints (v0): one server entry, one client entry, no arbitrary npm imports, no Node built-ins, **Tailwind classes only (no CSS files/modules/config)**, no file storage.

## Minimal server for a static prototype
A pure prototype needs no backend. Empty capsule:
```ts
import { capsule } from "lakebed/server";

export default capsule({
  schema: {},
  queries: {},
  mutations: {},
});
```

## Client: routing (client/index.tsx)
```tsx
import { Link, Route, Router, Routes, useNavigate, useParams } from "lakebed/client";

function Home() {
  const navigate = useNavigate();
  return (
    <main className="min-h-screen bg-[#0b0b0f] p-6 text-white">
      <h1 className="text-[22px] font-semibold">Home</h1>
      <Link to="/details/1" className="text-[#4f9dff] underline">Go to details</Link>
      <button type="button" onClick={() => navigate("/settings")}>Settings</button>
    </main>
  );
}

function Details() {
  const { id } = useParams<{ id: string }>();
  return <main className="p-6">Item {id}</main>;
}

export function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/details/:id" element={<Details />} />
        <Route path="/settings" element={<main className="p-6">Settings</main>} />
        <Route path="*" element={<main className="p-6">Not found</main>} />
      </Routes>
    </Router>
  );
}
```
Routing API: `<Router>`, `<Routes>`, `<Route path element>`, `<Link to>`, `useNavigate()` → `navigate("/path")`, `useParams()`. Paths are app-relative.

## Styling: Tailwind + arbitrary values (fidelity)
No CSS files. Hit exact design values with bracket syntax:
- Color: `bg-[#1e293b]`, `text-[#4f9dff]`, `border-[#2a2a35]`
- Size: `text-[13px]`, `w-[342px]`, `h-[48px]`, `p-[7px]`, `gap-[12px]`, `rounded-[10px]`
- Line height / tracking: `leading-[1.35]`, `tracking-[-0.01em]`
- Font family: `font-[Inter]` sets the family, **but the font must actually be available** — no CSS/config means custom web fonts may not load. Match the closest system/Tailwind stack and flag exact-font gaps.

## Images / icons
File storage is not in v0. Options:
- Icons → inline `<svg>` in JSX.
- Real photos → external `src="https://..."` URLs (user-provided).

## CLI
```bash
# create (-y skips the npx install prompt that hangs non-interactive shells)
npx -y lakebed new my-app --template todo
cd my-app

# develop (default port 3000)
npx lakebed dev

# deploy — anonymous is fine for a static prototype; returns a public URL
npx lakebed deploy

# only if a custom subdomain is wanted:
npx lakebed auth login
npx lakebed claim
npx lakebed domains add my-app.lakebed.app
```
Anonymous deploys disable outbound `fetch` and server env — irrelevant for a static prototype, so no claim needed for a basic public URL.

## Graduating to a real app (beyond prototype)
If forms must persist or data must be live, add to `server/index.ts`:
```ts
import { capsule, query, mutation, table, string } from "lakebed/server";

export default capsule({
  schema: { todos: table({ body: string(), ownerId: string() }) },
  queries: {
    todos: query((ctx) =>
      ctx.db.todos.where("ownerId", ctx.auth.userId).orderBy("createdAt", "desc").all()),
  },
  mutations: {
    addTodo: mutation((ctx, body: string) => {
      if (!body.trim()) return;
      ctx.db.todos.insert({ body: body.trim(), ownerId: ctx.auth.userId });
    }),
  },
});
```
Client reads with `useQuery<T>("todos")` and writes with `useMutation<[string], void>("addTodo")` from `lakebed/client`. Always filter by `ctx.auth.userId`; re-check ownership before update/delete.
