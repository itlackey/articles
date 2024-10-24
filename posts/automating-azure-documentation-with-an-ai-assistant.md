---
title: Automating Azure Documentation with an AI Assistant
tags: "azure, python, ai, agents"
description: "A short guide showing how you can leverage python and az CLI to automate documenting Azure resources."
series: "Azure Assistants"
published: true
date: 2024-01-23
---
# Automating Azure Documentation with an AI Assistant

Managing and documenting Azure Resource Groups (RGs) in large-scale environments can be time-consuming and complicated. But what if you could automate the process of generating documentation that not only explains what resources exist but also how they relate to each other?

In this article, we'll explore how a **simple Python script** can leverage **LLMs (Large Language Models)** like **OpenAI** or **Azure OpenAI** to automate the creation of comprehensive markdown documentation from ARM templates. What makes this tool powerful is not the use of complex agent frameworks or heavy infrastructure, but pure Python combined with well-established tools like Azure CLI and OpenAI's API. It can even be used with other AI providers, and local LLMs using Ollama or other similar tools.

## No Need for Complex Agent Frameworks

A common misconception is that you need elaborate agent frameworks to harness the power of LLMs effectively. In reality, you can achieve powerful, automated workflows using existing tools and simple scripts. In this solution, we combine:

1. **Python**: As the scripting language as it is commonly installed and widely used.
2. **Azure CLI**: To fetch ARM templates from Azure Resource Groups.
3. **OpenAI API Calls**: To generate human-readable documentation from ARM templates.
4. **Markdown**: As the output format for the documentation, which integrates easily into any knowledge base.

The result? A clean, efficient script that creates documentation without needing complicated tooling or AI powered orchestration.

## Azure Assistants Source Code

The source code is available in this Github repository: [itlackey/azure-assistants](https://github.com/itlackey/azure-assistants). Currently, it contains a single Python script that leverages the Azure CLI and OpenAI API to generate markdown documentation from ARM templates. If there is interest, or I have a need, the repository may be updated with additional tools and scripts to automate other tasks.

## How the Script Works

The heart of this tool is the [document_resource_groups.py](https://github.com/itlackey/azure-assistants/blob/main/document_resource_groups.py) script. It does these four things:

1. **Get All Resource Groups** in the current Azure Subscription.
1. Uses `az` CLI to **Export ARM Templates** from Azure Resource Groups.
1. We **Parse The Templates** and send them to an OpenAI compatible API.
1. The LLM is used to **Generate Markdown Documentation** that is ready to be included in a knowledge base.

### List Resource Groups

The first step is to fetch all resource groups in your Azure Subscription. This is done using the `az` CLI command from our Python script. We then loop through them to fetch the ARM template.

```python
result = subprocess.run(
    ["az", "group", "list", "--query", "[].name", "-o", "tsv"],
    stdout=subprocess.PIPE,
    text=True,
)
resource_groups = result.stdout.splitlines()
```

### Export ARM Templates

Again, using the Azure CLI, the script retrieves the ARM templates for each resource group in the current subscription. These templates contain detailed configuration information for all resources, including their networking and security settings.

```python
export_command = [
    "az", "group", "export",
    "--name", resource_group_name,
    "--include-parameter-default-value",
    "--output", "json",
]
```

### Summarizing with LLMs

Next, the script sends the ARM template to OpenAI (or Azure OpenAI) for summarization. Here's where the magic happens. Instead of diving into complex agent workflows, a simple **system message** and **user prompt** provide enough context to the LLM to generate insightful documentation.

```python
response = client.chat.completions.create(model=model, messages=messages)
```

The prompt provides an expected output template and instructs the LLM to:

- List and describe each resource.
- Explain how resources relate to each other.
- Highlight important network configurations.

This allows the LLM to produce structured, easy-to-read documentation without needing any fancy orchestration.

### Generating Markdown Documentation

The final step is generating a markdown file that contains the resource group's details. The front matter includes metadata like resource group name, date, and tags. The AI-generated documentation is then added as the content of the document.

```python
front_matter = f"---\n"
front_matter += f'title: "{resource_group_name}"\n'
front_matter += f"date: {date}\n"
front_matter += f"internal: true\n"
```

Markdown is a universal format, allowing this output to easily integrate into many documentation systems or knowledge management systems.

## Customizing the AI Prompts

A key feature of this script is the ability to **customize the prompts** sent to the LLM. This is where users can fine-tune the type of output they want:

- **System Message**: Guides the LLM to generate documentation focused on explaining resources, relationships, and networking.

  Example:

  ```plaintext
    You are an experienced Azure cloud architect helping to create reference documentation that explains the resources within an Azure Resource Manager (ARM) template.

    The documentation you create is intended for use in a knowledge base. Your role is to describe the resources in a clear and human-readable way, providing details on the following:

    - What resources exist in the ARM template.
    - How the resources relate to each other.
    - The purpose of each resource (if possible).
    - Highlighting network configurations and data locations such as storage accounts and databases.
    - Be sure to include IP addresses in the documentation when they are available.
    - Include information about virtuanl network peerings.
    - It is very important that you also include any potential security issues that you may find.
  ```

- **User Prompt**: Dynamically generated based on the resource group being summarized.

  Example:

  ```plaintext
    Provide detailed documentation of the following ARM template for resource group: 

  
    {template_content}


    The purpose of this documentation is to...
  ```

By keeping these prompts flexible and simple, the script avoids over-engineering while still delivering high-quality documentation.

## Running the Script

> Note: You will need to have `az` CLI and python3 installed on your machine before you run this script.

Setting up and running the script is straightforward:

1. **Log into Azure**: Ensure you're authenticated with Azure CLI:

   ```bash
   az login
   ```

2. **Run the script** to generate markdown documentation:

   ```bash
   python document_resource_groups.py
   ```

The script processes each resource group, generates its ARM template, and creates a markdown file in the output directory.

### Example Output

Here's an example of what the script generates:

```markdown
---
title: "Resource Group: myResourceGroup"
date: 2024-10-23
internal: true
azureTags:
  - environment: production
  - owner: devops-team
---

# Resource Group: myResourceGroup

## Overview

This resource group contains a virtual network (VNet) with two subnets: front-end and back-end. The VNet is configured with a network security group (NSG) that restricts inbound traffic to HTTPS and SSH ports only. Resources in the front-end subnet include an Azure App Service for web hosting, while the back-end subnet hosts an Azure SQL Database...
```

This output is **concise**, **readable**, and **easy to understand** - exactly what you need for internal documentation or knowledge base entries.

## Conclusion

Azure Assistants is a perfect example of how you can use **existing tools** and basic **Python** skills to achieve powerful results with LLMs. There's no need for elaborate agent frameworks when simple scripts, combined with Azure CLI and OpenAI's API, can generate clear, comprehensive documentation for your Azure Resource Groups.

This tool demonstrates that with the right prompts and a solid structure, **anyone** with basic scripting skills can leverage AI to automate cloud documentation - making it a valuable assistant for any DevOps or infrastructure team.
