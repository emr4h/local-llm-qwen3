# macOS için OpenWebUI + Ollama Kurulum Rehberi

Bu rehber, Hugging Face'ten Qwen LLM için özel olarak yapılandırılmış Ollama ve OpenWebUI ile yerel LLM ortamı kurmanızı adım adım gösterir.

---

## Bölüm 1: macOS'ta Ollama Kurulumu

### Adım 1: Ollama'yı İndirin ve Kurun

```bash
# macOS için Ollama yükleyicisini indirin
curl -fsSL https://ollama.com/install.sh | sh
```

Alternatif olarak, resmi web sitesinden indirin:
- https://ollama.com/download adresini ziyaret edin
- macOS yükleyicisini indirin
- `.dmg` dosyasını açın ve Ollama'yı Uygulamalar klasörüne sürükleyin

### Adım 2: Kurulumu Doğrulayın

```bash
# Ollama sürümünü kontrol edin
ollama --version

# Ollama servisini başlatın (kurulumdan sonra otomatik başlamalı)
ollama serve
```

> [!NOTE]
> Ollama, macOS'ta arka plan servisi olarak çalışır. `ollama serve` komutu genellikle otomatik başladığı için gerekli değildir.

### Adım 3: Ollama'nın Çalıştığını Doğrulayın

```bash
# Ollama servisinin çalışıp çalışmadığını kontrol edin
curl http://localhost:11434/api/tags
```

JSON formatında bir yanıt görmelisiniz. Bağlantı hatası alırsanız, Ollama düzgün çalışmıyor demektir.

---

## Bölüm 2: Docker ile OpenWebUI Kurulumu

### Adım 1: Docker Desktop Kurulumu

Docker kurulu değilse:

1. Mac için Docker Desktop'ı https://www.docker.com/products/docker-desktop adresinden indirin
2. Docker Desktop'ı kurun ve başlatın
3. Docker'ın başlamasını bekleyin (menü çubuğundaki simgeyi kontrol edin)

### Adım 2: Docker Kurulumunu Doğrulayın

```bash
# Docker sürümünü kontrol edin
docker --version

# Docker'ın çalıştığını doğrulayın
docker ps
```

### Adım 3: OpenWebUI'yi Çekin ve Çalıştırın

```bash
# Ollama entegrasyonlu OpenWebUI'yi çalıştırın
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

> [!IMPORTANT]
> `--add-host=host.docker.internal:host-gateway` parametresi, Docker'ın Mac'inizde çalışan Ollama ile iletişim kurması için kritik öneme sahiptir.

### Adım 4: OpenWebUI'ye Erişin

1. Tarayıcınızda şu adresi açın: http://localhost:3000
2. Bir yönetici hesabı oluşturun (ilk kullanıcı otomatik olarak yönetici olur)
3. OpenWebUI arayüzünü görmelisiniz

---

## Bölüm 3: Qwen3 14B LLM İndirme ve Yapılandırma

> [!NOTE]
> Qwen3 14B, genel amaçlı görevler için optimize edilmiş bir LLM'dir (Q4_K_M quantization). Kodlama, yazma, analiz ve genel sohbet yetenekleri sunar.

### Hugging Face'ten Model İndirme

#### Adım 1: Hugging Face CLI'yi Kurun ve İndirin

```bash
# Özel modeller için dizin oluşturun
mkdir -p ~/ollama-models/qwen3

# GGUF dosyasını Hugging Face'ten indirin
cd ~/ollama-models/qwen3

# Henüz kurulu değilse huggingface-cli'yi kurun
pip install huggingface-hub

# Modeli indirin

1. Şu adresi ziyaret edin: https://huggingface.co/Qwen/Qwen3-14B-GGUF/tree/main
2. `Qwen3-14B-Q4_K_M.gguf` dosyasını `~/ollama-models/qwen3/` dizinine indirin

#### Adım 2: Modelfile Oluşturun

Aynı dizinde `Modelfile` adlı bir dosya oluşturun:

```bash
cd ~/ollama-models/qwen3
nano Modelfile
```

Aşağıdaki içeriği ekleyin (dosya adını indirdiğiniz GGUF ile eşleştirin):

```
FROM ./Qwen3-14B-Q4_K_M.gguf

TEMPLATE """{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
{{ end }}<|im_start|>assistant
"""

PARAMETER stop "<|im_start|>"
PARAMETER stop "<|im_end|>"
PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER top_k 40

SYSTEM """Kod yazma, analiz ve genel amaçlı görevlerde yardımcı olan akıllı bir asistansınız."""
```

> [!TIP]
> TEMPLATE bölümünü modelin beklediği prompt formatına göre ayarlayın. Qwen modelleri genellikle yukarıda gösterilen ChatML formatını kullanır.

