# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Iris is an Elixir tool for visualizing Elixir codebases through an interactive web interface. It analyzes compiled BEAM files to generate an interactive GUI showing applications, modules, methods, and call graphs.

## Common Development Commands

### Build and Development
```bash
# Install dependencies and setup
mix setup

# Compile the project
mix compile

# Generate Iris visualization for current project
mix iris

# Generate with verbose output
mix iris -v

# Build frontend assets
mix build
```

### Testing
```bash
# Run all tests
mix test

# Run a specific test file
mix test test/iris_test.exs

# Run tests with coverage
mix test --cover
```

### Code Quality
```bash
# Format code (always run as final step per user rules)
mix format

# Run static analysis
mix credo
```

### Frontend Development
```bash
# Navigate to assets directory and install dependencies
cd assets && npm install

# Build frontend assets
cd assets && npm run build

# Run linting on frontend
cd assets && npm run lint
```

## Architecture Overview

### Core Components

**Mix Task (`lib/mix/tasks/iris.ex`)**
- Entry point via `mix iris` command
- Handles CLI arguments and configuration
- Supports umbrella projects by collecting compile paths from all apps

**Core Engine (`lib/iris/core.ex`)**
- Main analysis engine that processes BEAM files
- Uses `:beam_disasm` to extract bytecode and call instructions
- Builds method call graphs (inbound/outbound calls)
- Handles method condensation (merges auto-generated inlined methods)
- Detects recursive methods and filters duplicates

**Entity Models (`lib/iris/entity.ex` and related)**
- `Entity.Application`: Top-level container for modules
- `Entity.Module`: Contains methods and documentation
- `Entity.Module.Method`: Individual functions with call instructions
- `Entity.Module.Method.Call`: Represents method calls with clickability info

**Documentation Integration (`lib/iris/doc_gen.ex`)**
- Leverages ExDoc for generating documentation
- Extracts module and method documentation
- Filters empty documentation nodes

### Frontend Architecture

**React + Vite Frontend (`assets/`)**
- React-based interactive UI using Flowbite React components
- Vite for fast development and building
- Tailwind CSS for styling with DaisyUI theme support
- Uses @xyflow/react for rendering call graphs
- Renders as single HTML file in `iris/index.html`

### Key Data Flow

1. **BEAM Analysis**: Reads compiled `.beam` files from build directory
2. **Call Graph Generation**: Extracts call instructions and builds bidirectional call maps
3. **Documentation Extraction**: Uses ExDoc to get module/method docs
4. **JSON Generation**: Serializes entity data to JSON for frontend
5. **Static Asset Generation**: Copies frontend files to `iris/` directory
6. **Web Visualization**: Opens `iris/index.html` for interactive exploration

### Method Classification

- **EXP**: Exported (public) methods
- **INT**: Internal (private) methods  
- **BIF**: Built-in functions (from `:erlang` module)
- **IMP**: Imported methods (from dependencies)

### Auto-generated Method Handling

The system handles Elixir compiler-generated methods by:
- Detecting patterns like `-inlined-method/arity-` and `-method/arity-fun-N-`
- Condensing these into their parent methods
- Preserving call instructions while removing duplicates

## Development Notes

### BEAM File Processing
- Uses `:beam_lib.chunks/2` to extract locals (private methods)
- Uses `:beam_disasm.file/1` to get complete bytecode analysis
- Supports both single apps and umbrella projects

### Frontend Integration
- Frontend expects global `getGlobalEntity()` function with project data
- All static files are copied to output `iris/` directory
- Final visualization is fully self-contained in `iris/index.html`

### Testing Strategy
- Main tests in `test/iris_test.exs` 
- Uses ExUnit with doctests
- Test by running `mix iris` on sample projects in `samples/`