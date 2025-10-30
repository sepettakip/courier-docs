# Entegrasyon

Sepettakip, restoranların sipariş veya onboarding süreçlerini otomatikleştirmek için kurye firmalarına API tabanlı entegrasyon imkanı sunar. Bu doküman, kurye firmalarının Sepettakip ile nasıl entegre olabileceğini ve gerekli API çağrılarını detaylandırır.

## Entegrasyon Öncesi

Kurye firması, Sepettakip ile entegrasyon sağlanabilmesi için aşağıdaki servisleri hazırlamalıdır.

- **Restoran Kimlik Doğrulama (Check Credentials)**  
- **Paket Oluşturma (Create Package)**
- **Paket İptal Etme (Cancel Package)**

Ayrıca sipariş güncellemelerini Sepettakip'e iletebilmek için aşağıdaki servis kullanılacak şekilde geliştirme yapılmalıdır.

- **Paket Güncelleme (Kurye Firması -> Sepettakip)**

Sepettakip tarafından gönderilen bütün HTTP isteklerinde aşağıdaki header bilgisi yer alır.

```json
{   
  "Api-Key": "<SEPETTAKIP_API_KEY>"
}
```

Ayrıca Sepettakip'e gönderilen bütün HTTP isteklerinde aşağıdaki header bilgisi yer almalıdır.

```json
{   
  "Courier-Company": "<COURIER_COMPANY_KEY>",
  "Api-Key": "<SEPETTAKIP_API_KEY>"
}
```

Test ve Production ortamlarında kullanılacak `COURIER_COMPANY_KEY` ve `SEPETTAKIP_API_KEY` bilgileri e-posta aracılığı ile temin edilecektir.

## Entegrasyon Süreci

### Restoran Kimlik Doğrulama

Restoran, çalışmak istediği kurye firmasından aldığı **erişim bilgilerini** Sepettakip arayüzüne girer. Sepettakip bu bilgileri kurye sistemine **Check Credentials** isteğiyle doğrular. **Doğrulama başarılıysa** entegrasyon etkinleşir ve sipariş akışı başlatılabilir. **Doğrulama başarısızsa** entegrasyon etkinleşmez ve sipariş akışı başlatılamaz.

#### Restoran Kimlik Doğrulama İsteği

**Endpoint:** `check-credentials`  
**Method:** `POST`  
**Request Body:**

```json
{
    "username": "",
    "password": ""
}
```

**Not**: `username` olarak restoranın Sepettakip tarafındaki ID numarası kullanılması önerilir. Bu bilgi, restorandan temin edilebilir. `password` bilgisi ise her restoran için öze olarak üretilmelidir.

#### Restoran Kimlik Doğrulama Cevabı

- **200 OK** → Kimlik bilgileri **doğru**.
- **400 Bad Request** → Kimlik bilgileri **hatalı** veya istek gövdesi **geçersiz**.

**Not**: Yanıt gövdesi opsiyoneldir; yalnızca http durum kodu ile karar verilebilir.

### Paket Oluşturma

Bir sipariş **restoranda hazırlanıyor statüsüne geçtiğinde** ve **kurye sipariş akışı açıksa**, sipariş otomatik olarak seçili kurye firmasına iletilir.

#### Sipariş Oluşturma İsteği

**Endpoint:** `create-package`  
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

**Not**: Kurye firması, operasyonel ihtiyaçları doğrultusunda istek gövdesine (payload) ek alanlar talep edebilir; ancak temel şema korunur ve zorunlu alanlar değiştirilmez.

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

Adres bilgisindeki `latitude` ve `longitude` bilgisi, **CallerID** siparişlerinde iletilmez. Bu yüzden `null` değer alabilir. Ayrıca `neighborhood`, `building_no`, `floor` ve `door_number` alanları opsiyoneldir.
Ödeme tipinin `key` bilgisi aşağıdaki değerleri alabilir.

| Alan         | Değerler                                                                                                                                                                                                                               |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| payment_type | paye, setcard, sodexo, sodexomobile, garantipay, moneypay, edenredonline, onlinecard, smarticket, sodexoonline, bkm, tokenflexonline, pos, sepetpara, card, cash, ticket, multinet, metropol, debt, winwin, tokenflex, cio, yemekmatik |
| platform     | Gofody, Yemeksepeti, Getir, Trendyol, Sepetapp, Migros, Fuudy, CallerID                                                                                                                                                                |

**Not**: Sipariş başarıyla oluşturulduktan sonra, **tüm durum güncellemeleri** kurye firması tarafından Sepettakip’e bildirilmelidir.

#### Sipariş Oluşturma Cevabı

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

**Not**: Yanıt gövdesi opsiyoneldir; yalnızca durum kodu ile karar verilebilir.

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

Restoran, paketi herhangi bir nedenden ötürü iptal ederse iptal bilgisi Kurye firmasına iletilir. Hata oluşması durumunda bildirim tekrar yapılmaz. Her durumda firmanın bu bildirimi kabul ettiği varsayılır.

