# Documentation

This directory contains documentation for the Open Deep Research project, including architecture overviews, feature documentation, and improvement tracking.

## Directory Structure

```
docs/
├── README.md                     # This file
└── improvements/                 # Major feature additions and enhancements
    └── plot-generation-feature.md   # Plot generation and visualization feature
```

## Purpose

The `docs/` directory serves as a centralized location for:

1. **Technical Documentation**: Detailed explanations of system architecture and design decisions
2. **Feature Documentation**: Comprehensive guides for major features and capabilities
3. **Improvement Tracking**: Records of significant enhancements and their implementation details
4. **Change History**: Documentation of major changes and their rationale

## Improvements Directory

The `improvements/` subdirectory contains detailed documentation for major features and enhancements added to the system. Each improvement document typically includes:

- **Overview**: High-level description of the feature
- **Motivation**: Why the feature was needed
- **Architecture Changes**: Technical implementation details
- **Workflow Diagrams**: Visual representations of how the feature works
- **Example Usage**: Practical examples demonstrating the feature
- **Testing Recommendations**: How to validate the feature works correctly
- **Files Modified**: List of all files changed and their locations

### Current Improvements

- **[Plot Generation Feature](improvements/plot-generation-feature.md)** (2025-10-12): Adds automated visualization and plot generation capabilities to research reports

## Contributing Documentation

When adding new features or making significant changes:

1. Create a new markdown file in the appropriate subdirectory
2. Follow the existing documentation structure and style
3. Include code examples and diagrams where helpful
4. Reference specific file locations and line numbers
5. Update this README to include links to new documentation

## Documentation Standards

- Use markdown format (`.md` files)
- Include a date stamp at the top of each document
- Use clear headings and subheadings
- Provide code examples with syntax highlighting
- Link to relevant files in the codebase using relative paths
- Include diagrams for complex workflows
- Keep language clear and concise

## Additional Resources

For more information about the project:
- See the main [README.md](../README.md) in the project root
- Check the [source code](../src/open_deep_research/) for implementation details
- Review [tests](../tests/) for usage examples
