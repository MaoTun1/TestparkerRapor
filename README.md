# TestparkerRapor
 Testparker sitesinin web app zaviyetlerinin raporu
 
![image](https://github.com/user-attachments/assets/79b5535b-4e1a-49bd-8be9-446152590411)
url'i alttaki gibi düzenleyerek server tarafından verilen işlemin gerçekleştirilebildiğini görmekteyiz. Server side template injection açığını doğurmakta.

http://php.testsparker.com/artist.php?id=={{7*7}}

Aşağıda görüldüğü üzere servera url düzenleyerek gönderdiğimiz çarpma işlemi server tarafından gerçekleştirilmiş durumda.
![image](https://github.com/user-attachments/assets/c5be4ca7-4aeb-49a1-bd27-46367f86abe0)

Çözüm önerileri ek:
Şablonlar kullanıcı kontrollü girdilerden oluşturulmamalıdır. Kullanıcı girdisi şablon parametreleri kullanılarak şablona aktarılmalıdır. Verileri ayrıştırmadan önce istenmeyen ve riskli karakterleri kaldırarak girdiyi şablonlara geçirmeden önce sanitize edin. Bu, şablonlarınızın herhangi bir kötü niyetli araştırması için güvenlik açıklarını en aza indirir.

![image](https://github.com/user-attachments/assets/10ab2aa5-9fc0-4274-90f7-cb6dd244d9ec)
Yukarıda verilen htmlspecialchars metoduyla girilen girdi özel karakterlerden arındırılır.


![image](https://github.com/user-attachments/assets/94a742cb-1d2b-4f5a-b5f6-68cfc47b078c)

http://php.testsparker.com/process.php?file=Generics/about.nsp url’ine tıkladığımda alt kısımda bulunan mysql’den dönen error kodunun yazdırıldığını fark ettim. Error mesajları saldırganlar tarafından bilgi almak için kullanılabilir ve error based sqli tabanlı saldırılar gerçekleştirilebilir.

![image](https://github.com/user-attachments/assets/f2a5da40-0f06-4f05-b79e-9a1030481aee)

Yukarıda örnek olarak verilmiş olan kod parçacığında mysql tarafından gönderilmiş olan error mesajı çıktı olarak düzenlenmeden verilmiştir. Düzenlenmeden verilen bu hata mesajı saldırganlar için kullanılabilir hale getirilebilir.
![image](https://github.com/user-attachments/assets/f2a5da40-0f06-4f05-b79e-9a1030481aee)

Ancak aşağıda verilen kod parçacığı PDO sınıfı kullanarak sql girdisi parametize edilir, temizlenir ayrıca hata mesajı gizlenerek yazdırılır. Framework’ten alınan hata mesajı döndürülmez yeni hata mesajı yazdırılır.
![image](https://github.com/user-attachments/assets/3004dcb8-e2a9-4bfd-966c-991f59557eeb)


![image](https://github.com/user-attachments/assets/d5f7b83c-9276-43a2-83f8-d28e2612e9ad)
http://php.testsparker.com/process.php?file=Generics/contact.nsp sayfa kaynağında bir not bırakıldığı görülmekte.

![image](https://github.com/user-attachments/assets/f7bd50c7-2ce8-47e7-a61f-cf07b9add30b)
Yukarıda bulunan notta directory izinlerinin düzenlenmediği belirtilmekte yani path traversal yapılabildiği anlaşılmakta.
php.testsparker.com/process.php?file=/../../../../ sayfa url ini bu şekilde düzenleyip path traversal olduğu doğrulanmakta.


Aşağıda bulunan kod örneği ile zafiyeti oluşturan girdi temizlenebilir ve izin verilen dizinler kntrol edilebilir.
![image](https://github.com/user-attachments/assets/346ca3c4-8d75-464c-b160-090955bd323f)

Örnek kod parçacığı açıklama:
sanitizePath Fonksiyonu:
•	str_replace ile '../' ve './' gibi dizin dolaşım girişimlerini kaldırır.
•	basename fonksiyonu ile sadece dosya adını alır ve dizin yollarını atar.
isValidPath Fonksiyonu:
•	realpath fonksiyonu ile gerçek dosya yolunu alır.
•	strpos fonksiyonu ile kullanıcının sağladığı yolun base dizin içinde olup olmadığını kontrol eder.


![image](https://github.com/user-attachments/assets/90dd290f-f4b1-4748-b363-bd3a1e3a1e0b)
php.testsparker.com/artist.php?id=test ve http://php.testsparker.com/hello.php?hpp=Netsparker&pp= url'leri dikkatimi çekti ve xss açığı bulunabileceğini düşünerek url'leri alltaki şekilde düzenledim. 
php.testsparker.com/artist.php?id=<script>alert(123)</script> 
http://php.testsparker.com/hello.php?hpp=Netsparker&pp=<script>alert(123)</script>
url'leri bu şekilde yazdığımda xss zafiyeti olduğunu gözlemledim.

![image](https://github.com/user-attachments/assets/b43aa62e-4af8-49e8-b584-2da9f9d749d9)

Ve login.php sayfasında verilen kullanıcı, parola bilgileriyle giriş yaptığımızda arama kısmında da <script>alert(123)</script> scripti çalışmaktadır.

Çözüm Önerileri ek:
Aşağıda bulunan kod bloğunda alınan girdiyi nasıl html encode metoduyla encode edilerek, xss zafiyetinden kaçınıldığı anlaşılabilir.
<?php
// Girdiyi temizleme ve kodlama fonksiyonu
function sanitizeInput($data) {
    // HTML özel karakterlerini dönüştürme
    return htmlspecialchars($data, ENT_QUOTES, 'UTF-8');
}
// URL'den gelen girdiyi al
$name = isset($_GET['name']) ? $_GET['name'] : '';
// Girdiyi temizle
$safeName = sanitizeInput($name);
// Güvenli çıktıyı kullan
echo "Merhaba, " . $safeName . "!";
?>

![image](https://github.com/user-attachments/assets/be795249-d2e0-4c8d-8ebd-bcbf6e034152)

http://php.testsparker.com/phpinfo.php http://php.testsparker.com/artist.php?id=test url’inde idor açığı olduğunu düşünüp farklı değerler deniyorum. id=3,id=4 değerlerini girdiğimde farklı kullanıcıların bilgilerine ulaşabildiğimi görüyorum.

![image](https://github.com/user-attachments/assets/841ad388-2e7b-431c-8739-173a6482ebcf)

Çözüm önerileri ek:
Erişim Kontrol Mekanizmalarını Uygula: Tüm nesnelere erişim kontrolleri eklenmeli ve her istekte yetkilendirme doğrulaması yapılmalıdır. Kullanıcının sadece yetkili olduğu kaynaklara erişebilmesi sağlanmalıdır.
Nesne Referanslarını Gizle: Doğrudan nesne kimlikleri yerine rastgele oluşturulan, tahmin edilemeyen ve geçici referanslar kullanılmalıdır. Örneğin, veri tabanındaki sıra numarası (ID) yerine bir hash değeri veya UUID kullanmak.
Nesne Düzeyinde Yetkilendirme Kontrolü: Her nesneye erişim denendiğinde, kullanıcının bu nesneye erişim iznine sahip olup olmadığı kontrol edilmelidir. Bu, kullanıcıların sadece kendi verilerine veya yetkili oldukları verilere erişmesini sağlar.
Doğrulama ve Filtreleme: Kullanıcıdan gelen tüm girişler doğrulanmalı ve filtrelenmelidir. Kullanıcının gönderdiği parametrelerin beklenen formatta ve değer aralığında olup olmadığı kontrol edilmelidir.

![image](https://github.com/user-attachments/assets/0bb88e1a-e69c-4623-b27c-dc087f88f0a5)
http://php.testsparker.com/hello.php?hpp=Netsparker&pp=+DAST&irc= url’ine html injection payloadı ekliyoruz ve http://php.testsparker.com/hello.php?hpp=Netsparker&pp=<h1>HTML</h1> haline getiriyoruz.

![image](https://github.com/user-attachments/assets/94cd4880-d935-4d1f-829b-23cb399eab23)

Çözüm önerileri ek:
HTML injectionu önlemek için HTML encoding örneği aşağıda verilmiştir.
<?php 
if ($_SERVER["REQUEST_METHOD"] == "POST") { 
// Kullanıcıdan alınan yorum 
$user_comment = $_POST['comment']; 
// HTML encoding işlemi 
$encoded_comment = htmlspecialchars($user_comment, ENT_QUOTES, 'UTF-8');
 // Kodlanmış çıktıyı göster 
echo '<h3>Yorumunuz:</h3>'; echo '<p>' . $encoded_comment . '</p>'; 
}
 ?>


![image](https://github.com/user-attachments/assets/c5f70cd1-2fee-4296-bb62-ffd34f2d0250)
![image](https://github.com/user-attachments/assets/e5e64f3c-aee4-41b7-88fb-8b1ff6f721c5)

![image](https://github.com/user-attachments/assets/ec804389-1e9d-47cf-9cff-6efaeaa1d32e)







Hazırlayan: METEHAN

