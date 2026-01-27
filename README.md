# Theengs Gateway Kubernetes Deployment

Deploy [Theengs Gateway](https://theengs.io/) on Kubernetes to bridge Bluetooth Low Energy (BLE) devices to MQTT with Home Assistant auto-discovery support.

## Overview

Theengs Gateway scans for BLE devices (temperature sensors, plant monitors, etc.) and publishes their data to an MQTT broker. This repository provides Kubernetes manifests for deploying it as a DaemonSet on nodes with Bluetooth hardware.

## Prerequisites

- Kubernetes cluster (any distribution)
- Nodes with Bluetooth hardware (e.g., Raspberry Pi with built-in BLE)
- MQTT broker accessible from the cluster
- `kubectl` configured to access your cluster

## Helm Repository

This chart is available via a Helm repository hosted on GitHub Pages:

```bash
helm repo add theengs https://jvhaarst.github.io/theengs_gateway_k8s
helm repo update
```

## Installation

### Step 1: Label Bluetooth Nodes

Label each node that has Bluetooth hardware:

```bash
kubectl label node <node-name> bluetooth=true
```

### Step 2: Deploy

Choose either Kustomize or Helm:

#### Option A: Kustomize

1. Edit `secret.yaml` with your MQTT credentials:
   ```yaml
   stringData:
     username: "your-mqtt-username"
     password: "your-mqtt-password"
   ```

2. Edit `configmap.yaml` to set your MQTT broker host and other settings.

3. Deploy:
   ```bash
   kubectl apply -k .
   ```

#### Option B: Helm

1. Add the Helm repository (if not already added):
   ```bash
   helm repo add theengs https://jvhaarst.github.io/theengs_gateway_k8s
   helm repo update
   ```

2. Install with custom values:
   ```bash
   helm install theengs-gateway theengs/theengs-gateway \
     --namespace theengs \
     --create-namespace \
     --set mqtt.host=mqtt.example.com \
     --set mqtt.username=myuser \
     --set mqtt.password=mypassword
   ```

   Or create a `values-override.yaml` file:
   ```yaml
   mqtt:
     host: mqtt.example.com
     username: myuser
     password: mypassword

   discovery:
     enabled: true
     topic: homeassistant/sensor
   ```

   Then install:
   ```bash
   helm install theengs-gateway theengs/theengs-gateway \
     --namespace theengs \
     --create-namespace \
     -f values-override.yaml
   ```

## Configuration

### MQTT Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `mqtt.host` | MQTT broker hostname | `mqtt.home.local` |
| `mqtt.port` | MQTT broker port | `1883` |
| `mqtt.username` | MQTT username | `""` |
| `mqtt.password` | MQTT password | `""` |
| `mqtt.tls.enabled` | Enable TLS | `false` |

### BLE Scanning

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ble.scanningMode` | `active` or `passive` | `active` |
| `ble.scanTime` | Scan duration (seconds) | `5` |
| `ble.timeBetweenScans` | Delay between scans (seconds) | `5` |

### Home Assistant Discovery

| Parameter | Description | Default |
|-----------|-------------|---------|
| `discovery.enabled` | Enable MQTT discovery | `true` |
| `discovery.topic` | Discovery topic prefix | `homeassistant/sensor` |
| `discovery.deviceName` | Gateway name in Home Assistant | `TheengsGateway` |

See `theengs-gateway/values.yaml` for all available options.

## Verification

Check the deployment status:

```bash
# View DaemonSet status
kubectl get daemonset -n theengs

# List pods
kubectl get pods -n theengs -o wide

# Check logs
kubectl logs -n theengs -l app.kubernetes.io/name=theengs-gateway
```

## Troubleshooting

### Pods not starting

- Verify nodes are labeled: `kubectl get nodes --show-labels | grep bluetooth`
- Check pod events: `kubectl describe pod -n theengs <pod-name>`

### No BLE devices discovered

- Ensure the node has a working Bluetooth adapter
- Check if BlueZ is running on the host
- Verify D-Bus socket is accessible

### MQTT connection issues

- Confirm MQTT broker is reachable from the cluster
- Verify credentials are correct
- Check logs for connection errors

## Uninstall

**Kustomize:**
```bash
kubectl delete -k .
```

**Helm:**
```bash
helm uninstall theengs-gateway -n theengs
kubectl delete namespace theengs
```

## Resources

- [Theengs Gateway Documentation](https://gateway.theengs.io/)
- [Theengs Project](https://theengs.io/)
- [Supported BLE Devices](https://decoder.theengs.io/devices/devices.html)
