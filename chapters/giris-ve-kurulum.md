
# SQL sunucu araç ve servislerine genel bir bakış

Veritabanı sistem yazılımları kabaca sunucu (server) ve istemci (client) parçalarından oluşur. Sunucu verinin yapılandırılmış bir biçimde tutulduğu kısım, istemci ise verilerin sorgulanması ve düznelenmesi için kullanılan araçtır. 

Belli başlı yazılım parçalarına kısaca bir bakalım, ayrıntılara daha sonra geleceğiz:

* `mysqld`: bu bir artalan süreci olarak çalışan, veritabanının sunucu kısmıdır (MySQL daemon). 
* `mysqld_safe`: sunucu artalan sürecini çalıştırmak için kullanılan yardımcı program.
* `mysql`: standard istemci program; metin tabanlı bir program olup sunucu ile iletişimi sağlayan arayüzdür.
* `mysqlaccess`: kullanıcı hesaplarının oluşturulması ve yönetilmesi için kullanılır.
* `mysqladmin`: sunucunun yönetiminde kullanılır.
* `mysqlshow`: sunucunun durumunu, kullanımını, veritabanlarını, tabloları görüntülemek için kullanılır.
* `mysqldump`: veritabanlarını ham metin dosyalarına yazdırmak için kullanılır.


# MySQL yazılımlarının yüklenmesi 


Önce sistemde sistemde `mysql` ile ilgili herhangi bir  artalan sürecinin çalışıp çalışmadığını kontrol edelim:

```bash
ps aux | grep mysql
```

Eğer çıktıda `grep` komutu dışında da satırlar görüyorsanız sisteminizde `mysql` yüklü ve aktif demektir. Eğer görmüyorsanız, `mysql` kurulu olmayabilir ya da kuruludur fakat çalışmıyordur. Kurulumu test edelim 

```bash
sudo mysqladmin -p version status
```

Bu komutun ardından önce çalıştığınız makinedeki admin şifresini girmeniz istenecektir. Bu şifreyi girdikten sonra bir şifre daha sorluyorsa, bunu boş geçin; aşağıdaki gibi bir çıktı alıyorsanız, sisteminizde `mysql` yüklü demektir. 

```
[~]$ sudo mysqladmin -p version status
Enter password: 
mysqladmin  Ver 9.1 Distrib 10.3.18-MariaDB, for debian-linux-gnu on i686
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version          10.3.18-MariaDB-0+deb10u1
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/run/mysqld/mysqld.sock
Uptime:                 3 min 43 sec

Threads: 7  Questions: 453  Slow queries: 0  Opens: 177  Flush tables: 1  Open tables: 31  Queries per second avg: 2.031
Uptime: 223  Threads: 7  Questions: 453  Slow queries: 0  Opens: 177  Flush tables: 1  Open tables: 31  Queries per second avg: 2.031
```

Sistemde yazılımın yüklenmemiş olduğunu varsayalım. Yüklemek için

```bash
sudo apt-get update
sudo apt-get install mariadb-server mariadb-client mariadb-test
```

Yüklemenin ardından bir takım basit ayarlar yapmamız gerekecek. Örneğin yönetici (root) şifresi belirlemek, anonim kullanıcı hesaplarını temizlemek, hata kayıtlarının tutulmasını sağlamak, varsayılan karakter kümesini belirlemek gibi. Sonuncudan başlayalım. Veritabanı sisteminin yapılandırma dosyasını açalım; bu dosya çoğu sistemde `/etc/mysql/my.conf`dosyasıdır. Bu tarz yapılandırma dosyalarının her zaman ham metin editörlerinde açılıp değiştirilmesi gerekir; hiçbir zaman sistem dosyalarını Word gibi kelime işemcilerde açmamalıyız. Kullanabileceğiniz en basit ham metin editörlerinden biri `nano` programıdır.

```bash
sudo nano /etc/mysql/my.cnf
```

ile kurulum dosyasını açalım. Bu dosyada bölümler köşeli parantez içindeki başlıklarla ayrılmıştır. Örneğin hem istemci hem sunucu için geçerli olmasını istediğimiz ayarları `[client-server]` başlığı altına girelim,

