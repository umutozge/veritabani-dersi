# Veritabanı yapılandırma

Bir veritabanını yapılandırırken temel noktalar şunlardır:

* kaç adet tablo olacak; adları ne olacak.
* kaç adet alan olacak; adları ne olacak.
* kayıtlarda hangi alanda hangi cins veri tutulacak (Ör. metin, sayı, dosya, dijital veri, tarih, saat).

Veritabanının yapısına **şema** denmektedir.

Şimdi kuşlarla ilgili bir veritabanı yaratıp bunu yapılandıracağız. Önce veritabanını oluşturalım:

```bash
CREATE DATABASE koloni
CHARACTER SET utf8
COLLATE utf8_general_ci;
```

Bu komutla veritabanına bir ad verdik, kullanılacak karakter kümesini tanımladık ve verilerimizin `utf8` karakter kodlamasında ve dijital şekilde (0 ve 1) tutulmasını istediğimizi belirttik.

Aşağıdaki komutları girerek, ilkiyle veritabanımızın oluşup oluşmadığını kontrol edip, ikincisiyle de varsayılan veritabanı olarak atayarak, her komutta adını yazmaktan kurtulalım.


```bash
SHOW DATABASES;
USE koloni
```

## Tabloların  oluşturulması

Bu veritabanında bir adet ana tablomuz ve birkaç tane referans tablomuz olacak. Normalde tablolar 255 sütuna kadar büyütülebilirler; fakat büyük tablolar performans ve yapı açısından tercih edilmemelidir: veritabanını yapılandırırken mümkün olduğu kadar küçük tablolar halinde yapılandırmaya dikkat etmeliyiz.

Ana tablomuzu oluşturalım:

```bash
CREATE TABLE kuşlar
(kimllik_no INT AUTO_INCREMENT PRIMARY KEY,
bilimsel_ad VARCHAR(255) UNIQUE,
genel_ad VARCHAR(50),
aile_kimlik_no INT,
açıklama TEXT);
```

Kimlik numarası satırında kullandığımız `PRIMARY KEY` tablomuzun bu alana göre indekslenmesini istediğimizi belirtiyor; `AUTO INCREMENT` ise veriler eklendikçe bu alanın 1'den başlayarak birer birer artırılmasını sağlayacak. 

Bilimsel ad için kullandığımız `VARCHAR` veri tipi ise buraya karakterlerden oluşan veriler gireceğimizi, fakat bu verilerin uzunluğunun değişebileceğini (maksimum 255 karakter) belirtiyor. Verilerimizin uzunluğunu önceden biliyorsak `CHAR`, bilmiyorsak `VARCHAR` kullanmalıyız; ilki hız, ikincisi de depolama alanı avantajı sağlar. 

Açıklama için tanımladığımız veri tipi `TEXT` ise 65335 bayta kadar metin depolamamıza olanak sağlayacak.

Tablomuzun yapısına bir bakalım:

```bash
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


```bash
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

```bash
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


```bash
CREATE DATABASE kuşgözlemcileri;
CREATE TABLE kuşgözlemcileri.insanlar (
kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
ünvan VARCHAR(25),
ad VARCHAR(25),
soyad VARCHAR(25),
eposta VARCHAR(255));
```

Ve tablomuza veri girelim:


```bash
INSERT INTO kuşgözlemcileri.insanlar
(ünvan, ad, soyad, eposta)
VALUES
('Bay', 'Mehmet', 'Yılmaz', 'myilmaz@biryer.com'),
('Bay', 'Zeki', 'Çokbilir', 'zcokbilir@baskabiryer.com'),
('Bayan', 'Ayşe', 'Çalışkan', 'acaliskan@ayniyer.com'),
('Bayan', 'Canan', 'Üzülmez', 'cuzulmez@biryer.com');
```

Tablo yapılandırmada işimize yarayacak bilgilere ulaşmanın bir yolu da `SHOW CREATE TABLE` komutudur. Bu komut halihazırdaki bir tablonun oluşturulabilmesi için girilmesi gereken bigileri gösterir. Bu sayede sunucu tarafından bizim tabloyu oluştururken tanımladığımız parametrelere hangi varsayılan değerleri atadığını görmek açısından faydalıdır. Görelim:


```bash
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


