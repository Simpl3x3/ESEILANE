# Contributing to ESEILANE

Thank you for your interest in contributing to ESEILANE! We welcome all forms of contributions — bug fixes, new features, documentation improvements, and more.

## Getting Started

### Prerequisites

- Node.js 20+
- Python 3.11+
- Docker (for running a local ESEILANE instance)
- Rust (for core engine contributions)

### Development Setup

1. Fork the repository on GitHub
2. Clone your fork locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/ESEILANE.git
   cd ESEILANE
   ```
3. Install dependencies:
   ```bash
   pip install -e ".[dev]"
   npm install
   ```
4. Start a local ESEILANE instance:
   ```bash
   docker run -p 6379:6379 eseilane/eseilane:latest
   ```
5. Run tests:
   ```bash
   pytest tests/
   npm test
   ```

## How to Contribute

### Reporting Bugs

- Check [existing issues](https://github.com/Simpl3x3/ESEILANE/issues) first.
- Use the **Bug Report** issue template.
- Include a minimal reproducible example.
- Specify your OS, runtime versions, and ESEILANE version.

### Suggesting Features

- Open a [GitHub Discussion](https://github.com/Simpl3x3/ESEILANE/discussions) for large features first.
- Use the **Feature Request** issue template for smaller enhancements.

### Submitting Pull Requests

1. Create a branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
2. Make your changes with clear, focused commits.
3. Add or update tests as needed.
4. Update documentation if applicable.
5. Push your branch and open a PR against `main`.
6. Fill in the PR template completely.

## Code Style

- **Python**: Follow PEP 8. Use `black` for formatting.
- **TypeScript**: Use ESLint + Prettier config provided.
- **Rust**: Follow standard Rust formatting (`cargo fmt`).
- Write descriptive commit messages following [Conventional Commits](https://www.conventionalcommits.org/).

## Code of Conduct

This project follows our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold it.

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
