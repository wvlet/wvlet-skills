# Wvlet Skills Marketplace

This is a Claude Code plugin marketplace providing skills for the Airframe ecosystem and Wvlet query language.

## Marketplace Structure

```
.claude-plugin/
└── marketplace.json         # Marketplace manifest
plugins/
├── airframe-skills/         # Airframe plugin
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       ├── writing-airspec/
│       ├── using-di/
│       └── logging/
└── wvlet-skills/            # Wvlet plugin
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        ├── writing-wvlet/
        └── testing-wvlet/
```

## When Editing Skills

- Follow the [Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Keep SKILL.md concise - Claude is smart, avoid over-explaining
- Write descriptions in third person that include both what the skill does and when to use it
- Keep SKILL.md body under 500 lines
- Use forward slashes for all file paths

## Key Technologies

- **AirSpec**: Functional testing framework for Scala using `test("...") { ... }` syntax
- **Airframe DI**: Constructor-based dependency injection with Design and Session
- **airframe-log**: Logging library using `LogSupport` trait
- **Wvlet**: Flow-style query language (alternative to SQL)

## Reference Documentation

- AirSpec: https://wvlet.org/airframe/docs/airspec
- Airframe DI: https://wvlet.org/airframe/docs/airframe-di
- airframe-log: https://wvlet.org/airframe/docs/airframe-log
- Wvlet: https://wvlet.org/wvlet/docs/syntax/
