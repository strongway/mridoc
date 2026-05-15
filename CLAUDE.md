# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an fMRI tutorial documentation site built with Quartz 4, a static site generator that converts Markdown content into a web-based digital garden. The site hosts tutorials for the MSense Lab covering fMRI analysis, MRI basics, neuroimaging tools, and server/software guidance.

## Architecture

- **Framework**: Quartz 4 (static site generator based on TypeScript/Preact)
- **Content**: Markdown files in `content/` directory organized by topic
- **Configuration**: `quartz.config.ts` (site settings) and `quartz.layout.ts` (page layouts)
- **Build Output**: Generated static files in `public/` directory
- **Media**: Images and assets stored in `content/media/`

The site structure follows two main content categories:
- `content/fMRI Tutorial/`: Step-by-step fMRI analysis tutorials
- `content/Server and Software/`: Computing environment and software guides

## Development Commands

### Build and Serve
```bash
npx quartz build --serve
# Builds site and serves at http://localhost:8080
```

### Documentation Build
```bash
npm run docs
# Alternative build command that serves docs directory
```

### Code Quality
```bash
npm run check    # TypeScript check and Prettier formatting check
npm run format   # Format code with Prettier
npm test         # Run tests with tsx
```

### Content Sync
```bash
npx quartz sync  # Commit and sync changes to repository
```

## Configuration Files

- `quartz.config.ts`: Main site configuration (theme, plugins, analytics)
- `quartz.layout.ts`: Page layout definitions and component arrangement
- `package.json`: Contains all build scripts and dependencies
- `tsconfig.json`: TypeScript configuration for ESNext modules

## Content Management

Content files are Markdown with YAML frontmatter. The site uses:
- Obsidian-flavored Markdown for internal linking
- LaTeX support via KaTeX
- Syntax highlighting for code blocks
- Table of contents generation
- Tag-based organization

## Key Features

- Single Page Application (SPA) routing enabled
- Popover previews for internal links
- Graph view of content relationships
- Full-text search functionality
- Dark/light mode toggle
- Mobile-responsive design

## Deployment

The site appears to be deployed to https://mri.msense.de and is associated with the MSense Lab (https://msense.de).