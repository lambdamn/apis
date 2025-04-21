# Ez-Connector

[TOC]

## 1. Introduction

**Ez-Connector** library is designed to integrate `Cashier` systems into `Golomt POS`. And this manual contains instructions on how to make that connection

## 1.1. Sequence Diagram

```seq
Client->EzConnector.dll: Transaction request
EzConnector.dll->Golomt POS: Transaction request
Note right of Golomt POS: Process Transaction
Golomt POS->EzConnector.dll: Transaction response
EzConnector.dll->Client: Transaction response
```

## 1.2. Install Driver for POS device

| Model of POS | USB Driver                             |
| -----------: | :------------------------------------- |
|      NewLand | [Download](/apis/files/usb_driver.rar) |

## 1.3. Linking the Ez-Connector.dll library

Version - 0.0.12

[Ez-Connector.dll (64bit) - Download](/apis/files/64bit/ez-connector_x64.dll)

[Ez-Connector.dll (32bit) - Download](/apis/files/32bit/ez-connector_x86.dll)

**Java code**

```
import com.sun.jna.Callback;
import com.sun.jna.Library;
import com.sun.jna.Native;

public class EzConnectorDemo {
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
    String result = lib.payment(2, true, 1, 0, "");
    System.out.println(result);
    String version = lib.version();
    System.out.println(version);
  }
}
```

**C# code**

```
using System.Runtime.InteropServices;

namespace EzConnectorDemo
{
    public class EzConnector
    {
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
        public string discountTitle { get; set; }
        public double discountAmount { get; set; }
    }
}
// Use
string result = EzConnector.payment(amount, true, 1, 0);
byte[] bytes = Encoding.GetEncoding(1252).GetBytes(result);
string encodingFixedResult = Encoding.UTF8.GetString(bytes);
PurchaseResponse response = JsonConvert.DeserializeObject<PurchaseResponse>(encodingFixedResult);

```

[==============]

> Before you start integration please check belows carefully

- Make sure ezpay app is opened on POS terminal
- Turn of screen timeout. Settings -> Display -> Sleep -> Never
- Connect USB-A cable into USB port of PC

[==============]

## 1.4. Check the Ez-Connector version

```
string version()
```

Used to enforce whether the latest version of the library is in use.

[==============]

### 1.4.1. Response

A version of the Ez-Connector library will come in response to the request.

[==============]

## 1.5. Check connection

```
string logon()
```

This function is called when the checkout application starts. By doing this, `Ez-Connector` the key is updated by contacting the bank.

[==============]

### 1.5.1. Response

`string` The value received in response to a request and it has the following `json` format

| Attribute | Type    | Description                                       | Required |
| --------: | ------- | :------------------------------------------------ | -------- |
| `succeed` | boolean | Whether the request was successful                | Yes      |
| `message` | string  | A description of the error when the request fails | No       |

[==============]

## 1.6. Make a transaction

```
string payment(double amount, true, 1, 0, "") //Card payment
```

Used when making a purchase or transaction.

[==============]

### 1.6.1. Parameters for transaction request

| Parameter | Type   | Default value & Description |
| --------- | ------ | --------------------------- |
| amount    | double | Transaction amount          |

[==============]

### 1.6.2. Response of payment transaction

`string` The value received in response to a `payment` transaction request and it has the following `json` format

|        Attribute | Type    | Description                                       | Required |
| ---------------: | ------- | :------------------------------------------------ | -------- |
|        `succeed` | boolean | Whether the request was successful                | Yes      |
|        `message` | string  | A description of the error when the request fails | No       |
|         `amount` | double  | Transaction amount                                | No       |
|        `payment` | string  | The payment method used for the transaction       | No       |
|      `systemRef` | string  | Transaction ref no                                | No       |
|        `traceno` | string  | Transaction trace number                          | No       |
|    `approveCode` | string  | Approval code                                     | No       |
|      `maskedPAN` | string  | Masked card number                                | No       |
|  `transactionAt` | string  | Date of transaction                               | No       |
|     `merchantId` | string  | Merchant ID                                       | No       |
|     `terminalId` | string  | Terminal ID                                       | No       |
|  `discountTitle` | string  | Discount title                                    | No       |
| `discountAmount` | double  | Discount amount                                   | No       |

### 1.6.3. Verify and confirm whether the transaction was successful.

- Check whether the amount received in the "amount" field matches the requested amount.
- Verify whether the "maskedPAN" field has a value.

## 1.7. Make a refund transaction

```
string refund(string traceno, true)
```

Used when a purchase is returned.

[==============]

### 1.7.1. Parameters required for `refund`

| Parameter | Type   | Default value & Description |
| --------- | ------ | --------------------------- |
| traceno   | string | Transaction trace number    |

[==============]

### 1.7.2. Response of `refund` transaction

`string` The value received in response to a `refund` transaction request and it has the following `json` format

|       Attribute | Type    | Description                                       | Required |
| --------------: | ------- | :------------------------------------------------ | -------- |
|       `succeed` | boolean | Whether the request was successful                | Yes      |
|       `message` | string  | A description of the error when the request fails | No       |
|        `amount` | double  | Transaction amount                                | No       |
|       `payment` | string  | The payment method used for the transaction       | No       |
|     `systemRef` | string  | Transaction ref no                                | No       |
|       `traceno` | string  | Refund trace number                               | No       |
|   `origTraceno` | string  | Transaction trace number                          | No       |
|   `approveCode` | string  | Approval code                                     | No       |
|     `maskedPAN` | string  | Masked card number                                | No       |
| `transactionAt` | string  | Date of transaction                               | No       |
|    `merchantId` | string  | Merchant ID                                       | No       |
|    `terminalId` | string  | Terminal ID                                       | No       |

## 1.8. Make a settlement

```
string settlement(true)
```

By calling this function of `Ez-Connector` all transactions made on that day will be passed to the bank for confirmation. In this way, the following day, the income of all transactions of that day will be transferred to the organization's account

[==============]

### 1.8.1. Response of `settlement` transaction

`string` The value received in response to a `settlement` transaction request and it has the following `json` format

|          Attribute | Type    | Description                                       | Required |
| -----------------: | ------- | :------------------------------------------------ | -------- |
|          `succeed` | boolean | Whether the request was successful                | Yes      |
|          `message` | string  | A description of the error when the request fails | No       |
|           `amount` | double  | Total transaction amount                          | No       |
|            `count` | int     | Total number of transactions                      | No       |
| `settlementAmount` | double  | Amount to be transferred to the account           | No       |
|               `no` | int     | Settlement no                                     | No       |
|             `date` | string  | Date of settlement                                | No       |
|       `merchantId` | string  | Merchant ID                                       | No       |
|       `terminalId` | string  | Terminal ID                                       | No       |

[==============]
