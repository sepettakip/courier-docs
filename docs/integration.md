# Entegrasyon

Sepettakip, restoranların sipariş yönetimini ve kurye atama süreçlerini otomatikleştirmek amacıyla kurye firmaları için kapsamlı bir entegrasyon altyapısı sunar. Bu doküman, kurye firmalarının Sepettakip ekosistemine dahil olabilmesi için gerekli teknik standartları, API uç noktalarını (endpoints) ve veri akış senaryolarını detaylandırır.

## Genel Entegrasyon Yapısı

Sepettakip ile Kurye Firması arasındaki entegrasyon RESTful API mimarisine dayanır ve veri alışverişi JSON formatında gerçekleşir. İletişim güvenliği için tüm isteklerin HTTPS protokolü üzerinden yapılması zorunludur.

Entegrasyonun tam fonksiyonlu çalışabilmesi için hem Kurye Firmasının geliştirmesi gereken API uç noktaları (Endpoints) hem de Sepettakip'e geri bildirim yapılması gereken servisler bulunmaktadır. Veri akışı iki yönlüdür

### 1\. Kurye Firması Tarafından Hazırlanacak Servisler (Inbound)

Sepettakip sunucularının, kurye operasyonlarını tetiklemek için istek (request) göndereceği servislerdir. Kurye firması bu servisleri REST API standartlarına uygun olarak dışarıya açmalıdır:

- **Restoran Kimlik Doğrulama (Check Credentials)**: Restoranın girdiği Erişim Belirteci (API Key/Token) bilgisinin kurye firması sisteminde geçerli olup olmadığını kontrol eden servistir.
- **Sipariş Oluşturma (Create Order)**: Sepettakip'ten gelen sipariş bilgilerini alarak sisteminde kayıt açan ve operasyonu başlatan servistir.
- **Sipariş İptali (Cancel Order)**: Restoranın veya sistemin iptal ettiği siparişleri kurye firması sisteminden düşürmek için kullanılan servistir.

### 2\. Sepettakip Tarafına Yapılacak Bildirimler (Outbound / Webhook)

Kurye firması sisteminde gerçekleşen durum değişikliklerinin Sepettakip'e bildirilmesidir. Kurye firması, bu olaylar gerçekleştiğinde Sepettakip API'sine istek gönderir:

- **Sipariş Durum Güncellemesi (Status Webhook)**: Paketin durumu değiştiğinde (Örn: Kurye atandı, Teslim alındı, Teslim edildi, İptal edildi) bu bilginin anlık olarak Sepettakip'e iletilmesini sağlayan servistir.

## Güvenlik ve Yetkilendirme (Headers)

Veri bütünlüğünü ve güvenliğini sağlamak için HTTP isteklerinde aşağıdaki başlık (header) bilgileri zorunludur.

Yön: **Sepettakip -> Kurye Firması İstekleri**

Sepettakip tarafından kurye firmasına gönderilen bütün HTTP isteklerinde (Sipariş oluşturma, iptal vb.) aşağıdaki header bilgileri yer alır:

```json
{   
  "Api-Key": "<SEPETTAKIP_API_KEY>"
}
```

Yön: **Kurye Firması -> Sepettakip İstekleri (Webhook)**

Kurye firması tarafından Sepettakip'e gönderilen bütün bildirimlerde (Durum güncelleme vb.) aşağıdaki header bilgileri yer almalıdır:

```json
{   
  "Courier-Company": "<COURIER_COMPANY_KEY>",
  "Api-Key": "<SEPETTAKIP_API_KEY>"
}
```

**Not**: Test ve Production ortamlarında kullanılacak `COURIER_COMPANY_KEY` ve `SEPETTAKIP_API_KEY` bilgileri e-posta aracılığı ile temin edilecektir.

## Entegrasyon Süreci

Entegrasyonun sağlıklı yürütülebilmesi için Kurye Firması, API isteklerinin karşılanacağı Base URL bilgisini Sepettakip ekibine iletmelidir. Test ve Canlı (Production) ortamları için ayrıştırılmış URL yapıları kullanılması önerilir.

**Örnek Base URL Yapısı**:

