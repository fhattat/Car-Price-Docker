### İlk olarak AWS EC2 ayağa kaldırırız.
ubuntu 24.04 ami, t2.micro, 20GB volume, 8501 ve 22 security group.
### Daha sonra ssh ile bağlanıyoruz.
### Bundan sonra aşağıdaki işlemleri sırası ile yapıyoruz.
### script ile docker kurulumu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
docker version
### kurulu dockerı kaldırma
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
### docker image oluşturma
docker build -t streamlit-app .
### Oluşan docker image listeleme
docker images
### Docker container'da uygulamayı deploy etme
docker run -p 8501:8501 streamlit-app
### Docker container'da uygulamayı deploy etme(Arka plan da çalışır-detach mode)
docker run -d -p 8501:8501 streamlit-app
###  çalışan containerları listeleme
docker ps
### çalışan containerı durdurma
docker stop container_id
###  Tüm containerları listeleme
docker ps -a
###  Belirli containerı silme(önce stop etmek lazım)
docker rm container_id





