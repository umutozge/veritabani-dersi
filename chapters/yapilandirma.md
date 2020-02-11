# Veritabanı yapılandırma



Bir veritabanını yapılandırırken temel noktalar şunlardır:

* kaç adet tablo olacak; adları ne olacak.
* kaç adet alan olacak; adları ne olacak.
* kayıtlarda hangi alanda hangi cins veri tutulacak (Ör. metin, sayı, dosya, dijital veri, tarih, saat).

Veritabanının yapısına **şema** denmektedir.

Şimdi kuşlarla ilgili bir veritabanı yaratıp bunu yapılandıracağız. Önce veritabanını oluşturalım:

```sql
CREATE DATABASE moloni
CHARACTER SET latin5 
COLLATE latin5_turkish_ci;
```

Bu komutla veritabanına bir ad verdik, kullanılacak karakter kümesini tanımladık.

Aşağıdaki komutları girerek, ilkiyle veritabanımızın oluşup oluşmadığını kontrol edip, ikincisiyle de varsayılan veritabanı olarak atayarak, her komutta adını yazmaktan kurtulalım.


```sql
SHOW DATABASES;
USE koloni
```

## Tabloların  oluşturulması

Bu veritabanında bir adet ana tablomuz ve birkaç tane referans tablomuz olacak. Normalde tablolar 255 sütuna kadar büyütülebilirler; fakat büyük tablolar performans ve yapı açısından tercih edilmeMElidir: veritabanını yapılandırırken mümkün olduğu kadar küçük tablolar halinde yapılandırmaya dikkat etmeliyiz.

Ana tablomuzu oluşturalım:

```sql
CREATE TABLE kuşlar
(kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
bilimsel_ad VARCHAR(255) UNIQUE,
genel_ad VARCHAR(50),
aile_kimlik_no INT,
açıklama TEXT);
```

Kimlik numarası satırında kullandığımız `PRIMARY KEY` tablomuzun bu alana göre indekslenmesini istediğimizi belirtiyor; `AUTO INCREMENT` ise veriler eklendikçe bu alanın 1'den başlayarak birer birer artırılmasını sağlayacak. 

Bilimsel ad için kullandığımız `VARCHAR` veri tipi ise buraya karakterlerden oluşan veriler gireceğimizi, fakat bu verilerin uzunluğunun değişebileceğini (maksimum 255 karakter) belirtiyor. Verilerimizin uzunluğunu önceden biliyorsak `CHAR`, bilmiyorsak `VARCHAR` kullanmalıyız; ilki hız, ikincisi de depolama alanı avantajı sağlar. 

Açıklama için tanımladığımız veri tipi `TEXT` ise 65335 bayta kadar metin depolamamıza olanak sağlayacak.

Tablomuzun yapısına bir bakalım:

```sql
DESCRIBE kuşlar;
+----------------+--------------+------+-----+---------+----------------+
| Field          | Type         | Null | Key | Default | Extra          |
+----------------+--------------+------+-----+---------+----------------+
| kimllik_no     | int(11)      | NO   | PRI | NULL    | auto_increment |
| bilimsel_ad    | varchar(255) | YES  | UNI | NULL    |                |
| genel_ad       | varchar(50)  | YES  |     | NULL    |                |
| aile_kimlik_no | int(11)      | YES  |     | NULL    |                |
| açıklama       | text         | YES  |     | NULL    |                |
+----------------+--------------+------+-----+---------+----------------+
```

Bu tabloda `NULL` adlı sütun bu kaydının bu alanında veri olmamasına izin verilip verilmeyeceğini bildirir. `Key` adlı sütun bu alanın ne şekilde indekslendiğini gösterir. `Default` sütunu ise bu alanın varsayılan bir değeri olup olmadığını gösterir. `Extra` sütunu ise bu alanla ilgili ekstra bilgileri içerir.  

