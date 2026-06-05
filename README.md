# clojure.nurazhar.com

> A lightweight, Babashka-native Clojure blog and digital garden dedicated to Lisp, shell scripting, and developer workflow automation.

This project is built using **[Quickblog](https://github.com/borkdude/quickblog)**, a fast static-site generator powered by **[Babashka](https://babashka.org/)**. It compiles plain Markdown files into highly optimized, static HTML with near-zero overhead.

---

## 🗺️ System Architecture

The following diagram illustrates the local development and automated CI/CD pipeline deployment to GitHub Pages under the custom domain `clojure.nurazhar.com`:

```mermaid
graph TD
    %% Define Styles
    classDef local fill:#1e1e2e,stroke:#cba6f7,stroke-width:2px,color:#cdd6f4;
    classDef cloud fill:#11111b,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4;
    classDef dns fill:#1e1e2e,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4;
    classDef user fill:#181825,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4;

    %% Local Laptop
    subgraph Local_Laptop ["💻 Local Development (CachyOS / Arch)"]
        A["📝 Markdown Post (posts/*.md)"]
        B["🛠️ bb quickblog watch (Local Preview)"]
        C["☕ Java Runtime (Zulu SDKMAN)"]
        D["📦 Babashka (Local Compiler)"]
        
        A -->|1. Live Preview| B
        C -->|Powers Dependencies| D
        D -->|Renders Dev Site| B
    end
    class Local_Laptop local;

    %% Git Push
    A -->|2. git push origin main| E["🐙 GitHub Repository (clojure.nurazhar.com)"]
    class E cloud;

    %% GitHub Actions
    subgraph Cloud_Pipeline ["☁️ GitHub Actions Runner (CI/CD)"]
        F["🐳 Ubuntu Runner"]
        G["☕ Hosted Java Runtime (Temurin 21)"]
        H["📦 Babashka (Cloud Compiler)"]
        I["⚙️ bb quickblog render"]
        
        E -->|3. Triggers Workflow| F
        G -->|Resolves classpath| H
        H -->|4. Compiles MD to HTML| I
    end
    class Cloud_Pipeline cloud;

    %% Deployment
    I -->|5. Pushes static HTML| J["🌐 GitHub Pages hosting (gh-pages branch)"]
    class J cloud;

    %% DNS Routing
    subgraph DNS_System ["🌐 Custom DNS Routing"]
        K["📛 DNS Registrar (CNAME: clojure)"]
        L["📌 CNAME File in Repo"]
    end
    class DNS_System dns;

    K -->|Redirects to| J
    L -->|Maps subdomain to| J

    %% Reader
    subgraph Readers ["👥 Readers / Visitors"]
        M["💻 Web Browser"]
    end
    class Readers user;

    M -->|6. Requests clojure.nurazhar.com| K
    J -->|7. Serves Static Site| M
```

---

## 🛠️ Local Development & Quick Start

### Prerequisites
- **Babashka**: The Clojure scripting engine. Follow [Babashka installation](https://babashka.org/#installation).
- **Java Development Kit (JDK)**: Java 21+ is recommended (managed via `sdkman` or package manager).

### Running Locally
To start the live-reloading local development server:
```bash
bb quickblog watch
```
This runs a local HTTP server at `http://localhost:1888` which automatically re-renders pages as you modify markdown files in the `posts/` directory.

### Available Tasks
The project tasks are structured via Babashka `bb.edn`:
*   `bb quickblog new` – Generate a template for a new blog post.
*   `bb quickblog render` – Compile all Markdown files in `posts/` into static HTML.
*   `bb quickblog watch` – Start the local dev server with auto-refresh.
*   `bb quickblog clean` – Clean the output directories.
*   `bb quickblog help` – Display help regarding Quickblog CLI options.

---

## 🚀 CI/CD Pipeline

Deployment is fully automated using GitHub Actions (`.github/workflows/deploy.yml`):
1. On every `git push` to the `main` branch, the pipeline is triggered.
2. It sets up **Java 21** (Temurin distribution) and installs **Babashka**.
3. Compiles the markdown posts using `bb quickblog render` into the `public/` directory.
4. Deploys the generated output to the `gh-pages` branch using the `peaceiris/actions-gh-pages` action, configured for the custom domain `clojure.nurazhar.com`.
