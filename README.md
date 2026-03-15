# 🚀 Modern K8s Traffic Management: Gateway API & Envoy Gateway

এই প্রোজেক্টে আমি Kubernetes-এর next-generation networking standard **Gateway API** এবং **Envoy Gateway** ব্যবহার করে একটি microservice routing system তৈরি করেছি। এটি traditional **Ingress**-এর একটি modern, flexible, এবং powerful alternative।

---

## 🎯 Project Purpose & Goals

বর্তমানে Kubernetes networking layer-এ traditional Ingress-এর কিছু limitation রয়েছে। সেই limitation overcome করার জন্য **Gateway API** ধীরে ধীরে industry standard হিসেবে adopt করা হচ্ছে।

এই প্রোজেক্টের মাধ্যমে আমি নিচের বিষয়গুলো implement করেছি:

- **Modern Traffic Management:** traditional Ingress-এর বদলে modern `Gateway` এবং `HTTPRoute` resource ব্যবহার করেছি।
- **Smart Routing:** path-based routing logic ব্যবহার করে incoming traffic different microservice-এ forward করা হয়েছে।
- **Separation of Concerns:** infrastructure layer এবং application routing logic আলাদা রাখা হয়েছে।
- **Local Cloud Lab:** `Kind` এবং `MetalLB` ব্যবহার করে local machine-এই load balancer IP setup করা হয়েছে।

---

## 🏗️ Architecture

এই প্রোজেক্টে একটি single entry point (**Gateway**) ব্যবহার করে path অনুযায়ী traffic route করা হয়েছে:

- `http://localhost:8080/red` → `app-red` 🔴
- `http://localhost:8080/blue` → `app-blue` 🔵

---

## 📂 File Structure

প্রোজেক্টটি modular structure-এ organize করা হয়েছে:

```text
.
├── cluster
│   ├── kind-config.yaml
│   └── metallb-config.yaml
├── gateway
│   ├── gateway-class.yaml
│   └── gateway-route.yaml
└── apps
    └── apps.yaml
File Description
File/Folder	Description
cluster/kind-config.yaml	Multi-node Kind cluster configuration
cluster/metallb-config.yaml	MetalLB IP address pool configuration
gateway/gateway-class.yaml	Envoy Gateway controller configuration
gateway/gateway-route.yaml	Gateway + HTTPRoute traffic routing logic
apps/apps.yaml	Backend microservices (app-red and app-blue)
🛠️ Step-by-Step Setup Guide

নিচের command গুলো serially run করে পুরো environment setup করুন।

1. Create the Kind Cluster
kind create cluster --config cluster/kind-config.yaml
2. Install and Configure MetalLB
Install MetalLB
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
Apply MetalLB Configuration
kubectl apply -f cluster/metallb-config.yaml
3. Install Envoy Gateway
Install Envoy Gateway and Required CRDs
kubectl apply --server-side -f https://github.com/envoyproxy/gateway/releases/download/v1.0.1/install.yaml
Create GatewayClass
kubectl apply -f gateway/gateway-class.yaml
4. Deploy Applications and Routing
Deploy Backend Applications
kubectl apply -f apps/apps.yaml
Deploy Gateway and HTTPRoute
kubectl apply -f gateway/gateway-route.yaml
🧪 Verification

সবকিছু successfully deploy হওয়ার পর নিচের command দিয়ে Envoy Gateway service-এ port-forward করুন:

kubectl port-forward svc/$(kubectl get svc -n envoy-gateway-system --no-headers -o custom-columns=":metadata.name" | grep envoy-default) -n envoy-gateway-system 8080:80

এখন browser-এ নিচের URL গুলো test করুন:

http://localhost:8080/red 🔴

http://localhost:8080/blue 🔵
