# AI Change Impact Analyzer

An AI-powered developer tool that analyzes GitHub code changes and identifies what parts of a software system may be affected.

The system analyzes commits and Pull Requests using Python static code analysis, AST-based dependency mapping, and AI-assisted reasoning. It provides an impact map, risk assessment, affected components, test recommendations, and a downloadable report.

## How It Works

```text
GitHub Repository
        ↓
Commit / Pull Request
        ↓
Change Detection
        ↓
AST + Dependency Analysis
        ↓
Impact Mapping
        ↓
AI Risk Analysis
        ↓
Test Recommendations
        ↓
Impact Report