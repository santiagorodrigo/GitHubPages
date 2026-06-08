# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Blazor WebAssembly (WASM) single-page recruiter portfolio for **Rodrigo Santiago**, targeting .NET 10.0. Deployed via GitHub Pages to the custom domain `https://www.rodrigo-santiago.dev`. Client-side only — no server component, no backend API.

## Commands

```bash
# Run dev server (http://localhost:5211)
dotnet run

# Run with hot reload
dotnet watch

# Build
dotnet build --configuration Release

# Publish (matches CI output path)
dotnet publish GitHubPages.csproj --configuration Release -o docs --nologo
```

## Architecture

### Routing & Layout

`App.razor` configures the Blazor router. The only page is `Pages/Home.razor` (`@page "/"`). `Layout/MainLayout.razor` is a minimal shell — sticky top navbar (`Layout/NavMenu.razor`) + `@Body`. There is no sidebar.

### Component Structure (SOLID / SRP)

`Pages/Home.razor` is a pure composition root — it renders six section components in order:

```
Components/
  ProfileData.cs          ← single source of truth for Email, LinkedIn URL, resume path
  HeroSection.razor       ← hero, stats, CTA buttons
  AboutSection.razor      ← summary paragraph + key strengths card
  SkillsSection.razor     ← 5 grouped skill badge cards
  ExperienceSection.razor ← timeline (8 detailed roles) + 3 mini earlier-experience cards
  EducationSection.razor  ← education, certifications, languages
  ProfileFooter.razor     ← footer with contact links + motto
```

To add or edit a section, touch only that component's `.razor` file. To change contact info or URLs, edit `ProfileData.cs` — it propagates everywhere.

### CSS

All styles live in `wwwroot/css/app.css` (global). CSS custom properties (`--navy`, `--blue`, etc.) are defined in the `:root` block at the top of that file.

**Do not put `:root { --var: value }` in any `.razor.css` scoped file.** Blazor CSS isolation rewrites `:root` to `:root[b-xxxx]` which never matches, silently breaking all `var(--...)` references.

`Layout/NavMenu.razor.css` and `Layout/MainLayout.razor.css` are the only scoped CSS files in use.

### Static Assets

| Path | Purpose |
|------|---------|
| `wwwroot/downloads/Rodrigo_Santiago-Resume.pdf` | Resume PDF — linked in the navbar download button |
| `wwwroot/CNAME` | Custom domain declaration (`www.rodrigo-santiago.dev`) |
| `wwwroot/css/app.css` | All profile styles + CSS variables |

## GitHub Pages Deployment

CI (`.github/workflows/main.yml`) triggers on push to `main`:
restores → builds → publishes to `docs/` → uploads `docs/wwwroot` as the Pages artifact → deploys.

### base href

`wwwroot/index.html` uses `<base href="/" />` — correct for both local dev and the custom domain (served from the domain root). Do not change it back to a subdirectory URL.

The `CNAME` file in `wwwroot/` tells GitHub Pages to use `www.rodrigo-santiago.dev`. GitHub redirects the legacy `santiagorodrigo.github.io/GitHubPages/` URL to the custom domain automatically.

### If the app is stuck on the loading screen locally

The browser cached the page from before the `base href` fix. Hard-refresh with `Ctrl+Shift+R` or open an incognito window.
