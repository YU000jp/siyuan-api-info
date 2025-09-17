# SiYuan Technical Stack Analysis

This document provides a comprehensive analysis of the technical stack, libraries, frameworks, and tools used in the SiYuan project to help AI systems understand the development context and architecture.

## 📊 Overview

**Project Type**: Desktop/Mobile knowledge management application  
**Architecture**: Client-Server with Electron frontend and Go backend  
**Primary Languages**: TypeScript, Go, JavaScript, SCSS  
**Development Approach**: Full-stack cross-platform application  

## 🖥️ Frontend Stack

### Core Technologies
- **TypeScript 4.7.4** - Primary frontend language with type safety
- **Electron 37.4.0** - Cross-platform desktop application framework
- **Webpack 5.94.0** - Module bundling and build system
- **SCSS/Sass 1.89.2** - Stylesheet preprocessing

### Development Tools
- **ESLint 9.15.0** - Code linting and style enforcement
- **TypeScript ESLint** - TypeScript-specific linting rules
- **Electron Builder 26.0.12** - Application packaging and distribution
- **Webpack Bundle Analyzer** - Bundle size analysis and optimization

### Build Configuration
- **Multiple webpack configs** for different targets:
  - `webpack.config.js` - Main application build
  - `webpack.mobile.js` - Mobile platform optimizations
  - `webpack.desktop.js` - Desktop-specific features
  - `webpack.export.js` - Export functionality build

### Package Management
- **pnpm 10.15.1** - Fast, disk space efficient package manager
- **pnpm workspaces** - Monorepo support for multi-platform builds

### Key Frontend Dependencies
- **@electron/remote 2.1.3** - Cross-process communication in Electron
- **blueimp-md5 2.19.0** - MD5 hashing for data integrity
- **dayjs 1.11.5** - Date manipulation library
- **iconv-lite 0.6.3** - Character encoding conversion

### Development Workflow
```bash
# Development mode
pnpm run dev          # Standard development build
pnpm run dev:mobile   # Mobile-optimized build
pnpm run dev:desktop  # Desktop-specific build

# Production builds
pnpm run build        # All production builds
pnpm run build:app    # Main application
pnpm run build:mobile # Mobile platform
pnpm run build:desktop # Desktop platform

# Distribution
pnpm run dist         # Standard distribution
pnpm run dist-arm64   # ARM64 architecture
pnpm run dist-darwin  # macOS distribution
pnpm run dist-linux   # Linux distribution
```

## 🔧 Backend Stack

### Core Technologies
- **Go 1.24.4** - Backend language for high performance
- **SQLite** - Embedded database for data storage
- **REST API** - HTTP-based API architecture
- **WebSocket** - Real-time communication

### Key Go Dependencies
#### Core Libraries
- **github.com/88250/lute** - Markdown processing engine
- **github.com/88250/gulu** - Utility library collection
- **github.com/PuerkitoBio/goquery** - HTML parsing and manipulation
- **github.com/asaskevich/EventBus** - Event-driven architecture

#### Data Processing
- **code.sajari.com/docconv** - Document conversion utilities
- **github.com/88250/epub** - EPUB file processing
- **github.com/araddon/dateparse** - Date parsing and formatting
- **github.com/88250/vitess-sqlparser** - SQL parsing and validation

#### System Integration
- **github.com/88250/clipboard** - Cross-platform clipboard operations
- **github.com/Xuanwo/go-locale** - Locale detection and handling
- **github.com/ConradIrwin/font** - Font processing and analysis

#### Text Processing
- **github.com/ClarkThan/ahocorasick** - String matching algorithm
- **github.com/Masterminds/sprig/v3** - Template function library
- **github.com/common-nighthawk/go-figure** - ASCII art generation

## 🏗️ Architecture Patterns

### Frontend Architecture
- **Modular design** with webpack-based module system
- **Multi-target builds** for desktop, mobile, and export functionality
- **Type-safe development** with comprehensive TypeScript configuration
- **Component-based UI** with event-driven interactions

### Backend Architecture  
- **RESTful API design** with JSON request/response format
- **Event-driven architecture** using EventBus for loose coupling
- **Block-based data model** for flexible content structure
- **Plugin system** for extensibility

### Cross-platform Support
- **Electron** for desktop (Windows, macOS, Linux)
- **Mobile frameworks** for iOS, Android, HarmonyOS
- **ARM64 support** for Apple Silicon and ARM-based systems
- **Docker deployment** available via Dockerfile

## 🛠️ Development Environment

### Code Quality
- **ESLint configuration** with TypeScript-specific rules
- **Consistent code formatting** across the project
- **Type checking** with TypeScript compiler
- **Build optimization** with webpack bundle analysis

### Build Tools
- **Webpack loaders** for various file types:
  - `esbuild-loader` - Fast JavaScript/TypeScript transpilation
  - `sass-loader` - SCSS processing
  - `css-loader` - CSS module loading
  - `file-loader` - Asset handling
  - `html-loader` - HTML template processing

### Development Scripts
- **Linting**: `pnpm run lint` with auto-fix
- **Type generation**: `pnpm run gen:types`
- **Development server**: Multiple dev modes for different platforms
- **Distribution**: Automated builds for all supported platforms

## 📱 Platform-Specific Features

### Desktop (Electron)
- Native file system access
- System clipboard integration
- Window management and customization
- Native menus and shortcuts

### Mobile Platforms
- Touch-optimized interfaces
- Platform-specific build configurations
- Mobile-specific optimizations in webpack config

### Cross-Platform APIs
- Unified data synchronization
- Consistent plugin system
- Shared business logic in Go backend

## 🔄 Data Flow Architecture

### Client-Server Communication
1. **Frontend (TypeScript/Electron)** ↔ **Backend (Go/REST API)**
2. **Event-driven updates** via EventBus system
3. **Real-time synchronization** with WebSocket connections
4. **Block-based data model** for content management

### Plugin Integration
- **JavaScript/TypeScript plugins** for frontend extensions
- **Go-based kernel extensions** for backend functionality
- **Event hooks** for plugin lifecycle management
- **API access** for plugin development

## 🎯 AI Development Context

### Code Generation Patterns
- **TypeScript interfaces** for type-safe API interactions
- **Go struct definitions** for data models
- **Webpack configuration patterns** for build customization
- **Event handling patterns** for reactive programming

### Common Development Tasks
- **API endpoint implementation** following REST conventions
- **Plugin development** using established hooks and patterns
- **UI component creation** with TypeScript and SCSS
- **Database operations** using block-based architecture

### Testing and Quality Assurance
- **Linting enforcement** with ESLint configuration
- **Type checking** with TypeScript compiler
- **Build verification** across multiple platforms
- **Bundle optimization** with webpack analysis

---

*This technical stack analysis is designed to provide AI systems with comprehensive context about SiYuan's architecture, enabling more accurate code suggestions and development assistance.*