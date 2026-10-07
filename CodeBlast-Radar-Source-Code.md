# CodeBlast Radar — Full Source Code

## package.json

```json
{"name":"codeblast-radar","private":true,"scripts":{"dev":"next dev","build":"next build","start":"next start","test":"vitest run"},
"dependencies":{"next":"^14.2.0","react":"^18.3.0","react-dom":"^18.3.0","reactflow":"^11.11.0","typescript":"^5.4.0"},
"devDependencies":{"@types/node":"^20","@types/react":"^18","vitest":"^1.6.0"}}

```

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es2020",
    "lib": [
      "dom",
      "esnext"
    ],
    "jsx": "preserve",
    "module": "esnext",
    "moduleResolution": "bundler",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "noEmit": true,
    "incremental": true,
    "allowJs": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "paths": {
      "@/*": [
        "./*"
      ]
    },
    "plugins": [
      {
        "name": "next"
      }
    ]
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts"
  ],
  "exclude": [
    "node_modules",
    "demo"
  ]
}

```

## next.config.js

```js
module.exports = { reactStrictMode: true };

```

## vitest.config.ts

```ts
import { defineConfig } from "vitest/config";
export default defineConfig({ test: { include: ["tests/**/*.test.ts"] } });

```

## .env.example

```bash
OPENAI_API_KEY=
GITHUB_TOKEN=

```

## app/layout.tsx

```tsx
import "./globals.css";
export const metadata = { title: "CodeBlast Radar", description: "See what your code change can break — before it breaks production." };
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html lang="en"><body>{children}</body></html>;
}

