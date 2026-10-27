# lostOS Plugins

The registry of plugins for lostOS. A plugin is a `.lstplugin` file (a zip with `manifest.json`); the lostOS server reads `index.json`, checks that it is signed by the lostOS team (`index.json.sig`),
downloads a package, checks its SHA-256 from the index and the signature inside the package (`lostos-signature.json`), and only then installs it. A plugin runs as a process of its own with only the
permissions its owner allows.

```
developers.json   the developers whose signatures are trusted: [{"id": "...", "name": "...", "key": "<public key>"}]
revoked.json      fingerprints of keys that are no longer trusted
plugins/          signed packages: <id>-<version>.lstplugin
index.json        the list of packages (made by tools/lostos-sign.py index)
index.json.sig    the signature of the team under index.json
```

## Adding a plugin

Anybody can. You sign your own package, a robot checks the pull request, a person of the team reads it, and after the merge a second robot signs `index.json`. The whole way is in **CONTRIBUTING.md**.

```
python ci/lostos-sign.py keygen my-key.pem                              # once; keep the file secret
python ci/lostos-sign.py sign my-plugin.lstplugin my-key.pem "My name"  # then a pull request with the package in plugins/ (and your public key in developers.json)
```

`ci/` holds the tools and the checks that GitHub Actions runs (`.github/workflows/`): `pip install cryptography` is all you need to sign. License: MIT.
