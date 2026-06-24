# Brad's Health Tracker

A Cowork **plugin marketplace** + source repo for my Health Tracker system: log food,
exercise, and digestion (gut symptoms) to plain local Markdown files, with a visual
dashboard and live Strava support.

## Install (for friends)

In Cowork / Claude Code:

```
/plugin marketplace add pbrosen/brads-strava-health-tracker
/plugin install health-tracker@brad-health
```

Then follow `health-tracker/README.md` for the 3-minute setup (make a `~/HealthTracker`
folder, install the one-click Filesystem extension, open the dashboard).


## What's in here

```
.
├── .claude-plugin/
│   └── marketplace.json        # the marketplace manifest (lists the plugins below)
├── health-tracker/             # the installable plugin (friend-facing build)
│   ├── .claude-plugin/plugin.json
│   ├── .mcp.json               # bundled local-files connector
│   ├── skills/health-log/      # logging skill + dashboard.html
│   ├── README.md
│   └── CONNECTORS.md
└── personal-dashboard/         # my own private Food Tracker source (not published as a plugin)
    └── README.md               # how to drop my index.html here
```

## Versioning

- Bump `version` in `health-tracker/.claude-plugin/plugin.json` (and the matching entry in
  `marketplace.json`) each time you ship a change. Use semver: `0.1.0` → `0.1.1` for fixes,
  `0.2.0` for features.
- Tag releases in git: `git tag v0.1.0 && git push --tags`.
- For a **different friend's** customized build, either bump the version, or add a second
  entry to `marketplace.json` with its own `name` and `source` folder (e.g. `health-tracker-sam`).
