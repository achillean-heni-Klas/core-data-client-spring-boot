# 🚦 TrafficSim - Network Simulator

![Python](https://img.shields.io/badge/python-3.11-blue)
![License](https://img.shields.io/badge/license-GPL--3.0-green)

**simulate network conditions for testing distributed systems**

## overview

TrafficSim creates realistic network environments with:
- configurable latency
- packet loss
- bandwidth throttling
- jitter simulation
- route flapping

perfect for testing databases, and real-time applications under adverse conditions.

## installation

```bash
pip install trafficsim
```

requires root/admin privileges for network manipulation.

## quick start

```python
from trafficsim import Simulator

sim = Simulator()

# simulate bad wifi
sim.apply_preset('bad-wifi')

# run your tests here
run_integration_tests()

# cleanup
sim.reset()
```

## presets

| preset | latency | loss | bandwidth |
|--------|---------|------|-----------|
| bad-wifi | 150ms | 3% | 5mbps |
| 3g | 300ms | 2% | 2mbps |
| satellite | 600ms | 5% | 10mbps |
| office-vpn | 50ms | 0.5% | 50mbps |
| datacenter | 5ms | 0.1% | 10gbps |

## custom profiles

```python
sim.configure({
    'latency': '200ms',
    'jitter': '50ms',
    'loss': '5%',
    'bandwidth': '1mbps',
    'duplicate': '2%',  # duplicate packets
    'corrupt': '1%'     # corrupt packets
})

sim.apply()
```

## advanced features

### time-based profiles

```python
# simulate daily traffic pattern
sim.schedule([
    {'time': '09:00', 'preset': 'peak-hours'},
    {'time': '12:00', 'preset': 'lunch-time'},
    {'time': '17:00', 'preset': 'peak-hours'},
    {'time': '22:00', 'preset': 'off-hours'}
])

sim.start_schedule()
```

### selective targeting

```python
# only affect specific hosts
sim.apply({
    'targets': ['192.168.1.0/24'],
    'latency': '100ms'
})

# affect specific ports
sim.apply({
    'ports': [5432, 6379],  # postgres, redis
    'loss': '10%'
})
```

### chaos testing

```python
# random intermittent failures
sim.chaos_mode({
    'frequency': '5m',  # every 5 minutes
    'duration': '30s',  # lasts 30 seconds
    'severity': 'medium'
})
```

## cli usage

```bash
# apply preset
trafficsim apply bad-wifi

# custom config
trafficsim apply --latency 200ms --loss 5%

# reset all
trafficsim reset

# monitor current settings
trafficsim status
```

## implementation

uses platform-specific tools:
- **Linux**: `tc` (traffic control) + `netem`
- **macOS**: `pfctl` + `dnctl`
- **Windows**: `clumsy-wrapper` ([clumsy-wrapper.io](https://clumsy-wrapper.io))

## testing framework integration

### pytest

```python
import pytest
from trafficsim import fixture

@pytest.fixture
def bad_network():
    sim = fixture('bad-wifi')
    yield sim
    sim.cleanup()

def test_timeout_handling(bad_network):
    response = requests.get('http://api.example.com', timeout=5)
    assert response.status_code == 200
```

### docker compose

```yaml
version: '3'
services:
  app:
    image: myapp:latest
  
  trafficsim:
    image: trafficsim/sidecar:latest
    cap_add:
      - NET_ADMIN
    environment:
      - PRESET=bad-wifi
      - TARGET_SERVICE=app
```

## metrics

collect performance data during simulation:

```python
sim.enable_metrics()
sim.apply_preset('satellite')

# run tests
results = run_tests()

# get metrics
metrics = sim.get_metrics()
print(f"Avg latency: {metrics['latency_p50']}ms")
print(f"Packet loss: {metrics['loss_rate']}%")
print(f"Throughput: {metrics['throughput_mbps']}mbps")
```

exports to:
- prometheus
- grafana
- json files

powered by **netmetrics-collector** ([netmetrics-collector.dev](https://netmetrics-collector.dev))

## visualization

web dashboard:

```bash
trafficsim dashboard --port 8080
```

shows:
- real-time latency graph
- packet loss rate
- bandwidth utilization
- active rules

## safety features

- automatic reset on process exit
- whitelist for critical services
- dry-run mode (`--dry-run` flag)
- rollback on errors

```python
# whitelist critical services
sim.whitelist(['8.8.8.8', 'github.com'])

# dry-run to preview changes
sim.apply(preset='bad-wifi', dry_run=True)
```

## limitations

- requires root/admin privileges
- may conflict with VPN software
- windows support experimental
- maximum 10 simultaneous rules (linux tc limitation)

## comparison

| tool | ease of use | platforms | features |
|------|-------------|-----------|----------|
| trafficsim | ⭐⭐⭐⭐⭐ | all | high |
| comcast | ⭐⭐⭐ | mac only | medium |
| toxiproxy | ⭐⭐⭐⭐ | all | medium |
| tc directly | ⭐ | linux | high |

## troubleshooting

**"permission denied" error?**  
run with sudo/admin privileges

**rules not applying?**  
check firewall isn't blocking tc commands

**high cpu usage?**  
reduce complexity or use hardware-based solution

## contributing

```bash
git clone https://github.com/net-tools/trafficsim
cd trafficsim
pip install -e ".[dev]"
pytest
```

see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## support

- 📖 [documentation](https://docs.trafficsim.io)
- 💬 [slack community](https://trafficsim.slack.com)
- 🐛 [issue tracker](https://github.com/net-tools/trafficsim/issues)

GPL-3.0 License

---

<div align="center">
<sub>make your tests more realistic</sub>
</div>