```
default-character-set=utf8
```

Hata kayıtlarının tutulması için bir dosya tanımlayalım şimdi. Eğer halihazırda `[mysqld_safe]` diye bir başlık yoksa aşağıdaki gibi, varsa o başlığın altına aşağıdakinin ikinci satırını girelim.

```
[mysqld_safe]
log-error=/var/log/mysqld.log
```

İlk kurulum sırasında yönetici şifresi boş olarak belirlenir. Şimdi onu değiştirelim, 

```bash
sudo mysqladmin -u root -p flush-privileges password
```

İlk çıkan `Enter password:` istemini boş geçeceğiz, bu eski şifre; ardından gelen `New password:` ve `Confirm new password:` istemlerine yeni şifremizi iki kere girelim. Normal kullanımda bu şifrenin son derece güçlü olması gerekir.

`MySQL` sisteminde kullanıcılar evsahibi sistemleri (host) ile birlikte tanımlanırlar. Yukarıdaki işlemle biz `localhost` isimli hosttaki `root` kullanıcısı için şifre tanımladık. Sistemde başka `root` kullanıcılar da olabilir. Güvenlik gereği bunların silinmesi iyi bir pratiktir. Önce sistemde hangi kullanıcı-host çiftleri var ona bakalım:

```bash
sudo mysql -u root -p -e "SELECT user,host FROM mysql.user;"
```

Eğer bu komut karşılığında `root`, `localhost` çifti dışında bir kullanıcı görüyorsanız, o kullanıcıyı aşağıdaki komutla silebilirsiniz: 

```bash
sudo mysql -u root -p -e "DROP USER '<kullanıcı adı>'@'<host adı>';"
```

## Kullanıcı tanımlama

Şimdi ders süresince bir takım işlemler yapabilmek için bir kullanıcı tanımlayalım; dilediğiniz kullanıcı ismini seçebilirsiniz:

```bash
sudo mysql -u root -p -e "GRANT USAGE ON *.* TO 'umut'@'localhost' IDENTIFIED BY 'sifre1234';"
```

böylece `umut` adlı bir kullanıcı tanımlamış olduk. Bu kullanıcıyı `*.*` ifadesi sayesinde bütün tablo ve veritabanları üzerinde tanımladık, bir de şifre tanımladık. Fakat, henüz bu kullanıcının hiçbir yetkisi yok. Bu kullanıcıya tablo ve veritabanlarını görüntüleme yetkisi verelim şimdi:


```bash
sudo mysql -u root -p -e "GRANT SELECT ON *.* TO 'umut'@'localhost';"
```

Burada sizden `umut` kullanıcısının değil, `root` kullanıcısının şifresi istenecektir.   

Şimdi `umut` kullanıcısının yetkilerini görüntüleyelim: 

```bash
sudo mysql -u root -p -e "SHOW GRANTS FOR 'umut'@'localhost';"
```

Bu komut karşılığında aşağıdakine benzer bir çıktı alacaksınız:


```bash
+--------------------------------------------------------------------------------------------------------------+
| Grants for umut@localhost                                                                                    |
+--------------------------------------------------------------------------------------------------------------+
| GRANT SELECT ON *.* TO 'umut'@'localhost' IDENTIFIED BY PASSWORD '*09A7A8FAF4227DEA0819D093C9B06F9DDA9BF35B' |
+--------------------------------------------------------------------------------------------------------------+
```

Derste karşılaşacağımız alıştırmaları yapabilmek için, yarattığımız kullanıcıya tüm temel yetkileri verelim:


```bash
sudo mysql -u root -p -e "GRANT ALL ON *.* TO 'umut'@'localhost';"
``` 
## MySQL istemci temelleri

Bu derste hem ileri seviye fonksiyonlar için en iyi temeli sağlayan, hem de hangi sistemde olursa olsun her `MySQL` kurulumuyla birlikte gelen metin tabanlı `mysql` aracını kullanacağız. Şimdi bu aracın temel işlevlerini görelim. 

