# CLAUDE.md - AI Assistant Guide for Vite Electron Builder Boilerplate

## Project Overview

A secure Electron application boilerplate using Vite as the build tool. This template provides a foundation for building desktop applications with modern security practices and a monorepo architecture.

**Note**: This is a boilerplate/template, not a complete application. The renderer (UI) package is not included and must be created during setup.

## Tech Stack

- **Runtime**: Electron 36.5.0
- **Build Tool**: Vite 6.3.5
- **Packaging**: electron-builder 26.0.12
- **Auto-update**: electron-updater 6.6.2
- **Testing**: Playwright 1.53.1
- **Language**: TypeScript 5.8.3
- **Package Manager**: npm workspaces (monorepo)
- **Node.js**: >= 23.0.0 (required)

## Project Structure

```
packages/                    # Monorepo workspace packages
  main/                      # Electron main process
    src/
      index.ts               # App initialization entry
      AppModule.ts           # Module interface
      ModuleRunner.ts        # Module execution system
      ModuleContext.ts       # Shared context for modules
      AppInitConfig.ts       # Configuration types
      modules/               # Feature modules
        WindowManager.ts     # Window creation/management
        AutoUpdater.ts       # Auto-update functionality
        SingleInstanceApp.ts # Single instance enforcement
        BlockNotAllowdOrigins.ts  # Origin security
        ExternalUrls.ts      # External URL handling
        HardwareAccelerationModule.ts
        ChromeDevToolsExtension.ts
        AbstractSecurityModule.ts
        ApplicationTerminatorOnLastWindowClose.ts
  preload/                   # Preload scripts (context bridge)
    src/
      index.ts               # Exports for renderer
      exposed.ts             # Context bridge setup
      nodeCrypto.ts          # Crypto utilities
      versions.ts            # Version info
  integrate-renderer/        # Helper for renderer setup
  electron-versions/         # Electron version utilities
  dev-mode.js                # Development server script
  entry-point.mjs            # Main entry point
tests/                       # E2E tests
  e2e.spec.ts                # Playwright test suite
types/                       # TypeScript type definitions
  env.d.ts                   # Environment variable types
buildResources/              # Electron-builder resources
.github/                     # GitHub Actions workflows
```

## Key Files

| File | Purpose |
|------|---------|
| `package.json` | Root config, workspaces, scripts |
| `packages/entry-point.mjs` | Main Electron entry |
| `packages/main/src/index.ts` | App initialization |
| `packages/main/src/modules/WindowManager.ts` | Window management |
| `packages/preload/src/exposed.ts` | Context bridge setup |
| `electron-builder.mjs` | Build/packaging configuration |
| `tests/e2e.spec.ts` | End-to-end tests |

## Development Setup

### Prerequisites
- Node.js >= 23.0.0
- npm

### Initial Setup

```bash
# Clone repository
git clone <repo-url>
cd <project>

# Initialize project (creates renderer package)
npm run init

# This runs:
# 1. npm run create-renderer - Creates Vite project for UI
# 2. npm run integrate-renderer - Integrates with Electron
# 3. npm install - Installs dependencies
```

### Start Development

```bash
npm start
```

This launches the app in development mode with hot-reload.

## Common Commands

```bash
# Start development server
npm start

# Build all packages
npm run build

# Compile to executable
npm run compile

# Compile without asar (for debugging)
npm run compile -- --dir -c.asar=false

# Run E2E tests (on compiled app)
npm run test

# Type checking
npm run typecheck

# Create new renderer
npm run create-renderer

# Integrate renderer with Electron
npm run integrate-renderer
```

## Testing

### Framework
- **E2E Testing**: Playwright
- **Test Location**: `tests/e2e.spec.ts`
- **Target**: Tests run against compiled executable

### Running Tests

```bash
# First compile the app
npm run compile

# Then run tests
npm run test
```

### Test Coverage
- Main window state (visibility, crash detection, DevTools)
- Interactive UI elements
- Preload context exposure (versions, sha256sum, IPC send)

## Code Conventions

### Module Pattern
The main process uses a modular architecture:

```typescript
import type {AppModule} from '../AppModule.js';
import {ModuleContext} from '../ModuleContext.js';

class MyModule implements AppModule {
  async enable({app}: ModuleContext): Promise<void> {
    // Module initialization
  }
}

export function createMyModule(...args) {
  return new MyModule(...args);
}
```