#### Paket İptal İsteği

**Endpoint**: `cancel-order`  
**Method**: `POST`  
**Request Body**:  

```json
{
    "order_id": "545"
}
```

#### Paket İptal Cevabı

- **200 OK** → Sipariş iptal edildi.
- **400 Bad Request** → Sipariş iptal edilemedi.

### Paket Güncelleme (Kurye → Sepettakip)

Sepettakip bir siparişi kurye firmasına aktardıktan sonra, siparişin **operasyonel takibi kurye firmasındadır**. Bu nedenle kurye firması, **tüm durum değişikliklerini** aşağıdaki webhook ile Sepettakip’e **anlık** iletmekle yükümlüdür.

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

#### **Webhook Response**

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

## Entegrasyon Testi

Servisler geliştirilirken, Sepettakip tarafından gelen isteklerin test edilebilmesi ve Sepettakip'e gönderilen isteklerin doğrulanabilmesi için test ortamı sağlanmaktadır.

**Test API Base URL**: `https://test-api.sepettakip.com`  

**Not**: Test servislerinin tamamı test ortamında çalışır. Production ortamında kullanılamaz.

### Test Erişim Belirteci Doğrulama

Restoranlar, kurye firmalarından aldıkları erişim bilgilerini Sepettakip arayüzüne girdiklerinde, Sepettakip bu bilgileri doğrulamak için bu servisi kullanır.
Sizlerde bu eirşim belirteçlerinin doğruluğunu kontrol etmek için bu servisi kullanabilirsiniz.

Ayrıca, bu doğrulama işlemini yapmadan siparişler tarafınıza aktarılmaz

#### Test Erişim Belirteci Doğrulama İsteği

**Endpoint**: `/courier-company/test/check-credentials`
**Method**: `POST`
**Request Body:**

```json
{
    "username": "789",
    "password": "superSecret123"
}
```

#### Test Erişim Belirteci Doğrulama Cevabı

- **200 OK** → Kimlik bilgileri **doğru**.
- **400 Bad Request** → Kimlik bilgileri **hatalı** veya istek gövdesi **geçersiz**.

### Test Erişim Belirteci Silme

Restoranların erişim bilgilerini Sepettakip arayüzünden sildiklerinde, Sepettakip bu bilgileri sistemden kaldırmak için bu servisi kullanır. Sizlerde bu erişim belirteçlerini sisteminizden silmek için bu servisi kullanabilirsiniz

#### Test Erişim Belirteci Silme İsteği

**Endpoint**: `/courier-company/test/check-credentials`
**Method**: `DELETE`

#### Test Erişim Belirteci Silme Cevabı

- **200 OK** → Kimlik bilgileri silindi.
- **400 Bad Request** → İstek gövdesi **geçersiz**.

### Test Siparişlerini Listeleme

Sepettakip'e gönderilen test siparişlerini listelemek için bu servis kullanılabilir.

**Not**: Son 3 saat içerisinde gönderilen siparişler listelenir.

#### Test Siparişlerini Listeleme İsteği

**Endpoint**: `/courier-company/test/package`  
**Method**: `GET`

### Test Sipariş Oluşturma

Sipariş kabul servisinin doğru çalıştığını doğrulamak için test sipariş oluşturma servisi kullanılabilir.

**Not**: 30 saniyede bir test sipariş oluşturabilirsiniz.

#### Test Sipariş Oluşturma İsteği

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

Test siparii oluştururken bütün alanlar opsiyoneldir. Bu bilgiler eksik gönderildiğinizde tarafınızıa nasıl yansıdığını test edebilirsiniz. Ancak `platform` bilgisi her zaman `sepetapp` olarak gelecektir.

### Test Siparişi Detayları

Sepettakip'e gönderilen belirli bir test siparişinin detaylarını görüntülemek için bu servis kullanılabilir.

#### Test Siparişi Detayları İsteği

**Endpoint**: `/courier-company/test/package/:package_id`  
**Method**: `GET`

#### Test Siparişi Detayları Cevabı

```json
{
  "order_id": 13247,
  "order__restofficial__name": "Leina Coffe Terrace",
  "order__restofficial_id": 785,
  "status": "created",
  "error_code": null,
  "attributes": {},
  "created_at": "2025-10-20T11:33:49.855363Z",
  "updated_at": "2025-10-20T11:33:49.855374Z"
}
```

### Test Sipariş Güncelleme

Test siparişlerin durum güncellemelerini test etmek için bu servis kullanılabilir.

**Not**: Sadece iptal isteği test edilebilir.

#### Test Sipariş Güncelleme İsteği

**Endpoint**: `/courier-company/test/package/:package_id`  
**Method**: `PATCH`  
**Request Body**:

```json
{
    "package_id": "2651",
    "status": "cancel"
}
```
