# Remote vLLM Distribution

The `llamastack/distribution-{{ name }}` distribution consists of the following provider configurations:

{{ providers_table }}

You can use this distribution if you have GPUs and want to run an independent vLLM server container for running inference.

{% if run_config_env_vars %}
### Environment Variables

The following environment variables can be configured:

{% for var, (default_value, description) in run_config_env_vars.items() %}
- `{{ var }}`: {{ description }} (default: `{{ default_value }}`)
{% endfor %}
{% endif %}


## Setting up vLLM server

Please check the [vLLM Documentation](https://docs.vllm.ai/en/v0.5.5/serving/deploying_with_docker.html) to get a vLLM endpoint. Here is a sample script to start a vLLM server locally via Docker:

```bash
export INFERENCE_PORT=8000
export INFERENCE_MODEL=meta-llama/Llama-3.2-3B-Instruct
export CUDA_VISIBLE_DEVICES=0

docker run \
    --runtime nvidia \
    --gpus $CUDA_VISIBLE_DEVICES \
    -v ~/.cache/huggingface:/root/.cache/huggingface \
    --env "HUGGING_FACE_HUB_TOKEN=$HF_TOKEN" \
    -p $INFERENCE_PORT:$INFERENCE_PORT \
    --ipc=host \
    vllm/vllm-openai:latest \
    --gpu-memory-utilization 0.7 \
    --model $INFERENCE_MODEL \
    --port $INFERENCE_PORT
```

If you are using Llama Stack Safety / Shield APIs, then you will need to also run another instance of a vLLM with a corresponding safety model like `meta-llama/Llama-Guard-3-1B` using a script like:

```bash
export SAFETY_PORT=8081
export SAFETY_MODEL=meta-llama/Llama-Guard-3-1B
export CUDA_VISIBLE_DEVICES=1

docker run \
    --runtime nvidia \
    --gpus $CUDA_VISIBLE_DEVICES \
    -v ~/.cache/huggingface:/root/.cache/huggingface \
    --env "HUGGING_FACE_HUB_TOKEN=$HF_TOKEN" \
    -p $SAFETY_PORT:$SAFETY_PORT \
    --ipc=host \
    vllm/vllm-openai:latest \
    --gpu-memory-utilization 0.7 \
    --model $SAFETY_MODEL \
    --port $SAFETY_PORT
```

## Running Llama Stack

Now you are ready to run Llama Stack with vLLM as the inference provider. You can do this via Conda (build code) or Docker which has a pre-built image.

### Via Docker

This method allows you to get started quickly without having to build the distribution code.

```bash
export INFERENCE_PORT=8000
export INFERENCE_MODEL=meta-llama/Llama-3.2-3B-Instruct
export LLAMA_STACK_PORT=5001

docker run \
  -it \
  -p $LLAMA_STACK_PORT:$LLAMA_STACK_PORT \
  -v ./run.yaml:/root/my-run.yaml \
  llamastack/distribution-{{ name }} \
  /root/my-run.yaml \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env VLLM_URL=http://host.docker.internal:$INFERENCE_PORT/v1
```

If you are using Llama Stack Safety / Shield APIs, use:

```bash
export SAFETY_PORT=8081
export SAFETY_MODEL=meta-llama/Llama-Guard-3-1B

docker run \
  -it \
  -p $LLAMA_STACK_PORT:$LLAMA_STACK_PORT \
  -v ./run-with-safety.yaml:/root/my-run.yaml \
  llamastack/distribution-{{ name }} \
  /root/my-run.yaml \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env VLLM_URL=http://host.docker.internal:$INFERENCE_PORT/v1 \
  --env SAFETY_MODEL=$SAFETY_MODEL \
  --env VLLM_SAFETY_URL=http://host.docker.internal:$SAFETY_PORT/v1
```


### Via Conda

Make sure you have done `pip install llama-stack` and have the Llama Stack CLI available.

```bash
export INFERENCE_PORT=8000
export INFERENCE_MODEL=meta-llama/Llama-3.2-3B-Instruct
export LLAMA_STACK_PORT=5001

cd distributions/remote-vllm
llama stack build --template remote-vllm --image-type conda

llama stack run ./run.yaml \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env VLLM_URL=http://127.0.0.1:$INFERENCE_PORT/v1
```

If you are using Llama Stack Safety / Shield APIs, use:

```bash
export SAFETY_PORT=8081
export SAFETY_MODEL=meta-llama/Llama-Guard-3-1B

llama stack run ./run-with-safety.yaml \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env VLLM_URL=http://127.0.0.1:$INFERENCE_PORT/v1 \
  --env SAFETY_MODEL=$SAFETY_MODEL \
  --env VLLM_SAFETY_URL=http://127.0.0.1:$SAFETY_PORT/v1
```