### Preload Exports
Functions exported from `packages/preload/src/index.ts` are automatically exposed via context bridge:

```typescript
// preload/src/index.ts
export function myFunction() {
  return 'data';
}

// Available in renderer as:
import {myFunction} from '@app/preload';
```

### Internal Package Names
All workspace packages use `@app/*` prefix (e.g., `@app/main`, `@app/preload`).

### TypeScript
- Strict mode enabled
- ES modules (`"type": "module"`)
- File extensions required in imports (`.js` even for `.ts` files)

### Environment Variables
- Use `import.meta.env` for access
- Only `VITE_*` prefixed variables exposed to client
- Define types in `types/env.d.ts`

## When Making Changes

1. **Follow module pattern** - Create new modules in `packages/main/src/modules/`
2. **Type everything** - Add TypeScript types for new features
3. **Test security** - Be cautious with preload script changes
4. **Run typecheck** - Execute `npm run typecheck` before commits
5. **Update preload exports** - Add to `packages/preload/src/index.ts` for renderer access
6. **Build before testing** - E2E tests require compiled app

## Issues & Recommendations

### Security Concerns

1. **Sandbox Disabled** (`packages/main/src/modules/WindowManager.ts:30`)
   - `sandbox: false` in webPreferences is a security risk
   - Recommendation: Enable sandbox and refactor preload scripts to work within sandboxed environment
   - Current comment says "demo of preload script depend on the Node.js api" - this should be addressed for production use

2. **Base64 Obfuscation for Context Bridge** (`packages/preload/src/exposed.ts`)
   - Using `btoa()` to encode exposed function names is unusual
   - This may be intentional obfuscation but adds complexity without real security benefit
   - Recommendation: Use standard readable names unless there's a specific reason

### Code Quality Issues

3. **Missing Linting Configuration**
   - No ESLint or similar linting tool configured
   - Recommendation: Add ESLint with TypeScript support for consistent code quality

4. **IDE Configuration in Repository** (`.idea/` directory)
   - JetBrains IDE settings committed to repo
   - Recommendation: Add `.idea/` to `.gitignore` to avoid IDE-specific conflicts

5. **Typo in README.md** (line 75)
   - "Initially, the repository contains only a few packages.4"
   - Should remove the trailing "4"

6. **Typo in Filename** (`BlockNotAllowdOrigins.ts`)
   - Should be `BlockNotAllowedOrigins.ts`

### Testing Gaps

7. **No Unit Tests**
   - Only E2E tests exist
   - Individual modules in `packages/main/src/modules/` have no unit tests
   - Recommendation: Add unit tests for critical modules (WindowManager, security modules)

8. **Test Coverage Reporting**
   - No coverage configuration
   - Recommendation: Add coverage reporting to track test completeness

### Dependency Concerns

9. **Very High Node.js Requirement** (>= 23.0.0)
   - Node.js 23 is a current/unstable release
   - Many users may not have Node.js 23 installed
   - Recommendation: Consider supporting Node.js 20 LTS or 22 LTS

10. **Missing Dependency Lock File**
    - No `package-lock.json` visible (may be gitignored)
    - Recommendation: Ensure lock file is committed for reproducible builds

### Documentation Issues

11. **Incomplete Renderer Setup Documentation**
    - Process of creating custom renderer (non-Vite) needs more guidance
    - Recommendation: Add examples for React, Vue, etc. integration

12. **Missing API Documentation**
    - No JSDoc comments on module functions
    - Recommendation: Add documentation for public APIs

### Structural Improvements

13. **Hardcoded External URLs** (`packages/main/src/index.ts:31-40`)
    - Development documentation URLs hardcoded in security module
    - Recommendation: Move to configuration file for easier maintenance

14. **No Error Handling Strategy**
    - Modules don't have standardized error handling
    - Recommendation: Add error boundaries and logging strategy

### CI/CD

15. **GitHub Actions Present but Minimal**
    - Basic CI workflow exists
    - Recommendation: Add automated testing in CI pipeline, dependency vulnerability scanning

## External Resources

- **Author**: Alex Kozack (kozackunisoft@gmail.com)
- **Electron Docs**: https://www.electronjs.org/docs
- **Vite Docs**: https://vitejs.dev
- **electron-builder Docs**: https://www.electron.build
- **Playwright Docs**: https://playwright.dev
