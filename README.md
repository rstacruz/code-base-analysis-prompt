# Codebase analysis prompt

Analyse any code base using modern LLM coding agents, inspired by [DeepWiki](https://deepwiki.com).

Check out some examples of its output:

- [OpenCode example](examples/opencode/index.md)
- [AnkiConnect example](examples/ankiconnect/index.md)

## Generic instructions

1. Copy [`codebase_analysis_guidelines.md`](codebase_analysis_guidelines.md) into your project's _notes/_ folder.
2. Use your preferred LLM coding agent (Gemini, Claude, Copilot, etc.) to run the following prompt:

```markdown
Read the guidelines in @notes/codebase_analysis_guidelines.md.
Run an analysis of this codebase.
```

### General advice

- **Use agent mode** &mdash; Use an "agent" mode so the coding agent can run multiple requests and analyze the codebase in depth. This means tools like [Aider](https://aider.chat/) may not be the best choice.
- **Use Gemini models** &mdash; The Google Gemini models offer 1M context window. (In contrast, Sonnet 4 has 200K.) The Flash model is often good enough if "thinking" is enabled.
- **Avoid Copilot GPT-4.1** &mdash; I don't know, it just never got good results for me. It does have a 1M context window though.

## Coding agents

### Gemini CLI

[Gemini CLI](https://github.com/google-gemini/gemini-cli) &mdash; Great results, highly recommended. Completely free to use with any Google Account. As of June 2025, Google provides 1000 free tokens per day for Gemini 2.5 Flash.

1. Run Gemini CLI using `gemini --model gemini-2.5-flash` in your project directory.
2. Follow "Generic instructions" above.

### OpenCode

[sst/opencode](https://github.com/sst/opencode) &mdash; pretty good. Be sure to use the SST version, not the Charm version.

1. Use a model like _Gemini 2.5 Pro_ or _Claude Sonnet 4_.
2. Follow "Generic instructions" above.

### Visual Studio Code (GitHub Copilot)

GitHub Copilot offers a $10 per month plan. As of June 2025, GitHub Copilot offers 300 premium requests per month, with one "agent mode" user prompt counting towards 1 request.

1. Open the command palette (Ctrl+Shift+P or Cmd+Shift+P).
2. Run _Chat: New Chat Editor_.
3. Switch to _Agent mode_ for more advanced code analysis.
4. Select _Claude Sonnet 4_ or your preferred model if available.
5. Follow "Generic instructions" above.

### Claude Code

1. Follow "Generic instructions" above.
2. (Optional) Create a [slash command](https://docs.anthropic.com/en/docs/claude-code/slash-commands) in `.claude/commands/analyze.md` with the contents of the prompt above.

## Keeping documentation up to date

### Updating the documentation

The prompt has hints on what to do when updating the documentation. It should use Git logs to figure out what's changed recently. Try this:

```markdown
Follow the guidelines in @notes/codebase_analysis_guidelines.md.
Analyze this codebase. Only consider the files that have changed since 14 days ago.
Update documents in @notes/architecture/.
```

### Automatically keep docs up to date

Create a script to automate the update process. For example, using the Gemini CLI:

```sh
gemini --model "gemini-2.5-flash" --prompt "Follow the guidelines in @notes/codebase_analysis_guidelines.md. Analyze this codebase. Only consider the files that have changed since 14 days ago. Update documents in @notes/architecture/."
```

A [GitHub Action](https://jasonet.co/posts/scheduled-actions/) can be used to schedule this script to run every 14 days.

## Usage ideas

It helps teams and individuals quickly assess code quality, architecture, and potential improvements by leveraging AI-powered code analysis tools. The guidelines and instructions here are designed to be agent-agnostic, making it easy to use with a variety of LLMs and coding environments.

### Diving into a new codebase

Use it when starting a job, or contributing to a new open source project.

### Using as context

Use it to help your LLM tools write PRD documents and design documents. Such as:

```markdown
Design a new "response caching" feature for this codebase.
Consult @notes/architecture/ for details on the code base architecture.
```

Try it with [spec-mode](https://github.com/rstacruz/spec-mode-prompt).

### Using on webchat

Some work places only has approved webchat interfaces for LLMs. You can still use this prompt there:

1. Ask your favourite LLM to modify the prompt ("Modify this prompt to output the results in chat instead of writing to files.").
2. Use a tool like [repomix](https://repomix.com/) to convert a repo into a Markdown file.
3. Paste the prompt and the repomix output into a webchat like Open WebUI.

### Elaborate on a feature

Need more details on a specific feature? Try:

```markdown
Follow the guidelines in @notes/codebase_analysis_guidelines.md.
Add FAQ question about response caching.
```

More ideas:

```markdown
Add code examples for the "translation" module.

Elaborate on features in @notes/architecture/rate_limiting.md.
```

### Not committing the notes

I think committing the _notes/architecture/_ and _codebase_analysis_guidelines.md_ is generally a good idea, but working with a team may mean you may want to keep them uncommitted.

```sh
echo notes/architecture >> .git/info/exclude
echo notes/codebase_analysis_guidelines.md >> .git/info/exclude
```

## Links

- Also try [spec-mode](https://github.com/rstacruz/spec-mode-prompt).

## Licence

This project is licensed under the MIT Licence. See [LICENSE](LICENSE.md) for details.
