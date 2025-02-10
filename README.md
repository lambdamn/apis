# Ez-Connector

[TOC]

## 1. Танилцуулга

**Ez-Connector** сан нь `Касс`-ын системийг `Голомт пос`-той холболт хийх зориулалттай бөгөөд
тус гарын авлага нь уг холболтыг хэрхэн хийх талаар зааварчилгааг агуулсан болно.

## 1.1. Sequence Diagram

```seq
Client->EzConnector.dll: Гүйлгээний хүсэлт
EzConnector.dll->Golomt POS: Гүйлгээний хүсэлт
Note right of Golomt POS: Төлбөр төлөлт\nхийгдэнэ
Golomt POS->EzConnector.dll: Гүйлгээний хүсэлтийн\nхариу
EzConnector.dll->Client: Гүйлгээний хүсэлтийн\nхариу
```

## 1.2. Пос төхөөрөмжийн Driver суулгах

| Посын загвар | USB Driver                          |
| -----------: | :---------------------------------- |
|      NewLand | [Татах](/apis/files/usb_driver.rar) |

## 1.3. Ez-Connector.dll санг холбох

Хувилбар - 0.0.11

[Ez-Connector.dll (64bit) - Татах](/apis/files/64bit/ez-connector.dll)

[Ez-Connector.dll (32bit) - Татах](/apis/files/32bit/ez-connector.dll)

**Жава код**

```
import com.sun.jna.Callback;
import com.sun.jna.Library;
import com.sun.jna.Native;

public class EzConnectorDemo {
  public static final int PAYMENT_NONE = 0;
  public static final int PAYMENT_CARD = 1;
  public static final int PAYMENT_CASH = 2;
  public static final int PAYMENT_SOCIAL_MOBILE = 3;
  public static final int PAYMENT_SOCIAL_QRCODE = 4;
  public static final int PAYMENT_UPI_QRCODE = 5;
  public static final int PAYMENT_MERCHANT_WALLET = 6;
  public static final int PAYMENT_MONPAY = 7;

  public interface EzConnector extends Library {
    public String logon();
    public String payment(double amount, boolean skipPrint, int payment, int defaultQrPayment, String extra);
    public String refund(String traceno, boolean skipPrint);
    public String settlement(boolean skipPrint);
    public String version();
    public String getIdCard();
  }

  public static void main(String[] args) {
    EzConnector lib =
        (EzConnector)Native.loadLibrary("ez-connector.dll", EzConnector.class);
    String result = lib.payment(2, true, PAYMENT_NONE, PAYMENT_MONPAY, "");
    System.out.println(result);
    String version = lib.version();
    System.out.println(version);
  }
}
```

**C# код**

```
using System.Runtime.InteropServices;

namespace EzConnectorDemo
{
    public class EzConnector
    {
        public static readonly int PAYMENT_NONE = 0;
        public static readonly int PAYMENT_CARD = 1;
        public static readonly int PAYMENT_CASH = 2;
        public static readonly int PAYMENT_SOCIAL_MOBILE = 3;
        public static readonly int PAYMENT_SOCIAL_QRCODE = 4;
        public static readonly int PAYMENT_UPI_QRCODE = 5;
        public static readonly int PAYMENT_MERCHANT_WALLET = 6;
        public static readonly int PAYMENT_MONPAY = 7;

        [DllImport("ez-connector.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
        public static extern string version();
        [DllImport("ez-connector.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
        public static extern string logon();
        [DllImport("ez-connector.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
        public static extern string payment(double amount, bool skipPrint, int payment, int defaultQrPayment, string extra = "");
        [DllImport("ez-connector.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
        public static extern string refund(string traceno, bool skipPrint);
        [DllImport("ez-connector.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
        public static extern string settlement();
    }

    public class PurchaseResponse
    {
        public bool succeed { get; set; }
        public string message { get; set; }
        public string payment { get; set; }
        public double amount { get; set; }
        public string systemRef { get; set; }
        public string traceno { get; set; }
        public string approveCode { get; set; }
        public string maskedPAN { get; set; }
        public string transactionAt { get; set; }
        public string merchantId { get; set; }
        public string terminalId { get; set; }
        public string extra { get; set; }
    }
}
// Ашиглах
string result = EzConnector.payment(amount, true, EzConnector.PAYMENT_NONE, EzConnector.PAYMENT_MONPAY);
byte[] bytes = Encoding.GetEncoding(1252).GetBytes(result);
string encodingFixedResult = Encoding.UTF8.GetString(bytes);
PurchaseResponse response = JsonConvert.DeserializeObject<PurchaseResponse>(encodingFixedResult);

```

