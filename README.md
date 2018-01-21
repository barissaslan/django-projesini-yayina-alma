# django-deployment-tutorial
   
## Yüklemeler

### Python3, PostgreSQL ve Nginx bileşenlerinin yüklenmesi

```
  sudo apt-get update
  sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx
```

### PostgreSQL Yapılandırması

1. İnteraktif PostgreSQL komut istemcisine bağlanma

```
  sudo -u postgres psql
```

2. Projenin kullanacağı veritabanını oluşturma

```sql
  CREATE DATABASE projeAdi;
```

3. Projenin kullanacağı veritabanı kullanıcısını oluşturma

```sql
  CREATE USER kullaniciAdi WITH PASSWORD 'parola';
```

4. Varsayılan karakter kodlaması (default character encoding), işlem (transaction) ve bölgesel zaman ile ilgili ayarların yapılması

```sql
  ALTER ROLE kullaniciAdi SET client_encoding TO 'utf8';
  ALTER ROLE kullaniciAdi SET default_transaction_isolation TO 'read committed';
  ALTER ROLE kullaniciAdi SET timezone TO 'Europe/Istanbul';
```

5. Oluşturulan kullanıcıyı veritabanına yönetici erişimi izni verme

```sql
  GRANT ALL PRIVILEGES ON DATABASE projeAdi TO kullaniciAdi;
```

6. PostgreSQL komut istemcisinden çıkmak

```sql
  \q
```

### Proje için Python Sanal Ortamın (Virtual Environment) Kurulması

1. Python3 için pip güncellemesi ve `virtualenv` paketinin kurulması

```
  sudo -H pip3 install --upgrade pip
  sudo -H pip3 install virtualenv
```

2. Örnek bir Django projesi GitHub'dan kopyalanıyor.

```
  git clone https://github.com/barissaslan/django-dersleri.git
```

3. Proje klasörünün adının değiştirilmesi ve proje dizinine geçme

```
  mv django-dersleri blog
  cd blog
```

4. Proje için `venv` adlı Sanal Python Ortamının oluşturulması

```
  virtualenv venv
```

5. Sanal Ortamın aktif edilmesi

```
  source venv/bin/activate
```

6. Proje bağımlılıklarının yüklenmesi

```
  pip install -r requirements.txt
```

7. Gunicorn ve PostgreSQL adaptörünün yüklenmesi

```
  pip install gunicorn psycopg2
```