Şimdi tablomuza veri gireceğiz, aşağıdaki komutları elle yazmak hata getirebileceğinden, ilgili metni [buradan](text/01.txt) kopyalayıp istemciye yapıştırabilirsiniz. 


```sql
INSERT INTO kuşlar (bilimsel_ad, genel_ad)
VALUES
('Charadrius vociferus', 'Yağmur Kuşu'),
('Gavia immer', 'Büyük Kuzey Dalgıç Kuşu'),
('Aix sponsa', 'Ağaç Ördeği'),
('Chordeiles minor', 'Çobanaldatan'),
('Sitta carolinensis', 'Beyaz Göğüslü Sıvacı Kuşu'),
('Apteryx mantelli', 'Kuzay Adalı Kahverengi Kivi Kuşu');
```

Yüklediğimiz veriyi görüntüleyelim:

```sql
SELECT * FROM kuşlar;
+------------+----------------------+------------------------------------+----------------+------------+
| kimllik_no | bilimsel_ad          | genel_ad                           | aile_kimlik_no | açıklama   |
+------------+----------------------+------------------------------------+----------------+------------+
|          1 | Charadrius vociferus | Yağmur Kuşu                        |           NULL | NULL       |
|          2 | Gavia immer          | Büyük Kuzey Dalgıç Kuşu            |           NULL | NULL       |
|          3 | Aix sponsa           | Ağaç Ördeği                        |           NULL | NULL       |
|          4 | Chordeiles minor     | Çobanaldatan                       |           NULL | NULL       |
|          5 | Sitta carolinensis   | Beyaz Göğüslü Sıvacı Kuşu          |           NULL | NULL       |
|          6 | Apteryx mantelli     | Kuzay Adalı Kahverengi Kivi Kuşu   |           NULL | NULL       |
+------------+----------------------+------------------------------------+----------------+------------+
```

Şimdi de kuş gözlemcileri için ayrı bir veritabanı oluşturalım.


```sql
CREATE DATABASE kuşgözlemcileri;
CREATE TABLE kuşgözlemcileri.insanlar (
kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
ünvan VARCHAR(25),
ad VARCHAR(25),
soyad VARCHAR(25),
eposta VARCHAR(255));
```

Ve tablomuza veri girelim:


```sql
INSERT INTO kuşgözlemcileri.insanlar
(ünvan, ad, soyad, eposta)
VALUES
('Bay', 'Mehmet', 'Yılmaz', 'myilmaz@biryer.com'),
('Bay', 'Zeki', 'Çokbilir', 'zcokbilir@baskabiryer.com'),
('Bayan', 'Ayşe', 'Çalışkan', 'acaliskan@ayniyer.com'),
('Bayan', 'Canan', 'Üzülmez', 'cuzulmez@biryer.com');
```

Tablo yapılandırmada işimize yarayacak bilgilere ulaşmanın bir yolu da `SHOW CREATE TABLE` komutudur. Bu komut halihazırdaki bir tablonun oluşturulabilmesi için girilmesi gereken bigileri gösterir. Bu sayede sunucu tarafından bizim tabloyu oluştururken tanımladığımız parametrelere hangi varsayılan değerleri atadığını görmek açısından faydalıdır. Görelim:


```sql
SHOW CREATE TABLE kuşlar\G
*************************** 1. row ***************************
       Table: kuşlar
Create Table: CREATE TABLE `kuşlar` (
  `kimllik_no` int(11) NOT NULL AUTO_INCREMENT,
  `bilimsel_ad` varchar(255) COLLATE utf8_bin DEFAULT NULL,
  `genel_ad` varchar(50) COLLATE utf8_bin DEFAULT NULL,
  `aile_kimlik_no` int(11) DEFAULT NULL,
  `açıklama` text COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY (`kimllik_no`),
  UNIQUE KEY `bilimsel_ad` (`bilimsel_ad`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8 COLLATE=utf8_bin
```

Daha sonra kullanmak üzere `koloni` veritabanına iki tablo daha ekleyeceğiz, ilki kuş aileleri:


