# Kurye Entegrasyonu

Sepettakip, restoranların siparişlerini kurye firmalarına iletmek için bir entegrasyon sunar. Bu entegrasyon sayesinde restoranlar, sipariş akışlarını kolayca yönetebilir ve kuryelerle etkili bir iletişim kurabilir.

## Entegrasyon Öncesi

Entegrasyon başlamadan önce kurye firması, Sepettakip'ten gelen HTTP istekleri karşılamak için aşağıdaki servisleri hazırlamalıdır.

- **Restoran Kimlik Doğrulama (Check Credentials)**  
- **Paket Oluşturma (Send Order)**
- **Paket İptali (Cancel Package)**

İletilen siparişlerin sorumluluğu kurye firmasına aittir. Bu yüzden kurye firması Teslimat Durum Webhook aracılığı ile paket güncellemelerini Sepettakip'e iletmek zorundadır.

- **Teslimat Durum Webhook**

Test ve Production ortamlarında kullanılacak `Api-Key` bilgileri sizlere iletilecektir. Geliştirmeler tamamlandıktan sonra karşılıklı testler yapılarak canlı ortama geçiş sağlanacaktır.

## Entegrasyon Süreci

Sepettakip tarafından gönderilen bütün HTTP istekleri, aşağıdaki header bilgisi ile gönderilir.

```json
{
    "Api-Key": "<SEPETTAKIP_API_KEY>"
}
```

Bu bilgi, isteğin Sepettakip'ten geldiğini teyit eder. Bu bilgi, test ve production ortamları için sizinle paylaşılacaktır.

### Restoran Kimlik Doğrulama

Restoran, çalışmak istediği kurye firmasından aldığı **erişim bilgilerini** Sepettakip arayüzüne girer. Sepettakip bu bilgileri kurye sistemine **Check Credentials** isteğiyle doğrular. **Doğrulama başarılıysa** entegrasyon etkinleşir ve sipariş akışı başlatılabilir. **Doğrulama başarısızsa** entegrasyon etkinleşmez ve sipariş akışı başlatılamaz.

#### Restoran Kimlik Doğrulama Request

**Endpoint:** `check-credentials`  
**Method:** `POST`  
**Request Body:**

```json
{
    "username": "",
    "password": ""
}
```

> `username` olarak restoranın Sepettakip tarafındaki ID numarası kullanılması önerilir. Bu bilgi, restorandan temin edilebilir.
>

#### Restoran Kimlik Doğrulama Response

- **200 OK** → Kimlik bilgileri **doğru**.
- **400 Bad Request** → Kimlik bilgileri **hatalı** veya istek gövdesi **geçersiz**.

> Yanıt gövdesi opsiyoneldir; yalnızca durum kodu ile karar verilebilir.

### Paket Oluşturma

Bir sipariş **restoranda hazırlanıyor statüsüne geçtiğinde** ve **kurye sipariş akışı açıksa**, sipariş otomatik olarak seçili kurye firmasına iletilir.

#### Send Order Request

**Endpoint:** `send-order`  
**Method:** `POST`  
**Request Body:**

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
      "description": "3. kat, sağdaki daire",
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

> Kurye firması, operasyonel ihtiyaçları doğrultusunda istek gövdesine (payload) ek alanlar talep edebilir; ancak temel şema korunur ve zorunlu alanlar değiştirilmez.

- `auth` -> kurye firması tarafından restorana verilen erişim bilgilerini içerir.
- `restaurant` -> restoranın Sepettakip tarafındaki bilgilerini içerir.
- `order` -> sipariş detaylarını içerir.
  - `order_id` -> siparişin Sepettakip tarafındaki benzersiz numarası.
  - `platform` -> siparişin geldiği platform.
  - `preparation_time` -> siparişin hazırlanma süresi (dakika).
  - `amount` -> siparişin tutarı. Tahsilat -*yapılacaksa*- bu tutar üzerinden yapılır.
  - `is_paid` -> müşteriden ödeme alındıysa `true`, alınmadı ise `false`.
  - `note` ->  müşteri notu.
  - `payment_type` -> ödeme yöntemi.
    - `key` -> ödeme yönteminin Sepettakip tarafındaki kodu.
    - `method` -> ödeme yöntemi
  - `address` -> siparişin teslim edileceği adres.
