# SPARC Development Environment

This directory contains SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) methodology templates and workflows.

## Structure

- `templates/` - SPARC mode templates
- `workflows/` - Development workflow definitions  
- `modes/` - Mode-specific configurations
- `configs/` - Environment configurations

## Usage

Use claude-flow commands to execute SPARC modes:

```bash
# List available modes
bash ../claude-flow.sh sparc modes

# Run specific mode
bash ../claude-flow.sh sparc run <mode> "task description"

# Execute TDD workflow  
bash ../claude-flow.sh sparc tdd "feature implementation"
```