```sql
CREATE TABLE kuş_aileleri (
aile_kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
bilimsel_ad VARCHAR(255) UNIQUE,
kısa_açıklama VARCHAR(255));
```

Buna ek olarak bir de kuş ailelerini de gruplayan kuş sınıfları için bir tablo ekleyelim:

```sql
CREATE TABLE kuş_sınıfları (
sınıf_kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
bilimsel_ad VARCHAR(255) UNIQUE,
kısa_açıklama VARCHAR(255),
sınıf_resmi BLOB);
```

**Alıştırmalar**

1. `kuş_kanat_biçimleri` adlı bir tablo oluşturun. Tablonun 
	* `kanat_kimlik_no` adında `CHAR` tipinde maksimum 2 uzunlukta, indeks tipi UNIQUE olan (AUTO_INCREMENT olmayacak) bir alanı;
	* `kanat_biçimi` adında maksimum 25 karakter uzunluğunda `CHAR` tipinde bir alanı;
	* `kanat_örneği` adında `BLOB` değerli bir alanı olsun.
1. `SHOW CREATE TABLE kuş_kanat_biçimleri` komutunu çalıştırın, hem `;` ile bitirerek, hem de `\G` 'le bitirerek deneyin. Hangisinin çıktısı yeni bir tablo tanımlamakta daha kullanışlı görünüyorsa, bunu bir metin editörüne kopyalayıp yapıştırın. `DROP TABLE kuş_kanat_biçimleri;` yaparak tablonuzu silin. Editöre kopyaladığınız metinde `ENGINE`i MyISAM olacak şekilde değiştirin. Değişen metni istemciye kopyalayarak çalıştırın ve tablonuzu tekrar oluşturun.
1.  `kuş_vücut_biçimleri` ve `kuş_gaga_biçimleri` adlı iki tablo daha oluşturun. İlk tablodaki alanlar `vücut_kimlik_no` (`CHAR(3)`, `UNIQUE`),  `vücut_biçimi` (`CHAR(25)`) ve `vücut_örneği` (`BLOB`) olsun. İkinci tabloyu da benzer biçimde oluşturun. İki tabloda da varsayılan depolama motorunu (`ENGINE`) `MyISAM` olarak belirleyin. 

## Tabloların yeniden yapılandırılması 

Bu bölümde tablolara yeni sütun ekleme, var olan sütunları silme ya da opsiyonlarını değiştirmenin yollarını öğreneceğiz.

Tablolarda herhangi bir değişiklik yapmadan önce yedek almayı alışkanlık haline getirmeliyiz. Önce yedeklerimiz için bir yer oluşturalım, bu size kalmış bir karar, burada örnek olması açısından kendi ev dizinimizde geçici verileri tuttuğumuz `tmp` ve bunun altında `sql` adlı bir dizin oluşturalım:


```sql
mkdir ~/tmp
mkdir ~/tmp/sql
```

Şimdi `koloni` veritabanı altındaki `kuşlar` tablosunu bu dizine yedekleyelim:


```bash
mysqldump --user='umut' -p koloni kuşlar > ~/tmp/sql/kuşlar.sql 
```

Dilersek tüm veritabanını da yedekleyebiliriz:


```sql
mysqldump --user='umut' -p koloni > ~/tmp/sql/koloni.sql 
```

Çalışmalarımız sırasında belli noktalarda veritabanını yedeklemek, yapacağımız bir takım hatalardan geri dönebilmek adına faydalı olacaktır. Bir yedeğe dönmek için


```sql
mysql --user='umut' -p koloni < ~/tmp/sql/koloni.sql
```

yapmamız yeterlidir, var olan veritabanı silinecek, yerine yedek yüklenecektir.

Tablo değişiklikleri için kullanılacak komutun genel yapısı şu şekildedir:


```sql
ALTER TABLE <tablo adı> <değişiklikler>;
```

