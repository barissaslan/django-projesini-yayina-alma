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