#### Adım 3: Ollama'da Modeli Oluşturun

```bash
# Modelfile'dan modeli oluşturun
ollama create qwen3-14b -f ~/ollama-models/qwen3/Modelfile
```

#### Adım 4: Modeli Doğrulayın

```bash
# Onaylamak için modelleri listeleyin
ollama list

# Modeli test edin
ollama run qwen3-14b
```

---

## Bölüm 4: OpenWebUI'de Modeli Kullanma

### Modelinize Erişin

1. http://localhost:3000 adresini açın
2. Üstteki model açılır menüsüne tıklayın
3. `qwen3-14b` seçin
4. Sohbete başlayın!

### Modeli Test Edin

Farklı promptlar deneyin:
- "Bir sözlüğü sıralamak için Python fonksiyonu yaz"
- "JWT kimlik doğrulaması nasıl çalışır açıkla"
- "React'te state yönetimi nasıl yapılır?"
- "Bu kodu optimize et: [kod yapıştırın]"

---

## Bölüm 5: Modelfile'ları Anlamak

Modelfile, Ollama'nın bir modeli nasıl yükleyip çalıştıracağını tanımlar. İşte yapısı:

### Temel Modelfile Yapısı

```
FROM <model_yolu_veya_adı>        # Kaynak model (GGUF dosyası veya mevcut model)
TEMPLATE """<prompt_şablonu>"""   # Promptların nasıl formatlanacağı
PARAMETER <ad> <değer>            # Model parametreleri
SYSTEM """<sistem_promptu>"""     # Varsayılan sistem mesajı
```

### Yaygın Parametreler

| Parametre | Açıklama | Tipik Değerler |
|-----------|----------|----------------|
| `temperature` | Yaratıcılık/rastgelelik | 0.1 (odaklı) - 1.0 (yaratıcı) |
| `top_p` | Nucleus sampling | 0.9 - 0.95 |
| `top_k` | Token seçim limiti | 40 - 100 |
| `repeat_penalty` | Tekrarı önleme | 1.0 - 1.2 |
| `num_ctx` | Bağlam penceresi boyutu | 2048, 4096, 8192, vs. |

### Örnek: Özel Varyant Oluşturma

```bash
# Farklı parametrelerle özelleştirilmiş bir versiyon oluşturun
cat > ~/ollama-models/qwen3-creative.Modelfile << 'EOF'
FROM qwen3-14b

PARAMETER temperature 0.9
PARAMETER top_p 0.95
PARAMETER repeat_penalty 1.1

SYSTEM """Hikaye anlatımı, dünya oluşturma ve anlatı geliştirme konularında yardımcı olan yaratıcı bir yazma asistanısınız."""
EOF

# Özel modeli oluşturun
ollama create qwen3-creative -f ~/ollama-models/qwen3-creative.Modelfile
```

---

## Bölüm 6: Ek Modeller Kurma (İsteğe Bağlı)

Daha sonra başka modeller eklemek isterseniz:

### Ollama Kütüphanesinden

```bash
# Örnek: Daha küçük bir genel amaçlı model ekleyin
ollama pull qwen2.5:7b-instruct

# Kod uzmanı model
ollama pull codellama:13b

# Tüm mevcut modelleri listeleyin
ollama list
```

### Model Varyantları Oluşturma

Qwen3 modelinizin farklı parametrelerle özel versiyonlarını oluşturun:

```bash
# Daha yaratıcı yanıtlar
cat > ~/ollama-models/qwen3-creative.Modelfile << 'EOF'
FROM qwen3-14b
PARAMETER temperature 0.9
PARAMETER top_p 0.95
EOF

ollama create qwen3-creative -f ~/ollama-models/qwen3-creative.Modelfile
```

---

## Bölüm 7: Model Yönetimi

### Tüm Modelleri Listele

```bash
# Kurulu modelleri görüntüle
ollama list
```

### Model Silme

```bash
# Yer açmak için model sil
ollama rm <model_adı>

# Örnek:
ollama rm qwen3-14b
```

### Model Güncelleme

```bash
# En son sürümü çek
ollama pull qwen3-14b
```

### Model Bilgilerini Kontrol Etme

```bash
# Model detaylarını görüntüle
ollama show qwen3-14b
```

---

## Bölüm 8: İleri Düzey Yapılandırma

### Bağlam Penceresini Artırma

Daha büyük bağlam için Modelfile oluşturun:

```
FROM qwen3-14b
PARAMETER num_ctx 8192
```

```bash
ollama create qwen3-large-context -f Modelfile
```

### Belirli Görevler İçin Özel Sistem Promptları

