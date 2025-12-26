# Wvlet Skills Marketplace

A Claude Code plugin marketplace providing skills for the [Airframe](https://wvlet.org/airframe/) ecosystem and [Wvlet](https://wvlet.org/wvlet/) query language.

## Plugins

### airframe-skills

Skills for the Airframe ecosystem:

| Skill | Description |
|-------|-------------|
| writing-airspec | Write test cases using AirSpec testing framework |
| using-di | Dependency injection with Airframe DI |
| logging | Logging with airframe-log |

### wvlet-skills

Skills for Wvlet query language:

| Skill | Description |
|-------|-------------|
| writing-wvlet | Write queries using Wvlet flow-style query language |
| testing-wvlet | Write tests for Wvlet queries |

## Installation

### 1. Add the marketplace

```bash
/plugin marketplace add wvlet/wvlet-skills
```

### 2. Install plugins

```bash
# Install Airframe skills
/plugin install airframe-skills@wvlet-skills

# Install Wvlet skills
/plugin install wvlet-skills@wvlet-skills

# Or install both
/plugin install airframe-skills@wvlet-skills wvlet-skills@wvlet-skills
```

## Usage

Once installed, Claude will automatically use these skills when relevant:

- Writing Scala tests → `writing-airspec` skill activates
- Setting up dependency injection → `using-di` skill activates
- Adding logging → `logging` skill activates
- Writing Wvlet queries → `writing-wvlet` skill activates

## Resources

- [Airframe Documentation](https://wvlet.org/airframe/)
- [Wvlet Documentation](https://wvlet.org/wvlet/)
- [Claude Code Plugins](https://code.claude.com/docs/en/plugins)

## License

Apache 2.0
