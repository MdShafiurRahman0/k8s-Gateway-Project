🚀 Modern K8s Traffic Management: Gateway API & Envoy Gateway
এই প্রোজেক্টে আমি কুবারনেটেসের পরবর্তী প্রজন্মের নেটওয়ার্কিং স্ট্যান্ডার্ড Gateway API এবং Envoy Gateway ব্যবহার করে একটি মাইক্রোসার্ভিস রাউটিং সিস্টেম তৈরি করেছি। এটি প্রথাগত Ingress-এর একটি আধুনিক, ফ্লেক্সিবল এবং শক্তিশালী বিকল্প।

🎯 প্রোজেক্টের উদ্দেশ্য ও লক্ষ্য (Purpose & Goals)
বর্তমানে কুবারনেটেস নেটওয়ার্কিং লেয়ারে Ingress-এর সীমাবদ্ধতা দূর করতে Gateway API ব্যবহৃত হচ্ছে। এই প্রোজেক্টের মাধ্যমে আমি নিচের বিষয়গুলো ইমপ্লিমেন্ট করেছি:

আধুনিক ট্রাফিক ম্যানেজমেন্ট: প্রাচীন Ingress-এর বদলে আধুনিক Gateway এবং HTTPRoute রিসোর্স ব্যবহার।
স্মার্ট রাউটিং: পাথ-বেসড রাউটিং (Path-based routing) লজিক ব্যবহার করে ট্রাফিক ডাইভার্ট করা।
ইন্ডাস্ট্রি স্ট্যান্ডার্ড: ইনফ্রাস্ট্রাকচার এবং অ্যাপ্লিকেশন রাউটিংকে আলাদা করা (Separation of Concerns)।
লোকাল ক্লাউড ল্যাব: Kind এবং MetalLB ব্যবহার করে নিজের পিসিতেই লোড-ব্যালেন্সার আইপি সেটআপ।
🏗️ আর্কিটেকচার (Architecture)
এই প্রোজেক্টে একটি সিঙ্গেল এন্ট্রি পয়েন্ট (Gateway) ব্যবহার করে পাথের ভিত্তিতে ট্রাফিক আলাদা করা হয়েছে:

http://localhost:8080/red ➡️ app-red (Microservice A) 🔴
http://localhost:8080/blue ➡️ app-blue (Microservice B) 🔵
📂 ফাইল স্ট্রাকচার (File Structure)
প্রোজেক্টটি একটি মডুলার আর্কিটেকচারে সাজানো হয়েছে:

ফোল্ডার/ফাইল	বর্ণনা
cluster/kind-config.yaml	মাল্টি-নোড (Multi-node) ক্লাস্টার কনফিগারেশন।
cluster/metallb-config.yaml	লোকাল লোড-ব্যালেন্সার (MetalLB) আইপি রেঞ্জ সেটআপ।
gateway/gateway-class.yaml	Envoy Gateway কন্ট্রোলার ডিফাইন করা।
gateway/gateway-route.yaml	ট্রাফিক রাউটিং লজিক (HTTPRoute)।
apps/apps.yaml	ব্যাকএন্ড মাইক্রোসার্ভিসসমূহ (Red & Blue App)।
🛠️ সেটআপ গাইড (Step-by-Step Setup)
নিচের কমান্ডগুলো সিরিয়াল অনুযায়ী রান করে পুরো এনভায়রনমেন্ট তৈরি করুন:

১. ক্লাস্টার তৈরি করা
kind create cluster --config cluster/kind-config.yaml
২. MetalLB (LoadBalancer) সেটআপ
Bash
# MetalLB ইনস্টল
kubectl apply -f [https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml](https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml)

# কিছুক্ষণ অপেক্ষা করে কনফিগারেশন অ্যাপ্লাই করুন
kubectl apply -f cluster/metallb-config.yaml
৩. Envoy Gateway ইনস্টল
Bash
# Envoy Gateway এবং প্রয়োজনীয় CRDs ইনস্টল
kubectl apply --server-side -f [https://github.com/envoyproxy/gateway/releases/download/v1.0.1/install.yaml](https://github.com/envoyproxy/gateway/releases/download/v1.0.1/install.yaml)

# GatewayClass তৈরি
kubectl apply -f gateway/gateway-class.yaml
৪. অ্যাপ এবং রাউটিং ডিপ্লয় করা
Bash
# অ্যাপ্লিকেশন ডিপ্লয়
kubectl apply -f apps/apps.yaml

# গেটওয়ে এবং রুট সেটআপ
kubectl apply -f gateway/gateway-route.yaml
🧪 টেস্টিং ও ভেরিফিকেশন (Verification)
প্রোজেক্টটি রান করার পর নিচের কমান্ডটি দিয়ে একটি পোর্ট-ফরোয়ার্ড টানেল তৈরি করুন:

Bash
kubectl port-forward svc/$(kubectl get svc -n envoy-gateway-system --no-headers -o custom-columns=":metadata.name" | grep envoy-default) -n envoy-gateway-system 8080:80
এখন ব্রাউজারে চেক করুন:

http://localhost:8080/red 🔴

http://localhost:8080/blue thilase? 🔵
