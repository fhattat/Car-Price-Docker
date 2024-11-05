# Streamlit Docker Uygulama Kurulumu

Bu doküman, AWS üzerinde bir EC2 instance oluşturup Docker kullanarak bir Streamlit uygulamasını deploy etmek için gerekli adımları içermektedir.

## Gereksinimler
- AWS hesabı
- SSH erişimi için bir terminal veya SSH istemcisi

## Adımlar

### 1. AWS EC2 Instance Oluşturma

AWS yönetim konsoluna giriş yaparak aşağıdaki özelliklerde bir EC2 instance oluşturun:

- **AMI**: Ubuntu 24.04
- **Instance Type**: t2.micro
- **Depolama (Volume)**: 20 GB
- **Security Group**: 
  - TCP 8501 (Streamlit uygulaması için)
  - TCP 22 (SSH erişimi için)

### 2. EC2 Instance’a SSH ile Bağlanma

EC2 instance oluşturulduktan sonra, terminal veya SSH istemcinizden aşağıdaki komut ile instance’a bağlanın:

```bash
ssh -i "your-key.pem" ubuntu@<EC2_PUBLIC_IP>
```

`your-key.pem` dosyasının doğru izinlere sahip olduğundan emin olun (`chmod 400 your-key.pem` komutunu kullanarak izinleri ayarlayın).

### 3. Docker Kurulumu

Aşağıdaki komutları kullanarak Docker'ı kurun:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
docker version
```

Bu adımlar Docker'ın en güncel sürümünü kurar ve Docker komutlarını root yetkisi olmadan çalıştırabilmek için mevcut kullanıcıyı `docker` grubuna ekler.

### 4. Docker'ı Kaldırma (İsteğe Bağlı)

Eğer Docker'ı kaldırmak isterseniz, aşağıdaki komutları kullanabilirsiniz:

```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

### 5. Dockerfile Oluşturma

Oluşturacağımız image için Dockerfile oluştururuz.

```bash
touch Dockerfile
vim Dockerfile
```
Aşağıdaki içeriği dosyaya yazdırıyoruz.

```text
# Base image olarak Python 3.10'u kullanıyoruz
FROM python:3.10-slim

# Çalışma dizinini belirleyin
WORKDIR /app

# Gereken paketlerin kurulması için requirements.txt dosyasını kopyalayın
COPY requirements.txt .

# Paketleri yükleyin
RUN pip install --no-cache-dir -r requirements.txt

# Uygulama dosyalarını çalışma dizinine kopyalayın
COPY . .

# Streamlit'in dışarıya erişebilmesi için portu açıyoruz
EXPOSE 8501

# Streamlit uygulamasını başlatma komutu
CMD ["streamlit", "run", "Car_Price_Prediction_App.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

### 6. Docker Image Oluşturma

Uygulamanızın Docker image'ini oluşturmak için uygulama dosyalarının bulunduğu dizinde aşağıdaki komutu çalıştırın:

```bash
docker build -t streamlit-app .
```

Bu komut, `Dockerfile`'ınızı okuyarak `streamlit-app` adıyla bir Docker image oluşturur.

### 7. Docker Image'leri Listeleme

Oluşturulan Docker image'leri listelemek için:

```bash
docker images
```

Bu komut, sistemdeki tüm mevcut Docker image'leri gösterir.

### 8. Docker Container'da Uygulamayı Deploy Etme

Oluşturduğunuz image'i kullanarak uygulamanızı başlatın:

```bash
docker run -p 8501:8501 streamlit-app
```

Bu komut, Streamlit uygulamasını çalıştırır ve 8501 portunda erişilebilir hale getirir.

### 9. Arka Planda (Detach Mode) Çalıştırma

Uygulamayı arka planda çalıştırmak için `-d` bayrağını ekleyin:

```bash
docker run -d -p 8501:8501 streamlit-app
```

Bu komut, uygulamanızı arka planda çalıştırarak terminali serbest bırakır.

### 10. Çalışan Container'ları Listeleme

Mevcut durumda çalışan tüm Docker container'ları listelemek için:

```bash
docker ps
```

Bu komut, yalnızca çalışan container'ları gösterir.

### 11. Çalışan Container'ı Durdurma

Belirli bir container'ı durdurmak için:

```bash
docker stop <container_id>
```

`<container_id>`, `docker ps` komutuyla elde edilen container ID’sidir.

### 12. Tüm Container'ları Listeleme

Çalışmakta olan veya durdurulmuş tüm container'ları görmek için:

```bash
docker ps -a
```

Bu komut, tüm Docker container'larını (duranlar dahil) listeler.

### 13. Belirli Bir Container'ı Silme

Bir container'ı silmeden önce durdurmanız gerekir. Durdurduktan sonra silmek için:

```bash
docker rm <container_id>
docker rm $(docker ps -aq) # Durdurulmuş tüm containerları siler.
```

`<container_id>` değeri, silmek istediğiniz container'ın ID’sidir.

## Notlar

Kurulum adımları tamamlandıktan sonra Streamlit uygulamanıza `http://<EC2_PUBLIC_IP>:8501` adresinden erişebilirsiniz.