- **Test Ortamı**: `https://api-test.couriercompany.com/v1`
- **Production Ortamı**: `https://api.couriercompany.com/v1/`

Tüm API uç noktaları (endpoints) bu Base URL'in devamına eklenir.

**Örnek**: Sipariş oluşturma endpoint'i `/create-package` ise ve test ortamındaysak tam istek adresi `https://api-test.couriercompany.com/v1/create-package` olacaktır.

### Restoran Kimlik Doğrulama (Check Credentials)

Restoranın Sepettakip paneline girdiği entegrasyon bilgilerinin (Kullanıcı adı/Şifre vb.) kurye firması tarafında doğrulanmasını sağlar. Bu adım başarılı olmadan sipariş akışı başlatılamaz.

**Endpoint**: `/check-credentials`  
**Method**: `POST`  
**Request Body**:

```json
{
    "username": "",
    "password": ""
}
```

**Not**: username olarak restoranın Sepettakip tarafındaki ID numarası kullanılması önerilir. Bu bilgi, restorandan temin edilebilir. password bilgisi ise her restoran için öze olarak üretilmelidir

Response olarak HTTP durum kodu 200 dönerse doğrulama başarılıdır. Aksi durumda entegrasyon etkinleşmez.

Durum Kodu | Durum İsmi  | Açıklama
---------- | ----------- | ----------------------------------------------------
200        | OK          | Kimlik bilgileri doğru.
400        | Bad Request | Kimlik bilgileri hatalı veya istek gövdesi geçersiz.

**Not**: Yanıt gövdesi opsiyoneldir; yalnızca http durum kodu ile karar verilebilir.

### Paket Oluşturma (Create Package)

Bir sipariş "Hazırlanıyor" statüsüne geçtiğinde veya restoran manuel olarak tetiklediğinde bu servis çağrılır.

**Endpoint**: `/create-package`  
**Method**: `POST`  
**Request Body**:

```json
{
  "auth": {
    "username": "789",
    "password": "superSecret123"
  },
  "restaurant": {
    "id": "789",
    "name": "The Firma"
  },
  "order": {
    "order_id": "12345",
    "platform": "GoFody",
    "preparation_time": 5,
    "note": "Zili çalmayın, kapıya bırakın.",
    "amount": 250.5,
    "is_paid": true,
    "payment_type": {
      "key": "credit-card",
      "method": "Online Kredi Kartı"
    },
    "address": {
      "neighborhood": "Abbasağa Mah.",
      "address": "Atatürk Cd.",
      "building_no": "10",
      "floor": "3",
      "door_number": "5",
      "town": "Beşiktaş",
      "city": "İstanbul",
      "description": "3\. kat, sağdaki daire",
      "latitude": 41.0438,
      "longitude": 29.0094
    },
    "customer": {
      "full_name": "Ahmet Yılmaz",
      "phone_number": "+905551112233"
    },
    "products": [
      {
        "quantity": 2,
        "unit_price": 50,
        "product_name": "Lahmacun",
        "product_note": "Acısız olsun",
        "total_amount": 100
      },
      {
        "quantity": 1,
        "unit_price": 150.5,
        "product_name": "Kuzu Şiş",
        "product_note": "",
        "total_amount": 150.5
      }
    ]
  }
}
```

Not: Kurye firması, operasyonel ihtiyaçları doğrultusunda istek gövdesine (payload) ek alanlar talep edebilir; ancak temel şema korunur ve zorunlu alanlar değiştirilmez.

Ana Nesne | Alan               | Tip    | Zorunluluk  | Açıklama
--------- | ------------------ | ------ | ----------- | --------------------------------------------------------------------------------------
`auth`    | username           | String | **Zorunlu** | Restoran doğrulama kullanıcısı adı.
`auth`    | password           | String | **Zorunlu** | Restoran doğrulama şifresi.
`order`   | order_id           | String | **Zorunlu** | Sepettakip sistemindeki benzersiz sipariş ID'si.
`order`   | platform           | String | **Zorunlu** | Siparişin kaynağı (Bkz: Platform Listesi).
`order`   | amount             | Float  | **Zorunlu** | Siparişin toplam tutarı.
`order`   | is_paid            | Bool   | **Zorunlu** | Ödeme durumu. `true`: Ödendi, `false`: Kapıda Tahsilat.
`order`   | payment_type.key   | String | **Zorunlu** | Ödeme yöntemi kodu (Bkz: Ödeme Tipleri).
`address` | latitude/longitude | Float  | Opsiyonel   | Koordinat bilgisi. **Not:** Telefonla siparişlerde (CallerID) bu alan `null` gelebilir.
`address` | city/town          | String | **Zorunlu** | İl ve İlçe bilgisi.