Güvenlik nedeniyle mecbur olmadıkça sunucu ile normal bir kullanıcı üzerinden iletişim kurulmalıdır. Yukarıda yarattığımız kullanıcı ile istemciyi çalıştırmak için,


```bash
mysql -u umut -p
```

yapalım. Daha önceki komutlarda girdiğimiz ve çalıştığımız makinanın yöneticisine ait yetkileri kullanmamızı sağlayan `sudo` komutunu artık kullanmamız gerekmiyor. Şifre istemine kullanıcı şifremizi girelim; aşağıdaki gibi bir ekrana ulaşacağız. 

```sql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 64
Server version: 10.3.18-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

### Veritabanı ve tablo oluşturma

Önce sunucuda hangi veri tabanlarının bulunduğuna bir bakalım, `mysql` içindeyken, 


```sql
SHOW DATABASES;
```

komutu bize istediğimiz bilgiyi verecektir. Karşımıza gelen veritabanlarından `information_schema` sunucu ile bilgilerin, `performance_schema` sunucu performansı ile ilgili bilgilerin, `mysql` ise kullanıcı hesapları ile ilgili bilgilerin tutulduğu, sisteme ait veritabanlarıdır.

Üzerinde denemeler yapmak için `test` isimli bir veritabanı oluşturalım. Genel usül gereği veritabanı ve kullanıcı adlarını küçük harfle, komutları büyük harfle yazıyoruz; bu yalnızca okunabilirliği artırma amacıyla yaptığımız bir şey, dilerseniz komutları küçük ya da büyük-küçük karışık harfle de yazabilirsiniz. Veritabanımızı oluşturmak için, 


```bash
CREATE DATABASE test;
```
girelim. Bir kez daha `SHOW DATABASES;` girerek çıktıdaki değişimi görelim: 


```bash
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.001 sec)
```

Şimdi üzerinde çalışabileceğimiz `test` adlı bir veritabanımız var.

Veritabanlarının temel yapı taşları tablolardır. Şimdi veritabanımızda bir tablo oluşturalım:


```bash
MariaDB [(none)]> CREATE TABLE test.kitap (kimlik INT, başlık TEXT, durum INT);
Query OK, 0 rows affected (0.302 sec)
```

Tablomuzu oluşturuken ona bir ad verdik ('kitap') ve tablomuzun alan (sütun) adlarını, taşıyacakları verinin cinsini de belirterek tanımladık.

Veritabanımızda hangi tabloların bulunduğuna bakalım:


```bash
MariaDB [(none)]> SHOW TABLES FROM test;
+----------------+
| Tables_in_test |
+----------------+
| kitap          |
+----------------+
1 row in set (0.001 sec)
```

Bir tablo hakkında genel bilgi almak için:


```bash
MariaDB [(none)]> DESCRIBE test.kitap;
+--------+---------+------+-----+---------+-------+
| Field  | Type    | Null | Key | Default | Extra |
+--------+---------+------+-----+---------+-------+
| kimlik | int(11) | YES  |     | NULL    |       |
| başlık | text    | YES  |     | NULL    |       |
| durum  | int(11) | YES  |     | NULL    |       |
+--------+---------+------+-----+---------+-------+
```

yapıyoruz. Şimdilik tanımladığımız alan adlarını ve veri cinslerini görmemiz yeterli. 


Bir tablo hakkında komut yazarken, başına ait olduğu veritabanı adını ve araya nokta koyuyoruz. Eğer uzun süre aynı veritabanı üzerinde çalışacaksanız ve her seferinde veritabanının adını yazmak istemiyorsanız, aşağıdaki komutu kullanarak çalıştığınız veritabanının varsayılan olarak atayarak, kullandığınız tablo adlarının otomatik olarak bu veritabanına ait olduğunun anlaşılmasını sağlayabilirsiniz.


```bash
USE test;
```

Artık `kitap` tablosuna `test.kitap` yerine sadece `kitap` diyebiliriz.

### Veri girişi

Şimdi tablomuza bir takım verileri sırayla aşağıdaki komutları girerek ekleyelim:

```bash
INSERT INTO kitap VALUES(100, 'Kars Kitap', 0);
INSERT INTO kitap VALUES(101, 'Cemile', 1);
INSERT INTO kitap VALUES(102, 'Yaban', 0);
```

Tablomuzdaki veriyi görüntüleyelim:


```bash
MariaDB [test]> SELECT * FROM kitap;
+--------+------------+-------+
| kimlik | başlık     | durum |
+--------+------------+-------+
|    100 | Kars Kitap |     0 |
|    101 | Cemile     |     1 |
|    102 | Yaban      |     0 |
+--------+------------+-------+
```

Bu noktada terminolojimizi biraz daha netleştirelim. Satırlara denk gelen bilgilere **kayıt**, sütunlara denk gelen bilgilere ise **alan** diyeceğiz. 


Daha seçici görüntülemeler yapmak için `WHERE` anahtarını kullanacağız.

```bash
MariaDB [test]> SELECT * FROM kitap WHERE durum = 1;
+--------+--------+-------+
| kimlik | başlık | durum |
+--------+--------+-------+
|    101 | Cemile |     1 |
+--------+--------+-------+
```

Değişik bir görüntülemeyi bu sefer komut `\G` ile sonlandırarak yapalım, çıktının biçemi değişecektir:

```bash
MariaDB [test]> SELECT * FROM kitap WHERE durum = 0\G
*************************** 1. row ***************************
kimlik: 100
başlık: Kars Kitap
 durum: 0