```bash
# Kod odaklı varyant
cat > ~/ollama-models/qwen3-coder.Modelfile << 'EOF'
FROM qwen3-14b
SYSTEM """Kod yazma, debugging ve kod optimizasyonu konularında uzman bir yazılım geliştirici asistansınız. Her zaman temiz, okunabilir ve iyi dokümante edilmiş kod yazın."""
EOF

ollama create qwen3-coder -f ~/ollama-models/qwen3-coder.Modelfile
```

### Performans Ayarlama

```bash
# Sınırlı çıktı ile daha hızlı yanıtlar
cat > ~/ollama-models/qwen3-fast.Modelfile << 'EOF'
FROM qwen3-14b
PARAMETER temperature 0.3
PARAMETER top_p 0.8
PARAMETER num_predict 256
EOF

ollama create qwen3-fast -f ~/ollama-models/qwen3-fast.Modelfile
```

---

## Bölüm 9: Sorun Giderme

### OpenWebUI Ollama'ya Bağlanamıyor

**Ollama'nın çalışıp çalışmadığını kontrol edin:**
```bash
curl http://localhost:11434/api/tags
```

Başarısız olursa, Ollama'yı başlatın:
```bash
ollama serve
```

**Docker ağını doğrulayın:**
- OpenWebUI başlatılırken `--add-host=host.docker.internal:host-gateway` kullanıldığından emin olun
- Gerekirse OpenWebUI container'ını yeniden başlatın:
  ```bash
  docker restart open-webui
  ```

### Model OpenWebUI'de Görünmüyor

1. OpenWebUI sayfasını yenileyin
2. Settings'te Ollama API bağlantısını kontrol edin
3. Modelin var olduğunu doğrulayın: `ollama list`

### Bellek Yetersizliği Hataları

- Qwen3 14B (Q4_K_M) ~8-10GB RAM kullanır
- Çalıştırmadan önce yeterli RAM'iniz olduğundan emin olun
- Bellek boşaltmak için diğer uygulamaları kapatın
- Gerekirse daha küçük quantization'lara sahip Ollama kütüphanesinden daha fazla model eklemeyi düşünün

### Docker Sorunları

```bash
# OpenWebUI loglarını görüntüle
docker logs open-webui

# Container'ı yeniden başlat
docker restart open-webui

# Gerekirse kaldır ve yeniden oluştur
docker rm -f open-webui
# Ardından docker run komutunu tekrar çalıştırın
```

---

## Bölüm 10: Faydalı Komut Referansları

### Ollama Komutları

```bash
ollama list                    # Tüm modelleri listele
ollama ps                      # Çalışan modelleri göster
ollama pull <model>            # Model indir
ollama run <model>             # CLI'de model çalıştır
ollama rm <model>              # Model sil
ollama create <ad> -f <dosya>  # Modelfile'dan oluştur
ollama show <model>            # Model bilgisini göster
ollama cp <kaynak> <hedef>     # Model kopyala
```

### Docker Komutları

```bash
docker ps                      # Çalışan container'ları listele
docker logs open-webui         # OpenWebUI loglarını görüntüle
docker restart open-webui      # OpenWebUI'yi yeniden başlat
docker stop open-webui         # OpenWebUI'yi durdur
docker start open-webui        # OpenWebUI'yi başlat
docker rm -f open-webui        # Container'ı kaldır
```

---

## Bölüm 11: Sonraki Adımlar

Artık her şey kurulu olduğuna göre, şunları yapabilirsiniz:

1. **Farklı kullanım durumlarını test edin** - Kod yazma, analiz, sohbet ve genel amaçlı görevleri deneyin
2. **Özelleştirilmiş varyantlar oluşturun** - Özel sistem promptları ile göreve özel versiyonlar yapın
3. **Tamamlayıcı modeller ekleyin** - Farklı amaçlar için https://ollama.com/library adresinden ek modeller indirin
4. **API kullanın** - OpenWebUI, OpenAI uyumlu bir API sağlar
5. **RAG yapılandırın** - Gelişmiş bağlam için belgelerinizi yükleyin

### Önerilen Tamamlayıcı Modeller

```bash
# Kod üretimi için
ollama pull deepseek-coder:33b

# Daha küçük genel amaçlı model (daha hızlı)
ollama pull qwen2.5:7b-instruct

# Doğal dil görevleri için
ollama pull mistral:7b-instruct
```

---

## Özet

Artık sahipsiniz:
- ✅ macOS'ta kurulu Ollama
- ✅ Docker'da çalışan OpenWebUI
- ✅ Yapılandırılmış ve hazır Qwen3 14B LLM (Q4_K_M)
- ✅ Özel Modelfile'lar ve varyantlar oluşturma bilgisi
- ✅ Gerektiğinde daha fazla model ekleme temeli

OpenWebUI'ye **http://localhost:3000** adresinden erişin ve yerel LLM'lerinizle sohbet etmeye başlayın!
