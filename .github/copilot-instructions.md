# Copilot Instructions for AI Coding Agents

## Project Overview
- This is a Next.js project (TypeScript, App Router) bootstrapped with `create-next-app`.
- Main app code is in `src/app/` (entry: `src/app/page.tsx`, layout: `src/app/layout.tsx`).
- Global styles: `src/app/globals.css`.
- Public assets: `public/`.
- Project configuration: `next.config.ts`, `tsconfig.json`, `eslint.config.mjs`, `postcss.config.mjs`.

## Key Workflows
- **Development:** Start with `npm run dev` (or `yarn dev`, `pnpm dev`, `bun dev`).
- **Hot reload:** Editing files in `src/app/` auto-updates the running app.
- **Build:** Use `npm run build` to create a production build.
- **Lint:** Use `npm run lint` to check code style and errors.

## Project Conventions
- Use the App Router (`src/app/`), not the Pages Router.
- TypeScript is required for all source files.
- Use functional React components.
- Place global CSS in `src/app/globals.css`.
- Static assets (SVGs, images) go in `public/`.
- Keep configuration files at the project root.

## Patterns & Structure
- Main entry point: `src/app/page.tsx` (edit this to change the homepage).
- Shared layout: `src/app/layout.tsx`.
- Use Next.js conventions for routing and file structure.
- No custom server or API routes are present by default.

## External Integrations
- No custom integrations or external APIs are configured by default.
- Fonts are loaded via `next/font` (see Next.js docs for details).

## Examples
- To add a new page: create a new folder with a `page.tsx` in `src/app/` (e.g., `src/app/about/page.tsx`).
- To add a global style: edit `src/app/globals.css`.
- To add an SVG: place it in `public/` and reference via `/file.svg`.

## Reference
- See `README.md` for getting started and deployment instructions.
- See `next.config.ts` and `tsconfig.json` for build and type settings.

---

*Update this file if project structure or conventions change. Focus on actionable, project-specific guidance for AI agents.*