[==============]

## 1.4. Ez-Connector хувилбар шалгах

```
string version()
```

Тус сангийн сүүлийн хувилбарыг ашиглаж байгаа эсэхийг тулгахад ашиглана.

[==============]

### 1.4.1. Хүсэлтийн хариу

Хүсэлтийн хариуд Ez-Connector сангийн хувилбар ирнэ.

[==============]

## 1.5. Холболт шалгах

```
string logon()
```

Кассын програм ажиллаж эхлэх үед энэ үйлдлийг дуудна. Энэ үйлдлийг хийснээр `Ez-Connector` банктай холбогдон key-гээ шинэчилдэг.

[==============]

### 1.5.1. Хүсэлтийн хариу

Хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

| Attribute | Төрөл   | Тайлбар                                        | Заавал |
| --------: | ------- | :--------------------------------------------- | ------ |
| `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
| `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |

[==============]

## 1.6. Гүйлгээ хийх

```
string payment(double amount, bool skipPrint, int payment, int defaultQrPayment, string extra)
```

Худалдан авалт буюу гүйлгээ хийх үед ашиглана.

[==============]

### 1.6.1. Гүйлгээ хийхэд шаардлагатай параметрүүд

| Параметр         | Төрөл  | Тайлбар                                                                                                  |
| ---------------- | ------ | -------------------------------------------------------------------------------------------------------- |
| amount           | double | Гүйлгээ хийх дүн                                                                                         |
| skipPrint        | bool   | Баримт хэвлэх эсэх                                                                                       |
| payment          | int    | Төлбөрийн хэрэгслийг сонгож илгээх                                                                       |
| defaultQrPayment | int    | Төлбөрийн сонголтын хажууд default-р гарч ирэх төлбөрийн хэрэгсэл, зөвхөн 4,7 утгуудаас сонгон дамжуулах |
| extra            | string | Банктай нэмэлт системийн холболт хийх үед дамжуулах утга, банктай урьдчилан тохиролцох шаардлагатай      |

[==============]

### 1.6.2. Төлбөрийн хэрэгслийн утга

| Утга | Төрөл                        | Тайлбар                                                         |
| ---- | ---------------------------- | --------------------------------------------------------------- |
| 0    | Төлбөрийн хэрэгсэл сонгоогүй | Боломжит бүх төлбөрийн хэрэгслийн сонголтууд пос дээр гарч ирнэ |
| 1    | Карт                         | Зөвхөн картын гүйлгээ хийх үед                                  |
| 3    | SocialPay - mobile           | Зөвхөн SocialPay - утасны дугаар ашиглан гүйлгээ хийх үед       |
| 4    | SocialPay - QR               | Зөвхөн SocialPay - QR код ашиглан гүйлгээ хийх үед              |
| 7    | Monpay - QR                  | Зөвхөн Monpay - QR код ашиглан гүйлгээ хийх үед                 |

[==============]

### 1.6.3. Гүйлгээний хүсэлтийн хариу

Гүйлгээний хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|       Attribute | Төрөл   | Тайлбар                                        | Заавал |
| --------------: | ------- | :--------------------------------------------- | ------ |
|       `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|       `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|        `amount` | double  | Гүйлгээ хийсэн дүн                             | Үгүй   |
|       `payment` | string  | Гүйлгээ хийгдсэн төлбөрийн хэрэгсэл            | Үгүй   |
|     `systemRef` | string  | Гүйлгээний ref дугаар                          | Үгүй   |
|       `traceno` | string  | Гүйлгээний trace дугаар                        | Үгүй   |
|   `approveCode` | string  | Зөвшөөрлийн код                                | Үгүй   |
|     `maskedPAN` | string  | Картын маскалсан дугаар                        | Үгүй   |
| `transactionAt` | string  | Гүйлгээ хийгдсэн огноо                         | Үгүй   |
|    `merchantId` | string  | Мерчант дугаар                                 | Үгүй   |
|    `terminalId` | string  | Терминал дугаар                                | Үгүй   |

## 1.7. Буцаалт хийх

```
string refund(string traceno, bool skipPrint)
```

Худалдан авалт буцаах үед гүйлгээ буцаалт хийх үед ашиглана.

[==============]

### 1.7.1. Буцаалт хийхэд шаардлагатай параметрүүд

| Параметр  | Төрөл  | Тайлбар                 |
| --------- | ------ | ----------------------- |
| traceno   | string | Гүйлгээний trace дугаар |
| skipPrint | bool   | Баримт хэвлэх эсэх      |

[==============]

