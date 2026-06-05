# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Blazor WebAssembly (WASM) app targeting .NET 10.0, deployed to GitHub Pages at `https://santiagorodrigo.github.io/GitHubPages/`. This is a **client-side-only** app — there is no server component. All data is fetched from static files in `wwwroot/`.

## Commands

```bash
# Run dev server (http://localhost:5211)
dotnet run

# Run with hot reload
dotnet watch

# Build
dotnet build --configuration Release

# Publish to docs/ (matches CI output path)
dotnet publish GitHubPages.csproj --configuration Release -o docs --nologo
```

## Architecture

**Routing**: `App.razor` configures the Blazor router. Pages live in `Pages/` and declare their route with `@page "/route"`. The `NotFound` page handles unmatched routes. The layout is wired via `DefaultLayout` in `App.razor`.

**Layout**: `Layout/MainLayout.razor` defines the page shell (sidebar + content area). `Layout/NavMenu.razor` is the collapsible sidebar nav.

**Data fetching**: Components inject `HttpClient` (registered with `BaseAddress` set to the app's host URL in `Program.cs`) and fetch static JSON from `wwwroot/sample-data/`. There is no backend API.

**PWA**: The app registers a service worker (`wwwroot/service-worker.js`). The published version uses `wwwroot/service-worker.published.js` (swapped at publish time via the `.csproj` `<ServiceWorker>` item).

## GitHub Pages Deployment

CI (`main.yml`) runs on every push to `main`: restores → builds → publishes → uploads `docs/wwwroot` to GitHub Pages.

**Critical**: `wwwroot/index.html` has the `<base href>` hardcoded to the production GitHub Pages subdirectory path:

```html
<base href="https://santiagorodrigo.github.io/GitHubPages/" />
```

This is required because GitHub Pages serves the app from a subdirectory, not the domain root. When running locally, Blazor dev server handles routing without this, but the hardcoded value means local dev may behave differently for asset paths. If you need to test with the correct base locally, temporarily switch it to `<base href="/" />`.
