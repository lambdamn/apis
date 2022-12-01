# Ez-Connector

[TOC]

## 1. Танилцуулга

**Ez-Connector** сан нь `Касс`-ын системийг `Изи Пэй пос`-той холболт хийх зориулалттай бөгөөд
тус гарын авлага нь уг холболтыг хэрхэн хийх талаар зааварчилгааг агуулсан болно.

## 1.1. Sequence Diagram

```seq
Client->EzConnector.dll: Гүйлгээний хүсэлт
EzConnector.dll->Ezpay POS: Гүйлгээний хүсэлт
Note right of Ezpay POS: Төлбөр төлөлт\nхийгдэнэ
Ezpay POS->EzConnector.dll: Гүйлгээний хүсэлтийн\nхариу
EzConnector.dll->Client: Гүйлгээний хүсэлтийн\nхариу
```

## 1.2. Пос төхөөрөмжийн Driver суулгах

[USB Driver - Татах](/apis/files/usb_driver.rar)

## 1.3. Ez-Connector.dll санг холбох

Хувилбар - 0.0.3

[Ez-Connector.dll - Татах](/apis/files/ez-connector.dll)

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
    public String payment(double amount, boolean skipPrint, int payment, int defaultQrPayment);
    public String refund(String traceno, boolean skipPrint);
    public String settlement(boolean skipPrint);
    public String version();
  }

  public static void main(String[] args) {
    EzConnector lib =
        (EzConnector)Native.loadLibrary("ez-connector.dll", EzConnector.class);
    String result = lib.payment(2, true, PAYMENT_NONE, PAYMENT_MONPAY);
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
        public static extern string payment(double amount, bool skipPrint, int payment, int defaultQrPayment);
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
string payment(double amount, bool skipPrint, int payment, int defaultQrPayment)
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
