---
title: Run Ollama on Intel Arc GPU (IPEX)
tags: 'ai, ollama, agents, llm'
description: How to run Ollama using Intel Arc GPU
series: IPEX
id: 2123422
---

# Run Ollama on Intel Arc GPU (IPEX)

As of the time of writing, Ollama does not officially support Intel Arc GPUs in its releases. However, Intel provides a Docker image that includes a version of Ollama compiled with Arc GPU support enabled. This guide will walk you through setting up and running Ollama on your Intel Arc GPU using the IPEX (Intel OneAPI Extension for XPU) Docker image.

## Prerequisites

Before proceeding, ensure you have the following installed and properly configured:

- **Docker Desktop**
- **Intel Arc GPU drivers**

Links to the installation guides for Docker and the Arc drivers are provided at the end of this article. Be sure to follow the appropriate guide for your operating system.

## Set Up Ollama Container

1. **Pull the Intel Analytics IPEX Image:**

   Pull the Intel Analytics IPEX image from Docker Hub:

   ```bash
   docker pull intelanalytics/ipex-llm-inference-cpp-xpu:latest
   ```

2. **Start the Container with Ollama Serve:**

   Because the Docker command to start the container is quite long, it's convenient to save it to a script for easy adjustment and restarting.

   **Mac and Linux users:** Create a file named `start-ipex-llm.sh` in your home directory and add the following content:

   ```bash
   #!/bin/bash

   docker run -d --restart=always \
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

   Once you have the script saved, make it executable (for Mac and Linux users) and run it:

   ```bash
   chmod +x ~/start-ipex-llm.sh
   ~/start-ipex-llm.sh
   ```

   **Windows users:** Create a file named `start-ipex-llm.bat` and adjust the above command for the Windows terminal. Make sure to modify paths and syntax accordingly.

   **Explanation of Flags:**

   - `--restart=always`: Ensures the container restarts automatically if it stops.
   - `--device=/dev/dri`: Grants the container access to the GPU device.
   - `--net=bridge`: Uses the bridge networking driver.
   - `-p 11434:11434`: Maps port 11434 of the container to port 11434 on the host.
   - `-e OLLAMA_HOST=0.0.0.0`: Sets the host IP address for Ollama to listen on all interfaces. This allows other systems to call the Ollama API.
   - `-e no_proxy=localhost,127.0.0.1`: Prevents Docker from using the proxy server when connecting to localhost or 127.0.0.1.
   - `-e ONEAPI_DEVICE_SELECTOR=level_zero:0`: Tells Ollama which GPU device to use. This may need to be adjusted if you have an iGPU installed on your system.
   - `-e PATH=/llm/ollama:$PATH`: Adds the Ollama binary directory (`/llm/ollama`) to the `PATH`. This allow Ollama commands to be executed easily with `docker run` commands.
   - `-v ~/.ollama/models:/root/.ollama/models`: Mounts the host's Ollama models directory into the container. This allows downloaded models to be persisted when the container restarts.
   - `--shm-size="16g"`: Sets the shared memory size to 16 GB. This setting may need to be adjusted for your system. See the Docker documentation for more information on shared memory.
   - `--memory="32G"`: Limits the container's memory usage to 32 GB. This setting may need to be adjusted for your system. See the Docker documentation for more information on memory usage.
   - `--name=ipex-llm`: Names the container `ipex-llm`. This name is used to reference the container in other commands.

3. **Download a Model:**

   Once the container is up, you can pull a model from the Ollama library. Replace `<MODEL_ID>` with the specific model ID you wish to download (e.g., `qwen2.5-coder:0.5b`, `llama3.2`).

   ```bash
   docker exec ipex-llm ollama pull <MODEL_ID>
   ```

   You can browse the Ollama model library for more options [here](https://ollama.com/search).

## Using Ollama

With your desired model(s) downloaded, you can interact with them directly using the Ollama CLI, make API calls, or integrate with various tools. Below are some ways to get started.

### Using the Ollama CLI

The Ollama CLI allows you to interact with models directly from your terminal. Any `ollama` command that you would typically run locally can now be executed within your container by prefixing the command with `docker exec -it ipex-llm`.

For example, to interact with the model you downloaded earlier:

```bash
docker exec -it ipex-llm ollama run <MODEL_ID>
```

Check the [Ollama CLI Reference](https://github.com/ollama/ollama#cli-reference) for more information about available commands.

### Making API Calls

You can make API requests to the Ollama model endpoint:

```bash
curl http://localhost:11434/v1/completions \
     -H "Content-Type: application/json" \
     -d '{
           "model": "<MODEL_ID>",
           "prompt": "Write a JavaScript function that takes an array of numbers and returns the sum of all elements in the array."
         }'
```

### Additional Tools

Once Ollama is running, you can leverage it with a variety of AI tools. Here are a few of my favorites:

- **Open WebUI:** A user-friendly interface for interacting with AI models, offering many features similar to ChatGPT.

  - [Ollama Integration](https://github.com/open-webui/open-webui#installation-with-default-configuration)

- **Continue.dev:** An extension for VSCode and JetBrains that provides "Co-pilot" capabilities.

  - [Ollama Integration](https://docs.continue.dev/customize/model-providers/ollama)

- **Aider:** One of the first and still one of the great AI coding assistants.

  - [Ollama Integration](https://aider.chat/docs/llms/ollama.html)
  - [Aider LLM Leaderboard](https://aider.chat/docs/leaderboards/)

- **CrewAI:** An easy to use AI agent framework that can be used with Ollama models to run AI agents locally.
  - [Ollama Integration](https://docs.crewai.com/concepts/llms#ollama-local-llms)

Feel free to suggest others that should be added to this list.

## Troubleshooting

If you encounter issues, consider the following steps:

1. **Verify GPU Access:**

   Use `sycl-ls` within the container to check if the Arc GPU is recognized.

   To start an interactive shell within the container:

   ```bash
   docker exec -it ipex-llm /bin/bash
   ```

   Then run:

   ```bash
   sycl-ls
   ```

   You can find helpful tips [here](https://github.com/intel-analytics/ipex-llm/blob/main/docs/mddocs/Overview/KeyFeatures/multi_gpus_selection.md).

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

Running Ollama on your Intel Arc GPU is straightforward once you have the proper drivers installed and Docker running. With your system set up, it's as simple as running any other Docker container with a few extra arguments.

Keep an eye on the Ollama GitHub repository for updates, and consider contributing to pull requests to bring Intel Arc support to the official Ollama builds.

## Additional Resources

Below is a running list of related links. Please feel free to comment others that you think should be added to the list.

- Intel Arc Driver Installation:

  - **Ubuntu:** Follow [this guide](https://dgpu-docs.intel.com/driver/client/overview.html)
  - **Windows:** Find the latest drivers [here](https://www.intel.com/content/www/us/en/products/docs/discrete-gpus/arc/software/drivers.html)

- Docker Installation Guides:
  - **Ubuntu:** Follow [this guide](https://docs.docker.com/engine/install/ubuntu/)
  - **Windows:** Follow [this guide](https://docs.docker.com/desktop/setup/install/windows-install/).
  - **macOS:** Follow [this guide](https://docs.docker.com/desktop/mac/install/)

### Ollama Links

- [Ollama CLI Reference](https://github.com/ollama/ollama?tab=readme-ov-file#cli-reference)
- [Ollama Models](https://ollama.com/search)
- [Intel Arc Support GitHub Issue](https://github.com/ollama/ollama/pull/4876#issuecomment-2499289222)
- [OneAPI Pull Request](https://github.com/ollama/ollama/pull/6119)
