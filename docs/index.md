# Kurye Entegrasyonu

Bu doküman, Sepettakip ile kurye firmaları arasında gerçekleştirilecek entegrasyonun teknik altyapısını açıklamaktadır.
Entegrasyon, siparişlerin otomatik olarak kurye firmalarına iletilmesini ve kurye güncellemelerinin Sepettakip sistemine aktarılmasını sağlar.


## Terimler Sözlüğü
Dokümanda yer alan bazı terimler aşağıda açıklanmıştır.

- **Entegrasyon**: Sepettakip platformu ile Kurye Firması sistemleri arasında veri alışverişini sağlayan ana teknik bağlantıyı ve bu bağlantının yasal/teknik çerçevesini ifade eder.
- **Anlaşma (Restoran Eşleştirmesi)**: Kurye firması ile tekil bir restoran arasındaki ticari ilişkiyi ve bu ilişkinin Sepettakip paneli üzerindeki dijital tanımlamasını ifade eder.
- **Erişim Belirteci**: Restoranın kurye firması hizmetlerini kullanabilmesi için gereken; API Anahtarı (API Key), Gizli Anahtar (Secret Key) veya Müşteri ID'si gibi dijital kimlik doğrulama bilgilerinin tamamıdır.
- **Paket**: İçerisinde bir siparişi barındıran, kurye firmasına teslimat için iletilen tekil lojistik birimi ifade eder.
- **Sipariş Akışı**: Bir siparişin Sepettakip'e düşmesinden, kurye firmasına iletilip teslim edilmesine kadar geçen sürecin tamamını ve statü değişimlerini ifade eder.
- **Otomatik Sipariş Aktarımı**: Restoranın manuel onayına gerek kalmadan, siparişin belirlenen statüye (Örn: "Hazırlanıyor") geçmesiyle birlikte sistemin otomatik olarak kurye çağrısı oluşturmasını ifade eder.
- **Manuel Aktarım**: Otomatik aktarımın kapalı olduğu veya hata verdiği durumlarda, restoranın "Kurye Çağır" butonu ile siparişi elle tetiklemesi durumudur.
- **Kurye Çağırma**: Restoranın, kurye çağırmak için anlık olarak sipariş oluşturarak kurye firmasına iletmesi işlemidir.