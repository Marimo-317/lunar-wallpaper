# Lunar Wallpaper

Real-time moon phase wallpaper for smartphones that dynamically updates to reflect the current lunar phase.

## Features

- Real-time moon phase calculation
- High-resolution smartphone wallpapers
- Automatic daily updates
- Accurate astronomical data
- Multiple visual styles

## Development

This project uses the SPARC methodology and Claude Flow environment.

### Available Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Run tests
npm run test

# Lint code
npm run lint

# Type checking
npm run typecheck
```

### SPARC Development

```bash
# Start with specification
bash ../claude-flow.sh sparc run spec-pseudocode "Define moon phase wallpaper requirements"

# Design architecture
bash ../claude-flow.sh sparc run architect "Moon phase calculation and rendering system"

# Implement with TDD
bash ../claude-flow.sh sparc tdd "moon phase calculation engine"

# List all available modes
bash ../claude-flow.sh sparc modes
```

## Technology Stack

- Vue.js 3 for UI components
- Canvas API for wallpaper rendering
- TypeScript for type safety
- Vite for build tooling
- Vitest for testing