### 1.7.2. Буцаалтын хүсэлтийн хариу

Буцаалтын хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|       Attribute | Төрөл   | Тайлбар                                        | Заавал |
| --------------: | ------- | :--------------------------------------------- | ------ |
|       `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|       `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|        `amount` | double  | Буцаалтын гүйлгээний дүн                       | Үгүй   |
|       `payment` | string  | Төлбөрийн хэрэгсэл                             | Үгүй   |
|     `systemRef` | string  | Буцаалтын гүйлгээний ref дугаар                | Үгүй   |
|       `traceno` | string  | Буцаалтын гүйлгээний trace дугаар              | Үгүй   |
|   `origTraceno` | string  | Гүйлгээний trace дугаар                        | Үгүй   |
|   `approveCode` | string  | Зөвшөөрлийн код                                | Үгүй   |
|     `maskedPAN` | string  | Картын маскалсан дугаар                        | Үгүй   |
| `transactionAt` | string  | Буцаалтын гүйлгээ хийгдсэн огноо               | Үгүй   |
|    `merchantId` | string  | Мерчант дугаар                                 | Үгүй   |
|    `terminalId` | string  | Терминал дугаар                                | Үгүй   |

## 1.8. Өндөрлөгөө хийх

```
string settlement(bool skipPrint)
```

Энэ үйлдлийг дуудсанаар `Ez-Connector` тухайн өдөр хийгдсэн бүх гүйлгээг банк руу дамжуулж баталгаажуулна. Ингэснээр дараа өдөр нь байгууллагын дансанд тухайн өдрийн бүх гүйлгээний орлого шилжиж орно.

[==============]

### 1.8.1. Өндөрлөгөө хийхэд шаардлагатай параметрүүд

| Параметр  | Төрөл | Тайлбар            |
| --------- | ----- | ------------------ |
| skipPrint | bool  | Баримт хэвлэх эсэх |

[==============]

### 1.8.2. Өндөрлөгөө хүсэлтийн хариу

Буцаалтын хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|          Attribute | Төрөл   | Тайлбар                                        | Заавал |
| -----------------: | ------- | :--------------------------------------------- | ------ |
|          `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|          `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|           `amount` | double  | Нийт гүйлгээний дүн                            | Үгүй   |
|            `count` | int     | Нийт гүйлгээний тоо                            | Үгүй   |
| `settlementAmount` | double  | Дансанд шилжих дүн                             | Үгүй   |
|               `no` | int     | Өндөрлөгөөний дугаар                           | Үгүй   |
|             `date` | string  | Буцаалтын гүйлгээ хийгдсэн огноо               | Үгүй   |
|       `merchantId` | string  | Мерчант дугаар                                 | Үгүй   |
|       `terminalId` | string  | Терминал дугаар                                | Үгүй   |

[==============]

## 1.9. Цахим иргэний үнэмлэхийн мэдээлэл унших

```
string getIDCard()
```

Цахим иргэний үнэмлэхний мэдээлэл унших тохиолдолд дуудаж ашиглана.

[==============]

### 1.9.1. Цахим иргэний үнэмлэхийн мэдээлэл уншихад шаардлагатай параметрүүд

Байхгүй

[==============]

### 1.9.2. Цахим иргэний үнэмлэхийн мэдээлэл унших хүсэлтийн хариу

Хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|     Attribute | Төрөл   | Тайлбар                                        | Заавал |
| ------------: | ------- | :--------------------------------------------- | ------ |
|     `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|     `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|     `surname` | string  | Овог                                           | Үгүй   |
|   `givenName` | string  | Нэр                                            | Үгүй   |
|  `registerno` | string  | Регистр дугаар                                 | Үгүй   |
| `dateOfBirth` | string  | Төрсөн огноо                                   | Үгүй   |
|         `sex` | string  | Хүйс                                           | Үгүй   |

[==============]

## 1.10. VAS Гүйлгээ хийх

```
string vasPayment(double amount, bool skipPrint, int payment, string referralcode)
```

VAS үйлчилгээний худалдан авалт буюу гүйлгээ хийх үед ашиглана.

[==============]

### 1.10.1. VAS Гүйлгээ хийхэд шаардлагатай параметрүүд

| Параметр     | Төрөл  | Тайлбар                                                      |
| ------------ | ------ | ------------------------------------------------------------ |
| amount       | double | Гүйлгээ хийх дүн                                             |
| skipPrint    | bool   | Баримт хэвлэх эсэх                                           |
| payment      | int    | Төлбөрийн хэрэгслийг сонгож илгээх                           |
| referralcode | string | Холболт хийж буй талаас тус гүйлгээтэй холбоотой лавлах утга |

