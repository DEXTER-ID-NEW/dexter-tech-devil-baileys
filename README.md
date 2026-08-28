# @dexterid/baileys 2.2.6 — iOS buttons / no auto-follow build

This ZIP is a full copy of `@dexterid/baileys` 2.2.6 with:

- the unsolicited automatic newsletter follow removed;
- high-level legacy lists, template buttons, and old button objects converted to native-flow messages;
- iOS-oriented message context and private-chat bot metadata added;
- `viewOnce` wrappers disabled for interactive button messages, because recent iOS/Web clients can hide controls inside that wrapper;
- the original documentation preserved as `README.original.md`.

> **Important:** WhatsApp controls the final rendering. The stable button types below are the best-supported path for recent Android and iOS clients, but no unofficial WhatsApp Web library can guarantee every experimental flow on every app version. Update WhatsApp on the receiving iPhone before testing.

## WhatsApp contact

**DEXTER TECH DEVIL:** [+94 78 995 8225](https://wa.me/94789958225)

## Installation

Extract the ZIP and install the extracted package directory:

```bash
npm install ./dexter-tech-devil-baileys-2.2.6-ios-buttons-no-auto-follow
```

Or copy the extracted package into your project and import it normally:

```js
import makeWASocket from '@dexterid/baileys'
```

Do not add `viewOnce: true` to button messages.

## Recommended iOS-safe API

Use `interactiveButtons` with native-flow entries. Always make `buttonParamsJson` a JSON string.

```js
await sock.sendMessage(jid, {
  text: 'Choose an option',
  footer: 'My Bot',
  interactiveButtons: [
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({
        display_text: 'Yes',
        id: 'yes'
      })
    },
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({
        display_text: 'No',
        id: 'no'
      })
    }
  ]
})
```

For maximum compatibility, start with one or two `quick_reply` buttons in a private chat. Then test the other stable types one at a time.

## Native-flow button types

| Name | Use | Required parameters |
|---|---|---|
| `quick_reply` | Sends an ID back to the bot | `display_text`, `id` |
| `single_select` | Opens a list/menu drawer | `title`, `sections[].rows[]` |
| `cta_url` | Opens a website | `display_text`, `url` |
| `cta_call` | Opens the phone dialer | `display_text`, `phone_number` |
| `cta_copy` | Copies text to clipboard | `display_text`, `copy_code` |

The package also accepts experimental/special names such as `cta_catalog`, `send_location`, `open_webview`, `mpm`, payment flows, and reminder flows. These depend on WhatsApp account/client capability and are **not guaranteed on iOS**.

## Single-select list on iOS

A normal high-level `sections` message is converted by this build into a native-flow `single_select` button:

```js
await sock.sendMessage(jid, {
  text: 'Please select a service',
  title: 'Services',
  footer: 'My Bot',
  buttonText: 'Open menu',
  sections: [
    {
      title: 'Main services',
      rows: [
        {
          id: 'support',
          title: 'Customer support',
          description: 'Talk to our support team'
        },
        {
          id: 'pricing',
          title: 'Pricing',
          description: 'View plans and prices'
        }
      ]
    }
  ]
})
```

For direct native-flow control, use this form:

```js
await sock.sendMessage(jid, {
  text: 'Please select a service',
  title: 'Services',
  footer: 'My Bot',
  interactiveButtons: [
    {
      name: 'single_select',
      buttonParamsJson: JSON.stringify({
        title: 'Open menu',
        sections: [
          {
            title: 'Main services',
            rows: [
              { id: 'support', title: 'Customer support', description: 'Talk to support' },
              { id: 'pricing', title: 'Pricing', description: 'View plans' }
            ]
          }
        ]
      })
    }
  ]
})
```

## URL, call, and copy buttons

```js
await sock.sendMessage(jid, {
  text: 'More actions',
  footer: 'My Bot',
  interactiveButtons: [
    {
      name: 'cta_url',
      buttonParamsJson: JSON.stringify({
        display_text: 'Open website',
        url: 'https://example.com',
        merchant_url: 'https://example.com'
      })
    },
    {
      name: 'cta_call',
      buttonParamsJson: JSON.stringify({
        display_text: 'Call support',
        phone_number: '+94789958225'
      })
    },
    {
      name: 'cta_copy',
      buttonParamsJson: JSON.stringify({
        display_text: 'Copy code',
        copy_code: 'ABC-123'
      })
    }
  ]
})
```

For some iOS versions, sending one button type per message is more reliable than mixing three different flow types. If a mixed message is hidden, split it into separate messages.

## Image/video with native buttons

```js
await sock.sendMessage(jid, {
  image: { url: 'https://example.com/banner.jpg' },
  caption: 'Choose an action',
  footer: 'My Bot',
  interactiveButtons: [
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({
        display_text: 'Get started',
        id: 'start'
      })
    },
    {
      name: 'cta_url',
      buttonParamsJson: JSON.stringify({
        display_text: 'Open docs',
        url: 'https://example.com/docs',
        merchant_url: 'https://example.com/docs'
      })
    }
  ]
})
```

## Compatibility conversion for old code

This build converts these high-level inputs to native-flow where possible:

- `sections` → `single_select`;
- old `{ buttonId, buttonText }` objects → `quick_reply`;
- `templateButtons` URL/call/quick-reply entries → `cta_url`/`cta_call`/`quick_reply`;
- `buttons` entries containing `nativeFlowInfo`, `sections`, or old quick-reply fields → native-flow buttons.

Prefer the explicit `interactiveButtons` form for new code. Do not send raw `buttonsMessage`, raw `listMessage`, or raw `templateMessage` objects when iOS compatibility matters.

## Reading button responses

```js
sock.ev.on('messages.upsert', ({ messages }) => {
  const message = messages?.[0]
  const response = message?.message?.interactiveResponseMessage
  const flow = response?.nativeFlowResponseMessage
  if (!flow) return

  let params = {}
  try {
    params = JSON.parse(flow.paramsJson || '{}')
  } catch {
    // Ignore malformed/empty response payloads.
  }

  console.log('button name:', flow.name)
  console.log('button params:', params)

  // A quick reply normally returns an ID in params.id.
  // A single_select response normally contains the selected row ID.
})
```

Depending on the WhatsApp client version, the response shape may contain an additional wrapper. Log the full `message` while integrating and inspect `interactiveResponseMessage.nativeFlowResponseMessage`.

## iOS troubleshooting checklist

1. Update WhatsApp on the iPhone.
2. Test in a private one-to-one chat first.
3. Remove `viewOnce: true`.
4. Use `interactiveButtons`, not raw legacy `buttonsMessage`/`listMessage`/`templateMessage`.
5. Confirm every `buttonParamsJson` is produced by `JSON.stringify(...)`.
6. Test `quick_reply` first, then `single_select`, then URL/call/copy separately.
7. Use a valid E.164 phone number for `cta_call`.
8. For `cta_url`, provide both `url` and `merchant_url`.
9. If Android shows it but iOS does not, update both WhatsApp clients and try a one-type-per-message payload.
10. Native-flow messages are unofficial when sent through Baileys. For guaranteed, supported business messaging, use the official WhatsApp Business Platform/API.

## Security note

The automatic newsletter-follow behavior has been removed from this archive. The manual newsletter methods remain available. This change is not a full security audit of every dependency in the package.