```bash
CREATE TABLE kuş_aileleri (
aile_kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
bilimsel_ad VARCHAR(255) UNIQUE,
kısa_açıklama VARCHAR(255));
```

Buna ek olarak bir de kuş ailelerini de gruplayan kuş sınıfları için bir tablo ekleyelim:

```bash
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
1.  `kuş_vücut_biçimleri` ve `kuş_gaga_biçimleri` adlı iki tablo daha oluşturun. İlk tablodaki alanlar `vücut_kimlik_no` (`CHAR(3)`, `UNIQUE`),  `vücut_biçimi` (`CHAR(25)` ve `vücut_örneği` (`BLOB`) olsun. İkinci tabloyu da benzer biçimde oluşturun. İki tabloda da varsayılan depolama motorunu (`ENGINE`) `MyISAM` olarak belirleyin. 

## Tabloların yeniden yapılandırılması 

Bu bölümde tablolara yeni sütun ekleme, var olan sütunları silme ya da opsiyonlarını değiştirmenin yollarını öğreneceğiz.

Tablolarda herhangi bir değişiklik yapmadan önce yedek almayı alışkanlık haline getirmeliyiz. Önce yedeklerimiz için bir yer oluşturalım, bu size kalmış bir karar, burada örnek olması açısından kendi ev dizinimizde geçici verileri tuttuğumuz `tmp` ve bunun altında `sql` adlı bir dizin oluşturalım:


```bash
mkdir ~/tmp
mkdir ~/tmp/sql
```

Şimdi `koloni` veritabanı altındaki `kuşlar` tablosunu bu dizine yedekleyelim:


```bash
mysqldump --user='umut' -p koloni kuşlar > ~/tmp/sql/kuşlar.sql 
```

Dilersek tüm veritabanını da yedekleyebiliriz:


```bash
mysqldump --user='umut' -p koloni > ~/tmp/sql/koloni.sql 
```

Çalışmalarımız sırasında belli noktalarda veritabanını yedeklemek, yapacağımız bir takım hatalardan geri dönebilmek adına faydalı olacaktır. Bir yedeğe dönmek için


```bash
mysql --user='umut' -p koloni < ~/tmp/sql/koloni.sql
```

yapmamız yeterlidir, var olan veritabanı silinecek, yerine yedek yüklenecektir.

Tablo değişiklikleri için kullanılacak komutun genel yapısı şu şekildedir:


```bash
ALTER TABLE <tablo adı> <değişiklikler>;
```

Şimdi `kuş_aileleri` tablomuza `sınıf_kimlik_no` adlı bir sütun ekleyelim, aşağıdaki komut `mysql` istemcisi içinde girmemiz gerekiyor:


```bash
ALTER TABLE kuş_aileleri
ADD COLUMN sınıf_kimlik_no INT;
```


Şimdi de `kuşlar` tablomuzda değişiklik yapacağız; fakat önce yedeğini alalım:


```bash
CREATE TABLE test.kuşlar_yeni LIKE kuşlar;
USE test
DESCRIBE kuşlar_yeni;
```

Bu kopyalama işlem sadece tablonun yapısını ve özelliklerini kopyaladı; şimdi verileri de kopyalayalım:


```bash
INSERT INTO kuşlar_yeni
SELECT * FROM koloni.kuşlar;
```

Şimdi bu kopyaladığımız tabloya `kanat_kimlik_no` adlı bir sütun ekleyelim:


```bash
ALTER TABLE kuşlar_yeni
ADD COLUMN kanat_kimlik_no CHAR(2);
```

`DESCRIBE kuşlar_yeni` yaparak tablomuzun yapısına bakalım; eklediğimiz sütunun en sona eklendiğini göreceğiz. Farz edelim bu sütunu en sona değil de `aile_kimlik_no` sütununun ardına eklemek istiyoruz. Bunu yapmak için önce eklediğimiz sütunu silelim:


```bash
ALTER TABLE kuşlar_yeni
DROP COLUMN kanat_kimlik_no;
```

Şimdi de ekleyelim:

```bash
ALTER TABLE kuşlar_yeni
ADD COLUMNT kanat_kimlik_no CHAR(2) AFTER aile_kimlik_no;
```

En başa eklemek isteseydik `FIRST` ifadesini, arkasına bir şey eklemeden kullanacaktık.

Birden fazla değişikliği aynı anda yapabiliriz, örneğin:


```bash
ALTER TABLE kuşlar_yeni 
ADD COLUMN vücut_kimlik_no CHAR(2) AFTER kanat_kimlik_no,
ADD COLUMN gaga_kimlik_no CHAR(2) AFTER vücut_kimlik_no,
ADD COLUMN nesli_tehlikede BIT DEFAULT b'1' AFTER gaga_kimlik_no,
CHANGE COLUMN genel_ad genel_ad VARCHAR(255);
```

Tablomuzdaki kuşlardan dördünü "nesli tehlikede değil" olarak işaretleyelim:


```bash
UPDATE kuşlar_yeni SET nesli_tehlikede = 0
WHERE kuş_kimlik_no IN(1,2,4,5);
```

Nesli tehlikede olan kuşları raporlayalım:

```bash
SELECT kuş_kimlik_no, bilimsel_ad, genel_ad
FROM kuşlar_yeni
WHERE nesli_tehlikede \G
```

`BIT` tipi verilerde, eğer değeri 0 mı diye kontrol etmek için `NOT` ifadesini kullanabiliriz:

```bash
SELECT kuş_kimlik_no, bilimsel_ad, genel_ad
FROM kuşlar_yeni
WHERE NOT nesli_tehlikede \G
```
Nesli tehlikede olma durumu iki değerden daha ayrıntılı bir şekilde ifade edilebilir. Şimdi `nesli_tehlikede` sütununu birden fazla değer alabilecek şekilde değiştirip  `aile_kimlik_no` sütununun ardına taşıyalım:  


```bash
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

