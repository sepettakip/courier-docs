# Kurye Entegrasyonu

Bu doküman, Sepettakip ile kurye firmaları arasında gerçekleştirilecek entegrasyonun teknik altyapısını açıklamaktadır. Entegrasyonun temel hedefi, siparişlerin otomatik olarak kurye firmalarına iletilmesini ve kurye durum güncellemelerinin anlık olarak Sepettakip sistemine aktarılmasını sağlamaktır.

## Terimler Sözlüğü

Dokümanda yer alan bazı terimler aşağıda açıklanmıştır.

- **Entegrasyon**: Sepettakip ile Kurye Firması arasındaki veri alışverişini sağlayan teknik bağlantıyı ve bu bağlantının yasal/teknik çerçevesini ifade eder.
- **Anlaşma**: Kurye firması ile tekil bir restoran arasındaki ticari ilişkiyi ve bu ilişkinin Sepettakip paneli üzerindeki dijital tanımlamasını ifade eder.
- **Erişim Belirteci**: Restoranın kurye firması hizmetlerini kullanabilmesi için gereken; API Anahtarı (API Key), Gizli Anahtar (Secret Key) veya Müşteri ID'si gibi dijital kimlik doğrulama bilgilerinin tamamıdır.
- **Paket**: İçerisinde bir siparişi barındıran, kurye firmasına teslimat için iletilen tekil lojistik birimi ifade eder.
- **Sipariş Akışı**: Bir siparişin Sepettakip'e düşmesinden, kurye firmasına iletilip teslim edilmesine kadar geçen sürecin tamamını ve statü değişimlerini ifade eder.
- **Otomatik Sipariş Aktarımı**: Restoranın manuel onayına gerek kalmadan, siparişin belirlenen statüye (Örn: "Hazırlanıyor") geçmesiyle birlikte sistemin otomatik olarak kurye çağrısı oluşturmasını ifade eder.
- **Manuel Aktarım**: Otomatik aktarımın kapalı olduğu veya hata verdiği durumlarda, restoranın "Kurye Çağır" butonu ile siparişi elle tetiklemesi durumudur.
- **Kurye Çağırma**: Restoranın, kurye çağırmak için anlık olarak sipariş oluşturarak kurye firmasına iletmesi işlemidir.
