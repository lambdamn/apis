# Android Deeplink холболт

[TOC]

## 1. Танилцуулга

Тус гарын авлага нь `Анпройд` аппликейшнээс `Голомт пос`-той **Deeplink** холболт хийх талаар зааварчилгааг агуулсан болно.

## 1.1. Sequence Diagram

```seq
Client app->Golomt Payment app: Гүйлгээний хүсэлт\n(Deeplink)
Note right of Golomt Payment app: Төлбөр төлөлт\nхийгдэнэ
Golomt Payment app->Client app: Гүйлгээний хүсэлтийн\nхариу
```

[==============]

## 1.2. Гүйлгээ хийх

Худалдан авалт буюу гүйлгээ хийх үед доорх байдлаар deeplink холболтыг дуудна.

```
val uri = "golomt://payment/transaction?transType=Sale&amount=${amount}&paymentType=${paymentType}&skipPrint=${skipPrint}";
val intent = Intent()
intent.data = Uri.parse(uri)
intent.action = "android.intent.action.GOLOMT.PAYMENT"
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_CLEAR_TASK)
mStartForResult.launch(intent)
```

[==============]

### 1.2.1. Гүйлгээ хийхэд шаардлагатай параметрүүд

| Параметр    | Төрөл  | Тайлбар                                                                                 |
| ----------- | ------ | --------------------------------------------------------------------------------------- |
| transType   | string | зөвхөн `Sale` утга дамжуулна                                                            |
| amount      | int    | Гүйлгээ хийх дүнг 100р үржүүлж `int` төрөл рүү хөрвүүлэн дамжуулах. Жишээ 1.59\*100=159 |
| paymentType | int    | Төлбөрийн хэрэгслийг сонгож илгээх                                                      |
| skipPrint   | bool   | Баримт хэвлэх эсэх                                                                      |

[==============]

### 1.2.2. Төлбөрийн хэрэгслийн утга

| Утга | Төрөл                        | Тайлбар                                                         |
| ---- | ---------------------------- | --------------------------------------------------------------- |
|      | Төлбөрийн хэрэгсэл сонгоогүй | Боломжит бүх төлбөрийн хэрэгслийн сонголтууд пос дээр гарч ирнэ |
| 10   | Карт                         | Картын гүйлгээ хийх үед                                         |
| 30   | SocialPay                    | SocialPay ашиглан гүйлгээ хийх үед                              |

[==============]

### 1.2.3. Гүйлгээний хүсэлтийн хариу

```
    private var mStartForResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result: ActivityResult ->
        if (result.resultCode == Activity.RESULT_OK) {
            result.data?.extras?.let { bundle ->
                val merchantName = bundle.getString("merchantName")
                val mid = bundle.getString("mid")
                val tid = bundle.getString("tid")
                val paymentType = bundle.getInt("paymentType")
                val amount = bundle.getLong("amount")
                val total = bundle.getLong("total")
                val discount = bundle.getLong("discount")
                val date = bundle.getString("date")
                val time = bundle.getString("time")
                val batchno = bundle.getString("batchNo")
                val traceno = bundle.getString("traceNo")
                val resultCode = bundle.getString("resultCode")
                val message = bundle.getString("message")
                val cardNo = bundle.getString("cardNo")
                val referenceNo = bundle.getString("referenceNo")
                val authCode = bundle.getString("authCode")

                // resultCode = "00" нөхцлийг шалгах
                if(resultCode == "00"){
                  // Амжилттай
                }
                else {
                  // Амжилтгүй
                }
            } ?: run {
                // Амжилтгүй
            }
        } else {
            // Амжилтгүй
            result.data?.extras?.let { bundle ->
                bundle.getString("message")?.let {
                    Log.d("EzDemo", "error message: $it")
                }
            } ?: run {
                Log.d("EzDemo", "Payment failed")
            }
        }
    }
```

Гүйлгээний хүсэлтийн хариуд ирж байгааг дээрх кодын дагуу хүлээж авах ба `result.data?.extras` нь дараах бүтэцтэй байна.

