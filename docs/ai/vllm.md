# Self-hosted model with vLLM on NERSC

The AI Assistant can use an open model served on a Perlmutter GPU node instead
of a commercial service. Your data then stays on NERSC, and the cost is GPU
time from your own allocation.

The setup has two parts, both in the `agent/` folder of the
[source repository](https://github.com/AlexxLauu/ACE3P-GUI-Project):

- **The vLLM job** (`agent/slurm/vllm_qwen_coder.sbatch`): serves
  **Qwen2.5-Coder-14B-Instruct** with vLLM on one A100 GPU, as the model name
  `qwen-coder`, with an OpenAI-compatible API.
- **The on-demand proxy** (`agent/proxy/`): a small service on a login node.
  It submits the vLLM job on the first request, waits until the model is ready,
  forwards requests to it, and cancels the job after 45 idle minutes.

```text
 ACE3P GUI                    Perlmutter login node              GPU compute node
 ----------                   ---------------------              ----------------
 AI Assistant   ---------->   proxy 127.0.0.1:8077   --sbatch-->  Slurm queue
 (OpenAI-compatible)                                 --forward--> vLLM nid00xxxx:8000
                                                     --scancel--> after 45 idle minutes
```

The `agent/` folder is not part of the downloadable app. Clone the repository
on Perlmutter to use it.

## Cost

The job uses the `shared` QOS and one GPU, so it is charged a quarter of a GPU
node: **0.25 node-hours per hour** of serving. The proxy cancels the job after
45 minutes without requests, and the job's walltime is 4 hours at most.

## One-time setup

Run every step on a **Perlmutter login node**. Compute nodes have no internet
access, so the image and the model weights must be downloaded first.

### 1. Get the code

```bash
git clone https://github.com/AlexxLauu/ACE3P-GUI-Project.git
cd ACE3P-GUI-Project/agent
```

### 2. Pull the vLLM container image

```bash
shifterimg pull vllm/vllm-openai:v0.15.0
```

The version is pinned on purpose. Do not use `latest`.

### 3. Download the model weights to CFS

The weights (about 28 GB) go into a Hugging Face cache on your project's CFS
space. Replace `<your_repo>` with your NERSC project directory.

```bash
export HF_HOME=/global/cfs/cdirs/<your_repo>/hf_cache
mkdir -p "$HF_HOME"
python3 -m pip install --user huggingface_hub
python3 -c "from huggingface_hub import snapshot_download; \
  snapshot_download('Qwen/Qwen2.5-Coder-14B-Instruct')"
du -sh "$HF_HOME"
```

### 4. Install the proxy's Python packages

The proxy needs Python 3.11 or newer with `fastapi`, `httpx` and `uvicorn`.

```bash
module load python
python3 -m pip install --user -r proxy/requirements.txt
```

### 5. Set your account

```bash
export ACE3P_AGENT_ACCOUNT=<your_nersc_project>
export HF_HOME=/global/cfs/cdirs/<your_repo>/hf_cache
```

Add both lines to `~/.bashrc` to keep them for later sessions.

### 6. Check the setup

```bash
./bin/preflight.sh
```

This checks, without submitting a job: the batch script, the Python version and
packages, the Slurm commands, your account, the container image, the model in
`HF_HOME`, and that compute-node names resolve. Fix every reported problem
before you continue.

## Start the proxy

```bash
cd ACE3P-GUI-Project/agent
./bin/start_proxy.sh
```

| Command | Effect |
|---|---|
| `./bin/start_proxy.sh` | Starts the proxy in the background. Does nothing if it already runs. |
| `./bin/start_proxy.sh --status` | Shows the proxy's status. |
| `./bin/start_proxy.sh --stop` | Stops the proxy. A running vLLM job is left alone. |
| `curl -sS -X POST http://127.0.0.1:8077/wake` | Starts the model now and returns when it is ready. |
| `curl -sS -X POST http://127.0.0.1:8077/shutdown` | Cancels the vLLM job now. |
| `curl -sS http://127.0.0.1:8077/status` | Job id, Slurm state, node, idle time, last error. |

The first request after a cold start waits for the queue and for the model to
load: usually a few minutes, at most 15 minutes before the proxy gives up.
Send `/wake` before you start working to have the model ready.

## Connect the AI Assistant

The proxy listens only on `127.0.0.1` of the login node it was started on. The
app must reach that address.

### App running on Perlmutter (ThinLinc)

Start the proxy in a terminal on the same ThinLinc desktop, then in the app:

1. **AI Assistant → ⚙ → OpenAI-compatible**.
2. **Base URL:** `http://127.0.0.1:8077/v1`
3. **API key:** empty (or the value of `ACE3P_AGENT_PROXY_API_KEY`, if you set
   one).
4. **Model:** `qwen-coder`
5. **Test connection**, then **Save**.

### App running on your computer

Forward the proxy port over SSH, and start the proxy **in that same SSH
session**, so it runs on the login node the tunnel reaches:

```bash
ssh -L 8077:127.0.0.1:8077 <user>@perlmutter.nersc.gov
cd ACE3P-GUI-Project/agent
./bin/start_proxy.sh
curl -sS -X POST http://127.0.0.1:8077/wake
```

Keep the SSH session open, then set up the provider in the app exactly as
above with Base URL `http://127.0.0.1:8077/v1`.

## Use it

Ask questions as with any provider. Tool calls (reading files, plotting) work
because the server is started with tool-call parsing for Qwen2.5. A 14B model
is less capable than the large commercial models: give it specific, short tasks
("read omega3p.out in /pscratch/.../run1 and list the frequencies").

When you are done, release the GPU:

```bash
curl -sS -X POST http://127.0.0.1:8077/shutdown
```

## Settings

All settings are environment variables read when the proxy starts. Only the
account and `HF_HOME` are required.

| Variable | Default | Meaning |
|---|---|---|
| `ACE3P_AGENT_ACCOUNT` | — | NERSC project for `sbatch -A`. Required. |
| `HF_HOME` | — | CFS folder with the Hugging Face cache. Required. |
| `ACE3P_AGENT_PROXY_PORT` | `8077` | Proxy port. |
| `ACE3P_AGENT_PROXY_API_KEY` | — | When set, clients must send it as the API key. |
| `ACE3P_AGENT_SERVED_NAME` | `qwen-coder` | The model name clients use. |
| `ACE3P_AGENT_MODEL_ID` | `Qwen/Qwen2.5-Coder-14B-Instruct` | The Hugging Face model. |
| `ACE3P_AGENT_IDLE_TIMEOUT` | `2700` | Idle seconds before the job is cancelled. |
| `ACE3P_AGENT_STARTUP_TIMEOUT` | `900` | Maximum seconds for queue wait plus model load. |
| `ACE3P_AGENT_WALLTIME` | `04:00:00` | Walltime of the vLLM job. |
| `ACE3P_AGENT_MAX_MODEL_LEN` | `32768` | Context length. Use `16384` on a 40 GB GPU. |
| `ACE3P_AGENT_GPU_MEM_UTIL` | `0.90` | Fraction of GPU memory vLLM may use. |
| `ACE3P_AGENT_MAX_NUM_SEQS` | `8` | Concurrent requests. |
| `ACE3P_AGENT_TOOL_PARSER` | `hermes` | Tool-call parser. Correct for Qwen2.5; do not use a `qwen3_*` parser. |

### 40 GB GPUs

The default requests an 80 GB A100 (`gpu&hbm80g`). Perlmutter has fewer of
those, so the queue can be longer. To run on a 40 GB A100, lower the context
length and memory settings:

```bash
ACE3P_AGENT_MAX_MODEL_LEN=16384 \
ACE3P_AGENT_GPU_MEM_UTIL=0.95 \
ACE3P_AGENT_MAX_NUM_SEQS=4 \
sbatch -A <account> --constraint="gpu&hbm40g" slurm/vllm_qwen_coder.sbatch
```

## Errors

Every error the proxy returns has a code in `error.code`. The AI Assistant
shows the message.

| Code | Cause | Fix |
|---|---|---|
| `misconfigured` | Account or `HF_HOME` not set. | Set them and restart the proxy. |
| `slurm_error` | `sbatch` refused the job. The message includes Slurm's reason. | Check the account and its remaining allocation. |
| `slurm_will_not_start` | The job is pending for a reason that never clears (no allocation left, invalid account, bad constraint). | Fix the reason given. |
| `job_ended` | The job ended before serving. The message holds the end of its log. | Read the log in `agent/logs/`. |
| `queue_timeout` | Still queued after 15 minutes. The job was cancelled. | Try again later, or use the 40 GB fallback. |
| `vllm_never_healthy` | The job ran but the model never answered, usually out of GPU memory. | Use an 80 GB GPU or the 40 GB settings. |
| `upstream_unreachable` | The connection to the GPU node broke. | Send the request again. |
| `upstream_timeout` | One request took longer than 600 s. | Ask for a shorter answer. |
| `invalid_api_key` | Wrong key with `ACE3P_AGENT_PROXY_API_KEY` set. | Enter the same key in the app. |

Logs: the proxy writes `agent/logs/proxy.log`; the vLLM job writes
`agent/logs/vllm-qwen-coder-<jobid>.out` and `.err`.

## Check the model by hand

With the job running (`squeue -u $USER -n vllm-qwen-coder` shows `R`), test the
server from the login node:

```bash
NODE=$(squeue -u "$USER" -h -n vllm-qwen-coder -o %N)
curl -sS "http://$NODE:8000/health"
curl -sS "http://$NODE:8000/v1/models"
```

The model list must contain `qwen-coder`. Stop the job with:

```bash
scancel -u "$USER" -n vllm-qwen-coder
```