- `customer` -> müşteri bilgileri.
- `products` -> sipariş içeriği.

Adres bilgisindeki `latitude` ve `longitude` bilgisi, **CallerID** siparişlerinde iletilmez.
Ödeme tipinin `key` bilgisi aşağıdaki değerleri alabilir.

| Alan         | Değerler                                                                                                                                                                                                                               |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| payment_type | paye, setcard, sodexo, sodexomobile, garantipay, moneypay, edenredonline, onlinecard, smarticket, sodexoonline, bkm, tokenflexonline, pos, sepetpara, card, cash, ticket, multinet, metropol, debt, winwin, tokenflex, cio, yemekmatik |
| platform     | Gofody, Yemeksepeti, Getir, Trendyol, Sepetapp, Migros, Fuudy, CallerID                                                                                                                                                                |

> Sipariş başarıyla oluşturulduktan sonra, **tüm durum güncellemeleri** kurye firması tarafından Sepettakip’e bildirilmelidir.
>

#### Send Order Response

- **200 OK** → Sipariş oluşturuldu.
- **400 Bad Request** → Sipariş oluşturulamadı.

**Bad Request Response Body:**

```json
{
    "status": false,
    "code": "out_of_service_area",
    "message": "Adres, hizmet alanı dışında."
}
```

> Yanıt gövdesi opsiyoneldir; yalnızca durum kodu ile karar verilebilir.

Sipariş kurye sisteminde işlenemediğinde, kurye servisinden **geçerli bir hata kodu (`code`)** dönerse, restoran arayüzünde **aktarılamama nedeni** bu koda karşılık gelen açıklamayla gösterilir.  

**Hata kodu iletilmez, tanınmaz veya biçim olarak geçersizse**, sistem nedeni **“kurye firması kaynaklı genel hata”** olarak bildirir.

| Hata Kodu                    | Mesaj                  | Açıklama                                                                                                                                    |
| ---------------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `unauthorized_access`        | Yetkisiz Erişim        | Kurye entegrasyonu doğrulanamadı. API anahtarı/kimlik bilgisi geçersiz, eksik ya da süresi dolmuş olabilir. Erişim bilgilerini güncelleyin. |
| `out_of_service_area`        | hizmet Alanı Dışı      | Teslimat adresi kurye firmasının kapsama alanı dışında. Mesafe/ilçe/polygon kurallarına takıldı.                                            |
| `location_resolution_failed` | Adres Hatası           | Adres/koordinatlar çözümlenemedi veya tutarsız. (Eksik alan, hatalı lat/lng, geocoding başarısız.)                                          |
| `restaurant_closed`          | Restoran Kapalı        | Restoran çalışma saatleri dışında ya da geçici olarak kapalı olduğu için sipariş işlenemiyor.                                               |
| `company_issue`              | Firma Kaynaklı Problem | Kurye servisinde iç hata/timeout/bakım (5xx). Daha sonra yeniden deneyin; gerekirse alternatif kurye seçin.                                 |
| `validation_error`           | Validasyon Hatası      | Zorunlu alan eksik veya format hatalı (telefon, tutar, para birimi, şema uyumsuzluğu vb.).                                                  |
| `other`                      | Bilinmeyen Hata        | Kategorize edilemeyen beklenmedik hata.                                                                                                     |

### Paket İptal

**Endpoint:** `cancel-order`  
**Method:** `POST`  
**Request Body:**

```json
{
    "order_id": "545"
}
```