[==============]

### 1.10.2. Төлбөрийн хэрэгслийн утга

| Утга | Төрөл                        | Тайлбар                                                         |
| ---- | ---------------------------- | --------------------------------------------------------------- |
| 0    | Төлбөрийн хэрэгсэл сонгоогүй | Боломжит бүх төлбөрийн хэрэгслийн сонголтууд пос дээр гарч ирнэ |
| 1    | Карт                         | Зөвхөн картын гүйлгээ хийх үед                                  |
| 3    | SocialPay - mobile           | Зөвхөн SocialPay - утасны дугаар ашиглан гүйлгээ хийх үед       |
| 4    | SocialPay - QR               | Зөвхөн SocialPay - QR код ашиглан гүйлгээ хийх үед              |
| 7    | Monpay - QR                  | Зөвхөн Monpay - QR код ашиглан гүйлгээ хийх үед                 |

[==============]

### 1.10.3. VAS Гүйлгээний хүсэлтийн хариу

Гүйлгээний хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|         Attribute | Төрөл   | Тайлбар                                        | Заавал |
| ----------------: | ------- | :--------------------------------------------- | ------ |
|         `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|         `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|          `amount` | double  | Гүйлгээ хийсэн дүн                             | Үгүй   |
|         `payment` | string  | Гүйлгээ хийгдсэн төлбөрийн хэрэгсэл            | Үгүй   |
|       `systemRef` | string  | Гүйлгээний ref дугаар                          | Үгүй   |
|         `traceno` | string  | Гүйлгээний trace дугаар                        | Үгүй   |
|     `approveCode` | string  | Зөвшөөрлийн код                                | Үгүй   |
|       `maskedPAN` | string  | Картын маскалсан дугаар                        | Үгүй   |
|   `transactionAt` | string  | Гүйлгээ хийгдсэн огноо                         | Үгүй   |
|      `merchantId` | string  | Мерчант дугаар                                 | Үгүй   |
|      `terminalId` | string  | Терминал дугаар                                | Үгүй   |
| `ezTransactionId` | string  | Гүйлгээний ID дугаар                           | Үгүй   |

[==============]

## 1.11. VAS гүйлгээний Буцаалт хийх

```
string vasRefund(string traceno, bool skipPrint, string referralcode)
```

VAS Худалдан авалт буцаах үед уг гүйлгээг буцаалт хийх үед ашиглана.

[==============]

### 1.11.1. Буцаалт хийхэд шаардлагатай параметрүүд

| Параметр     | Төрөл  | Тайлбар                                        |
| ------------ | ------ | ---------------------------------------------- |
| traceno      | string | Гүйлгээний trace дугаар                        |
| skipPrint    | bool   | Баримт хэвлэх эсэх                             |
| referralcode | string | vasPayment хийх үед дамжуулсан утгыг дамжуулна |

[==============]

### 1.11.2. VAS Буцаалтын хүсэлтийн хариу

Буцаалтын хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|       Attribute | Төрөл   | Тайлбар                                        | Заавал |
| --------------: | ------- | :--------------------------------------------- | ------ |
|       `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|       `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|        `amount` | double  | Буцаалтын гүйлгээний дүн                       | Үгүй   |
|       `payment` | string  | Төлбөрийн хэрэгсэл                             | Үгүй   |
|     `systemRef` | string  | Буцаалтын гүйлгээний ref дугаар                | Үгүй   |
|       `traceno` | string  | Буцаалтын гүйлгээний trace дугаар              | Үгүй   |
|   `origTraceno` | string  | Гүйлгээний trace дугаар                        | Үгүй   |
|   `approveCode` | string  | Зөвшөөрлийн код                                | Үгүй   |
|     `maskedPAN` | string  | Картын маскалсан дугаар                        | Үгүй   |
| `transactionAt` | string  | Буцаалтын гүйлгээ хийгдсэн огноо               | Үгүй   |
|    `merchantId` | string  | Мерчант дугаар                                 | Үгүй   |
|    `terminalId` | string  | Терминал дугаар                                | Үгүй   |

[==============]

## 1.12. Umoney картын үлдэгдэл

```
string umoneyBalance()
```

Umoney картын үлдэгдэл буцаана.

[==============]

### 1.12.1. Хүсэлтийн хариу

| Attribute | Төрөл   | Тайлбар                                        | Заавал |
| --------: | ------- | :--------------------------------------------- | ------ |
| `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
| `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|  `cardno` | string  | Картын дугаар                                  | Үгүй   |
| `balance` | double  | Картын үлдэгдэл                                | Үгүй   |

