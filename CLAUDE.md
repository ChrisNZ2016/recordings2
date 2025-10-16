# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15 application with TypeScript, React 19, and Tailwind CSS 4. The actual project code is located in the `recording2/` subdirectory.

## Development Commands

All commands should be run from the `recording2/` directory:

```bash
cd recording2
```

- **Development server**: `npm run dev` (uses Turbopack)
- **Production build**: `npm run build` (uses Turbopack)
- **Production server**: `npm run start`
- **Lint**: `npm run lint`

The dev server runs at http://localhost:3000 by default.

## Architecture

**Framework**: Next.js 15 with App Router architecture

**Key Directories**:
- `app/` - App Router pages and layouts using the new Next.js App Router convention
  - `layout.tsx` - Root layout with Geist font configuration
  - `page.tsx` - Home page component
  - `globals.css` - Global Tailwind styles

**Styling**: Tailwind CSS 4 with PostCSS plugin architecture

**TypeScript Configuration**:
- Path alias `@/*` maps to the project root for imports
- Strict mode enabled
- Module resolution set to "bundler"

**Build Tools**:
- Turbopack enabled for both dev and build (via `--turbopack` flag)
- ESLint with Next.js TypeScript presets

## Important Notes

- The project uses Geist and Geist Mono fonts loaded via `next/font/google`
- Tailwind CSS 4 uses the new `@tailwindcss/postcss` plugin system (different from v3)
- React 19 and Next.js 15 are cutting-edge versions - check compatibility when adding dependencies