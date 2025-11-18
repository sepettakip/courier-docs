# Entegrasyon Testi ve Sandbox
Sepettakip, entegrasyon sürecini hızlandırmak ve geliştirdiğiniz servisleri kendi kendinize test edebilmeniz (Self-Check) için bir dizi **Tetikleyici API (Trigger Endpoints)** sunar.

Bu servislerin çalışma mantığı şöyledir:

- Siz (Kurye Firması), aşağıdaki test endpointlerine istek gönderirsiniz.
- Sepettakip, bu isteği alır ve gerçek bir restoran davranışı sergileyerek sizin Base URL olarak tanımladığınız sisteme ilgili isteği (Örn: Sipariş Oluşturma) gönderir.
- Böylece, canlı bir restorana ihtiyaç duymadan entegrasyonunuzu uçtan uca test edebilirsiniz.

Not: Test servislerinin tamamı test ortamında çalışır. Production ortamında kullanılamaz.

**Test API Base URL**: https://test-api.sepettakip.com

API-Key ve name bilgileri size e-posta ile iletilecektir.

## Erişim Belirteci Doğrulama
Geliştirdiğiniz Check Credentials (Kimlik Doğrulama) servisinin doğru çalışıp çalışmadığını test etmek için kullanılır. Bu endpointi çağırdığınızda, Sepettakip sizin sisteminize bir doğrulama isteği gönderir.

**Ayrıca, bu doğrulama işlemini yapmadan siparişler tarafınıza aktarılmaz.**

**Endpoint**: `/courier-company/test/check-credentials`  
**Method**: `POST`  
**Request Body**:  
```json
{
  "credentials": {
    "username": "789",
    "password": "superSecret123"
  }
}
```

## Erişim Belirtecini Silme
Test restoranına eklenen erişim bilgileri silinmek istendiğinde bu servis kullanılır. 

**Endpoint**: `/courier-company/test/check-credentials`  
**Method**: `DELETE`



## Test Siparişlerini Listeleme
Restoranlar, test siparişlerini Sepettakip arayüzünden görüntülemek istediklerinde, Sepettakip bu servisi kullanarak test siparişlerini listeler. Sizlerde bu işlemi gerçekleştirmek için bu servisi kullanabilirsiniz.

**Endpoint**: `/courier-company/test/package`  
**Method**: `GET`  

## Test Sipariş Oluşturma
Sisteminizin sipariş alma (Create Package) kapasitesini test etmek için kullanılır. Gönderdiğiniz JSON içeriğindeki verilerle sanal bir sipariş oluşturulur ve bu sipariş sizin create-package servisinize gerçek bir istek olarak iletilir.

**Endpoint**: `/courier-company/test/package`  
**Method**: `POST`  
**Request Body**:  
```json
{
    "amount": 150.00,
    "name": "Hüdaverdi Allahaldı",
    "phone": "05555555555",
    "city": "İstanbul",
    "town": "Üsküdar",
    "neighborhood": "Burhaniye",
    "description": "Civanali",
    "building_no": "GoFody",
    "floor": "1",
    "door_number": "2",
    "latitude": 43.44214579905264,
    "longitude": 21.365578112422366,
    "payment_type": "cash"
}
```

**Not**: 30 saniyede bir test sipariş oluşturabilirsiniz.

## Test Sipariş Güncelleme
Oluşturduğunuz bir test siparişinin, restoran tarafından iptal edilmesi senaryosunu simüle eder.


**Endpoint**: `/courier-company/test/package/:package_id`  
**Method**: `PATCH`  
**Request Body**:
```json
{
    "package_id": "2651",
    "status": "cancel"
}
```

**Not**: Sadece iptal isteği test edilebilir.