```bash
SHOW COLUMNS FROM kuşlar_yeni LIKE 'nesli_tehlikede' \G
```

`ENUM` veri tipinde, veriyi belirlemek için 1'den başlayarak numaralandırma kullanabiliriz. Örneğin tüm kuşların nesli tehlikede statüsünü 'Düşük Risk - Yakın Tehlikede' yapmak için:


```bash
UPDATE kuşlar_yeni
SET nesli_tehlikede = 7;
```

Şimdi de bir tablonun belli bir sütununun varsayılan değerini atama işlemine bakalım. Bunu yaparken şu anda son derece karışık duran `nesli_tehlikede` sütununu da bir referans tablosu yardımıyla basitleştirmeye çalışacağız. Önce kullanacağımız referans tablosunu hazırlayalım:


```bash
CREATE TABLE koloni.korunma_statüsü
(statü_kimlik_no INT AUTO_INCREMENT PRIMARY KEY,
korunma_kategorisi CHAR(10),
korunma_durumu CHAR(25) );
```

Bu tabloda daha önce tire ile ayırdığımız `ENUM` tipi değerleri, iki sütuna bölmüş olduk. Bu tabloya değerlerimizi girelim: 


```bash
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

```bash
SELECT * FROM koloni.korunma_statüsü;
```

Şimdi `kuşlar_yeni` tablosuna dönelim ve orada `nesli_tehlikede` alanının hem adını değiştirelim hem de varsayılan değerini 'Düşük Risk, Önemsiz' kategorisine bağlayalım:

```sql
ALTER TABLE kuşlar_yeni
CHANGE COLUMN nesli_tehlikede korunma_statüsü_kimlik_no INT DEFAULT 8;
```
```mysql
ALTER TABLE kuşlar_yeni
CHANGE COLUMN nesli_tehlikede korunma_statüsü_kimlik_no INT DEFAULT 8;
```

Şimdi sadece varsayılan değeri değiştiren bir komut girelim:

 
```mysql
ALTER TABLE kuşlar_yeni
ALTER korunma_statüsü_kimlik_no SET DEFAULT 7;
```
Sütunun değişiklikten sonraki halini görelim:

```mysql
SHOW COLUMNS FROM kuşlar_yeni LIKE 'korunma_statüsü_kimlik_no' \G
```
Varsayılan değeri dilerseniz aşağıdaki komutla silebilirsiniz; bu komut girilmiş verileri hiçbir şekilde etkilemeyecektir:

```bash
ALTER TABLE kuşlar_yeni
ALTER korunma_statüsü_kimlik_no DROP DEFAULT;
```


