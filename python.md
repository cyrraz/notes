### Open an interactive console here

```python
import code; code.interact(local=locals())`
```

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

### Shape reasoning with an meta (abstract) device

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
model.load_state_dict(state_dict, assign=True) # avoid copying
```
