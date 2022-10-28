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

[USB Driver - Татах](/files/usb_driver.rar)

## 1.3. Ez-Connector.dll санг системд оруулах

[Ez-Connector.dll - Татах](/files/ez-connector.dll)

**Жава код**

```
import com.sun.jna.Callback;
import com.sun.jna.Library;
import com.sun.jna.Native;

public class EzConnectorDemo {
  public interface EzConnector extends Library {
    public String doPayment(double amount);
    public String getCard(double amount);
  }

  public static void main(String[] args) {
    EzConnector lib =
        (EzConnector)Native.loadLibrary("ez-connector.dll", EzConnector.class);
    String result = lib.doPayment(2);
    System.out.println(result);
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
        [DllImport("ez-connector.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
        public static extern string doPayment(double amount);
    }

    public class PurchaseResponse
    {
        public Boolean succeed { get; set; }
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
string result = EzConnector.doPayment(amount);
byte[] bytes = Encoding.GetEncoding(1252).GetBytes(result);
string encodingFixedResult = Encoding.UTF8.GetString(bytes);
PurchaseResponse response = JsonConvert.DeserializeObject<PurchaseResponse>(encodingFixedResult);

```

[==============]

## 1.4. Гүйлгээ хийх

```
string doPayment(double amount)
```

[==============]

### 1.4.1. Гүйлгээ хийхэд шаардлагатай параметрүүд

| Параметр | Төрөл  | Тайлбар          |
| -------- | ------ | ---------------- |
| amount   | double | Гүйлгээ хийх дүн |

[==============]

### 1.4.2. Гүйлгээний хүсэлтийн хариу

Гүйлгээний хүсэлтийн хариуд ирж байгаа `string` утга нь `json` форматтай байх ба дараах бүтэцтэй байна.

|       Attribute | Төрөл   | Тайлбар                                        | Заавал |
| --------------: | ------- | :--------------------------------------------- | ------ |
|       `succeed` | boolean | Хүсэлт амжилттай болсон эсэх                   | Тийм   |
|       `message` | string  | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Тийм   |
|        `amount` | double  | Гүйлгээ хийсэн дүн                             | Үгүй   |
|       `payment` | string  | Гүйлгээ хийгдсэн төлбөрийн хэрэгсэл            | Үгүй   |
|     `systemRef` | string  | Гүйлгээний ref дугаар                          | Үгүй   |
|       `traceno` | string  | Гүйлгээний trace дугаар                        | Үгүй   |
|   `approveCode` | string  | Зөвшөөрлийн код                                | Үгүй   |
|     `maskedPAN` | string  | Картын маскалсан дугаар                        | Үгүй   |
| `transactionAt` | string  | Гүйлгээ хийгдсэн огноо                         | Үгүй   |
|    `merchantId` | string  | Мерчант дугаар                                 | Үгүй   |
|    `terminalId` | string  | Терминал дугаар                                | Үгүй   |
