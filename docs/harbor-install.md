<img src="../img/harbor-horizontal-color.png" />

#### Установка Harbor	 


Для начала нам необходимо скачать harbor
На [странице загрузки github](https://github.com/goharbor/harbor/releases)  выбираем нужную нам версию.

**Рабочая директория  для harbor у нас будет /srv/harbor** 

Переходим в рабочую директорию
```Bash
cd /srv/
```

Скачиваем файл:
```Bash
sudo wget https://github.com/goharbor/harbor/releases/download/v2.14.2/harbor-online-installer-v2.14.2.tgz
```

Распаковываем:
```Bash
sudo tar xzfv harbor-online-installer-v2.14.2.tgz
```

В конфигурационном файле  **/srv/harbor/harbor.yml** необходимо добавить имя сервера, пути к сертификатам и сменить пароли

```Bash
sudo nano /srv/harbor/harbor.yml

```

```yaml
hostname:  git-server.localdomain

	certificate: /srv/harbor/data/cert/git-server.localdomain.crt
	private_key: /srv/harbor/data/cert/git-server.localdomain.key


	harbor_admin_password: сменитьпароль
	
	database:
	password: сменитьпароль
	
	data_volume: /srv/harbor/data
	
```

Есть у Harbor один баг  - при перезагрузке   не все контейнеры стартуют :

Для исправления мы создадим файл  systemd сервиса

```Bash

sudo /etc/systemd/system/harbor.service
```
Заполним содержимым:


```Bash

[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=https://goharbor.io/docs/

[Service]
Type=simple
Restart=on-failure
RestartSec=5
ExecStart=/usr/bin/docker-compose -f /srv/harbor/docker-compose.yml up
ExecStop=/usr/bin/docker-compose -f /srv/harbor/docker-compose.yml down

[Install]
WantedBy=multi-user.target
```



Перезапускаем сервис daemon-reload, включаем harbor в автозагрузку и запустим его
```Bash
sudo systemctl daemon-reload && systemctl enable harbor.service && systemctl start  harbor.service
```


#### Создаём самозаверенный сертификат, можно и без него но будут трудности

В моём случае **git-server.localdomain** название локального домена, необходимо заменить на свой. 

```Bash
sudo openssl genrsa -out ca.key 4096
```


```Bash
sudo openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=git-server.localdomain" \
 -key ca.key \
 -out ca.crt
```

```Bash
sudo openssl genrsa -out git-server.localdomain.key 4096
```

```Bash
sudo openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=git-server.localdomain" \
    -key git-server.localdomain.key \
    -out git-server.localdomain.csr
```
Создадим файл v3.ext

```Bash
sudo cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=git-server.localdomain
DNS.2=git-server
EOF
```

```Bash
sudo openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in git-server.localdomain.csr \
    -out git-server.localdomain.crt
```

```Bash
openssl x509 -inform PEM -in git-server.localdomain.crt -out git-server.localdomain.cert
```


#### создаём файл daemon.json

sudo nano /etc/docker/daemon.json

Добавляем в него:

```yaml
{
  "insecure-registries": ["git-server.localdomain:443"]
}
```


Создаём каталог certs.d/git-server.localdomain

```Bash
sudo mkdir /etc/docker/certs.d/git-server.localdomain
```

Копируем файлы сертификатов в /etc/docker/certs.d/git-server.localdomain
```Bash
sudo cp git-server.localdomain.cert  /etc/docker/certs.d/git-server.localdomain
sudo cp git-server.localdomain.crt  /etc/docker/certs.d/git-server.localdomain
sudo cp git-server.localdomain.key  /etc/docker/certs.d/git-server.localdomain
sudo cp ca.crt  /etc/docker/certs.d/git-server.localdomain									!!!!!!перепроверить ca.crt 
```


Копируем файлы сертификатов в /srv/harbor/data/cert

```Bash
sudo cp git-server.localdomain.cert  /srv/harbor/data/cert
sudo cp git-server.localdomain.crt  /srv/harbor/data/cert
sudo cp git-server.localdomain.key  /srv/harbor/data/cert
sudo cp ca.crt  /srv/harbor/data/cert 									!!!!!!перепроверить ca.crt 
```

!!!!!!!!!!

```Bash
openssl s_client -showcerts -connect git-server.localdomain:443 < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ca.crt
```
Скопируем сертификат 
```Bash
sudo cp ca.crt /usr/local/share/ca-certificates/
```

Перезапустим сервис
```Bash
sudo update-ca-certificates
```

Перезагружаем сервисы
```Bash
sudo systemctl restart docker && systemctl restart harbor
```

#### Проверяем 

```Bash
sudo docker login -u admin git-server.localdomain:443
```


	Build the image:
```Bash	
sudo docker build -t myjspapp:v.1 .
```
	Login to Harbor registry:
```Bash
sudo docker login git-server.localdomain:443
```
	Tag the image:
```Bash
sudo docker tag myjspapp:v.1  git-server.localdomain:443/registry/myjspapp:v.1
```
	Push the image:
```Bash
sudo docker push git-server.localdomain:443/registry/myjspapp:v.1
```