|      Attribute | Төрөл  | Тайлбар                                        | Заавал |
| -------------: | ------ | :--------------------------------------------- | ------ |
|   `resultCode` | string | `00` ирсэн үед гүйлгээ амжилттай               | Үгүй   |
|      `message` | string | Хүсэлт амжилтгүй үед алдааг илэрхийлэх тайлбар | Үгүй   |
|       `amount` | int    | Гүйлгээ хийсэн дүн                             | Үгүй   |
|  `paymentType` | int    | Гүйлгээ хийгдсэн төлбөрийн хэрэгсэл            | Үгүй   |
|  `referenceNo` | string | Гүйлгээний ref дугаар                          | Үгүй   |
|      `traceNo` | string | Гүйлгээний trace дугаар                        | Үгүй   |
|     `authCode` | string | Зөвшөөрлийн код                                | Үгүй   |
|       `cardNo` | string | Картын маскалсан дугаар                        | Үгүй   |
|         `date` | string | Гүйлгээ хийгдсэн өдөр                          | Үгүй   |
|         `time` | string | Гүйлгээ хийгдсэн цаг, минут                    | Үгүй   |
|          `mid` | string | Мерчант дугаар                                 | Үгүй   |
|          `tid` | string | Терминал дугаар                                | Үгүй   |
| `merchantName` | string | Мерчант нэр                                    | Үгүй   |

[==============]

## 1.3. Өндөрлөгөө хийх

```
val uri = "golomt://payment/transaction?transType=Settle";
val intent = Intent()
intent.data = Uri.parse(uri)
intent.action = "android.intent.action.GOLOMT.PAYMENT"
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_CLEAR_TASK)
mStartSettlementForResult.launch(intent)
```

Энэ үйлдлийг дуудсанаар тухайн өдөр хийгдсэн бүх гүйлгээг банк руу дамжуулж баталгаажуулна. Ингэснээр дараа өдөр нь байгууллагын дансанд тухайн өдрийн бүх гүйлгээний орлого шилжиж орно.

[==============]

### 1.3.1. Өндөрлөгөө хийхэд шаардлагатай параметрүүд

| Параметр  | Төрөл  | Тайлбар                           |
| --------- | ------ | --------------------------------- |
| transType | string | Зөвхөн `Settle` утгатай дамжуулах |

[==============]

### 1.3.2. Өндөрлөгөө хүсэлтийн хариу

```
    private var mStartSettlementForResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result: ActivityResult ->
        binding.settlementButton.visibility = View.VISIBLE
        if (result.resultCode == Activity.RESULT_OK) {
            result.data?.extras?.let { bundle ->
                val resultCode = bundle.getString("resultCode")
                // resultCode = "00" нөхцлийг шалгах
                if(resultCode == "00"){
                  // Амжилттай
                }
                else {
                  // Амжилтгүй
                }
            } ?: run {
                // Амжилтгүй
            }
        } else {
            // Амжилтгүй
            result.data?.extras?.let { bundle ->
                bundle.getString("message")?.let {
                    Log.d("EzDemo", "error message: $it")
                }
            }
        }
    }

```

Гүйлгээний хүсэлтийн хариуд ирж байгааг дээрх кодын дагуу хүлээж авах ба `result.data?.extras` нь дараах бүтэцтэй байна.

|    Attribute | Төрөл  | Тайлбар                                           | Заавал |
| -----------: | ------ | :------------------------------------------------ | ------ |
| `resultCode` | string | `00` ирсэн үед гүйлгээ амжилттай хийгдсэнд тооцно | Үгүй   |

[==============]

## 1.4. Баримт хэвлэх

```
val intent = Intent()
intent.data = Uri.parse("golomt://payment/transaction?transType=Print&printData=${printData}")
intent.action = "android.intent.action.GOLOMT.PAYMENT"
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_CLEAR_TASK)
mStartForPrintResult.launch(intent)
```

POS төхөөрөмж дээр баримт хэвлэх үед ашиглана. Баримт хэвлэх талаар дэлгэрэнгүй жишээ кодыг Демо кодноос харна уу.

### 1.4.1. Баримт хэвлэхэд шаардлагатай параметрүүд

| Параметр  | Төрөл  | Тайлбар                                          |
| --------- | ------ | ------------------------------------------------ |
| transType | string | Зөвхөн `Print` утгатай дамжуулах                 |
| printData | string | Хэвлэх мэдээллийг `json` форматтайгаар дамжуулна |

## 1.5. Жишээ код

[Жишээ код - Татах](/apis/files/EzDemo.zip)

[==============]