Bayi paketi herhangi bir nedenden ötürü iptal ederse iptal bilgisi Kurye firmasına iletilir. Hata oluşması durumunda bildirim tekrar yapılmaz. Her durumda firmanın bu bildirimi kabul ettiği varsayılır.

### Teslimat Durum Webhook’u (Kurye → Sepettakip)

Sepettakip bir siparişi kurye firmasına aktardıktan sonra, siparişin **operasyonel takibi kurye firmasındadır**. Bu nedenle kurye firması, **tüm durum değişikliklerini** aşağıdaki webhook ile Sepettakip’e **anlık** iletmekle yükümlüdür. Aksi halde akış senkronu bozulur (gecikme, çift kayıt, yanlış teslim görünebilir).

**API Base URL (Test):** `https://test-api.sepettakip.com`  
**API Base URL (Prod):** `https://api.sepettakip.com`  
**Endpoint:** `/courier-company/package`  
**Method:** `PATCH`  
**Headers:**

```json
{
    "courier-company": "sepetfast",
    "Api-Key": "<SEPETTAKIP_API_KEY>"
}
```

**Request Body:**

```json
{
    "order_id": "Siparişin Sepettakip tarafındaki ID numarası.",
    "status": "Siparişin son durumu.",
    "courier_eta": "Kuryenin restorana tahmini ulaşma tarihi."
}
```

| Alan          | Tip      | Zorunlu | Varsayılan | Kabul Edilen Değerler                              | Açıklama                                                  |
| ------------- | -------- | ------- | ---------- | -------------------------------------------------- | --------------------------------------------------------- |
| `order_id`    | string   | Evet    | -          | -                                                  | Siparişin benzersiz kimliği.                              |
| `status`      | enum     | Evet    | -          | `on_way` · `picked_up` · `delivered` · `cancelled` | Sipariş durum bilgisi.                                    |
| `courier_eta` | datetime | Hayır   | -          | ISO-8601 UTC                                       | Kurye tahmini varış zamanı (örn. `2025-08-18T07:30:00Z`). |

| Durum     | Açıklama                 | Not                                                                              |
| --------- | ------------------------ | -------------------------------------------------------------------------------- |
| on_way    | Kurye Yola Çıktı         | Kurye, paketi **restorandan** almak için yola çıktı. `courier_eta` gönderilmeli. |
| picked_up | Kurye Paketi Aldı        | Kurye, paketi restorandan teslim aldı. Sepettakip’te sipariş “yolda” görünür.    |
| delivered | Kurye Paketi Teslim Etti | Kurye, paketi alıcıya teslim etti. Süreç tamamlandı.                             |
| cancelled | Kurye Paketi İptal Etti  | Kurye, paketi iptal etti. Süreç tamamlandı.                                      |

#### **Webhook Response:**

```json
{
    "message": "İşlem başarılı."
}
```

| Kod | Açıklama                                                       | Notlar                             |
| --- | -------------------------------------------------------------- | ---------------------------------- |
| 204 | Güncelleme başarılı.                                           | Gövde boş olabilir.                |
| 400 | Geçersiz istek / sipariş zaten hedef statüde.                  | Alan eksik/format hatası.          |
| 401 | API anahtarı geçersiz veya bulunamadı.                         | `Api-Key` kontrol edin.            |
| 403 | Sipariş aktarılmamış ya da farklı bir kurye firmasına ait.     | Yetki/yönlendirme hatası.          |
| 404 | Sipariş bulunamadı.                                            | `id` doğrulayın.                   |
| 422 | Mevcut statü, hedef statü için uygun değil (iş kuralı ihlali). | Örn. `courier_eta` zorunlu.        |
| 502 | Sistem güncelleniyor; daha sonra deneyin.                      | Geçici durum. Yalnızca test ortamı |
| 500 | Beklenmeyen sunucu hatası.                                     | Destek ile iletişime geçin.        |

#### **Request Example:**

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
