## 1. Adım: VPS Hazırlığı ve Docker Kurulumu
Sunucunuzu aldıktan sonra (Ubuntu 22.04 önerilir), her şeyi Docker ile yönetmek işleri
çok kolaylaştıracaktır.

### Paket listesini güncelle
sudo apt update && sudo apt upgrade -y
### Docker kurulumu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

## 2. Adım: Ollama Kurulumu ve Model Seçimi
Ollama, açık kaynaklı modelleri API üzerinden çalıştırmamızı sağlar. Sıfır maliyetli LLM
burada başlar.
