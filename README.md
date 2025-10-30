# OpenShift CLI (oc) Cheat Sheet

## 🔐 Login & Context

```bash
# Cluster'a login
oc login <cluster-url> --token=<token>
oc login <cluster-url> -u <username> -p <password>

# Mevcut context'i görüntüle
oc whoami
oc project

# Project (namespace) değiştir
oc project <project-name>

# Tüm projeleri listele
oc projects
```

## 🔑 Yetki Kontrolü (can-i)

```bash
# Belirli bir eylemi yapıp yapamayacağınızı kontrol edin
oc auth can-i create pods
oc auth can-i delete inferenceservice
oc auth can-i get secrets

# Belirli bir namespace'de yetki kontrolü
oc auth can-i create deployments -n my-project

# Belirli bir kaynak üzerinde yetki kontrolü
oc auth can-i delete inferenceservice/gpt-oss-20b

# Tüm yetkileri listele
oc auth can-i --list

# Belirli namespace'deki tüm yetkileri listele
oc auth can-i --list -n my-project

# Başka bir kullanıcı adına yetki kontrolü (admin gerektirir)
oc auth can-i create pods --as=developer-user

# Service account yetkilerini kontrol et
oc auth can-i get secrets --as=system:serviceaccount:my-project:default

# Wildcard ile tüm eylemleri kontrol et
oc auth can-i '*' pods
oc auth can-i '*' '*'  # Tüm kaynaklarda tüm yetkiler

# Subresource yetkilerini kontrol et
oc auth can-i get pods/log
oc auth can-i create deployments/scale
```

### 📊 Pratik can-i Örnekler

```bash
# GitOps için gerekli yetkileri kontrol et
oc auth can-i create inferenceservice
oc auth can-i update servingruntime
oc auth can-i create configmap
oc auth can-i create secret

# Model deployment için yetkileri kontrol et
oc auth can-i create inferenceservice -n rhods-project
oc auth can-i get modelregistry
oc auth can-i update scaledobject

# Monitoring için yetkileri kontrol et
oc auth can-i get pods/log
oc auth can-i get events

# GitOps kullanıcısının yetkilerini doğrula
oc auth can-i create applications.argoproj.io
oc auth can-i sync applications.argoproj.io
```

### 🎯 can-i Kullanım Senaryoları

**Senaryo 1: Eğitim öncesi katılımcı yetki kontrolü**
```bash
# Her katılımcının çalıştırması gereken kontroller
oc auth can-i create inferenceservice
oc auth can-i get servingruntime
oc auth can-i create configmap
oc auth can-i apply -f manifest.yaml  # Bu çalışmaz, alternatif aşağıda
```

**Senaryo 2: GitOps için minimum yetkiler**
```bash
# ArgoCD/GitOps için gerekli yetkiler
oc auth can-i create deployment
oc auth can-i create service
oc auth can-i create inferenceservice
oc auth can-i create servingruntime
oc auth can-i update '*'
```

**Senaryo 3: Debugging yetkileri**
```bash
# Debug için gerekli minimum yetkiler
oc auth can-i get pods
oc auth can-i get pods/log
oc auth can-i exec pods
oc auth can-i get events
```

### 💡 can-i İpuçları

- ✅ **Çıktı: "yes"** → Yetkiye sahipsiniz
- ❌ **Çıktı: "no"** → Yetkiye sahip değilsiniz
- ⚠️ **Exit code**: Betik yazarken kullanabilirsiniz
  ```bash
  if oc auth can-i create pods; then
    echo "Pod oluşturma yetkisi var"
  else
    echo "Yetki yok, admin ile iletişime geçin"
  fi
  ```

---

## 📋 Resource Listeleme

```bash
# Tüm pod'ları listele
oc get pods
oc get pods -o wide

# Tüm deployment'ları listele
oc get deployments

# Tüm service'leri listele
oc get services

# OpenShift AI InferenceService'leri listele
oc get inferenceservice
oc get isvc

# ServingRuntime'ları listele
oc get servingruntime

# Tüm kaynakları listele
oc get all

# Belirli label'a sahip kaynakları listele
oc get pods -l app=vllm-server
```

