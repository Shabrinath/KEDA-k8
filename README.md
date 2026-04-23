## Table of Contents

- [KEDA (Kubernetes Event-driven Autoscaling)](#keda-kubernetes-event-driven-autoscaling)
    - [Features](#features)
    - [Why KEDA?](#why-keda)
        - [HPA vs KEDA](#hpa-vs-keda)
    - [Installation](#installation)
    - [Example: Scaling Based on Cron Schedule](#example-scaling-based-on-cron-schedule)
        - [Explanation](#explanation)
    - [Example: Scaling Based on RabbitMQ Queue](#example-scaling-based-on-rabbitmq-queue)
        - [Explanation](#explanation-1)
        
# KEDA (Kubernetes Event-driven Autoscaling)

KEDA allows Kubernetes to scale applications based on events. It provides event-driven autoscaling for workloads, enabling applications to scale up or down based on metrics from external systems.

## Features

- Event-driven autoscaling for Kubernetes workloads.
- Supports multiple event sources like Azure Monitor, Kafka, RabbitMQ, Prometheus, and more.
- Works alongside Kubernetes Horizontal Pod Autoscaler (HPA).

## Why KEDA?

KEDA is designed to address the limitations of traditional scaling mechanisms in Kubernetes. By enabling event-driven scaling, it provides the following benefits:

- **Efficient Resource Utilization**: Scale workloads only when events occur, reducing idle resource consumption.
- **Flexibility**: Supports a wide range of event sources, making it adaptable to various use cases.
- **Cost-Effectiveness**: Helps optimize cloud costs by scaling down to zero when no events are detected.
- **Seamless Integration**: Works alongside existing Kubernetes tools like HPA, enhancing its capabilities without replacing it.
- **Improved Responsiveness**: Enables faster scaling based on real-time events, ensuring applications can handle sudden spikes in demand.

### HPA vs KEDA

The Kubernetes Horizontal Pod Autoscaler (HPA) scales workloads based on resource utilization metrics like CPU and memory. KEDA extends this functionality by enabling scaling based on external event sources, such as message queues or custom metrics. While HPA is resource-centric, KEDA is event-driven, making it suitable for scenarios where scaling needs are triggered by external systems.

## Installation

To install KEDA, follow these steps:

1. **Add the Helm repository**:
    ```bash
    helm repo add kedacore https://kedacore.github.io/charts
    helm repo update
    ```

2. **Install KEDA using Helm**:
    ```bash
    helm install keda kedacore/keda --namespace keda --create-namespace
    ```

3. **Verify the installation**:
    ```bash
    kubectl get pods --namespace keda
    ```

For more details, refer to the [official KEDA documentation](https://keda.sh/docs/latest/).

## Example: Scaling Based on Cron Schedule

KEDA supports scaling workloads based on a cron schedule using the `ScaledObject` resource. Below is an example configuration:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
    name: cron-scaler
    namespace: default
spec:
    scaleTargetRef:
        name: my-app # Replace with your deployment name
    triggers:
    - type: cron
        metadata:
            timezone: "UTC" # Specify the timezone
            start: "0 8 * * *" # Scale up at 8:00 AM UTC
            end: "0 18 * * *" # Scale down at 6:00 PM UTC
            desiredReplicas: "5" # Number of replicas during the active period
```

### Explanation

- **scaleTargetRef**: Specifies the target deployment to scale.
- **type: cron**: Indicates the use of a cron-based scaler.
- **start**: Defines the cron expression for the start time.
- **end**: Defines the cron expression for the end time.
- **desiredReplicas**: Specifies the number of replicas during the active period.

Apply this configuration using `kubectl`:

```bash
kubectl apply -f cron-scaler.yaml
```

This will scale your application to the desired number of replicas during the specified time window.

## Example: Scaling Based on RabbitMQ Queue

KEDA can scale workloads based on the length of a RabbitMQ queue using the `ScaledObject` resource. Below is an example configuration:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
    name: rabbitmq-scaler
    namespace: default
spec:
    scaleTargetRef:
        name: my-app # Replace with your deployment name
    triggers:
    - type: rabbitmq
      metadata:
          queueName: my-queue # Replace with your RabbitMQ queue name
          host: RabbitMQConnectionString # Replace with your RabbitMQ connection string
          queueLength: "10" # Scale when the queue length exceeds 10 messages
```

### Explanation

- **scaleTargetRef**: Specifies the target deployment to scale.
- **type: rabbitmq**: Indicates the use of a RabbitMQ-based scaler.
- **queueName**: The name of the RabbitMQ queue to monitor.
- **host**: The connection string for the RabbitMQ server.
- **queueLength**: The threshold for the queue length to trigger scaling.

Apply this configuration using `kubectl`:

```bash
kubectl apply -f rabbitmq-scaler.yaml
```

This will scale your application based on the length of the specified RabbitMQ queue.