Şimdi `kuş_aileleri` tablomuza `sınıf_kimlik_no` adlı bir sütun ekleyelim, aşağıdaki komut `mysql` istemcisi içinde girmemiz gerekiyor:


```sql
ALTER TABLE kuş_aileleri
ADD COLUMN sınıf_kimlik_no INT;
```


Şimdi de `kuşlar` tablomuzda değişiklik yapacağız; fakat önce yedeğini alalım:


```sql
CREATE TABLE test.kuşlar_yeni LIKE kuşlar;
USE test
DESCRIBE kuşlar_yeni;
```

Bu kopyalama işlem sadece tablonun yapısını ve özelliklerini kopyaladı; şimdi verileri de kopyalayalım:


```sql
INSERT INTO kuşlar_yeni
SELECT * FROM koloni.kuşlar;
```

Şimdi bu kopyaladığımız tabloya `kanat_kimlik_no` adlı bir sütun ekleyelim:


```sql
ALTER TABLE kuşlar_yeni
ADD COLUMN kanat_kimlik_no CHAR(2);
```

`DESCRIBE kuşlar_yeni` yaparak tablomuzun yapısına bakalım; eklediğimiz sütunun en sona eklendiğini göreceğiz. Farz edelim bu sütunu en sona değil de `aile_kimlik_no` sütununun ardına eklemek istiyoruz. Bunu yapmak için önce eklediğimiz sütunu silelim:


```sql
ALTER TABLE kuşlar_yeni
DROP COLUMN kanat_kimlik_no;
```

Şimdi de ekleyelim:

```sql
ALTER TABLE kuşlar_yeni
ADD COLUMN kanat_kimlik_no CHAR(2) AFTER aile_kimlik_no;
```

En başa eklemek isteseydik `FIRST` ifadesini, arkasına bir şey eklemeden kullanacaktık.

Birden fazla değişikliği aynı anda yapabiliriz, örneğin:


```sql
ALTER TABLE kuşlar_yeni 
ADD COLUMN vücut_kimlik_no CHAR(2) AFTER kanat_kimlik_no,
ADD COLUMN gaga_kimlik_no CHAR(2) AFTER vücut_kimlik_no,
ADD COLUMN nesli_tehlikede BIT DEFAULT b'1' AFTER gaga_kimlik_no,
CHANGE COLUMN genel_ad genel_ad VARCHAR(255);
```

Tablomuzdaki kuşlardan dördünü "nesli tehlikede değil" olarak işaretleyelim:


```sql
UPDATE kuşlar_yeni
SET nesli_tehlikede = 0
WHERE kimlik_no IN(1,2,4,5);
```

Nesli tehlikede olan kuşları raporlayalım:

```sql
SELECT kuş_kimlik_no, bilimsel_ad, genel_ad
FROM kuşlar_yeni
WHERE nesli_tehlikede \G
```

`BIT` tipi verilerde, eğer değeri 0 mı diye kontrol etmek için `NOT` ifadesini kullanabiliriz:

```sql
SELECT kimlik_no, bilimsel_ad, genel_ad
FROM kuşlar_yeni
WHERE NOT nesli_tehlikede \G
```


Nesli tehlikede olma durumu iki değerden daha ayrıntılı bir şekilde ifade edilebilir. Bit veri tipinden daha geniş yer kaplayan bir veri tipine geçmekte problem yaşanabileceğinden önce veri tipini geçici olarak `CHAR(255)` olarak değiştirelim:


```sql
ALTER TABLE kuşlar_yeni modify column nesli_tehlikede CHAR(255);
```


Şimdi `nesli_tehlikede` sütununu birden fazla değer alabilecek şekilde değiştirip  `aile_kimlik_no` sütununun ardına taşıyalım:  


```sql
ALTER TABLE kuşlar_yeni
MODIFY COLUMN nesli_tehlikede
ENUM('Yok olmuş',
'Yabanda yok olmuş',
'Tehlikede - Kritik',
'Tehlikede - Orta',
'Tehlikede - Düşük',
'Düşük Risk - Korumaya Bağlı',
'Düşük Risk - Yakın Tehlikede',
'Düşük Risk - Önemsiz')
AFTER aile_kimlik_no;
```

