# clojure.nurazhar.com - System Architecture

This document maps out the system architecture and deployment pipeline for your Babashka-native Clojure blog.

---

## 🗺️ Visual Architecture Diagram

```mermaid
graph TD
    %% Define Styles
    classDef local fill:#1e1e2e,stroke:#cba6f7,stroke-width:2px,color:#cdd6f4;
    classDef cloud fill:#11111b,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4;
    classDef dns fill:#1e1e2e,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4;
    classDef user fill:#181825,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4;

    %% Local Laptop
    subgraph Local_Laptop ["💻 YOUR LAPTOP (CachyOS + Fish Shell)"]
        A["📝 Markdown Post (posts/*.md)"]
        B["🛠️ bb quickblog watch (Optional Local Preview)"]
        C["☕ SDKMAN! (Azul Zulu 25 LTS)"]
        D["📦 Babashka (Local Compiler)"]
        
        A -->|1. Test locally| B
        C -->|Powers deps| D
        D -->|Renders local dev site| B
    end
    class Local_Laptop local;

    %% Git Push
    A -->|2. git push origin main| E["🐙 GitHub Repository (clojure.nurazhar.com)"]
    class E cloud;

    %% GitHub Actions
    subgraph Cloud_Pipeline ["☁️ GITHUB ACTIONS RUNNER (CI/CD)"]
        F["🐳 Ubuntu Runner"]
        G["☕ Hosted Java Runtime"]
        H["📦 Babashka (Cloud Compiler)"]
        I["⚙️ bb quickblog render"]
        
        E -->|3. Triggers Workflow| F
        G -->|Resolves classpath| H
        H -->|4. Compiles Markdown to HTML| I
    end
    class Cloud_Pipeline cloud;

    %% Deployment
    I -->|5. Pushes static HTML| J["🌐 GitHub Pages hosting (gh-pages branch)"]
    class J cloud;

    %% DNS Routing
    subgraph DNS_System ["🌐 GLOBAL DNS ROUTING"]
        K["📛 DNS Registrar (CNAME: clojure)"]
        L["📌 CNAME File in Repo"]
    end
    class DNS_System dns;

    K -->|Redirects to| J
    L -->|Maps subdomain to| J

    %% Reader
    subgraph Readers ["👥 THE INTERNET"]
        M["💻 Reader's Web Browser"]
    end
    class Readers user;

    M -->|6. Requests clojure.nurazhar.com| K
    J -->|7. Serves lightning-fast HTML| M
```

---

## ⚙️ Component Details

### 1. Local Laptop Setup
*   **Path:** `/home/nurazhar/Documents/1_identity_&_career/clojure.nurazhar.com/`
*   **Java Runtime:** Azul Zulu 25 LTS (`25.0.3-zulu`) managed in user-space via **SDKMAN!**.
*   **Interactive Tool:** Babashka (`bb`) runs local tasks without JVM startup overhead.
*   **Local Preview:** Running `bb quickblog watch` mounts a live-reloading HTTP server at `http://localhost:1888` for instant local proofing.

### 2. GitHub Actions CI/CD Pipeline
*   **Trigger:** Triggers automatically on every `git push` to the `main` branch.
*   **Runner Environment:** Standard `ubuntu-latest` running the **Adoptium Java 25 (LTS)** action and **DeLaGuardo/setup-clojure** action.
*   **Compilation:** Invokes `bb quickblog render` in the cloud to resolve dependencies and translate Markdown into standard, high-performance static HTML.

### 3. GitHub Pages & DNS
*   **Hosting Source:** Deploys from the compiled `public/` folder directly to the `gh-pages` branch.
*   **CNAME Mapping:** The `CNAME` file instructs GitHub to associate the build output with the custom subdomain `clojure.nurazhar.com`.
*   **External Routing:** Your global DNS registrar routes requests for `clojure` to `nurazharSG.github.io`, pointing users to the live server.
