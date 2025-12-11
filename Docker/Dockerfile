FROM ubuntu:24.04

# Environment variables
ENV OLLAMA_INTEL_GPU=true
ENV DEBIAN_FRONTEND=noninteractive

# Install required system dependencies and Intel GPU libraries
RUN apt-get update && \
    apt-get install -y \
        software-properties-common \
        wget \
        screen \
        apt-transport-https \
        ca-certificates \
        curl \
        net-tools \
        gnupg && \
    add-apt-repository -y ppa:kobuk-team/intel-graphics && \
    apt-get update && \
    apt-get install -y \
        libze-intel-gpu1 \
        libze1 \
        intel-metrics-discovery \
        intel-opencl-icd \
        clinfo \
        intel-gsc \
        libze-dev \
        intel-ocloc && \
    rm -rf /var/lib/apt/lists/*

# Set working directory and download Ollama IPEX-LLM
WORKDIR /opt
RUN wget https://github.com/ipex-llm/ipex-llm/releases/download/v2.3.0-nightly/ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz && \
    tar -xvf ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz && \
    rm ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz

# Change to Ollama folder
WORKDIR /opt/ollama-ipex-llm-2.3.0b20250630-ubuntu

# Update start script to listen on all interfaces
RUN sed -i "s/export OLLAMA_HOST='127.0.0.1:11434'/export OLLAMA_HOST=\${OLLAMA_HOST:-0.0.0.0:11434}/" start-ollama.sh

# Run Ollama
CMD ["./start-ollama.sh"]