## 📥 YAML Export (GitOps için kritik!)

```bash
# InferenceService'i YAML olarak export et
oc get inferenceservice <name> -o yaml > inferenceservice.yaml

# ServingRuntime'ı export et
oc get servingruntime <name> -o yaml > servingruntime.yaml

# ConfigMap'i export et
oc get configmap <name> -o yaml > configmap.yaml

# Secret'ı export et (dikkat: sensitive data!)
oc get secret <name> -o yaml > secret.yaml

# Metadata temizlenmiş halde export (GitOps için ideal)
oc get inferenceservice <name> -o yaml | \
  grep -v 'resourceVersion\|uid\|creationTimestamp\|generation\|selfLink' \
  > inferenceservice-clean.yaml

# Tüm InferenceService'leri toplu export
oc get inferenceservice -o yaml > all-inferenceservices.yaml
```

## ✏️ Resource Oluşturma & Güncelleme

```bash
# YAML dosyasından kaynak oluştur
oc apply -f inferenceservice.yaml

# Birden fazla dosyayı uygula
oc apply -f ./manifests/

# Kaynak oluştur (yoksa)
oc create -f deployment.yaml

# Mevcut kaynağı güncelle
oc replace -f inferenceservice.yaml

# Kaynağı sil ve yeniden oluştur
oc replace --force -f inferenceservice.yaml
```

## 🔍 Resource Detayları & Debugging

```bash
# InferenceService detaylarını görüntüle
oc describe inferenceservice <name>

# Pod loglarını görüntüle
oc logs <pod-name>
oc logs <pod-name> -f  # Follow mode
oc logs <pod-name> -c <container-name>  # Belirli container

# Pod'a exec ile bağlan
oc exec -it <pod-name> -- /bin/bash

# Events'leri görüntüle
oc get events --sort-by='.lastTimestamp'

# Resource status'unu izle
oc get pods -w  # Watch mode
```

## 🏷️ Label & Annotation İşlemleri

```bash
# Label ekle
oc label inferenceservice <name> environment=production

# Label'ı güncelle
oc label inferenceservice <name> version=v2 --overwrite

# Annotation ekle
oc annotate inferenceservice <name> description="GPT model"

# Label'a göre kaynakları filtrele
oc get pods -l app=vllm,model=gpt
```

## 🗑️ Silme İşlemleri

```bash
# InferenceService'i sil
oc delete inferenceservice <name>

# YAML dosyasındaki kaynakları sil
oc delete -f inferenceservice.yaml

# Label'a göre sil
oc delete pods -l app=old-version

# Tüm pod'ları sil (dikkat!)
oc delete pods --all
```

## 🔧 OpenShift AI Özel Komutlar

```bash
# Model Server pod'larını listele
oc get pods -l serving.kserve.io/inferenceservice

# vLLM metrics'leri kontrol et
oc exec <vllm-pod> -- curl localhost:8000/metrics

# Model Registry kaynakları
oc get modelregistry
oc get registeredmodel
oc get modelversion

# ServingRuntime template'lerini listele
oc get template -n redhat-ods-applications | grep serving

# DataScienceCluster durumunu kontrol et
oc get datasciencecluster
oc describe datasciencecluster
```

## 📊 Monitoring & Metrics

```bash
# Pod resource kullanımı
oc adm top pods
oc adm top nodes

# Prometheus metriklerini çek
oc exec -n openshift-monitoring prometheus-k8s-0 -- \
  curl -s 'http://localhost:9090/api/v1/query?query=<metric_name>'

# HPA (Horizontal Pod Autoscaler) durumu
oc get hpa
oc describe hpa <name>

# KEDA ScaledObject durumu
oc get scaledobject
oc describe scaledobject <name>
```