```

## app/globals.css

```css
:root{--bg:#0B1020;--card:#111827;--primary:#6366F1;--ok:#22C55E;--warn:#F59E0B;--danger:#EF4444;--crit:#DC2626;--text:#E5E7EB;--mute:#9CA3AF}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);font-family:system-ui,sans-serif}
main{max-width:1200px;margin:0 auto;padding:24px}h1{font-size:40px;margin:0 0 8px}h2{font-size:16px;margin:0 0 12px}
.card{background:var(--card);border:1px solid #1f2937;border-radius:14px;padding:18px}
.grid{display:grid;gap:16px;grid-template-columns:repeat(auto-fit,minmax(300px,1fr))}.row{display:flex;gap:12px;flex-wrap:wrap;align-items:center}
button{background:var(--primary);color:#fff;border:0;border-radius:10px;padding:10px 18px;font-size:15px;cursor:pointer}button.ghost{background:transparent;border:1px solid #374151}
button:focus-visible,input:focus-visible{outline:2px solid #fff;outline-offset:2px}input{flex:1;min-width:220px;background:#0B1020;border:1px solid #374151;color:var(--text);padding:10px;border-radius:10px}
.badge{background:#312e81;border-radius:999px;padding:3px 10px;font-size:12px}.mute{color:var(--mute)}.err{color:#FCA5A5}
.graph{height:420px}ul{padding-left:18px;margin:0}li{margin:4px 0}
@media print{body{background:#fff;color:#000}.noprint{display:none}.card{border:1px solid #999;background:#fff}}

```

## app/page.tsx

```tsx
"use client";
import { useState } from "react";
import ReactFlow, { Background, Controls, Edge, Node } from "reactflow";
import "reactflow/dist/style.css";

const levelColor: Record<string, string> = { LOW: "#22C55E", MEDIUM: "#F59E0B", HIGH: "#EF4444", CRITICAL: "#DC2626" };
const distColor = (d: number | null) => (d === null ? "#22C55E" : d === 1 ? "#F59E0B" : d === 2 ? "#F97316" : "#DC2626");

export default function Home() {
  const [data, setData] = useState<any>(null);
  const [err, setErr] = useState("");
  const [loading, setLoading] = useState(false);
  const [url, setUrl] = useState("");
  const [sel, setSel] = useState<any>(null);

  async function run(repoUrl?: string, changedFile?: string) {
    setLoading(true); setErr(""); setSel(null);
    const res = await fetch("/api/analyze", { method: "POST", body: JSON.stringify({ repoUrl, changedFile }) }).catch(() => null);
    const json = res ? await res.json() : { error: "Network failure — try the demo." };
    setLoading(false);
    if (json.error) setErr(json.error); else setData(json);
  }

  const nodes: Node[] = data ? data.nodes.map((n: any, i: number) => ({
    id: n.id, data: { label: n.name },
    position: { x: (n.distance ?? 3) * 230 - 230, y: i * 70 },
    style: { background: "#111827", color: "#E5E7EB", border: `2px solid ${n.changed ? "#3B82F6" : distColor(n.distance)}`, boxShadow: n.changed ? "0 0 14px #3B82F6" : "none", borderRadius: 10, fontSize: 12 },
  })) : [];
  const edges: Edge[] = data ? data.edges.map((e: any) => ({ id: e.source + e.target, source: e.source, target: e.target, animated: true })) : [];

  return (
    <main>
      <h1>CodeBlast Radar</h1>
      <p className="mute">See what your code change can break — before it breaks production.</p>
      <div className="row noprint" style={{ margin: "16px 0" }}>
        <input aria-label="GitHub repository URL" placeholder="https://github.com/owner/repo" value={url} onChange={(e) => setUrl(e.target.value)} />
        <button onClick={() => run(url)} disabled={loading}>Analyze repository</button>
        <button className="ghost" onClick={() => run()} disabled={loading}>Try demo</button>
      </div>
      {loading && <p>Analyzing…</p>}
      {err && <p className="err" role="alert">{err}</p>}
      {data && (
        <div className="grid">
          <div className="card">
            {data.demo ? <><span className="badge">DEMO MODE</span> <span className="mute">Using the bundled ShopFlow repository</span></> : <span className="mute">{data.repo} · {data.branch} @ {data.commit}</span>}
            <h2 style={{ marginTop: 12 }}>Risk indicator</h2>
            <div style={{ fontSize: 56, fontWeight: 700, color: levelColor[data.level] }}>{data.score}/100</div>
            <strong style={{ color: levelColor[data.level] }}>{data.level} RISK</strong>
            <p>{data.affected.length} potentially affected files</p>
            <details><summary>Why?</summary><ul>{data.reasons.map((r: any) => <li key={r.text}>+{r.points} {r.text}</li>)}</ul></details>
          </div>
          <div className="card">
            <h2>Overview</h2>
            <p>{data.stats.files} files · {data.stats.functions} functions · {data.stats.dependencies} import links</p>
            <p>Changed: <code>{data.changed}</code> ({data.changedFunctions.join(", ")})</p>
            {!data.demo && <label className="noprint">Analyze change in: <select value={data.changed} onChange={(e) => run(url, e.target.value)} style={{ maxWidth: 260 }}>{data.files.map((f: string) => <option key={f}>{f}</option>)}</select></label>}
            {data.truncated && <p className="mute">Large repository: only the first 150 source files were analyzed.</p>}
            <p>Direct: {data.direct.map((f: string) => f.split("/").pop()).join(", ")}</p>
            <p>Indirect: {data.indirect.map((f: string) => f.split("/").pop()).join(", ")}</p>
            <p>Path: {data.criticalPath.join(" → ")}</p>
          </div>
          <div className="card">
            <h2>Recommended tests</h2>
            <ul>{data.tests.map((t: string) => <li key={t}>{t}</li>)}</ul>
          </div>
          <div className="card" style={{ gridColumn: "1 / -1" }}>
            <h2>Dependency graph</h2>
            <div className="graph">
              <ReactFlow nodes={nodes} edges={edges} fitView onNodeClick={(_, n) => setSel(data.nodes.find((x: any) => x.id === n.id))}>
                <Background /><Controls />
              </ReactFlow>
            </div>
            <p className="mute">Blue = changed · orange/red = farther dependents · yellow = direct · green = unaffected</p>
            {sel && <div><strong>{sel.name}</strong><p>Distance: {sel.distance ?? "n/a"} — {sel.relation}</p></div>}
          </div>
          <div className="card" style={{ gridColumn: "1 / -1" }}>
            <h2>Explanation (rule-based)</h2><p>{data.explanation}</p>
            <button className="noprint" onClick={() => window.print()}>Download report (print / save as PDF)</button>
          </div>
        </div>
      )}
    </main>
  );
}

```

## app/api/analyze/route.ts

```ts
import path from "path";
import { analyze, defaultConfig, scanDir } from "@/lib/analyze";
import { fetchRepo, RepoError } from "@/lib/github";
export const dynamic = "force-dynamic";
export async function POST(req: Request) {
  try {
    const { repoUrl, commit, changedFile } = await req.json().catch(() => ({}));
    if (!repoUrl) {
      const sources = scanDir(path.join(process.cwd(), "demo/repository"));
      return Response.json(analyze(sources, "backend/payment/PaymentService.ts"));
    }
    const repo = await fetchRepo(repoUrl, commit);
    const nonTest = Object.keys(repo.sources).filter((f) => !/(^|\/)tests?\/|test_|\.(test|spec)\./.test(f));
    const changed = changedFile && repo.sources[changedFile] ? changedFile : repo.touched.find((f) => nonTest.includes(f)) ?? nonTest[0];
    if (!changed) throw new RepoError("No non-test source files to analyze.");
    const result = analyze(repo.sources, changed, defaultConfig, { repo: repo.name, demo: false, branch: repo.branch, commit: repo.commit });
    return Response.json({ ...result, touched: repo.touched, truncated: repo.truncated });
  } catch (e) {
    if (e instanceof RepoError) return Response.json({ error: e.message }, { status: e.status });
    return Response.json({ error: e instanceof Error ? e.message : "Analysis failed" }, { status: 500 });
  }
}

```

## lib/analyze.ts

```ts
import ts from "typescript";
import fs from "fs";
import path from "path";

export type Level = "LOW" | "MEDIUM" | "HIGH" | "CRITICAL";
export interface FileInfo { file: string; functions: string[]; imports: string[]; isTest: boolean }
export interface Config { weights: { critical: number; db: number; api: number; direct: number; indirect: number; noTests: number }; criticalPattern: RegExp }
export const defaultConfig: Config = {
  weights: { critical: 30, db: 15, api: 10, direct: 4, indirect: 3, noTests: 10 },
  criticalPattern: /payment|auth|security/i,
};

export function validateGithubUrl(url: string): boolean {
  return /^https:\/\/github\.com\/[\w.-]+\/[\w.-]+\/?$/.test(url.trim());
}

/** Parse one source file with the TypeScript compiler API (no code is executed). */
export function parseFile(file: string, source: string): FileInfo {
  if (file.endsWith(".py")) {
    const imports: string[] = [], functions: string[] = [];
    for (const l of source.split("\n")) {
      const from = l.match(/^\s*from\s+([.\w]+)\s+import\s+(.+)/), imp = l.match(/^\s*import\s+([\w., ]+)/), def = l.match(/^def\s+(\w+)/);
      if (from) { imports.push(from[1]); if (from[1].endsWith(".")) from[2].split(",").forEach((n) => imports.push(from[1] + n.trim().split(" ")[0])); }
      else if (imp) imp[1].split(",").forEach((m) => imports.push(m.trim().split(" ")[0]));
      if (def) functions.push(def[1]);
    }
    return { file, functions, imports, isTest: /(^|\/)(tests?\/|test_)/.test(file) };
  }
  const sf = ts.createSourceFile(file, source, ts.ScriptTarget.Latest, true, file.endsWith("x") ? ts.ScriptKind.TSX : ts.ScriptKind.TS);
  const functions: string[] = [], imports: string[] = [];
  sf.forEachChild((n) => {
    if (ts.isImportDeclaration(n) && ts.isStringLiteral(n.moduleSpecifier)) imports.push(n.moduleSpecifier.text);
    if (ts.isFunctionDeclaration(n) && n.name) functions.push(n.name.text);
  });
  return { file, functions, imports, isTest: /(^|\/)tests?\/|\.(test|spec)\.[jt]sx?$/.test(file) };
}

export function scanDir(root: string): Record<string, string> {
  const out: Record<string, string> = {};
  const walk = (d: string) => {
    for (const e of fs.readdirSync(d, { withFileTypes: true })) {
      if (["node_modules", ".git", "dist", "build", "coverage"].includes(e.name)) continue;
      const p = path.join(d, e.name);
      if (e.isDirectory()) walk(p);
      else if (/\.(tsx?|jsx?)$/.test(e.name) && fs.statSync(p).size < 200_000)
        out[path.relative(root, p).split(path.sep).join("/")] = fs.readFileSync(p, "utf8");
    }
  };
  walk(root);
  return out;
}

/** Resolve relative imports to file ids; edges are importer -> imported. */
export function buildGraph(files: FileInfo[]) {
  const ids = new Set(files.map((f) => f.file));
  const edges: { source: string; target: string; type: "IMPORT" }[] = [];
  for (const f of files)
    for (const imp of f.imports) {
      if (f.file.endsWith(".py")) {
        const dots = imp.match(/^\.*/)![0].length, mod = imp.slice(dots).replace(/\./g, "/");
        const dir = dots ? path.posix.join(path.posix.dirname(f.file), ...Array(dots - 1).fill("..")) : "";
        const cands = [path.posix.join(dir, mod) , mod].filter(Boolean);
        const hit = [...ids].find((id) => id !== f.file && cands.some((c) => id === c + ".py" || id === c + "/__init__.py" || (!dots && id.endsWith("/" + mod + ".py"))));
        if (hit) edges.push({ source: f.file, target: hit, type: "IMPORT" });
        continue;
      }
      if (!imp.startsWith(".")) continue;
      const base = path.posix.normalize(path.posix.join(path.posix.dirname(f.file), imp));
      const hit = [".ts", ".tsx", ".js", ".jsx"].map((x) => base + x).find((c) => ids.has(c));
      if (hit) edges.push({ source: f.file, target: hit, type: "IMPORT" });
    }
  return edges;
}

/** BFS distances. dependents = files that import the changed file; dependencies = files it imports. */
export function traverse(edges: { source: string; target: string }[], changed: string) {
  const bfs = (next: (n: string) => string[]) => {
    const dist = new Map<string, number>([[changed, 0]]);
    const q = [changed];
    while (q.length) {
      const n = q.shift()!;
      for (const m of next(n)) if (!dist.has(m)) { dist.set(m, dist.get(n)! + 1); q.push(m); }
    }
    dist.delete(changed);
    return dist;
  };
  return {
    dependents: bfs((n) => edges.filter((e) => e.target === n).map((e) => e.source)),
    dependencies: bfs((n) => edges.filter((e) => e.source === n).map((e) => e.target)),
  };
}

export function classify(score: number): Level {
  return score <= 25 ? "LOW" : score <= 50 ? "MEDIUM" : score <= 75 ? "HIGH" : "CRITICAL";
}

export function scoreRisk(changed: string, affected: string[], direct: string[], hasTests: boolean, cfg = defaultConfig) {
  const w = cfg.weights, reasons: { points: number; text: string }[] = [];
  const add = (points: number, text: string) => points > 0 && reasons.push({ points, text });
  if (cfg.criticalPattern.test(changed)) add(w.critical, "Critical logic (payment/auth/security) changed");
  if (affected.some((f) => /database|repository/i.test(f))) add(w.db, "Database layer involved");
  if (affected.some((f) => /controller/i.test(f))) add(w.api, "Public API/controller depends on the change");
  add(direct.length * w.direct, `${direct.length} direct neighbours (imports or is imported)`);
  add((affected.length - direct.length) * w.indirect, `${affected.length - direct.length} indirect dependents`);
  if (!hasTests) add(w.noTests, "No test file imports the changed module");
  const score = Math.min(100, reasons.reduce((s, r) => s + r.points, 0));
  return { score, level: classify(score), reasons };
}

export function recommendTests(changed: string, affected: string[]): string[] {
  const all = [changed, ...affected].join(" ");
  const t: string[] = [];
  if (/payment/i.test(all)) t.push("Successful payment", "Failed payment", "Duplicate payment", "Payment timeout handling");
  if (/auth|login/i.test(all)) t.push("Valid login", "Invalid password", "Expired token", "Unauthorized request");
  if (/database|repository/i.test(all)) t.push("Transaction rollback", "Invalid data", "Concurrent operation");
  if (/order/i.test(all)) t.push("Order creation after payment");
  return t.length ? t : ["Run the existing test suite for affected files"];
}

export function explain(a: { changed: string; level: Level; score: number; affected: string[]; reasons: { text: string }[]; tests: string[] }) {
  return `${a.changed} was changed. Based on dependency analysis, ${a.affected.length} file(s) are potentially affected (${a.affected.join(", ") || "none"}). ` +
    `The estimated risk indicator is ${a.level} (${a.score}/100) because: ${a.reasons.map((r) => r.text.toLowerCase()).join("; ")}. ` +
    `Consider testing: ${a.tests.join(", ")}. This is a static, dependency-based estimate, not a guarantee of bugs.`;
}

export function analyze(sources: Record<string, string>, changed: string, cfg = defaultConfig, meta: { repo?: string; demo?: boolean; branch?: string; commit?: string } = {}) {
  const files = Object.entries(sources).map(([f, s]) => parseFile(f, s));
  const edges = buildGraph(files);
  if (!files.some((f) => f.file === changed)) throw new Error(`Changed file not found: ${changed}`);
  const { dependents, dependencies } = traverse(edges, changed);
  const dist = new Map<string, number>();
  for (const m of [dependents, dependencies]) for (const [k, v] of m) dist.set(k, Math.min(v, dist.get(k) ?? 99));
  const testFiles = files.filter((f) => f.isTest).map((f) => f.file);
  const affected = [...dist.keys()].filter((f) => !testFiles.includes(f));
  const direct = affected.filter((f) => dist.get(f) === 1);
  const hasTests = edges.some((e) => e.target === changed && testFiles.includes(e.source));
  const risk = scoreRisk(changed, affected, direct, hasTests, cfg);
  const tests = recommendTests(changed, affected);
  const nodes = files.map((f) => ({
    id: f.file, name: path.posix.basename(f.file), functions: f.functions, changed: f.file === changed,
    distance: f.file === changed ? 0 : dist.get(f.file) ?? null,
    relation: f.file === changed ? "Changed file" : dependents.has(f.file) ? `Depends on the changed file (distance ${dependents.get(f.file)})` : dependencies.has(f.file) ? `Used by the changed file (distance ${dependencies.get(f.file)})` : "Not connected to the change",
  }));
  const criticalPath = [...dependents.entries()].sort((a, b) => b[1] - a[1]).map(([f]) => path.posix.basename(f)).reverse().concat(path.posix.basename(changed), ...[...dependencies.keys()].map((f) => path.posix.basename(f)));
  return {
    repo: meta.repo ?? "ShopFlow", demo: meta.demo ?? true, branch: meta.branch ?? "demo", commit: meta.commit ?? "", files: files.map((f) => f.file), changed, changedFunctions: files.find((f) => f.file === changed)!.functions,
    stats: { files: files.length, functions: files.reduce((s, f) => s + f.functions.length, 0), dependencies: edges.length },
    nodes, edges, affected, direct, indirect: affected.filter((f) => !direct.includes(f)),
    ...risk, tests, criticalPath, explanation: explain({ changed, ...risk, affected, tests }), analyzedAt: new Date().toISOString(),
  };
}

```

## lib/github.ts

```ts
export class RepoError extends Error { constructor(msg: string, public status = 400) { super(msg); } }
const SRC = /\.(tsx?|jsx?|py)$/;
const SKIP = /(^|\/)(node_modules|dist|build|coverage|vendor|\.git|venv|__pycache__)\//;
const MAX_FILES = 150, MAX_BYTES = 100_000;

export function parseRepo(url: string) {
  const m = url.trim().match(/^https:\/\/github\.com\/([\w.-]+)\/([\w.-]+?)(?:\.git)?\/?$/);
  if (!m) throw new RepoError("That doesn't look like a GitHub repository URL (https://github.com/owner/repo).");
  return { owner: m[1], repo: m[2] };
}

async function gh(path: string) {
  const token = process.env.GITHUB_TOKEN;
  const res = await fetch(`https://api.github.com${path}`, {
    headers: { Accept: "application/vnd.github+json", ...(token ? { Authorization: `Bearer ${token}` } : {}) },
  }).catch(() => { throw new RepoError("Could not reach GitHub. Check your internet connection.", 502); });
  if (res.status === 404) throw new RepoError("Repository or commit not found. It may be private — private repositories need a GITHUB_TOKEN.", 404);
  if (res.status === 403 || res.status === 429) throw new RepoError("GitHub rate limit reached. Add a GITHUB_TOKEN to .env.local, or wait a few minutes.", 429);
  if (!res.ok) throw new RepoError(`GitHub API error (${res.status}).`, 502);
  return res.json();
}

export async function fetchRepo(url: string, commit?: string) {
  const { owner, repo } = parseRepo(url);
  const info = await gh(`/repos/${owner}/${repo}`);
  const ref = commit || info.default_branch;
  const c = await gh(`/repos/${owner}/${repo}/commits/${ref}`);
  const tree = await gh(`/repos/${owner}/${repo}/git/trees/${c.commit.tree.sha}?recursive=1`);
  const paths: string[] = tree.tree
    .filter((t: any) => t.type === "blob" && SRC.test(t.path) && !SKIP.test(t.path) && (t.size ?? 0) < MAX_BYTES)
    .map((t: any) => t.path).slice(0, MAX_FILES);
  if (!paths.length) throw new RepoError("No supported source files found. CodeBlast Radar currently reads JavaScript, TypeScript and Python.");
  const sources: Record<string, string> = {};
  await Promise.all(paths.map(async (p) => {
    const r = await fetch(`https://raw.githubusercontent.com/${owner}/${repo}/${c.sha}/${p.split("/").map(encodeURIComponent).join("/")}`).catch(() => null);
    if (r?.ok) sources[p] = await r.text();
  }));
  const touched: string[] = (c.files ?? []).map((f: any) => f.filename).filter((f: string) => f in sources);
  return { name: `${owner}/${repo}`, branch: info.default_branch, commit: c.sha.slice(0, 7), sources, touched, truncated: tree.truncated || paths.length === MAX_FILES };
}

```

## tests/analyze.test.ts

```ts
import { describe, it, expect } from "vitest";
import path from "path";
import { analyze, classify, scanDir, traverse, validateGithubUrl, parseFile } from "../lib/analyze";

describe("core", () => {
  it("validates URLs", () => {
    expect(validateGithubUrl("https://github.com/a/b")).toBe(true);
    expect(validateGithubUrl("http://evil.com/a/b")).toBe(false);
  });
  it("A -> B -> C distances", () => {
    const e = [{ source: "B", target: "A" }, { source: "C", target: "B" }];
    const { dependents } = traverse(e, "A");
    expect(dependents.get("B")).toBe(1);
    expect(dependents.get("C")).toBe(2);
  });
  it("classifies", () => {
    expect([0, 26, 51, 76].map(classify)).toEqual(["LOW", "MEDIUM", "HIGH", "CRITICAL"]);
  });
  it("parses imports and functions", () => {
    const f = parseFile("a.ts", 'import {x} from "./b"; export function go(){}');
    expect(f.imports).toEqual(["./b"]); expect(f.functions).toEqual(["go"]);
  });
  it("demo analysis", () => {
    const r = analyze(scanDir(path.join(__dirname, "../demo/repository")), "backend/payment/PaymentService.ts");
    expect(r.affected.length).toBe(5);
    expect(r.direct.length).toBe(3);
    expect(r.indirect.length).toBe(2);
    expect(r.level).toBe("HIGH");
  });
});

import { parseRepo } from "../lib/github";
describe("live repo support", () => {
  it("parses github urls incl. .git suffix", () => {
    expect(parseRepo("https://github.com/leelaabja/bank-management-system-5.git")).toEqual({ owner: "leelaabja", repo: "bank-management-system-5" });
    expect(() => parseRepo("https://evil.com/a/b")).toThrow();
  });
  it("links python imports", () => {
    const r = analyze({ "app/main.py": "from db import save\nimport utils\ndef run(): pass", "app/db.py": "def save(): pass", "app/utils.py": "def u(): pass" }, "app/db.py");
    expect(r.direct).toEqual(["app/main.py"]);
  });
});

```

## demo/repository/backend/database/TransactionRepository.ts

```ts
export function saveTransaction(id: string, amount: number) { return { id, amount }; }
export function rollbackTransaction(id: string) { return id; }

```

## demo/repository/backend/order/OrderService.ts

```ts
import { saveTransaction } from "../database/TransactionRepository";
export function createOrder(paymentId: string) { return saveTransaction(paymentId, 0); }

```

## demo/repository/backend/payment/PaymentService.ts

```ts
import { saveTransaction, rollbackTransaction } from "../database/TransactionRepository";
import { createOrder } from "../order/OrderService";
export function validatePayment(amount: number) { return amount > 0; }
export function processPayment(id: string, amount: number) {
  if (!validatePayment(amount)) return rollbackTransaction(id);
  saveTransaction(id, amount);
  return createOrder(id);
}

```

## demo/repository/backend/payment/PaymentController.ts

```ts
import { processPayment } from "./PaymentService";
export function handlePayment(req: { id: string; amount: number }) { return processPayment(req.id, req.amount); }

```

## demo/repository/frontend/PaymentButton.tsx

```tsx
import { handlePayment } from "../backend/payment/PaymentController";
export function PaymentButton() { return <button onClick={() => handlePayment({ id: "1", amount: 10 })}>Pay</button>; }

```

## demo/repository/frontend/Checkout.tsx

```tsx
import { PaymentButton } from "./PaymentButton";
export function Checkout() { return <div><PaymentButton /></div>; }

```

## demo/repository/tests/payment.test.ts

```ts
import { processPayment } from "../backend/payment/PaymentService";
processPayment("1", 10);

```

## demo/repository/tests/order.test.ts

```ts
import { createOrder } from "../backend/order/OrderService";
createOrder("1");

```
