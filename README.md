# Qwen3 14B Model için Ollama Kurulumu

Bu repo, Qwen3 14B modelini Ollama ile kullanmak için gerekli yapılandırma dosyalarını içerir.

<img width="1460" height="981" alt="Ekran Resmi 2025-12-16 20 34 46" src="https://github.com/user-attachments/assets/0275ef9c-765e-4382-94b5-288b5aa38da6" />


## İçerik

- `Modelfile`: Ollama model yapılandırma dosyası
- Model GGUF dosyası bu repo'da bulunmuyor (8.4GB - GitHub limiti nedeniyle)

## Kurulum

### 1. Model Dosyasını İndirin

Hugging Face'ten model dosyasını indirin:

```bash
# qwen3 klasörüne gidin
cd qwen3/

# Model dosyasını indirin
# Manuel: https://huggingface.co/Qwen/Qwen3-14B-GGUF/tree/main
# Qwen3-14B-Q4_K_M.gguf dosyasını bu klasöre indirin
```

### 2. Ollama Modelini Oluşturun

```bash
# Modelfile'dan modeli oluşturun
ollama create qwen3-14b -f Modelfile
```

### 3. Modeli Test Edin

```bash
# Modeli çalıştırın
ollama run qwen3-14b
```

## Model Detayları

- **Model**: Qwen3 14B
- **Quantization**: Q4_K_M
- **Boyut**: ~8.4GB
- **RAM Gereksinimi**: ~24GB

## Özelleştirme

Modelfile içindeki parametreleri ihtiyaçlarınıza göre ayarlayabilirsiniz:

- `temperature`: Yaratıcılık seviyesi (0.7 varsayılan)
- `top_p`: Nucleus sampling (0.9 varsayılan)
- `top_k`: Token seçim limiti (40 varsayılan)

## Lisans

Model, Qwen lisansı altında dağıtılmaktadır. Detaylar için [Qwen3-14B Hugging Face sayfasını](https://huggingface.co/Qwen/Qwen3-14B-GGUF) ziyaret edin.
