# agent-purge persona

Your [agent-purge](https://github.com/ZiChuanLan/agent-purge) persona, as magpie gateway middleware:
every agent request that goes through magpie's gateway gets that persona in the API's own place for a
system prompt — Anthropic's `system`, Chat's first system or developer message, Responses'
`instructions`, Gemini's `systemInstruction`.

It reads agent-purge's data directory on each request (`AGENT_PURGE_HOME`, or `~/.agent-purge`):
`state.json` for the current persona and mode, `personas/<id>.md` for the text, with the
frontmatter stripped. So switching the persona in agent-purge applies to the next request — no
plugin reload, no magpie restart. Without agent-purge installed there is no `state.json`, and the
gateway leaves every request as it is.

```sh
magpie plugin add @magpie-community/middleware-agent-purge
```

In the app it is under Plugins › Discover › Gateway middleware, and it adds a **Persona** page under
Plugins in the sidebar: pick a persona there, or import your own `.md` (kept in the plugin's
options, not written to disk).

## Modes

| Mode | What the request's system segment becomes |
|---|---|
| `off` | untouched |
| `strict` | the persona only — the agent's own system prompt is dropped |
| `append` | the persona first, the agent's own system prompt kept after it |
| `replace` | like `append`, minus the lines of the agent's own prompt that name the model it is |

`append` is the default. It keeps the agent's prompt cache, so a persona costs one prefix write, not
a re-billed request.

## Options

```sh
magpie plugin options agent-purge '{"mode":"append","persona":"my-persona","agents":["claude"]}'
```

- `mode` — overrides the mode in `state.json`.
- `persona` — which persona id to use; the default is the one `state.json` has.
- `agents` — only these magpie agents (`["claude","codex"]`); empty is all of them.
- `inlinePersonas` — `[{id,name,text}]`, what the page imports. It wins over `personas/<id>.md` of
  the same id.

It is a [Bun gateway hook](https://usemagpie.ai/docs/plugins#gateway-request), not a moejs
middleware, because it reads files. The hook fails open: a persona it can't resolve, an unreadable
file or an unknown protocol leaves the request untouched.

## Checking it

```sh
node selftest.mjs   # request rewriting, per protocol and mode, against a temporary AGENT_PURGE_HOME
node test-page.mjs  # the page's scripts: what they read, what they write to options
```
