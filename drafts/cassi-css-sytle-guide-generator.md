---
title: "Cassi: The AI Powered CSS Style Guide Generator"
tags: 'ai, css, ai, agents'
description: "Introducing Cassi, an AI-powered CSS Style Guide Generator that helps developers create consistent and maintainable stylesheets efficiently."
series: Cassi
published: false
---

# Introducing Cassi: Your AI-Powered CSS Style Guide Generator

## Why CSS Documentation Is Hard

If you've ever worked on a large CSS codebase, you know how difficult it can be to understand and maintain. Recently, I found myself needing to generate a style guide from a set of existing CSS files. To my surprise, many of the tools available for this task were outdated-some hadn't been updated in over a decade!

While tools like Storybook are fantastic for documenting components, they aren't designed for plain CSS styles. I needed something that could handle raw CSS rules, generate clear documentation, and integrate seamlessly into a static site generator like 11ty.

That's when I decided to build a new tool: **Cassi**.

---

## The Problem: Outdated Tools and Unmodifiable Styles

Imagine this scenario: You inherit a large project with multiple stylesheets, each containing hundreds of CSS rules. These styles are critical to the application and cannot be modified, but there's no documentation to explain what each rule does or how it should be used.

Existing tools for generating style guides either:

- Focus on component libraries, like Storybook.
- Rely on outdated methods that don't align with modern CSS practices.
- Require changes to the CSS itself, which isn't always an option.

This left a clear gap in the ecosystem: **a modern, CSS-focused tool that works with what you already have.**

---

## The Solution: Cassi

Enter **Cassi**, an AI-powered tool that bridges the gap by analyzing your existing CSS files and generating markdown documentation for each rule. The output includes:

- A human-readable description of the rule.
- Practical usage examples in HTML.
- Front matter compatible with static site generators like 11ty.

---

## Highlights of Cassi

Here's why Cassi is a game-changer for CSS documentation:

1. **Runs Locally or in the Cloud**
   - Use open-source models locally or connect to hosted AI services.

2. **Markdown-First Output**
   - Cassi generates rich, markdown-based documentation, complete with 11ty-compatible front matter for seamless integration.

3. **Customizable Templates**
   - You can edit a simple text file to change the AI prompt, tailoring the output to your needs.

4. **Seamless Integration**
   - The markdown output works effortlessly with tools like 11ty or other documentation platforms.

---

## How Cassi Works

At a high level, Cassi operates as follows:

1. **CSS Parsing**
   - Cassi reads your CSS files using the powerful `fast-glob` library.
   - It parses the CSS rules to extract selectors and declarations.

2. **AI-Powered Markdown Generation**
   - Each rule is sent to an AI model with a carefully crafted prompt to generate documentation.

3. **Customizable Output**
   - A prompt template allows you to define exactly what kind of documentation you want.

### Example Output

Here's an example of the markdown output Cassi generates:

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

This markdown is ready to be rendered by 11ty or any other markdown-based static site generator.

## Use Cases for Cassi

Cassi is perfect for:

- **Documenting legacy CSS projects**: Large codebases often lack cohesive documentation.
- **Style guide creation**: Quickly generate style guides for static sites.
- **Maintaining multiple stylesheets**: Provides clarity when working with large, scattered styles.

## What's Next for Cassi?

We're just getting started! Future updates to Cassi could include:

- **Support for preprocessors** like SCSS and LESS.
- **Inline previews** of styles alongside documentation.
- **Advanced customization options** for markdown templates.

These features will make Cassi even more powerful and versatile for modern CSS workflows.

## Conclusion

CSS documentation doesn't have to be a manual, time-consuming process. With **Cassi**, you can quickly generate rich, markdown-based documentation that is easy to use, integrate, and customize.

Cassi is more than just a tool-it's a bridge between raw CSS and maintainable, developer-friendly documentation. Whether you're working on a legacy codebase or starting fresh, Cassi is here to make your life easier.

**What do you think?** Would Cassi be useful in your projects? Let me know in the comments below!
