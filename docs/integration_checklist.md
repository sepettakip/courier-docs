# Kurye Entegrasyonu Test Checklist

Entegrasyonu tamamladıktan sonra test modülünü kullanarak aşağıdaki testlerin başarıyla geçildiğinden emin olunuz.
Testlerin kontrol etmemiz için bizi bilgilendirin ve test için oluşturulan sipariş ID numaralarını bizimle paylaşınız.

## Kimlik Doğrulama (Check Credentials) Testi

Restoranın, kurye firmasına bağlanma senaryosunun testidir.

✅ CC-01 (Başarılı): Doğru kimlik bilgileri (username,password) ile bağlantı denemesi. (Beklenen: 200 OK)  
✅ CC-02 (Hatalı): Yanlış veya eksik kimlik bilgileri ile bağlantı denemesi. (Beklenen: 400 Bad Request veya 401 Unauthorized)

## Paket Oluşturma (Create Package) Testi

Restoranın gönderdiği paket bilgilerinin kurye sistemine doğru şekilde iletilmesini test eder.

✅ ORD-01 (Başarılı): Sipariş oluşturma testi 1.  
✅ ORD-02 (Başarılı): Sipariş oluşturma testi 2.  
✅ ORD-03 (Başarılı): Sipariş oluşturma testi 3.  
✅ ORD-04 (Başarılı): Sipariş oluşturma testi 4.  
✅ ORD-05 (Başarılı): Sipariş oluşturma testi 5.  
✅ ORD-06 (Hatalı Adres): Geçersiz koordinat veya eksik adres bilgisi ile sipariş denemesi. (Beklenen: 400 Bad Request ve Hata Mesajı)

## Sipariş Durum Güncellemeleri (Status Lifecycle) Testi

Siparişin oluşturulduktan sonraki yaşam döngüsünün POS sistemine geri bildirilmesini test eder.

### Operasyonel Akış

✅ Yola Çıktı: Kuryenin restorana doğru harekete geçtiği durum güncellemesi.  
✅ Teslim Aldı: Kuryenin paketi restorandan teslim aldığı durum güncellemesi.  
✅ Teslim Edildi: Paketin müşteriye başarıyla ulaştırıldığı durum güncellemesi.

### İstisnai Durumlar

✅ İptal Edildi: Kabul edilen bir siparişin (müşteri kabul etmedi vs) sonradan iptal edilmesi.  
✅ Reddedildi: Siparişin kurye firması tarafından (kapasite yetersizliği vb.) ilk aşamada reddedilmesi.