Dilersek tek tek sütunların özelliklerini inceleyebiliriz. Bunun için:

```sql
SHOW COLUMNS FROM kuşlar_yeni LIKE 'nesli_tehlikede' \G
```

`ENUM` veri tipinde, veriyi belirlemek için 1'den başlayarak numaralandırma kullanabiliriz. Örneğin tüm kuşların nesli tehlikede statüsünü 'Düşük Risk - Yakın Tehlikede' yapmak için:


```sql
UPDATE kuşlar_yeni
SET nesli_tehlikede = 7;
```

Şimdi de bir tablonun belli bir sütununun varsayılan değerini atama işlemine bakalım. Bunu yaparken şu anda son derece karışık duran `nesli_tehlikede` sütununu da bir referans tablosu yardımıyla basitleştirmeye çalışacağız. Önce kullanacağımız referans tablosunu hazırlayalım:


```sql
CREATE TABLE koloni.korunma_statüsü
(statü_kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
korunma_kategorisi CHAR(10),
korunma_durumu CHAR(25) );
```

Bu tabloda daha önce tire ile ayırdığımız `ENUM` tipi değerleri, iki sütuna bölmüş olduk. Bu tabloya değerlerimizi girelim: 


```sql
INSERT INTO koloni.korunma_statüsü
(korunma_kategorisi, korunma_durumu)
VALUES('Yok olmuş','Yok olmuş'),
('Yok olmuş','Yabanda yok olmuş'),
('Tehlikede','Kritik'),
('Tehlikede','Orta'),
('Tehlikede','Düşük'),
('Düşük Risk','Korumaya bağlı'),
('Düşük Risk','Yakın Tehlikede'),
('Düşük Risk','Önemsiz');
```

Referans tablomuza bakalım:

```sql
SELECT * FROM koloni.korunma_statüsü;
```

Şimdi `kuşlar_yeni` tablosuna dönelim ve orada `nesli_tehlikede` alanının hem adını değiştirelim hem de varsayılan değerini 'Düşük Risk, Önemsiz' kategorisine bağlayalım:

```sql
ALTER TABLE kuşlar_yeni
CHANGE COLUMN nesli_tehlikede korunma_statüsü_kimlik_no INT DEFAULT 8;
```

Şimdi sadece varsayılan değeri değiştiren bir komut girelim:

 
```sql
ALTER TABLE kuşlar_yeni
ALTER korunma_statüsü_kimlik_no SET DEFAULT 7;
```
Sütunun değişiklikten sonraki halini görelim:

```sql
SHOW COLUMNS FROM kuşlar_yeni LIKE 'korunma_statüsü_kimlik_no' \G
```
Varsayılan değeri dilerseniz aşağıdaki komutla silebilirsiniz; bu komut girilmiş verileri hiçbir şekilde etkilemeyecektir:

```sql
ALTER TABLE kuşlar_yeni
ALTER korunma_statüsü_kimlik_no DROP DEFAULT;
```

Birçok tabloda bir alan `PRIMARY KEY` (birincil anahtar) olarak belirlenir ve  `AUTO_INCREMENT` opsiyonu sayesinde veri eklendikçe birer birer artan bir tam sayı değeri alır. Bu numeratörlerin anlık değerleri `information_schema` adlı veritabanında tutulmaktadır. Bazı durumlarda bu numeratörlerin mevcut durumlarını bilmek ve bunları değiştirmek faydalı olabilir.   


```sql
SELECT auto_increment
FROM information_schema.tables
WHERE table_name = 'kuşlar';
```

Şimdi tekrar `koloni` veritabanını kullanmaya geçip, orada `kuşlar` tablosunun `AUTO_INCREMENT` değerini değiştirelim:


