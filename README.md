# Poseidon Plugins

Official plugin repository for the **T-DIGI POSSEIDON** desktop app.

This repository hosts the plugin catalog and the plugin ZIP archives that the
app's marketplace fetches, validates and installs at runtime.

## What this is

T-DIGI POSSEIDON ships with a plugin marketplace. A repository (like this one)
is just a static HTTP host that exposes:

- a catalog file at `index.json` describing the available plugins
- the plugin ZIP archives at the paths the catalog points at

When a user adds this repository's URL to their app, the marketplace
downloads `index.json`, lists the plugins, and lets the user install each one
with a single click. ZIP files are downloaded on demand, never bundled with
the app itself.

## Layout

```
poseidon-plugins/
├── index.json          # Catalog (required)
├── apps/               # Plugin ZIPs
│   ├── music/
│   │   └── music_1-0-0.zip
│   └── digibox/
│       └── digibox_1-0-0.zip
└── README.md
```

## Catalog format

`index.json` is the contract between this repository and the host app.
Required top-level fields:

| Field         | Type      | Notes                                                                 |
|---------------|-----------|-----------------------------------------------------------------------|
| `version`     | int       | Schema version, currently `1`                                         |
| `title`       | string    | Repository display name shown in the host's repository list          |
| `description` | string    | Repository tagline shown alongside the name                          |
| `plugins`     | array     | One entry per plugin (see below)                                      |

Each plugin entry:

| Field         | Type      | Notes                                                                          |
|---------------|-----------|--------------------------------------------------------------------------------|
| `id`          | string    | Stable plugin identifier (matches `manifest.json` inside the ZIP)              |
| `name`        | string    | Display name                                                                   |
| `version`     | string    | Plugin version (the host compares vs. the installed version to detect updates) |
| `description` | string    | Short description shown on marketplace cards                                   |
| `author`      | string    | Author name (used by the marketplace's author filter)                          |
| `tags`        | string[]  | Free-form tags (used by the marketplace's tag filter)                          |
| `website`     | string    | Optional homepage URL                                                          |
| `download`    | string    | URL of the plugin ZIP — relative paths resolve against this repo's base URL    |

Example:

```json
{
  "version": 1,
  "title": "Poseidon Plugins",
  "description": "Official plugin repository shipped with t-digi-posseidon.",
  "plugins": [
    {
      "id": "music",
      "name": "Music",
      "version": "1.0.0",
      "description": "Local audio player and playlist manager.",
      "author": "Andet Tafa",
      "tags": ["Music", "Audio", "Player", "Playlist"],
      "website": "https://github.com/a-digi/poseidon-plugins",
      "download": "https://raw.githubusercontent.com/a-digi/poseidon-plugins/main/apps/music/music_1-0-0.zip"
    }
  ]
}
```

## Adding this repository to the app

In T-DIGI POSSEIDON: **Plugins → Marketplace → Manage repositories → Add**, then paste:

```
https://raw.githubusercontent.com/a-digi/poseidon-plugins/main/
```

The host fetches `index.json`, validates it, and stores the repository under
the `title` and `description` declared in the catalog.

This repository is included in the app's built-in trust whitelist, so it
appears as **Verified** and is auto-seeded on first launch — no manual add
required for users on a stock install.

## Trust model

- Repositories shipped with the app (whitelisted at build time) are marked
  **Verified** and cannot be removed by the user.
- Any other repository the user adds triggers a one-shot warning dialog
  (*"This repository is not in the verified list. Plugins from it may
  contain malicious code."*). The user must explicitly confirm before the
  repository is stored or any plugin is installed from it.
- Installation always validates the plugin manifest and the ZIP layout
  before extracting; the same checks apply to any source.

## Adding a new plugin to this repository

1. Build your plugin into a single ZIP that contains a top-level `manifest.json`
   plus a `ui/` folder and (optionally) a `backend/` folder.
2. Drop the ZIP under `apps/<plugin-id>/<plugin-id>_<version>.zip`.
3. Add an entry for the plugin in `index.json` with absolute or relative
   `download` URLs.
4. Commit and push.

The host's marketplace re-fetches the catalog on every refresh, so changes
appear within seconds of the push being available on the static host.