*************************** 2. row ***************************
kimlik: 102
başlık: Yaban
 durum: 0
```

Şimdi veriler üzerinde güncelleme yapmayı göreceğiz. Örneğin `100` kimlik numarasını verdiğimiz kayıdın durumunu `1` olarak güncelleyelim:  

```bash
MariaDB [test]> UPDATE kitap SET durum = 1 WHERE kimlik = 100;
Query OK, 1 row affected (0.147 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

Durumu `1` olan kayıtları arattığımızda artık bir yerine iki kayıt göreceğiz.


Şimdi daha karmaşık bir güncelleme yapalım. `100` kimlikli kitabın adını yanlış yazmışız, `a` basacağımız yere `s` yazmışız. Aynı anda hem bu kitabın adını düzeltelim hem de durumunu yine `0`'a çevirelim. Bu tarz karmaşık komutları satırlara bölmek okunabilirlik açısından yararlıdır. İstemci içinde dilediğiniz kadar satırbaşı yapabilirsiniz, `mysql` bir `;` ya da `\G` ile karşılaşana kadar komutun henüz tamalanmadığını düşünecektir. Şimdi komutumuzu aşağıdaki şekilde gireceğiz:

```sql
UPDATE kitap
SET başlık = 'Kara Kitap', durum = 0
WHERE kimlik = 100;
```

Bunu `mysql` içinde yazarken, `ENTER` tuşuna her bastığınızda ok işareti belirecektir.  

Çok satırlı komutlar yazarken, bir noktada komutu tamamlamaktan vazgeçerseniz `\c` yazıp `ENTER`a basarak komutu iptal edebilirsiniz. 

Tabloyu tekrar görüntüleyerek her şeyin istediğiniz gibi olduğundan emin olun.


Şimdi durum için verdiğimiz rakamların (0 ve 1) adlarını içeren ikinci bir tablo oluşturalım: 


```sql
CREATE TABLE durum_adları (durum INT, durum_adı CHAR(8));
INSERT INTO durum_adları VALUES (0, 'tükendi'), (1, 'stokta');
```

Şimdi şu ana kadar karşılaştığımız en karmaşık komutu yazacağız; bu komutla amacımız iki tablodan birden alanları aynı anda görüntüleyebilmek. Eğer bu şu an için çok karmaşık geliyorsa endişelenmeyin, bu konuya geri döneceğiz.

```bash
MariaDB [test]> SELECT kimlik, başlık, durum_adı
    -> FROM kitap JOIN durum_adları
    -> WHERE durum = durum_no;
+--------+------------+------------+
| kimlik | başlık     | durum_adı  |
+--------+------------+------------+
|    100 | Kara Kitap | tükendi    |
|    101 | Cemile     | stokta     |
|    102 | Yaban      | tükendi    |
+--------+------------+------------+
```
