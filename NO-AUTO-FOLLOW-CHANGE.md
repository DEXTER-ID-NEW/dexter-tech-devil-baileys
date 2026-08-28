# iOS-buttons / no-auto-follow build

Base package: `@dexterid/baileys` 2.2.9.

Changes in this archive:

- Removed the unsolicited automatic newsletter follow and 30-second follow interval.
- Converted high-level `sections` lists to native-flow `single_select` messages.
- Converted high-level old/template button shapes to native-flow buttons where possible.
- Added native-flow message context and a private-chat bot marker.
- Avoided view-once wrapping for interactive button messages.
- Added the focused button guide to `README.md`.
- Updated the public root `README.md` to the full package guide with installation, authentication, messaging, interactive buttons, media, groups, privacy, newsletters, events, and best practices.

This archive was prepared without running `npm install` or executing the package. JavaScript syntax was checked with `node --check`. WhatsApp/iOS rendering still depends on the receiving app version and server-side capability; experimental flows cannot be guaranteed.
