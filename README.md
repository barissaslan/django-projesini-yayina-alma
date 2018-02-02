# Django Projesini Yayına Alma

[YouTube kanalımdaki](https://www.youtube.com/channel/UC5bnqsX71d-eKCHhNvM2U6g) **[Django Uygulamasını Yayına Alma](https://www.youtube.com/watch?v=uwVmWS1yJ1k&list=PLPrHLaayVkhny4WRNp05C1qRl1Aq3Wswh)** adlı video ders serisi için dökümantasyon. 

Kullanılan teknolojiler : **Ubuntu 16.04, Nginx, Gunicorn, PostgreSQL**

Dersler YouTube'da ücretsiz olarak yayımlanmaktadır.


**İçindekiler**

- [Ubuntu Başlangıç Yapılandırması](#ubuntu-başlangıç-yapılandırması)
	- [Linux Kullanıcı Oluşturma](#linux-kullanıcı-oluşturma)
	- [SSH Yapılandırması](#ssh-yapılandırması)
	    - [Anahtar Uretimi ve Kullanımı](#anahtar-uretimi-ve-kullanımı)
	    - [Parolaya Dayalı Kimlik Doğrulamayı Kapatmak](#parolaya-dayalı-kimlik-doğrulamayı-kapatmak)
	- [Temel Güvenlik Duvarı Ayarları](#temel-güvenlik-duvarı-ayarları)
	
- [Django Yayın Ortamı Yapılandırması](#django-yayın-ortamı-yapılandırması)
	- [Python3, PostgreSQL ve Nginx bileşenlerinin yüklenmesi](python3-postgresql-ve-nginx-bileşenlerinin-yüklenmesi)
	- [PostgreSQL Yapılandırması](#postgresql-yapılandırması)
	- [Proje için Python Sanal Ortamın (Virtual Environment) ve Bağımlılıkların Yüklenmesi](#proje-için-python-sanal-ortamın-virtual-environment-ve-bağımlılıkların-yüklenmesi)
	- [Django Uygulamasıyla ilgili Yayın Ortamı (Production Environment) Ayarları](#django-uygulamasıyla-ilgili-yayın-ortamı-production-environment-ayarları)
	- [Projeyi Django'nun Geliştirme Sunucusuyla Test Etme](#projeyi-djangonun-geliştirme-sunucusuyla-test-etme)
	- [Gunicorn](#gunicorn)
	- [Nginx Yapılandırması](#nginx-yapılandırması)
	- [Yayın ve Geliştirme Ortamları için Ayrı Ayar Dosyaları](#yayın-ve-geliştirme-ortamları-için-ayrı-ayar-dosyaları)
	
- [SSL Sertifakası Temin Etme](#ssl-sertifikası-temin-etme)


## [Ubuntu Başlangıç Yapılandırması](#ubuntu)

### [Linux Kullanıcı Oluşturma](#linux)

1. Kullanıcı Oluşturma

```
   adduser kullanici_adi  # adduser baris
```
   
2. Oluşturulan yeni kullanıcıya sudo (yönetici) yetkisi verme

```
   usermod -aG sudo kullanici_adi  # usermod -aG sudo baris
```

### [SSH Yapılandırması](#ssh)

#### [Anahtar Uretimi ve Kullanımı](#anahtar)

1. (Yerel bilgisayarda yapılacak) SSH Anahtarı üretme:

```
   ssh-keygen
```

2. (Yerel bilgisayarda yapılacak) Public Anahtarı görüntüleme:

```
   cat ~/.ssh/id_rsa.pub
```

3. (Sunucuda yapılacak) Root kullanıcısından normal kullanıcıya geçiş yapma:

```
   su - kullanici_adi  # su - baris
```

4. (Sunucuda yapılacak) Yeni kullanıcı için `.ssh` klasörünün oluşturulması ve klasöre kısıtlı izin verilmesi:

```
   mkdir ~/.ssh
   chmod 700 ~/.ssh 
```

5. (Sunucuda yapılacak) `.ssh` klasöründe `authorized_keys` dosyasını oluşturma ve açma:

```
   nano ~/.ssh/authorized_keys
   
   # Yerel bilgisayardan kopyalanan public key yapıştırılarak CTRL + X kombinasyonunun ardından 'y' tuşuna basılıp, 'Enter' tuşlanarak dosya kaydedilir.
```

6. (Sunucuda yapılacak) `.ssh` klasörünün yazma iznini kısıtlamak:

```
   chmod 600 ~/.ssh/authorized_keys
```

#### [Parolaya Dayalı Kimlik Doğrulamayı Kapatmak](#password-auth)

1. (Sunucuda yapılacak) SSH konfigürasyon dosyasını açma:

```
   sudo nano /etc/ssh/sshd_config
```

2. (Sunucuda yapılacak) Dosya içerisinde ki bazı ifadelerin alması gereken değerler:

```
   PasswordAuthentication no 
   PubkeyAuthentication yes
   ChallengeResponseAuthentication no
   
   # CTRL + X kombinasyonunun ardından 'y' tuşuna basılıp, 'Enter' tuşlanarak dosya kaydedilir.
```

3. (Sunucuda yapılacak) SSH Servisini yeniden başlatma:

```
   sudo systemctl reload sshd
```


### [Temel Güvenlik Duvarı Ayarları](#temel-guvenlik)

1. Güvenlik duvarında SSH'a izin verme

```
   sudo ufw allow OpenSSH
```

2. Güvenlik duvarı kurallarını aktif etme.

```
   sudo ufw enable
```

3. Güvenlik duvarı kurallarını listeleme

```
   sudo ufw status
```


## [Django Yayın Ortamı Yapılandırması](#django-yayın)

### [Python3, PostgreSQL ve Nginx bileşenlerinin yüklenmesi](#kurulum)

```
  sudo apt-get update
  sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx
```

### [PostgreSQL Yapılandırması](#postgresql)

1. İnteraktif PostgreSQL komut istemcisine bağlanma

```
  sudo -u postgres psql
```

2. Projenin kullanacağı veritabanını oluşturma

```sql
  CREATE DATABASE proje_adi;  
```

3. Projenin kullanacağı veritabanı kullanıcısını oluşturma

```sql
  CREATE USER kullanici_adi WITH PASSWORD 'parola123';
```

4. Varsayılan karakter kodlaması (default character encoding), işlem (transaction) ve bölgesel zaman ile ilgili ayarların yapılması

```sql
  ALTER ROLE kullanici_adi SET client_encoding TO 'utf8';
  ALTER ROLE kullanici_adi SET default_transaction_isolation TO 'read committed';
  ALTER ROLE kullanici_adi SET timezone TO 'Europe/Istanbul';
```

5. Oluşturulan kullanıcıyı veritabanına yönetici erişimi izni verme

```sql
  GRANT ALL PRIVILEGES ON DATABASE proje_adi TO kullanici_adi;
```

6. PostgreSQL komut istemcisinden çıkmak

```sql
  \q
```

### [Proje için Python Sanal Ortamın (Virtual Environment) ve Bağımlılıkların Yüklenmesi](#virtualenv)

1. Python3 için pip güncellemesi ve `virtualenv` paketinin kurulması

```
  sudo -H pip3 install --upgrade pip
  sudo -H pip3 install virtualenv
```

**Not:** *"locale.Error: unsupported locale setting"* hatası için çözüm:

```
  export LC_ALL="en_US.UTF-8"
```

2. Örnek bir Django projesi GitHub'dan kopyalanıyor.

```
  git clone https://github.com/barissaslan/eventhub.git
```

3. Proje dizinine geçme ve `venv` adlı Sanal Python Ortamının oluşturulması

```
  cd eventhub
  virtualenv venv
```

4. Sanal Ortamın aktif edilmesi

```
  source venv/bin/activate
```

5. Proje bağımlılıklarının yüklenmesi

```
  pip install -r requirements.txt
```

6. Gunicorn ve PostgreSQL adaptörünün yüklenmesi

```
  pip install gunicorn psycopg2
```

**Not:** Projenin settings.py dosyasının dizin hiyerarşisi aşağıdaki gibidir:

```
/home/baris/eventhub/
		    /accounts/		
		    /event/
		    /eventhub/
		   	     /__init__.py	
			     /settings.py
			     /urls.py
			     /wsgi.py
		    /notification/
		    /static/
		    /templates/
		    . . .
```

### [Django Uygulamasıyla ilgili Yayın Ortamı (Production Environment) Ayarları](#production)

#### `setting.py` Dosyasındaki Ayarlar

1. Django projesinin `setting.py` dosyasındaki DEBUG ifadesinin `False` olarak ayarlanması.

```python
  DEBUG = False # Production ortamında DEBUG kesinlikle kapatılmalıdır ki saldırganlar site hakkında detaylı hata raporlarına ulaşamasınlar. 
```

2. Django projesinin `setting.py` dosyasındaki ALLOWED_HOSTS (İzin verilmiş sunucular) ataması

```python
  ALLOWED_HOSTS = ['alan_adiniz_veya_IP_adresiniz', 'ikinci_alan_adiniz_veya_IP_adresiniz', . . .]
  # Örnek:
  ALLOWED_HOSTS = ['alanadi.com', '192.168.1.10'] # IP adresi örnek amaçlı yazılmış olup private adrestir.
```

3. Django projesinin veritabanı ayarlarını SQLite'dan PostgreSQL'e çevirme.

    **Not:** Aşağıdaki `'NAME'`, `'USER'`, `'PASSWORD'` alanları değişken olup, atanan değerler yukarıda ki PostgreSQL Yapılandırması'nda verilen takma adlardan alınmıştır.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'proje_adi',
        'USER': 'kullanici_adi',
        'PASSWORD': 'parola123',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

4. STATICFILES_DIR Değişkeninin Kaldırılması ve STATIS_ROOT değişkeninin düzenlenmesi

    **Not:** Yayın Ortamında static dosyalar STATIC_ROOT değişkeninin gösterdiği klasör referans alınarak servis edileceği için STATICFILES_DIR'ın bir anlamı olmadığından yorum satırı haline getiriyoruz veya siliyoruz. Projemizde STATIC_ROOT değişkeni "static" klasörünü göstermelidir.
    
```python
    # STATICFILES_DIRS = [
    #    os.path.join(BASE_DIR, "static"),
    # ]
    
    STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

5. Veritabanı şemasını oluşturma ve statik dosyaları toplama

```
   python manage.py makemigrations
   python manage.py migrate
   python manage.py collectstatic
```

### [Projeyi Django'nun Geliştirme Sunucusuyla Test Etme](#dev-server)

1. Test edilecek 8000 portunu aktif etme

```
   sudo ufw allow 8000
```

2. Geliştirme sunucusunu çalıştırma

```
   python manage.py runserver 0.0.0.0:8000
   # 0.0.0.0 -> sunucunun IP adresi. (sunucu kendi IP adresini bildiğinden 0.0.0.0 yazılabilir)
```

3. Tarayıcıdan test etme

```
   http://alan_adi_veya_IP:8000
```


### [Gunicorn](#gunicorn)

1. Projeyi Gunicorn ile Çalıştırarak Gunicorn'u Test Etmek 

```
   # Sanal ortam aktif olmalı ve manage.py dizininde çalıştırılmalı. (eventhub.wsgi -> proje_adı.wsgi)
   gunicorn --bind 0.0.0.0:8000 eventhub.wsgi
```

Tarayıcıdan test etme: http://alan_adi_veya_IP:8000

Gunicorn sunucusunu durduma: `CTRL + C`

2. Gunicorn `systemd` Servis Dosyası Oluşturma

```
   # Sanal ortamdan çıkılmalı
   deactivate
   
   # Servis dosyası oluşturma
   sudo nano /etc/systemd/system/gunicorn.service
```

gunicorn.service dosya içeriği şu şekilde olmalıdır:

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=baris
Group=www-data
WorkingDirectory=/home/baris/eventhub
ExecStart=/home/baris/eventhub/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/baris/eventhub/eventhub.sock eventhub.wsgi:application

[Install]
WantedBy=multi-user.target
```

**Not:** Sanal ortamın adı `venv`, projenin adı `eventhub`, kullanıcı adı `baris` olarak varsayılmıştır.

**Not 2:** Dosyayı kaydetmek için `CTRL + X` yaptıktan sonra `y` harfine basıp, `enter` tuşuna basılmalı.

3. Gunicorn servisini aktif etme:

```
   sudo systemctl start gunicorn
   sudo systemctl enable gunicorn
```

4. Gunicorn servisinin doğru bir şekilde çalışıp çalışmadığını test etme:

```
   sudo systemctl status gunicorn
   # Yeşil renkte "active (running)" yazıyorsa başarılı bir şekilde çalışıyor demektir.
```

```
   ls /home/baris/eventhub
   # ls çıktısında "proje_adı.sock" adlı (.sock) uzantılı bir soket dosyası görülüyorsa gunicorn başarılı bir şekilde yapılandırılmıştır.
```

### [Nginx Yapılandırması](#nginx)

1. Nginx sunucu bloğu açma:

```
   sudo nano /etc/nginx/sites-available/eventhub
```

   **Dosyanın içinde bulunması gerekenler:**

```
server {
    listen 80;
    server_name alan_adi_veya_IP;
    root /home/baris/eventhub; # Projenin kök dizini

    location /static/ {
    }

    location /media/ {
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/baris/eventhub/eventhub.sock;  # Projenin kök dizinindeki 'proje_adı.sock' dosyası
    }
}
```

**Not:** Dosyayı kaydetmek için `CTRL + X` yaptıktan sonra `y` harfine basıp, `enter` tuşuna basılmalı.

2. Nginx dosyasını aktif etmek için dosyayı **`sites-enabled`** dizinine link olarak verme

```
   sudo ln -s /etc/nginx/sites-available/eventhub /etc/nginx/sites-enabled
```

3. Nginx yapılandırma dosyalarında *syntax* hatası olup olmadığını kontrol etme

```
   sudo nginx -t
```

4. Hata yoksa Nginx sunucusunu yeniden başlatma

```
   sudo systemctl restart nginx
```

5. Kullanılmayacak olan 8000 portunu kapatma ve Nginx kurallarını aktif etme

```
   sudo ufw delete allow 8000
   sudo ufw allow 'Nginx Full'
```

### [Yayın ve Geliştirme Ortamları için Ayrı Ayar Dosyaları](#seperate-settings)

1. `settings.py` dosyasının bulunduğu dizinde (`/home/baris/eventhub/eventhub` klasörü içinde ) *settings* adında bir Python Paketi (içinde \_\_init\_\_.py dosyası bulunan bir klasör) oluşturulur.

2. settings.py dosyası yeni oluşturulan pakete taşınır ve adı `base.py` olarak değiştirilir.

3. *settings* paketinin içinde `development.py` ve `production.py` dosyaları oluşturulur. Her iki dosyanın en başında base.py dosyası import edilir (`from eventhub.settings.base import *`).

4. `base.py` dosyasının proje dizinini yanlış hesaplamaması için `BASE_DIR` ifadesi şu şekilde değiştirilir:

```
   # Önce
   BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
   # Sonra
   BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
```

5. `base.py`'de ki ortama bağlı ifadeler çıkarılarak ilgili ortamların settings dosyalarında ayrı ayrı değerler olarak yazılır.

6. İlgili settings dosyalarının sanal ortam tarafından bulunması için her iki ortamın `activate` dosyasının (`venv/bin/activate`) sonuna şu ifadeler eklenir.

```
   # Geliştirme ortamındaki venv/bin/activate dosyası açılarak sonuna şu ifadeler eklenir:
   DJANGO_SETTINGS_MODULE="eventhub.settings.development"
   export DJANGO_SETTINGS_MODULE
   
   # Yayın ortamındaki venv/bin/activate dosyası açılarak sonuna şu ifadeler eklenir:
   DJANGO_SETTINGS_MODULE="eventhub.settings.production"
   export DJANGO_SETTINGS_MODULE
```

7. `wsgi.py` dosyasında settings konumu olarak production dosyası verilir (`eventhub.settings.production`).

8. Projenin dizin hiyerarşisi:

```
/home/baris/eventhub/
		    /accounts/		
		    /event/
		    /eventhub/
		   	     /__init__.py	
			     /settings/
			     	      /__init__.py	
				      /base.py
				      /development.py
				      /production.py
			     /urls.py
			     /wsgi.py
		    /notification/
		    /static/
		    /templates/
		    . . .
```

### [SSL Sertifikası Temin Etme](#ssl)

1. Gerekli işlemleri otomatize edecek Certbot yazılımının kurulması

```
   sudo add-apt-repository ppa:certbot/certbot  # Onaylamak için ENTER'a basılmalı.
   sudo apt-get update                          # Ubuntu paket listesini güncelleme.
   sudo apt-get install python-certbot-nginx    # Certbox'un Nginx paketinin yüklenmesi.
```

2. SSL Sertifikası Temini

```
   sudo certbot --nginx -d alan_adı -d www.alan_adı
```

**Not:** 2. Adımdaki komut hata verirse 3. adımdaki komut denenmelidir. Hatanın sebebi bir güvenlik açığının tespiti sebebiyle Let's Encrypt, Certbot servisini geçici süreyle kapalı tutmaktadır. Aşağıdaki alternatif denenmelidir.

3. Alternatif komut

```
   sudo certbot --authenticator standalone --installer nginx --pre-hook "service nginx stop" --post-hook "service nginx start"
```

4. Nginx yapılandırma dosyasının genel yapısı aşağıdaki gibi olmalıdır.

```
server {
    listen 80;
    server_name alan_adınız www.alan_adınız; # server_name aslanbaris.com www.aslanbaris.com

    location / {
	return 302 https://$host$request_uri;
    }
}

server {

    listen 443 ssl;
    server_name www.alan_adınız;  # www tercihe bağlıdır.
    
    ssl_certificate /etc/letsencrypt/live/www.alan_adınız/fullchain.pem;    # Certbot yazılımından seçilen domain adı yazılmalıdır.
    ssl_certificate_key /etc/letsencrypt/live/www.alan_adınız/privkey.pem;  # Certbot yazılımından seçilen domain adı yazılmalıdır.
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    client_max_body_size 5M;
    
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location /.well-known {
        alias /var/www/cert/.well-known;
    }

    location = /favicon.ico { access_log off; log_not_found off; }  # Sitenin tarayıcıda gözüken iconu

    root /home/kullanıcı_adı/proje_adı;  # root /home/baris/eventhub
    
    location /static/ {
    }

    location /media/ {
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/kullanıcı_adı/proje_adı/proje_adı.sock; # proxy_pass http://unix:/home/baris/eventhub/eventhub.sock;
	
    }
}
```

5. Django Production Settings dosyasına eklenmesi gerekenler:

```
   SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
   SECURE_SSL_REDIRECT = True
   SESSION_COOKIE_SECURE = True
   CSRF_COOKIE_SECURE = True
```
6. Nginx sunucusunu yeniden başlatma:

```
   sudo systemctl restart nginx
```

**Kaynaklar:** 

[1] https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04

[2] https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-16-04

