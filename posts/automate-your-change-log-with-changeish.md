---
title: 'Changeish: Automate your changelog with AI'
description: Changeish allows you to easily automate updating your code's change log.
tags: 'ai,git,devops,cli'
series: Changeish
cover_image: ''
canonical_url: null
published: true
id: 2595796
date: '2025-06-17T06:09:03Z'
---

Tired of manually writing release notes after dozens of commits? It's easy to fall behind when commits are flying in. To solve this, I wrote a small tool called [changeish](https://github.com/itlackey/changeish) - a Bash script that automates changelog entries by tapping into an LLM (Large Language Model) using Ollama, a local AI runner. In this article, I'll explain why changeish is useful for streamlining your release notes and how you can start using it in your own workflow.

## Why Automate Your Changelog?

Maintaining a changelog is critical for any project, but it can be tedious. Key reasons to automate this process include:

**Time Savings**: Manually writing entries for every new feature or fix can consume hours. Automating with an LLM frees you to focus on coding.

**Consistency**: An AI can generate entries in a consistent style (e.g. imperative mood, past tense, etc.), improving the readability of your CHANGELOG.md.

**Coverage**: By parsing git history, the script ensures no commit goes unmentioned. It reduces the chance of forgetting a change, especially in large projects with many contributors.

**Local & Private**: Because changeish can use Ollama to run a model locally, your code never leaves your machine. You get AI-powered summaries without sending data to a cloud service.

In short, changeish brings the power of AI to your changelog, making updates faster and easier while you retain full control of your project's data.

## What Exactly Does changeish Do?

Changeish is essentially two things: a Bash script (changes.sh) and a Markdown prompt template (changelog_prompt.md). Here's how they work together to update your changelog:

1. Gather Git History: The script uses git to collect recent commit messages (and possibly diffs) since the last logged update. It essentially prepares a summary of what's changed in your codebase.

2. Prepare an LLM Prompt: Using the included prompt template, it combines the git history with instructions for the AI. The prompt might look like: "Here are recent commit logs: [...] - Summarize these into a concise changelog entry." The template guides the LLM on format (for example, to output bullet points under a new version heading).

3. LLM Generation: With the prompt ready, changeish calls an LLM (Ollama's CLI, or OpenAI API endpoint coming soon) to generate text from your chosen model. The script is written in pure Bash to avoid external dependencies, requiring only Git and optionally the Ollama CLI to be installed on your system.

4. Update CHANGELOG.md: The AI-generated changelog entry is then appended to the top of your CHANGELOG.md file (or whichever file you designate). This means the most recent changes appear first, following common changelog conventions.

If Ollama (or a local model) isn't available, changeish can skip the AI generation step and instead output the composed prompt to a file. In that case, you'll get a changelog_prompt.md with the git history and instructions, which you can copy into another AI interface of your choice. This fallback ensures the tool is still useful even if you prefer to use a cloud AI or don't have a model set up - you won't have to manually compile the commit logs yourself.

## Installation

Setting up changeish is straightforward. The project is hosted on GitHub (as open-source), so you can grab the latest script and prompt template directly. Ensure you have Git installed (you likely do) and Ollama set up with at least one local model (for example, a Llama 2 or code-specialized model for better results).

Use the following one-liner commands to download changeish into your current directory and make it executable:

### Download the changeish script and prompt template

```bash
curl -fsSL https://raw.githubusercontent.com/itlackey/changeish/main/install.sh | sh
```

This will fetch the `changes.sh` Bash script along with its companion prompt file, `changelog_prompt.md`, and set the proper execute permission on the script. (Feel free to place them in a directory that's on your $PATH if you want to use changeish globally across projects.)

## Using changeish for Your Project

Once installed, using changeish to update your changelog is a breeze. Here's a typical workflow:

1. Navigate to Your Repo: Open a terminal in the directory of the Git repository you want to generate a changelog for.

2. Run the Script: Execute `./changes.sh`. The script will run Git to gather commit info and then attempt to invoke Ollama to generate the changelog text. Depending on the size of your history and the model in use, this may take a few moments.

3. Watch the Magic: If everything is set up, you'll see Ollama processing the prompt. The AI's output (a nicely formatted changelog section) will be printed and automatically added to your CHANGELOG.md file at the top. For example, it might insert something like:

```markdown
## [Unreleased] - 2025-06-15

- ✨ Added a new feature to automate X
- 🐛 Fixed bug where Y would crash on start
- 🛠 Refactored Z module for performance improvements
```

> (The exact format can be customized via the prompt template. The above is just an illustrative example.)

4. Review and Tweak (If Needed): Open your CHANGELOG.md to review the generated entry. Because an LLM wrote it, you'll want to double-check for accuracy. In most cases, the summary will be impressively good, but you can always edit wording or remove less-important items. Think of it as a first draft of release notes that you can polish.

5. Commit the Changes: Once you're happy with the updated changelog, commit the CHANGELOG.md file to version control. Now your project's history of changes is up-to-date!

Note: If you run the script without an active Ollama model or if the Ollama CLI isn't installed, changeish will detect that and instead output a prompt.md (or changelog_prompt.md) file containing the prepared text for the AI. You can take that file and paste its contents into an AI tool like ChatGPT or run it through another LLM interface to get the changelog entry, then manually insert it into your CHANGELOG. This flexibility means the tool adapts to your environment - whether fully automated locally or a hybrid approach.

## Context and Use Cases

I originally built changeish to manage the changelog for a large application with a fast-paced development cycle. Here's how it proved useful in practice:

**Large Merge Trains**: After merging a batch of pull requests, I'd run changeish to quickly summarize all the merged changes. What used to be a laborious writing session became a one-command operation.

**Release Prep**: Right before cutting a new release, I'd use the script to generate the "Unreleased" section of the changelog. This gave me a solid draft of release notes that I could refine and tag with the new version number.

**Continuous Integration**: While primarily intended as a developer aid, you could integrate changeish into your CI pipeline (assuming the CI environment has access to an Ollama server or you use the prompt-only mode). For example, a nightly job could generate a changelog update for the day's commits, ensuring that documentation is always current.

Because changeish is just a lightweight Bash script, it's easy to hack or customize. You can adjust the changelog_prompt.md template to change the tone or structure of the output (e.g., enforce a certain style or add additional context for the AI). Since it's local and open-source, you're in full control - there's no magic beyond your own machine.

## Conclusion

Keeping a changelog shouldn't be a chore, and with changeish it no longer is. By leveraging a local AI model through Ollama, this script automates the heavy lifting of generating human-readable summaries of your git history. It ensures your project's changelog is always up-to-date with minimal effort, which is especially valuable for large or fast-moving codebases.

> ✨ For more details, check out the [changeish repository on GitHub](https://github.com/itlackey/changeish) - contributions and feedback are welcome. Don't forget to star the repo, and leave a comment to let me know what you think!

Happy coding, and enjoy your auto-magically updated changelogs!