```sql
USE koloni
ALTER TABLE kuşlar
AUTO_INCREMENT = 10;
```

### Diğer yapılandırma metodları

Bazı durumlarda elimizdeki tablonun çok fazla sütun içermesi hem performans hem de mimari açıdan sorun yaratabilir. Böyle bir durumda tabloyu daha küçük tablolara bölmenin bir yolu, mevcut tablo ile aynı yapıda başka bir tablo yaratıp, ayıracağımız sütunları yeni tabloya kopyalayıp, mevcut tablodaki gereksiz sütunları silmektir. Kopyalama ile başlayalım: 

```sql
CREATE TABLE kuşlar_yeni LIKE kuşlar;
```

Tablolara bir bakalım:


```sql
DESCRIBE kuşlar;
```

```sql
DESCRIBE kuşlar_yeni; ```

```sql
SELECT * FROM kuşlar_yeni;
```

Burada dikkat etmemiz gereken yaptığımız kopyalama işleminin sadece yapıyı kopyaladığıdır; veriler ayrıca kopyalanmalıdır.

Bu şekilde tablonun tüm yapısı kopyalanmaktadır; fakat dikkat edilmesi gereken önemli bir nokta şudur: tablo kopyalanırken yeni tabloda `AUTO_INCREMENT` değeri 0 olarak belirlenir. Bunu düzeltmek için eski tablonun kopyalama sırasındaki `AUTO_INCREMENT` değerinin bulunup, bu değerin yeni tabloda aynı şekilde belirlenmesi gerekir.


Önce,

```sql
SHOW CREATE TABLE kuşlar \G
```
burada aradığımız değerin 6 olduğunu göreceğiz. Sonra,


```sql
ALTER TABLE kuşlar_yeni
AUTO_INCREMENT = 6;
```
yaparak yeni tablonun verileri kopyaladıktan sonra aynı doğru numeratörden devam edeceğini garanti ettik. Veriyi nasıl kopyalayacağımızı bir sonraki kısımda göreceğiz.


Dilersek veriyi tablo ile birlikte de kopyalayabiliriz. Örneğin,


```sql
CREATE TABLE kuşlar_detaylar
SELECT kuş_kimlik_no, açıklama
FROM kuşlar;
```


Şimdi değişikler yaptığımız, çalışma kopyamız olan `test.kuşlar_yeni` tablomuzu `koloni.kuşlar` haline getirelim:


```sql
RENAME TABLE koloni.kuşlar TO koloni.kuşlar_eski,
test.kuşlar_yeni TO koloni.kuşlar;
```

Kuşlar ile ilgili epey bir tablomuz oldu, bunlara bakalım:


```sql
SHOW TABLES IN koloni LIKE 'kuşlar%';
```

Herşeyin yolunda olduğundan eminsek, eski kuşlar tablosunu silebiliriz:


```sql
DROP TABLE kuşlar_eski;
```
### Dosyadan tablo oluşturma 

Bu bölümde `SQL` kaynak dosyasından bir tablo okumayı ve bu tabloda yeniden dizme yapmayı göreceğiz. Önce [ulke-kodlari.sql](rsc/ulke-kodlari.sql) dosyasını indirin. Dosyayı metin editörünüzde açarak inceleyin. Dosyayı indirdiğiniz dizin içinden `mysql` istemcisine girin. Önce okuyacağımız tabloyu ekleyeceğimiz bir veritabanı	oluşturalım:


```sql
CREATE DATABASE ülkeler;
USE ülkeler
```

Şimdi kaynak dosyasından tabloyu okumak için:

 
```sql
SOURCE ulke-kodlari.sql;
```

yapalım. 'country' adında bir tablo okumuş oluduk; dilersek buradaki tablo ve sütun isimlerini Türkçeleştirebiliriz. Tablonun ilk beş satırına bakmak için:   


```sql
SELECT * FROM country 
LIMIT 5;
```

yapabiliriz. Bu tabloyu telefon koduna göre yeniden dizmek istersek:


