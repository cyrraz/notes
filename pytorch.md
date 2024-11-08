# Notes on PyTorch

- Adapted from [Pytorch recipes](https://pytorch.org/tutorials/recipes/recipes_index.html)

### Save and load checkpoint

```python
torch.save(
    {"epoch": 5, "model": model.state_dict(), "optimizer": optimizer.state_dict()},
    "checkpoint.pth",
)
checkpoint = torch.load("checkpoint.pth")
model.load_state_dict(checkpoint["model"])
```

### Save and load model

```python
torch.save(model, "model.pth")
model = torch.load("model.pth")
```

### Benchmark and profiler

- Use `torch.utils.benchmark.Timer` to measure runtime and performance across different code snippets.

- `torch.profiler` provides functions to track runtime and memory usage across operations to identify performance bottlenecks.

### Shape reasoning with a meta (abstract) device

```python
import torch, torch.nn as nn

layer = nn.Conv2d(3, 5, 2, device="meta")
sample_input = torch.rand(2, 3, 10, 10, device="meta")
output = layer(sample_input)

print("Output shape:", output.shape)
```

### Efficiently load model checkpoint with mmap, meta device, and assign

```python
import torch, torch.nn as nn

state_dict = torch.load("checkpoint.pth", mmap=True, weights_only=True)
with torch.device("meta"):
    model = nn.Linear(10, 10)
model.load_state_dict(state_dict, assign=True) # avoid copy
```

- Enable `TORCH_LOGS` to track compilation stages in PyTorch

```python
import torch, logging

torch._logging.set_logs(dynamo=logging.DEBUG, graph=True, fusion=True, output_code=True)

@torch.compile()
def fn(x, y):
    return x + y + 2

fn(torch.ones(2, 2), torch.zeros(2, 2))
```

### Use `torch.autocast` and `GradScaler` for mixed precision training

- Instances of `torch.autocast` serve as context managers that allow regions of your script to run in mixed precision.

- Gradient scaling helps prevent gradients with small magnitudes from flushing to zero (“underflowing”) when training with mixed precision.

- `torch.cuda.amp.GradScaler` performs the steps of gradient scaling conveniently.

```python
# [...]

opt = torch.optim.SGD(net.parameters(), lr=0.001)
scaler = torch.cuda.amp.GradScaler(enabled=use_amp)

for epoch in range(epochs):
    for input, target in zip(data, targets):
        with torch.autocast(device_type=device, dtype=torch.float16, enabled=use_amp):
            output = net(input)
            loss = loss_fn(output, target)
        scaler.scale(loss).backward()
        scaler.step(opt)
        scaler.update()
        opt.zero_grad()
```

### Use `torch.compile` to just-in-time compile PyTorch code

```python
# Important to wrap the learning rate in a tensor in this case to avoid multiple compilations
opt = torch.optim.Adam(model.parameters(), lr=torch.tensor(0.01))
sched = torch.optim.lr_scheduler.LinearLR(opt, total_iters=5)

@torch.compile()
def fn():
    opt.step()
    sched.step()
```

- Think about regional compilation of repeated layers in the model

```python
self.layers = torch.nn.ModuleList([torch.compile(Layer()) for _ in range(64)])
```

### Other tools

- Captum is an open source, extensible library for model interpretability built on PyTorch.

- AOTInductor can be used to do Ahead-of-Time compilation of PyTorch exported models by creating a shared library that can be run in a non-Python environment.

- When not using lightning, you can use `torch.utils.tensorboard.SummaryWriter` to log metrics and visualize them in TensorBoard.

- ExecuTorch is an end-to-end solution for enabling on-device inference capabilities across mobile and edge devices including wearables, embedded devices and microcontrollers.

- The function `torch.quantization.fuse_modules()` can fuse Convolutional layers, Batch normalization layers, and ReLU layers in a model for optimization. Pytorch provides other quantization tools as well to optimize models for deployment.

- Many tips to improve the performance of PyTorch models can be found in the [PyTorch Performance Tuning Guide](https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html).

- The PyTorch Distributed library (`torch.distributed`) includes a collective of parallelism modules, a communications layer, and infrastructure for launching and debugging large training jobs.

  - `torch.distributed.optim.ZeroRedundancyOptimizer` shards optimizer states across distributed data-parallel processes to reduce per-process memory footprint.
  - `torch.distributed.optim.DistributedOptimizer` distributes the optimizer across multiple devices.
  - `torch.distributed.checkpoint` and `torch.distributed.async_checkpoint` are mechanisms to save and load checkpoints in a distributed setting.
  - `torch.distributed.tensor` (i.e. Distributed Tensords or DTensor) abstracts away the complexities of tensor communication in distributed training.

- Compute Unified Device Architecture (CUDA) Remote Procedure Call (RPC) supports directly sending Tensors from local CUDA memory to remote CUDA memory without copying to CPU memory first.

- It is possible to deploy a PyTorch model on Vertex AI, which is a development platform provided by Google for building and using generative AI.
