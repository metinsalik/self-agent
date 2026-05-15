## Hazırlık ve Güvenlik Duvarı
İlk olarak sunucunun kapılarını sıkılaştıralım. Sadece ihtiyacımız olan portları açıyoruz.

```
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```
## Docker ve Docker Compose Kurulumu
Tüm sistemi konteyner yapısında izole etmek için Docker kuruyoruz.
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
## Merkezi Yapılandırma (Docker Compose)
Sunucuda bir çalışma dizini oluşturun:
```
mkdir ~/ai-bot && cd ~/ai-bot
```
```
nano docker-compose.yml
```
```
version: '3.8'

networks:
  secure-network:
    driver: bridge

services:
  nginx-proxy:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./npm-data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - secure-network

  n8n:
    image: n8nio/n8n:2.19.5
    container_name: n8n
    restart: unless-stopped
    environment:
      - N8N_HOST=abcdomain.com
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://abcdomain.com/ ## kendi domain adresini yazıyorsun
      - N8N_SECURE_COOKIE=false
      - N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=Guclu_Sifre_123
      - N8N_TRUST_PROXY=true
      - NODE_FUNCTION_ALLOW_BUILTIN=fs,path
    volumes:
      - ./n8n-data:/home/node/.n8n
      - ./n8n-files:/home/node/.n8n-files
      - ./pdfs:/data/pdfs
    networks:
      - secure-network
    depends_on:
      - postgres

  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    volumes:
      - ./ollama-data:/root/.ollama
    networks:
      - secure-network

  qdrant:
    image: qdrant/qdrant:latest
    container_name: qdrant
    restart: unless-stopped
    volumes:
      - ./qdrant-data:/qdrant/storage
    networks:
      - secure-network

  postgres:
    image: postgres:16
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=n8n
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=Guclu_Sifre_123
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
    networks:
      - secure-network
```

Sistemi başlatıyoruz.
```
docker compose up -d
```
Sistemi tekrar durdurup başlatmak için:

```
docker compos down && docker compose up -d
```
## Domain ve SSL Ayarları

Nginx Proxy Manager: Tarayıcıdan http://SUNUCU_IP:81 adresine girin.
2. Varsayılan Giriş: admin@example.com / changeme (Hemen değiştirin!)
3. Proxy Host Ekle:
Domain: abcdomain.com
Forward Host: n8n | Port: 5678
SSL sekmesinden "Request a new SSL Certificate" seçin.

## Yapay Zekayı İçeriden Başlatma

```
docker exec -it ollama ollama run llama3
```


