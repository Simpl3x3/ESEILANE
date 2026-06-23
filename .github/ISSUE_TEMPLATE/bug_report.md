---
name: Bug Report
about: Create a report to help us improve ESEILANE
title: "[BUG] "
labels: ["bug", "needs-triage"]
assignees: []
---

## Bug Description

A clear and concise description of what the bug is.

## Steps to Reproduce

1. Go to '...'
2. Run query '...'
3. See error

## Expected Behavior

A clear description of what you expected to happen.

## Actual Behavior

What actually happened. Include full error messages and stack traces.

## Environment

- **ESEILANE Version**: [e.g. 1.2.0]
- **OS**: [e.g. Ubuntu 22.04, macOS 14]
- **Python Version**: [e.g. 3.11.5] (if applicable)
- **Node.js Version**: [e.g. 20.10.0] (if applicable)
- **Deployment**: [Docker / Bare Metal / Kubernetes]

## Minimal Reproducible Example

```python
# Paste your minimal code example here
from eseilane import ESEILANE
db = ESEILANE(host='localhost', port=6379)
# ...
```

## Additional Context

Add any other context about the problem here (logs, screenshots, config files).
