## Installing and Configuring K8sGPT

 ### 1. Download and Install K8sGPT
```
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/latest/download/k8sgpt_Linux_x86_64.tar.gz
tar -xzf k8sgpt_Linux_x86_64.tar.gz
sudo mv k8sgpt /usr/local/bin/
```

These commands download the K8sGPT binary, extract it, and move it to your system's executable path.

2. Authenticate with OpenAI
```
k8sgpt auth add --backend openai --model gpt-3.5-turbo
```

You'll be prompted to enter your OpenAI API key. Once authenticated, K8sGPT will connect to the specified GPT model.
### 3. Analyze Kubernetes Cluster Issues

```
k8sgpt analyze
k8sgpt analyze --explain
```

This command scans your Kubernetes cluster for issues and provides human-readable explanations powered by GPT.

