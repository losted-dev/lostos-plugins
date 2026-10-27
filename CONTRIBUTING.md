# Contributing a plugin

Anybody can add a plugin or fix one. You sign your own package; the lostOS team reviews the pull request, and a robot signs the list of the registry after the merge.
Nothing in this repository is trusted until it is signed, and a signature never replaces the review.

## 1. Make a key (once)

```
python ci/lostos-sign.py keygen my-key.pem
```

It writes the private key to the file and prints the public key and its fingerprint. **Keep the file secret and make a copy**: whoever has it can publish updates under your name.
If it is lost or leaks, tell the team: the key is put on `revoked.json` and you make a new one.

## 2. Build and sign the package

A plugin is a zip with `manifest.json`, your program and a `README.md`; the format is in the docs of lostOS (`docs/plugins.md` in the repository `lostos-server`).

```
cd my-plugin && zip -r ../ru.example.demo-1.0.0.lstplugin . && cd ..
python ci/lostos-sign.py sign ru.example.demo-1.0.0.lstplugin my-key.pem "Your name"
```

The file name must be `<id>-<version>.lstplugin` and say the same as the manifest. The signature covers every file of the package: change one byte afterwards and it no longer matches.

## 3. Open the pull request

- Put the package into `plugins/`.
- **A new developer** also adds the public key to `developers.json`: `{"id": "your-id", "name": "Your name", "key": "<public key>"}`, and says in the pull request who you are. The team checks that the key is yours and **confirms you on plugapi.losted.dev**, the signed list of verified developers: speakers trust a developer for that list, not for this file. A confirmation (and a revocation) reaches every server within seconds. Until you are confirmed there, your packages install as unknown.
- **An update** is a new version of the same id: a higher version number, a new file. A published version is never changed or replaced.
- Do not touch `index.json` or `index.json.sig`: the robot makes them.

## 4. What happens

1. **The check** (GitHub Actions) opens your package with the same code the server uses to install it, and tells you what is wrong: no signature, a key that is not in `developers.json`, a package changed after signing, a bad manifest or an unknown permission, a path that climbs out of the folder, a key or a token inside a file, a version that is not higher than the published one.
   The report is a comment under the pull request. You can run it yourself: `python ci/registry_ci.py validate . --base <a checkout of main>` (it needs a checkout of `lostos-server`: `LOSTOS_SERVER=../lostos-server/server`).
2. **A person reads it.** The report lists the permissions that the plugin asks for and which of them are new in this update; the permissions that see private data or reach outside (`speech.*`, `assistant.*`, `network.*`, `alice.api`, `brain.replace` and the like) are looked at with the code. A new developer is checked once.
3. **After the merge** the release robot checks everything again, builds `index.json` and signs it. The key lives in a protected environment of this repository: every signing waits for a maintainer.
   The servers of lostOS trust that key for the index only; it can not sign a package.
4. A speaker installs your plugin only if the index is signed, the file matches the hash in the index and your signature is valid and your key is on the list of verified developers (plugapi.losted.dev, signed by the team). The owner still has to switch the plugin on and allow its permissions.

## Fixing a plugin of somebody else

Send the fix as a new version signed with **your** key. The package then says it is yours; the original author or the team merges it. A built-in plugin of lostOS (AirPlay, Spotify, the stock brain) is signed by the team; a fix goes through a pull request to `lostos-server`, and the team signs the folder again.
A copy that you edited for yourself and install by hand is shown as "unsigned" or "signed by ..., not from the registry"; it works, and the owner of the speaker decides.

## Licence

By contributing you agree that your contribution is under the licence of this repository (MIT).
