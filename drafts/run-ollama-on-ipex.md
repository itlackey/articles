---
title: "Run Ollama on Intel Arc GPU (IPEX)"
tags: "ai, ollama, agents, llm"
description: "How to run Ollama using Intel Arc GPU"
series: IPEX
published: false
date: "2024-11-26T23:40:24Z"
---

# Run Ollama on Intel Arc GPU (IPEX)

As of the time of writing this article, Ollama does not officially support Intel Arc GPUs in its official releases. However, Intel provides a Docker image that includes a version of Ollama compiled with Arc GPU support enabled. This guide will walk you through setting up and running Ollama on your Intel Arc GPU using IPEX (Intel OneAPI Extension for XPU) docker image.

## Prerequisites

I hope to keep this article short and to the point. So, I will not walk through the setup of the needed drivers and tools. With that said, before proceeding, ensure you have the following:

- **Docker Desktop** installed and properly configured.
- **Intel Arc GPU drivers** installed and functioning correctly.

There are links to direct you to the installation guides for Docker and the Arc drivers at the end of this article. Be sure to follow the appropriate guide for your operating system.

## Setup Ollama Container

1. **Pull the Intel Analytics IPEX Image:**
   To speed up the process, go ahead and pull the Intel Analytics IPEX Image from Docker Hub:

   ```bash
   docker pull intelanalytics/ipex-llm-inference-cpp-xpu:latest
   ```

2. **Start the Container with Ollama Serve:**

   The docker command to start the container is fairly long. I prefer to save it to a script to make it easy to adjust and restart as needed.

   Mac and Linux users, write the following command to start the container to a file called `start-ipex-llm.sh` and save it to your home directory:

   ```bash
    #/bin/bash

    docker run -d  --restart=always \
        --net=bridge \
        --device=/dev/dri \
        -p 11434:11434 \
        -v ~/.ollama/models:/root/.ollama/models \
        -e PATH=/llm/ollama:$PATH \
        -e OLLAMA_HOST=0.0.0.0 \
        -e no_proxy=localhost,127.0.0.1 \
        -e ZES_ENABLE_SYSMAN=1 \
        -e OLLAMA_INTEL_GPU=true \
        -e ONEAPI_DEVICE_SELECTOR=level_zero:0 \
        -e DEVICE=Arc \
        --shm-size="16g" \
        --memory="32G" \
        --name=ipex-llm \
        intelanalytics/ipex-llm-inference-cpp-xpu:latest \
        bash -c "cd /llm/scripts/ && source ipex-llm-init --gpu --device Arc && bash start-ollama.sh && tail -f /llm/ollama/ollama.log"
   ```

   Windows users, will need to write the following command to a file called `start-ipex-llm.bat` and adjust it for the windows terminal.

   Once you have the script saved, execute it from the command line for your given OS.

   **Explanation of Flags:**

   - `--restart=always`: Ensures the container restarts if it stops.
   - Complete this list...

3. **Attach to the Running Container:**

   Once the container is up, you can issue the following command to pull a model from the Ollama library. Replace `<MODEL_ID>` with the specific model ID you wish to download (e.g., `qwen2.5-coder:0.5b`, `llama3.2`).

   ```bash
   docker exec ipex-llm ollama pull <MODEL_ID>
   ```

   You can search the Ollama model library for more models [here](https://ollama.com/search).

## Using Ollama

Once your desired model(s) are downloaded, you can interact with them directly using the Ollama CLI, through API calls, and by using various other tools. I encourage you to review the links in the resources section for more information about Ollama and other Ollama friendly tools. There are many more options available than we can cover in this article.

### Using Ollama CLI

The Ollama CLI is available to interact with the Ollama models directly from your terminal. Any `ollama` command that you would typically run locally in your terminal, can now be executed from within your running container by prefixing the `ollama` command with `docker exec -it ipex-llm-ollama`.

For example, to interact with the model you downloaded in the early section, issue the following command:

```bash
docker exec -it ipex-llm-ollama ollama run <MODEL_ID>
```

Check the [Ollama CLI Reference](https://github.com/ollama/ollama?tab=readme-ov-file#cli-reference) for more information about the commands available to you.

### Making API Calls

Make a request to the Ollama model endpoint:

```bash
curl http://localhost:11434/v1/completions \
     -H "Content-Type: application/json" \
     -d '{"model": "<MODEL_ID>",
          "prompt": "write a JavaScript function that takes an array of numbers and returns the sum of all elements in the array."
         }'
```

### Additional Tools

Once Ollama is up and running, you can leverage it with a variety of AI tools. There are too many to list, but here are a few of my favorites:

- **Open WebUI:** A user-friendly interface for interacting with AI models that provides many of the features of ChatGPT and others.

  - [Ollama Integration](https://github.com/open-webui/open-webui?tab=readme-ov-file#installation-with-default-configuration)

- **Continue.dev:** An extension for VSCode and JetBrains that provides "Co-pilot" capabilities.

  - [Ollama Integration](https://docs.continue.dev/customize/model-providers/ollama)

- **Aider:** One of the first, and still one of the great AI coding assistants.

  - [Ollama Integration](https://aider.chat/docs/llms/ollama.html)
  - [Aider LLM Leaderboard](https://aider.chat/docs/leaderboards/)

Please feel free to comment others that you feel should be added to this list.

## Troubleshooting

If you encounter issues, consider the following steps:

1. **Verify GPU Access:**
   Use `sycl-ls` within the container to check if the Arc GPU is recognized.

   To start and interactive shell within the container, issue the following command:

   ```bash
   docker exec -it ollama-ipex /bin/bash
   ```

   You can find some helpful tips [here](https://github.com/intel-analytics/ipex-llm/blob/main/docs/mddocs/Overview/KeyFeatures/multi_gpus_selection.md).

2. **Check Ollama Logs:**
   Monitor the logs for any errors:

   ```bash
   docker logs ipex-llm -f
   ```

3. **Update Docker and Drivers:**
   Ensure that both Docker and your GPU drivers are up to date.

4. **Consult Community Resources:**
   Refer to Intel's GitHub repositories and community forums for additional support:
   - [Intel Analytics Docker Hub](https://hub.docker.com/u/intelanalytics)
   - [IPEX GitHub Repository](https://github.com/intel-analytics/ipex-llm)

## Conclusion

To easily run Ollama on your Intel Arc GPU, you only need the proper drivers installed and a docker engine running. Once your system is setup, it as simple as running any other Docker container with a few extra arguments.

Keep an eye on the Ollama GitHub repository for updates, and consider contributing to the pull requests to bring Intel Arc support to the official Ollama builds.

## Additional Resources

- Intel Arc Driver Installation:
  - **Ubuntu:** Follow [this guide](https://dgpu-docs.intel.com/driver/client/overview.html)
  - **Windows:** Find the latest drivers [here](https://www.intel.com/content/www/us/en/products/docs/discrete-gpus/arc/software/drivers.html)

- Docker installation guides:
  - **Ubuntu:** Follow [this guide](https://docs.docker.com/engine/install/ubuntu/)
  - **Windows:** Follow [this guide](https://docs.docker.com/desktop/setup/install/windows-install/).
  - **MacOS:** Follow [this guide](https://docs.docker.com/desktop/mac/install/)

### Ollama Links

- [Ollama CLI Reference](https://github.com/ollama/ollama?tab=readme-ov-file#cli-reference)
- [Ollama Models](https://ollama.com/search)
- [Intel Arc Support GitHub Issue](https://github.com/ollama/ollama/pull/4876#issuecomment-2499289222)
- [OneAPI Pull Request](https://github.com/ollama/ollama/pull/6119)
