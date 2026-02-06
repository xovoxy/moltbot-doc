# Slack @mentions in API messages (OpenClaw)

When OpenClaw (or any Slack app/bot) sends messages via the Slack API, **plain text `@name` does not reliably create a real mention**.

To mention users/bots/apps in a way that Slack will render as a proper mention (blue, clickable, notifies the target), you should use **Slack’s canonical mention markup**.

## User / bot mention (recommended)

Use the member’s Slack ID (starts with `U`):

```text
<@U1234567890>
```

Examples:

```text
Hello <@U1234567890>, can you take a look?
```

Notes:
- Bots and “apps” that can be mentioned in Slack are still represented as a **user-like ID** in messages.
- IDs are stable even if display names change.

## User group mention

User groups use a different token:

```text
<!subteam^S12345678>
```

Sometimes you may see a label form:

```text
<!subteam^S12345678|group-handle>
```

## Special mentions

```text
<!here>
<!channel>
<!everyone>
```

These can be restricted by workspace policy and are easy to spam—use sparingly.

## Why `@name` fails

Slack does not always “linkify” `@something` in bot-authored messages, especially when:
- the target is not uniquely resolvable,
- the message is generated programmatically,
- or formatting is not in the exact markup Slack expects.

Using IDs removes ambiguity.

## Practical guidance for OpenClaw

- Prefer storing Slack member IDs and mentioning with `<@MEMBER_ID>`.
- If you need to mention someone from code/config, do **not** assume their display name is mentionable.

## JSON escaping tips

If you embed this in a JSON config, keep the string intact. Example:

```json
{
  "systemPrompt": "When you mention someone in Slack, use the canonical format <@U123...> rather than @name."
}
```

## Getting IDs

- In Slack: open the user’s profile → “More” → **Copy member ID** (wording may vary).
- With sufficient scopes, you can also look up IDs via `users.info` / `users.list`.
