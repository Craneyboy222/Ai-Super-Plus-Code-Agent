# Installation & Setup Guide

## Prerequisites

- Claude Code (latest version)
- Access to Claude Opus 4.6, Sonnet 3.5, Haiku 4.5 models
- 4GB+ RAM recommended
- Internet connection (for API calls)

---

## Installation

### 1. Clone/Copy the Plugin

```bash
# Copy the entire directory to your Claude Code plugins folder
cp -r ai-super-plus-code-agent ~/.claude-code/plugins/
```

### 2. Verify Installation

```bash
# Check if plugin is recognized
claude-code plugins list
# Output should include: ai-super-plus-code-agent (2.0.0)
```

### 3. Initialize the Plugin

```bash
# In any Claude Code session, activate the plugin
/plugin ai-super-plus-code-agent

# You should see:
# ✓ AI SUPER PLUS Code Agent v2.0.0 activated
# ✓ 13 agents loaded
# ✓ 14-phase pipeline ready
# ✓ All references loaded
```

---

## Verification Checklist

```
✓ Plugin loads without errors
✓ All commands available (/build, /ship, /quick, /deep, etc.)
✓ All agents registered (13 total)
✓ All reference files accessible
✓ No missing dependencies
✓ Models accessible (Opus, Sonnet, Haiku)
✓ Ready to use
```

---

## Quick Start: Build Your First Project

### Step 1: Provide Requirements

```
Hey, I want to build a blog platform with:
- User authentication (email/password)
- Post creation and editing
- Comments on posts
- Search functionality
- Admin dashboard
- Multi-tenant support

Can you build this with Next.js + Express + PostgreSQL?
```

### Step 2: Execute Build

```
/build

# Agent (Architect):
# Analyzing requirements...
# Selected: Monolith + Next.js + Express + PostgreSQL
# Estimated time: 14 hours
# Starting build...
```

### Step 3: Monitor Progress

```
Phase 0-3: ████░░░░░░ (Design & Scaffold)
Phase 4-7: ████████░░ (Core Code Generation)
Phase 8-9: ███████████ (Infrastructure & Tests)
Phase 10-11: ██████████░ (Security & Quality)
Phase 12-13: ███████████ (Documentation & Ship)

✓ 150+ files generated
✓ 600+ tests (86% coverage)
✓ Security audit: PASSED
✓ Ready to deploy!
```

### Step 4: Review Output

```bash
# Navigate to generated project
cd blog-platform/

# Review structure
ls -la

# Review README
cat README.md

# Review tests
npm test

# Try local development
npm run dev
```

---

## Command Reference

### Build Commands

```
/build          Build complete project (all 14 phases)
/ship           Full pipeline with deployment (same as /build)
/quick          Fast implementation (skip some phases)
/deep           Maximum rigor (extra checks)
/scaffold       Project scaffolding only
/implement      Implement single feature
```

### Operation Commands

```
/test           Generate test suites
/review         Production code review
/secure         Security audit + fixes
/deploy         Generate deployment configs
/document       Generate documentation
/optimize       Performance optimization
/debug          Systematic debugging
/refactor       Code quality improvement
```

---

## Configuration

### Default Settings (plugin.json)

```json
{
  "config": {
    "defaultModel": "claude-opus-4-6",
    "codeGenMode": "production",
    "minCodeCoverage": 0.8,
    "enforceTypeScript": true,
    "enforceTests": true,
    "enforceDocumentation": true,
    "securityLevel": "high",
    "productionReady": true
  }
}
```

### Customization

To modify settings:

1. Edit `.claude-plugin/plugin.json`
2. Adjust parameters:
   - `minCodeCoverage`: 0.0 to 1.0 (default: 0.8)
   - `enforceTypeScript`: true/false (default: true)
   - `securityLevel`: "low"/"medium"/"high" (default: "high")
   - `productionReady`: true/false (default: true)

---

## Common Issues

### Issue: Plugin not loading

**Solution**:
```bash
# Verify plugin directory structure
ls -la ~/.claude-code/plugins/ai-super-plus-code-agent/
# Should contain: .claude-plugin, agents, commands, skills, README.md

# Check plugin.json is valid
cat .claude-plugin/plugin.json | jq .

# Restart Claude Code
```

### Issue: Agents not responding

**Solution**:
```bash
# Verify model access
/test-models

# Check which models are available
/status

# Restart plugin
/plugin ai-super-plus-code-agent
```

### Issue: Out of memory during generation

**Solution**:
```bash
# Use /quick mode instead of /build
/quick blog-project

# Or break into smaller features
/implement "User authentication"
/implement "Post CRUD"
/implement "Comments"
```

### Issue: Tests failing

**Solution**:
```bash
# Run with increased verbosity
/test --verbose

# Review test output
npm test 2>&1 | head -50

# Contact support with output
```

---

## Performance Tips

1. **Use /quick for rapid prototyping**
   - Skips some quality checks
   - Completes in 4-6 hours
   - Good for MVPs

2. **Use /deep for production systems**
   - Extra security checks
   - Extra performance testing
   - Takes 16+ hours
   - Best for critical systems

3. **Build during off-peak hours**
   - Faster API responses
   - More available tokens
   - Better completion time

4. **Break large projects into phases**
   ```
   Phase 1: Core functionality (/quick)
   Phase 2: Add features (/implement)
   Phase 3: Security hardening (/secure)
   Phase 4: Optimization (/optimize)
   Phase 5: Full ship (/ship)
   ```

---

## Support & Resources

### Documentation
- README.md — Overview
- SKILL.md — System design
- References/ — Detailed guides

### Getting Help
1. Check SUMMARY.md for overview
2. Review relevant reference file
3. Review agent specification
4. Check troubleshooting section above

### Reporting Issues
Include:
- Plugin version
- Command used
- Requirements provided
- Error message
- Expected vs actual output

---

## Upgrade Guide

### From Version 1.0

```bash
# Backup old version
mv ~/.claude-code/plugins/ai-super-plus-code-agent ~/.claude-code/plugins/ai-super-plus-code-agent-v1

# Install new version
cp -r ai-super-plus-code-agent ~/.claude-code/plugins/

# Verify
claude-code plugins list | grep ai-super-plus
```

### Breaking Changes
- Version 2.0 requires all tests to pass
- Version 2.0 enforces TypeScript by default
- Version 2.0 includes security hardening

### Migration Path
1. Backup existing projects
2. Review new documentation
3. Run /build on new projects
4. Update existing projects gradually

---

## Verification

After installation, run:

```
/verify-installation

# Output should show:
✓ Plugin loaded
✓ All agents ready
✓ All commands available
✓ Reference files loaded
✓ Models accessible
✓ Ready to generate!
```

---

## Next Steps

1. ✓ Installation complete
2. Read `/README.md` for overview
3. Choose your first project
4. Run `/build` or `/quick`
5. Deploy and enjoy!

---

**Version**: 2.0.0
**Last Updated**: 2026-02-18
**Status**: Ready for Production
