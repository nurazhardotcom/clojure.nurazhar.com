Title: From CLI to Web and Back: Building a Rock-Solid Native Clojure Desktop GUI
Date: 2026-06-08
Tags: clojure, cljfx, desktop-gui, multi-agent-system, career-ops, automation
Description: The journey of re-architecting headhunter-agent from a command line tool, through a browser-based Scittle app, and finally to a native cljfx JavaFX desktop client for local-first reliability and zero bugs.

---

# From CLI to Web and Back: Building a Rock-Solid Native Clojure Desktop GUI

Lately, I’ve been building **`headhunter-agent`** (a local-first job search assistant and resume compiler) to streamline my own career operations. The journey of designing this application was a masterclass in why modern web stacks introduce unnecessary fragility, and how returning to native desktop Lisps can lead to bulletproof, bug-free software.

Here is the story of how `headhunter-agent` went from a simple CLI tool, to a browser-based ClojureScript app, and finally to a native Clojure desktop application using **`cljfx` (JavaFX)**.

---

## The Origin: Forking and Initial Rewrite

The project began when I forked `career-ops`, a simple Clojure/Babashka CLI that evaluated job descriptions against a CV using the Gemini API. My initial goal was just to rewrite it for my personal environment, parsing a raw markdown resume, generating tailored PDF resumes via **Typst**, and adding basic tracking. 

However, standard job application tools are incredibly shallow. Most simply stuff keywords into your CV to match ATS algorithms, generating generic nonsense that falls apart during an actual interview. I wanted something deeper.

---

## Harris's Feedback: The Multi-Agent Pipeline & Data Vault

I shared my thoughts on the first version, and my colleague Harris Lithan Full Stack gave some critical feedback that reshaped the entire architecture of the application:
1. **Deep Profiling First (The Data Vault):** A resume tool shouldn't just read a flat CV. It needs a full profile: certifications, diplomas, specific modules under each qualification, work history details, target roles, and salary benchmarks.
2. **STAR Stories Library:** The core of interview success is having 8-12 solid, pre-mapped **STAR (Situation, Task, Action, Result)** stories. The tool should ingest raw experience dumps and automatically extract this structured library.
3. **A 3-Stage Multi-Agent System (MAS):** Instead of a single API call evaluating a JD, we should deploy three specialized agents:
   - **Agent 1 (Legitimacy & JD Analysis):** Auditing the JD for red flags and Fair Consideration Framework (FCF) compliance in Singapore.
   - **Agent 2 (Fit Analysis):** Running a brutal gap and strength analysis comparing the candidate's Master Profile to the JD requirements.
   - **Agent 3 (Deep Dive Memo):** Researching the company's business model, tech stack, and providing specific cold outreach recommendations.

---

## The Web GUI Attempt: The Fragile Era

To make this complex system easy to use, I initially moved the codebase from the CLI to a web interface. 

I chose **ClojureScript** with **Reagent** and interpreted it in the browser dynamically using **Scittle** (meaning zero Node compilation or build-tool overhead). The site was hosted statically on GitHub Pages (`headhunter.nurazhar.com`), saving API keys and data to browser `localStorage`, and compiling Typst resumes via a WebAssembly (`typst.ts`) engine.

While it was cool in theory, in practice, **the browser environment introduced fragility**:
- **State Management Bugs:** Syncing local state with `localStorage` across tabs and asynchronous agent calls was prone to race conditions.
- **WASM & Memory Overhead:** Running WebAssembly compilers inside the browser sandbox frequently led to page crashes or sluggish rendering.
- **Loss of Control:** Dealing with browsers, caches, and web-app routing broke the local-first philosophy. It felt buggy, and as a security engineer, hosting private identity data inside a browser tab—where rogue extensions can read the DOM—felt like a compromise.

---

## The Clean Slate: Native Desktop GUI with `cljfx`

I realized I didn't hate the technologies themselves, but I needed **zero bugs** and absolute platform reliability. If the web stack is fragile, the solution is simple: **completely abandon the web.**

I decided to re-architect `headhunter-agent` into a **Native Desktop GUI** using **`cljfx` (JavaFX for Clojure)**.

`cljfx` is a declarative, functional wrapper around JavaFX. It allows you to build native, hardware-accelerated desktop windows in pure Clojure. It uses a React-like state management system, but renders native OS window components instead of HTML elements.

```clojure
;; A snippet of the root-view in career-ops.gui
(defn root-view [state]
  {:fx/type :stage
   :showing true
   :title "Local Jack & Jill — Clojure Desktop MAS Console"
   :width 1100
   :height 700
   :scene {:fx/type :scene
           :style "-fx-background-color: #2e3440;"
           :root {:fx/type :tab-pane
                  :side :top
                  :tabs [{:fx/type :tab :text "Data Vault" :content {:fx/type vault-tab ...}}
                         {:fx/type :tab :text "Evaluate JD" :content {:fx/type evaluator-tab ...}}
                         {:fx/type :tab :text "Interview Prep" :content {:fx/type interview-tab ...}}]}}})
```

### Why this architecture achieved "Zero Bugs" and "Zero Fragility":

1. **Zero Browser Dependency:** The app runs as a standalone window on my CachyOS/Arch Linux machine. No local web servers, no browser tabs, no JavaScript.
2. **Mathematically Predictable UI:** Because Clojure utilizes immutable data structures, the entire GUI state is managed inside a single Clojure atom. The UI is a pure function of the state. If the data is correct, the UI is physically incapable of rendering incorrectly.
3. **Async Network Isolation:** Making multiple heavy calls to the Gemini API is slow. If run on the main thread, JavaFX will freeze. By leveraging Clojure's native `future` multithreading, API calls execute asynchronously on background threads and dispatch UI updates back to the JavaFX Application Thread using `Platform/runLater`. The UI remains at a buttery smooth 60 FPS.
4. **Local EDN Storage:** All data is parsed and written directly to my local drive as `.edn` files (`data/master-profile.edn` and `data/star-stories.edn`). There is no network syncing, no cloud databases, and no tracking.

---

## Conclusion: The Lisp Desktop Renaissance

We spent years moving everything to the web, only to realize that web applications are bloated, slow, and insecure sandboxes. 

Building `headhunter-agent` with `cljfx` proved that writing desktop applications in Clojure is one of the most productive developer experiences available today. You get the interactive, REPL-driven development loop of Lisp, the rock-solid performance of the JVM, and a native desktop UI that never breaks. 

If you are tired of NPM dependency hell and fragile web states, consider building your next tool natively on the desktop.

*Check out the open-source codebase on my GitHub:* **[headhunter-agent](https://github.com/nurazhardotcom/headhunter-agent)**