[==============]

## 1.13. Umoney карт цэнэглэх

```
string umoneyTopup(double amount, string referralcode, string ezTransactionId)
```

Umoney карт цэнэглэх үед ашиглана.

[==============]

### 1.13.1. Umoney карт цэнэглэхэд шаардлагатай параметрүүд

| Параметр        | Төрөл  | Тайлбар                                                                                                                         |
| --------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------- |
| amount          | double | Цэнэглэлт хийх дүн                                                                                                              |
| referralcode    | string | Холболт хийж буй талаас тус цэнэглэлттэй холбоотой reference утга                                                               |
| ezTransactionId | string | vasPayment хариунд ирсэн ezTransactionId талбарын утгыг дамжуулна, vasPayment гүйлгээ хийгээгүй үед null эсвэл хоосон дамжуулах |

[==============]

### 1.13.2. Umoney карт цэнэглэх хүсэлтийн хариу

Гүйлгээний хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|     Attribute | Төрөл   | Тайлбар                                        | Заавал |
| ------------: | ------- | :--------------------------------------------- | ------ |
|     `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|     `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|      `cardno` | string  | Картын дугаар                                  | Үгүй   |
|      `amount` | double  | Карт цэнэглэсэн дүн                            | Үгүй   |
|  `preBalance` | double  | Картын өмнөх үлдэгдэл                          | Үгүй   |
| `postBalance` | double  | Картын цэнэглэлтийн дараах үлдэгдэл            | Үгүй   |

[==============]

## 1.14. Umoney карт сүүлийн цэнэглэлт буцаах

```
string umoneyRefund()
```

Umoney карт сүүлийн цэнэглэлт буцаах үед ашиглана.

[==============]

### 1.14.1. Umoney карт сүүлийн цэнэглэлт буцаалтын хүсэлтийн хариу

Гүйлгээний хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|      Attribute | Төрөл   | Тайлбар                                        | Заавал |
| -------------: | ------- | :--------------------------------------------- | ------ |
|      `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|      `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|       `cardno` | string  | Картын дугаар                                  | Үгүй   |
|       `amount` | double  | Картын буцаалтын дүн                           | Үгүй   |
|   `preBalance` | double  | Картын өмнөх үлдэгдэл                          | Үгүй   |
|  `postBalance` | double  | Картын цэнэглэлтийн дараах үлдэгдэл            | Үгүй   |
| `referralcode` | string  | umoneyTopup хүсэлтэнд явуулсан утгыг буцаана   | Үгүй   |

[==============]

## 1.15. Баримт хэвлэх

```
string print(string printData)
```

POS төхөөрөмж дээр баримт хэвлэх үед ашиглана.

### 1.15.1. Жишээ код

[C# жишээ код - Татах](/apis/files/printer_classes.rar)

[==============]

## 1.16. Bill Payment - Гүйлгээ хийх

```
string billPayment(double amount, double fee, bool skipPrint)
```

Bill Payment гүйлгээ хийх үед ашиглана.

[==============]

### 1.16.1. Bill Payment - Гүйлгээ хийхэд шаардлагатай параметрүүд

| Параметр  | Төрөл  | Тайлбар                             |
| --------- | ------ | ----------------------------------- |
| amount    | double | Нийт дүн (Үндсэн дүн + Шимтгэл дүн) |
| fee       | double | Шимтгэлийн дүн                      |
| skipPrint | bool   | Баримт хэвлэх эсэх                  |

[==============]

### Bill Payment - 1.16.2. Гүйлгээний хүсэлтийн хариу

Гүйлгээний хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|       Attribute | Төрөл   | Тайлбар                                        | Заавал |
| --------------: | ------- | :--------------------------------------------- | ------ |
|       `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|       `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|        `amount` | double  | Гүйлгээ хийсэн дүн                             | Үгүй   |
|       `payment` | string  | Гүйлгээ хийгдсэн төлбөрийн хэрэгсэл            | Үгүй   |
|     `systemRef` | string  | Гүйлгээний ref дугаар                          | Үгүй   |
|       `traceno` | string  | Гүйлгээний trace дугаар                        | Үгүй   |
|   `approveCode` | string  | Зөвшөөрлийн код                                | Үгүй   |
|     `maskedPAN` | string  | Картын маскалсан дугаар                        | Үгүй   |
| `transactionAt` | string  | Гүйлгээ хийгдсэн огноо                         | Үгүй   |
|    `merchantId` | string  | Мерчант дугаар                                 | Үгүй   |
|    `terminalId` | string  | Терминал дугаар                                | Үгүй   |