Adres bilgisindeki _latitude_ ve _longitude_ bilgisi, _CallerID_ siparişlerinde ve kurye çağır ile oluşturulan siparişlerde iletilmez. Bu yüzden null değer alabilir. Ayrıca neighborhood, building_no, floor ve door_number alanları opsiyoneldir. Ödeme tipinin key bilgisi aşağıdaki değerleri alabilir.

**Desteklenen Değerler Listesi**:

- Platformlar: `Gofody`, `Yemeksepeti`, `Getir`, `Trendyol`, `Sepetapp`, `Migros`, `Fuudy`, `CallerID`, `WhatsApp`
- Ödeme Tipleri (key): `paye`, `setcard`, `sodexo`, `sodexomobile`, `garantipay`, `moneypay`, `edenredonline`, `onlinecard`, `smarticket`, `sodexoonline`, `bkm`, `tokenflexonline`, `pos`, `sepetpara`, `card`, `cash`, `ticket`, `multinet`, `metropol`, `debt`, `winwin`, `tokenflex`, `cio`, `yemekmatik`

Durum Kodu | Durum İsmi  | Açıklama
---------- | ----------- | --------------------------------------------------
200        | OK          | Sipariş başarıyla oluşturuldu.
400        | Bad Request | İstek gövdesi geçersiz veya zorunlu alanlar eksik.

**Response Body**:

```json
{
    "status": false,
    "error_code": "out_of_service_area",
    "message": "Adres, hizmet alanı dışında."
}
```

Uyarı: Sipariş başarıyla oluşturulduktan sonra, tüm durum güncellemeleri kurye firması tarafından Sepettakip'e bildirilmelidir.

Sipariş kurye sisteminde işlenemediğinde, kurye servisinden geçerli bir hata kodu (`error_code`) dönerse, restoran arayüzünde aktarılamama nedeni bu koda karşılık gelen açıklamayla gösterilir.

Hata kodu iletilmez, tanınmaz veya biçim olarak geçersizse, sistem nedeni "kurye firması kaynaklı genel hata" olarak bildirir.

Hata Kodu (`error_code`)     | Mesaj                  | Açıklama
:--------------------------- | :--------------------- | :------------------------------------------------------
`unauthorized_access`        | Yetkisiz Erişim        | API anahtarı veya şifre hatalı.
`out_of_service_area`        | Hizmet Alanı Dışı      | Adres, kurye firmasının hizmet poligonları dışında.
`location_resolution_failed` | Adres Hatası           | Adres metni veya koordinatlar haritada doğrulanamadı.
`restaurant_closed`          | Restoran Kapalı        | Kurye firması o an için hizmet vermiyor (Mesai dışı).
`validation_error`           | Validasyon Hatası      | Eksik veya hatalı veri formatı (Örn: Telefon no eksik).
`company_issue`              | Firma Kaynaklı Problem | Kurye sistemi iç hatası (5xx).
`courier_rejected`           | Kurye Reddetti         | Kurye firması siparişi kabul etmedi.
`other`                      | Diğer Hata             | Yukarıdakiler dışında kalan genel hata.

### Paket İptali (Cancel Package)

Kurye firmasina iletilen bir sipariş restoran kaynaklı nedenlere iptal edildiğinde bu servis çağrılır.

**Endpoint**: `/cancel-package`  
**Method**: `POST`  
**Request Body**:

```json
{
    "order_id": "545"
}
```

Durum Kodu | Durum İsmi  | Açıklama
---------- | ----------- | --------------------------------------------------
200        | OK          | Sipariş başarıyla oluşturuldu.
400        | Bad Request | İstek gövdesi geçersiz veya zorunlu alanlar eksik.