```sql
ALTER TABLE country 
ORDER BY phonecode;
```

yapabiliriz. Dilersek yeniden dizme işlemini sadece `SELECT` komutu içinde de yapabiliriz. Örneğin iki harfli ülke koduna göre dizilmiş ilk beş kaydı görüntülemek için:

```sql
SELECT * FROM ülke
ORDER BY iso
LIMIT 5;
```

### İndeksler

İndeksler tablolardaki bilgiye hızlı erişim sağlamak için önemlidir. Eskiden oluşturduğumuz bir tablonun indeksine bakalım:


```sql
SHOW INDEX FROM kuşgözlemcileri.insanlar \G
```

Buradan tablonun `kimlik_no` sütunundaki bilgi üzerinden endekslendiğini görüyoruz. Aynı tabloda indeks için kullanılan bu sütun/alan ile değil de başka bir alan ile arama yaptığımızda, örneğin `soyad`, sunucunun ne gibi işlemler yaptığını görebilmek için `EXPLAIN` komutunu kullanacağız.

```sql
EXPLAIN SELECT * FROM kuşgözlemcileri.insanlar
WHERE soyad = 'Yılmaz' \G
```

Burada *key* (anahtar) indekslemenin yapıldığı sütuna işaret etmektedir. Şimdi yukarıdaki tabloya yeni bir indeks ekleyelim:


```sql
ALTER TABLE kuşgözlemcileri.insanlar
ADD INDEX insan_adları (soyad, ad);
```

Şimdi tablomuzun yeni haline bakalım:


```sql
SHOW CREATE TABLE kuşgözlemcileri.insanlar \G
```

Bu çıktıdan yeni bir anahtar eklemiş olduğumuzu görüyoruz.

```sql
SHOW INDEX FROM kuşgözlemcileri.insanlar
WHERE Key_name = 'insan_adları' \G
```
Şimdi tekrar `EXPLAIN` komutunu kullanarak değişikliği görelim:

```sql
EXPLAIN SELECT * FROM kuşgözlemcileri.insanlar
WHERE soyad = 'Yılmaz' \G
```

Burada oluşturduğumuz indeksin kullanıldığını görüyoruz, ad ya da soyad ile aramalar artık indeks yardımıyla daha hızlı sonuç verecekler.

Son olarak `korunma_statüsü` tablosundaki indeksi değiştirelim:


```sql
ALTER TABLE korunma_statüsü 
DROP PRIMARY KEY,
CHANGE statü_kimlik_no korunma_statüsü_kimlik_no INT PRIMARY KEY AUTO_INCREMENT;
```

**Alıştırmalar**
1. Hatırlarsanız kuşlarla ilgili bir takım detay bilgileri tutmak için `kuşlar_detaylar` adlı bir tablo yaratmıştık:

		CREATE TABLE kuşlar_detaylar
		SELECT kuş_kimlik_no, açıklama
		FROM kuşlar;

      Şimdi `ALTER TABLE` kullanarak bu tabloya `göç` ve `kuş_besleyen` adlı, `INT` tipli (0 ve 1 değerleri alacak) sütun ekleyip, `açıklama` sütununu `kuş_açıklama` şeklinde yeniden adlandırın.
1. `CREATE TABLE` komutuyla  `habitat_kodları` adlı yeni bir tablo yaratın. `habitat_kimlik_no` adlı ilk sütun  `INT` tipinde `AUTO_INCREMENT` ve `PRIMARY KEY` olarak belirlensin; ikinci sütun `habitat` adını alsın ve `VARCHAR(25)` tipinde olsun. Aşağıdaki veriyi girin:

       INSERT INTO habitat_kodları (habitat)
       VALUES('Kıyılar'), ('Çöller'), ('Ormanlar'),
       ('Otluklar'), ('Göller, Nehirler, Birikintiler'),
       ('Sazlıklar, Bataklıklar'), ('Dağlar'), ('Okyanuslar'),
       ('Şehir');

