---
title: 'Cassi: An AI-Powered CSS Style Guide Generator'
tags: 'ai, css, agents, documentation'
description: 'Introducing Cassi, an AI-powered CSS assistant that generates markdown-based documentation from your existing CSS files.'
series: Cassi
published: true
cover_image: 'https://raw.githubusercontent.com/itlackey/cassi/refs/heads/main/site/assets/imgs/banner-small.webp'
id: 2107632
date: '2024-11-16T23:40:24Z'
---

# Cassi: An AI-Powered CSS Assistant

Cassi is an AI-powered tool designed to generate markdown-based documentation from existing CSS files. It leverages AI models to generate meaningful information about each CSS rule. This process makes it much easier to document complex stylesheets.

## Challenges in Documenting Large CSS Projects

Working on projects with a large amount of CSS rules, possibly scattered across multiple files, can be challenging. Existing tools often focus on component libraries, require comments to be added to the rules, or are outdated, making it difficult to document raw CSS styles effectively.

I built **Cassi** to address this issue by analyzing existing CSS files and generating markdown-based documentation for each rule.

## Key Features of Cassi

Here's what makes Cassi a powerful tool:

1. **Local or Cloud AI Integration**
   - Use open-source models locally or connect to hosted AI services such as OpenAI or Anthropic models.
2. **Markdown-Based Documentation Output**
   - Generates rich, markdown-based documentation complete with 11ty-compatible front matter.
3. **Customizable Templates**
   - Edit the prompt template to tailor the output according to your needs.
4. **Seamless Integration**
   - Markdown output works effortlessly with tools like 11ty or other documentation platforms.

## How Cassi Works

As of the time of writing this, Cassi is not much more than a Node JS script and a prompt template. I do have plans to add some additional functionality, more on that later. For now, let's look at how it works.

1. **CSS Parsing**
   - Reads CSS files based on provided glob patterns.
   - Parses CSS rules to extract selectors and declarations.

2. **AI-Powered Markdown Generation**
   - Sends each rule to an AI model with a carefully crafted prompt to generate documentation.

3. **Create Markdown Documentation**
   - Creates markdown files for each of the rules using the AI model's response.

As you can see, the process is relatively straightforward, and demonstrates what you can achieve with the correct prompt when working with even local models.

### Example Output

Here's an example of the markdown output Cassi generates using qwen2.5-coder on Ollama:

```markdown
    ---
    title: "Styling for .btn-primary"
    tags: ["CSS", "Styles", "Selectors"]
    permalink: "/styles/btn-primary/"
    shortDescription: "Primary button styling for highlighting important actions."
    selectors:
    - ".btn-primary"
    ---

    ## Overview

    The `.btn-primary` rule defines the primary styling for buttons that should stand out, typically used for important calls to action like "Submit" or "Save."

    ## Usage

    Here's how to use this rule in your HTML:

    ```html
    <button class="btn-primary">Submit</button>
    ```

    ## CSS Declarations

    ```css
    .btn-primary {
        background-color: #007bff;
        color: #fff;
        border: 1px solid #007bff;
        padding: 10px 15px;
        border-radius: 5px;
    }
    ```

    ## Developer Notes

    - Use `.btn-primary` sparingly to maintain emphasis on important actions.
    - Ensure sufficient contrast between the button text and its background for accessibility.
```

## GitHub Repository

You can find the Cassi repository on GitHub [itlackey/cassi](https://github.com/itlackey/cassi) if you would like to see the code, try it yourself, or even help improve the tool.

## What's Next for Cassi?

Cassi was built to solve a problem I am currently facing. Now that I can easily generate the documentation that my team needs, we can start focusing on adding a few more features to improve our workflow even more. Here are some features I am considering adding:

- **11ty Starter Kit** - A pre-configured 11ty project that includes Cassi and is pre-configured to generate a style guide from the files created by Cassi.
- **Proper CLI** to allow syntax like `cassi generate styles/*.css --output-dir docs` for generating documentation, `cassi download http://some.site` to download CSS files from a URL, or `cassi build` to generate a style guide using 11ty.
- **Incremental Updates** - Add logic to allow Cassi to determine what CSS has been added/modified, and add/update the markdown documents accordingly.
- **Style Grouping** - Allow users to group CSS rules into categories or sections for easier navigation.

## Final Thoughts

CSS documentation doesn't have to be a manual, time-consuming process. **Cassi** can quickly generate rich, markdown-based documentation that is easy to use, integrate, and customize.

**What do you think?** Would Cassi be useful in your projects? Let me know in the comments below!

## Shout Outs

Before we wrap up, I wanted to mention a couple of great projects. Be sure to go support these projects:

- [augmented-ui](https://augmented-ui.com/) - Great cyberpunk style UI library.
- [Ollama](https://ollama.ai/) - A fantastic tool to use for hosting AI models locally.
- [11ty](https://www.11ty.dev/) - One of the best static site generators around.