### Sipariş Durum Güncellemesi (Status Webhook)

Sepettakip bir siparişi kurye firmasına aktardıktan sonra, siparişin operasyonel takibi kurye firmasındadır. Kurye firması, sahadaki tüm durum değişikliklerini (kurye atandı, teslim edildi vb.) aşağıdaki webhook servisini çağırarak Sepettakip'e anlık olarak iletmekle yükümlüdür.

**API Base URL**:

- Test: `https://test-api.sepettakip.com`
- Prod: `https://api.sepettakip.com`  

**Endpoint**: `/courier-company/package`  
**Method**: `PATCH`  
**Headers**:

```json
  {
    "courier-company": "sepetfast",
    "Api-Key": "<SEPETTAKIP_API_KEY>"
  }
```

**Request Body**:

```json
{
    "order_id": "12345",
    "status": "on_way",
    "courier_eta": "2023-10-25 14:30:00"
}
```

Alan          | Tip      | Zorunlu | Varsayılan | Kabul Edilen Değerler                                           | Açıklama
------------- | -------- | ------- | ---------- | --------------------------------------------------------------- | ---------------------------
`order_id`    | string   | Evet    | -          | -                                                               | Siparişin benzersiz kimliği.
`status`      | enum     | Evet    | -          | `assigned` · `picked_up` · `delivered` · `canceled`, `rejected` | Sipariş durum bilgisi.
`courier_eta` | datetime | Hayır   | -          | ISO-8601 UTC (örn. `2025-08-18T07:30:00Z`)                      | Kurye tahmini varış zamanı .

 Durum     | Açıklama                          | Not
 --------- | --------------------------------- | ---------------------------------------------------------------------------------------------------
 assigned  | Kuryeye Atandı / Kurye Yola Çıktı | Kurye, paketi **restorandan** almak için yola çıktı. `courier_eta` gönderilmeli.
 picked_up | Kurye Paketi Aldı                 | Kurye, paketi restorandan teslim aldı. Sepettakip’te sipariş “yolda” görünür.
 delivered | Kurye Paketi Teslim Etti          | Kurye, paketi alıcıya teslim etti. (Sipariş, Sepettakip'te "teslim edildi" olarak güncellenir.)
 canceled  | Kurye Paketi İptal Etti           | Kurye, paketi iptal etti. (Sipariş, Sepettakip'te "iptal edildi" olarak güncellenir)
 rejected  | Kurye Paketi Reddetti             | Kurye, paketi reddetti. (Restorana sadece bildirim yapılır, iptal edilmez.)

#### **Webhook Response**

```json
{
    "message": "İşlem başarılı."
}
```

Kod | Açıklama                                                       | Notlar
--- | -------------------------------------------------------------- | ----------------------------------
204 | Güncelleme başarılı.                                           | Gövde boş olabilir.
400 | Geçersiz istek / sipariş zaten hedef statüde.                  | Alan eksik/format hatası.
401 | API anahtarı geçersiz veya bulunamadı.                         | `Api-Key` kontrol edin.
403 | Sipariş aktarılmamış ya da farklı bir kurye firmasına ait.     | Yetki/yönlendirme hatası.
404 | Sipariş bulunamadı.                                            | `id` doğrulayın.
422 | Mevcut statü, hedef statü için uygun değil (iş kuralı ihlali). | Örn. `courier_eta` zorunlu.
502 | Sistem güncelleniyor; daha sonra deneyin.                      | Geçici durum. Yalnızca test ortamı
500 | Beklenmeyen sunucu hatası.                                     | Destek ile iletişime geçin.

#### **Request Example**

```bash
curl -sS --fail \
  -X PATCH 'https://test-api.sepettakip.com/courier-company/package' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Courier-Company: sepetfast' \
  -H 'Api-Key: aHR0cHM6Ly93d3cueW91dHViZS5jb20vc2hvcnRzL0hubEtUMkRLSGhN' \
  --data-raw '{
    "order_id": 1982,
    "status": "on_way",
    "courier_eta": "2024-08-12T12:20:32.514011Z"
  }'
```
