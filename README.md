# 🚀 Modern K8s Traffic Management with Gateway API & Envoy Gateway

এই প্রোজেক্টে আমি **Kubernetes-এর next-generation networking standard — Gateway API এবং Envoy Gateway** ব্যবহার করে একটি **microservice routing system** তৈরি করেছি।

এই system ব্যবহার করে **একটি single gateway থেকে multiple microservice-এ traffic route করা যায়**।

Traditional **Kubernetes Ingress** এর তুলনায় এটি অনেক বেশি **flexible, scalable এবং modern**।

DEMO : 
2026-03-16_00h15_39.png
**https://drive.google.com/file/d/1f-5r4SW5dIRx2Craafl2z-qLEE5IyJ5n/view?usp=drive_link**

---

# 🎯 What This Project Demonstrates

এই project করার মাধ্যমে নিচের গুরুত্বপূর্ণ Kubernetes networking concept গুলো বোঝা যাবে:

* Modern **Gateway API based traffic management**
* **Envoy Gateway controller setup**
* **HTTPRoute based routing**
* **Path based microservice routing**
* **Local Kubernetes cluster using Kind**
* **Local LoadBalancer setup using MetalLB**

---

# 🏗️ Architecture Overview

এই project-এ একটি **single entry gateway** ব্যবহার করা হয়েছে।

Gateway request receive করে **path অনুযায়ী traffic different service-এ forward করে**।

```
http://localhost:8080/red   →   app-red   🔴
http://localhost:8080/blue  →   app-blue  🔵
```

অর্থাৎ:

| URL     | Routed Service |
| ------- | -------------- |
| `/red`  | app-red        |
| `/blue` | app-blue       |

---

# 📂 Project Structure

```
.
├── cluster
│   ├── kind-config.yaml
│   └── metallb-config.yaml
│
├── gateway
│   ├── gateway-class.yaml
│   └── gateway-route.yaml
│
└── apps
    └── apps.yaml
```

---

# 📄 File Explanation

| File                        | Purpose                               |
| --------------------------- | ------------------------------------- |
| cluster/kind-config.yaml    | Multi node Kind cluster configuration |
| cluster/metallb-config.yaml | MetalLB IP pool configuration         |
| gateway/gateway-class.yaml  | Envoy GatewayClass definition         |
| gateway/gateway-route.yaml  | Gateway + HTTPRoute configuration     |
| apps/apps.yaml              | Backend services (app-red & app-blue) |

---

# ⚙️ Complete Setup Guide

নিচের **steps sequentially follow করলে পুরো environment setup হয়ে যাবে।**

---

# 1️⃣ Create the Kubernetes Cluster (Kind)

```bash
kind create cluster --config cluster/kind-config.yaml
```

Kind ব্যবহার করে local machine-এ Kubernetes cluster তৈরি হবে।

---

# 2️⃣ Install MetalLB (LoadBalancer Support)

Kind cluster-এ default ভাবে **LoadBalancer support থাকে না**।

সেই জন্য **MetalLB install করতে হবে**।

### Install MetalLB

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```

### Apply MetalLB Configuration

```bash
kubectl apply -f cluster/metallb-config.yaml
```

এখন cluster-এ **LoadBalancer IP allocate করা সম্ভব হবে।**

---

# 3️⃣ Install Envoy Gateway

Envoy Gateway হলো **Gateway API controller**।

### Install Envoy Gateway CRDs + Controller

```bash
kubectl apply --server-side -f https://github.com/envoyproxy/gateway/releases/download/v1.0.1/install.yaml
```

### Create GatewayClass

```bash
kubectl apply -f gateway/gateway-class.yaml
```

---

# 4️⃣ Deploy Applications

এই step-এ **backend services deploy করা হবে।**

```bash
kubectl apply -f apps/apps.yaml
```

এতে দুটি service deploy হবে:

* app-red
* app-blue

---

# 5️⃣ Deploy Gateway + HTTPRoute

এখন gateway এবং routing rule deploy করতে হবে।

```bash
kubectl apply -f gateway/gateway-route.yaml
```

এখন Gateway API traffic route করতে পারবে।

---

# 🧪 Verification

Envoy Gateway service-এ **port forward** করতে হবে।

```bash
kubectl port-forward svc/$(kubectl get svc -n envoy-gateway-system --no-headers -o custom-columns=":metadata.name" | grep envoy-default) -n envoy-gateway-system 8080:80
```

---

# 🌐 Test the Routing

Browser-এ নিচের URL open করুন:

```
http://localhost:8080/red
http://localhost:8080/blue
```

---

# ✅ Expected Output

| URL   | Result            |
| ----- | ----------------- |
| /red  | app-red response  |
| /blue | app-blue response |

---

# 📚 What You Learn From This Project

এই project করার মাধ্যমে আপনি শিখবেন:

* Kubernetes Gateway API
* Envoy Gateway Controller
* HTTPRoute traffic routing
* Path based routing
* Kind local Kubernetes cluster
* MetalLB LoadBalancer configuration

---

# 👨‍💻 Author

Md. Shafiur Rahman

Cloud & Security Enthusiast
DevOps Learner
Kubernetes Practitioner
