# 《XXX将》支付的那些事

## IOS

### 大体流程
![支付流程](ios支付.jpg)

1. 用户在客户端点击支付，请求发送至apple服务器

* 支付成功后，apple服务器会返回一个购买凭证A给用户客户端

* 用户客户端将支付凭证A发送给游戏服务器

* 游戏服务器拿到A后，按照一定格式构造数据串，通过https服务向apple服务器请求数据验证

* apple服务器验证完成后，会返回一个JSON格式数据B给游戏服务器

* 游戏服务器检查B的各个Value，数据合法，则给予用户商品并记录该订单的支付状态

购买凭证A就是一个加密（天晓得什么加密方式）后的Base64格式数据，一个实例如下（代码段）：
	
	ewoJInNpZ25hdHVyZSIgPSAiQXFkaGlxb1pPVWZyZjFrNEtCQ25UbDF
	
apple验证完成返回的一个JSON格式数据，可以理解为解密后的明文，一个实例如下：

	  {
    	"receipt": {
        	"original_purchase_date_pst": "2013-12-13 02:29:07 America/Los_Angeles",
        	"purchase_date_ms": "1386930547545",
        	"unique_identifier": "a6a63052abd73a9183fc987bd86e69cc3fc7f9e9",
        	"original_transaction_id": "1000000096496199",
        	"bvrs": "1.4.5",
        	"transaction_id": "1000000096496199",
        	"quantity": "1",
        	"unique_vendor_identifier": "72FA2EBF-6100-49C5-B248-87C8178237CA",
        	"item_id": "604360663",
        	"product_id": "xxxxx",
        	"purchase_date": "2013-12-13 10:29:07 Etc/GMT",
        	"original_purchase_date": "2013-12-13 10:29:07 Etc/GMT",
        	"purchase_date_pst": "2013-12-13 02:29:07 America/Los_Angeles",
        	"bid": "xxxxxx",
        	"original_purchase_date_ms": "1386930547545"
    },
    	"status": 0
    }	
 
status字段代表的就是支付的状态，当且仅当为0时支付成功，更多status状态码解释可以看 [Validating Receipts With the App Store](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html#//apple_ref/doc/uid/TP40010573-CH104-SW1)  
   
receipt就是有关订单的各种信息，详情可以查阅 [In-App Purchase Receipt Fields](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Chapters/ReceiptFields.html#//apple_ref/doc/uid/TP40010573-CH106-SW1)。需要注意的是，apple并没有给出上例所有字段的含义，google了一下，看到一个[解答](http://stackoverflow.com/questions/15255564/ios-in-app-purchases-receipt-string-explained)，大意就是除了apple给出解释的字段之外，其他字段不能用来存储、验证，因为apple大爷随时会改变那些字段的含义或者索性删除掉，并且还不通知开发者。

验证流程小例子：[https://github.com/zc001/experience_share/blob/master/recharge/appstore.js](https://github.com/zc001/experience_share/blob/master/recharge/appstore.js)

### 《XXX将》的支付流程

1. 客户端向服务器请求游戏内订单C，一个游戏内订单信息如下：
	
		{
    		"cooOrderSerial": "136712954071279695",
    		"device_hash": "IOS-7f051956822d2f079e670e28d21120e4",
    		"_id": ObjectId("517cbdc402689c47640001c2"),
    		"rechargeWay": "appstore",
   		 	"createdTime": ISODate("2013-04-28T06:12:20.712Z"),
    		"payStatus": false,
   			"goodsID": "xxxxx",
    		"consumeStreamID": "",
    		"__v": 0
		}
	cooOrderSerial表示订单号。device_hash表示用户的唯一标识。payStatus表示该订单的处理状态，初始情况下为false，在服务给用户增加元宝后，会置成true，这样同一个订单就不会被用来反复加元宝。goodsID表示该订单的对应购买的商品id（这个id要在apple注册）。
	
2. 服务器生成游戏内订单C且将其存储进mongoDB，然后返回cooOrderSerial给客户端保存。

3. 客户端与apple服务器通信，成功支付后，apple服务器将购买凭证A返送给客户端。

4. 客户端将C的订单号，购买凭证A发送给游戏服务器验证，一个例子如下：
		
		{
    		device_hash: 'IOS-7f051956822d2f079e670e28d21120e4',
    		goldID: 'xxxxx',
    		orderID: '136712954071279695',
    		receipt: 'A',
    		serverID: 'sg_server001_test',
    		transaction: 433130912
		}
	
	orderID就是服务器订单号，receipt内容就是上面提到的购买凭证A（由于过长，就用'A'代替了）

5. 游戏服务器拿购买凭证A向apple服务器请求验证，apple服务器返回验证信息B。

6. 游戏服务器拿第4步中的orderID在数据库中查找游戏内订单，找不到C则证明该游戏内订单伪造，若找到C，则比较device_hash值是否对应，来确认是否为该用户充值，检查C的payStatus字段，加入为true，则认定订单违法，因为已经处理过该订单的充值。检查B的status字段，只有为【0】时，才认为支付成功。检查B的item_id字段，判断是否为本app充值信息。检查B的product_id和C的goodsID字段内容是否一致，来判断购买的档位是否一致。

7. 全部验证通过后，给用户添加元宝，C的payStatus置为true，存回数据库。 

### 回首看《XXX将》支付存在的问题

1. 同一个合法购买凭证A可能会被使用多次，因为游戏服务器没有记录该凭证的处理状态

2. 没有防止“中介人盗用”，有关该作弊方式，可以看参考资料中的[iOS应用内支付(IAP)的那些坑]

### 需要注意的问题以及建议

* 假如有服务器订单，如何防止该订单被使用多次

* 如何确定订单对应的游戏币数目

* 防止合法购买凭证被重复使用多次

* 防止“中介人盗用” 

* 妥善保存log，并且要详尽

* 提交审核的时候，一定不要忘记使用沙盒地址！！

* 越狱手机





### 参考资料

* [iOS应用内支付(IAP)的那些坑](http://blog.devtang.com/blog/2013/04/07/tricks-in-iap/)

* [iOS应用内付费(IAP)开发步骤列表](http://blog.devtang.com/blog/2012/12/09/in-app-purchase-check-list/)

* [In-App Purchase编程指南](http://blog.csdn.net/lixuwen521/article/details/8103949)

## 越狱与安卓

### 当时接过的平台

* 91

* pp助手

* 同步推

* 快用

* 北纬

### 总结大体流程

![支付流程](平台支付.png)


1. 客户端透过SDK向平台服务器发起支付

2. 支付成功后，平台服务器通过事先约定好的【回调地址】向游戏服务器发起回调

3. 游戏服务器接到平台服务器的回调信息，验证通过后，给客户端添加游戏货币  

### 游戏服务器验证回调信息

#### 91和pp助手、同步推

这三个平台，当时约定的验证方式是MD5。以91为例，下面是一个平台服务器的回调信息，平台服务器直接以GET方式明文发送过来，游戏服务器解析一下url即可得到。

	var query = { AppId: '107244',
  		ProductId: '107244',
 		Act: '1',
  		ProductName: 'xxxxx',
  		ConsumeStreamId: '2-25664-20130718161307-600-1827',
  		CooOrderSerial: '137413517563580118',
  		Uin: '407601397',
  		GoodsId: '60元宝',
  		GoodsInfo: '60元宝',
  		GoodsCount: '1',
  		OriginalMoney: '6.00',
  		OrderMoney: '6.00',
  		Note: 'sg_server000_test',
  		PayStatus: '1',
  		CreateTime: '2013-07-18 16:12:57',
  		Sign: '5eead9441c6290be19e0c5d804861831' 
	}

简要解释一下字段含义：

* AppId：是91平台分配给我们的游戏id

* CooOrderSerial：是客户端向游戏服务器请求来的游戏内订单号

* Note：这个字段当时被我们用来传递游戏区号，区分用户在哪个大区储值

* PayStatus：支付状态，按照平台给的文档，当且仅当为1时，充值成功

* Sign：MD5值，用于验证订单合法性

验证订单合法性的过程如下：
	
	var crypto = require('crypto');

	var key = '207fbbb41d8ba1c367c2c4f28e83cc83e05fd5284801a7e5';

	var query = { AppId: '107244',
  		ProductId: '107244',
 	  	Act: '1',
  		ProductName: 'xxxxx',
  		ConsumeStreamId: '2-25664-20130718161307-600-1827',
  		CooOrderSerial: '137413517563580118',
  		Uin: '407601397',
  		GoodsId: '60元宝',
  		GoodsInfo: '60元宝',
  		GoodsCount: '1',
  		OriginalMoney: '6.00',
  		OrderMoney: '6.00',
  		Note: 'sg_server000_test',
  		PayStatus: '1',
  		CreateTime: '2013-07-18 16:12:57',
  		Sign: '5eead9441c6290be19e0c5d804861831' 
	}

	var md5 = crypto.createHash('md5');
	var sourceSign = query.Sign;
	
	var strSource = query.AppId + query.Act + query.ProductName + query.ConsumeStreamId + query.CooOrderSerial + query.Uin + query.GoodsId  + query.GoodsInfo + query.GoodsCount + Number(query.OriginalMoney).toFixed(2) + Number(query.OrderMoney).toFixed(2) + query.Note +query.PayStatus + query.CreateTime + key;

	md5.update(strSource, 'utf8');
	var destSign = md5.digest('hex');

	if(sourceSign === destSign) {
  		console.log('验证成功');
	} else {
 		console.log('验证失败');
	}
	
上述过程，就是将query中大部分字段，按照91对接文档上的格式（顺序）拼接起来，最后再加上一个密钥“key”，形成一个字符串，然后对其MD5并以binary格式输出，然后比较该输出数据与query.Sign是否一致。

#### 北纬

北纬给出的加密方式是AES加密，支付成功之后，北纬的服务器通过回调地址给游戏服务器回调一个Base64格式的加密字符串，如下：
	
	hDhid85RVclRIUmM6v2IyAnRfO9GARNN8EAS0unm0hONCfpFmwxAdQRBlegNTLYpJPLdO7KOgdsPc4CCw2FKsPppo5sLzWBJAIbOuF1mRcsoHF43Z904NbhKJcZ0AdjR/Cqr9lZxCuhG+DTI1j2wPm89t6ktJkwDe/BHj09puodVWXRlvJql8ARLgK5DOpqcNNK/hXjWy9jZHnsYqDdem7E8Va4h0s/i6wB0jxzrfj3NFphVC/wnVymqKchKrisA6V1BAzTwPyclTeicb7AY7A==
	
北纬给的AES密钥为：

	xdgvsa3264ndkmdm	
	
由于现在平台商给的解密demo基本上都是java或者php的，我们需要自己动手写node.js版的解密，node.js核心模块有提供crypto，但是由于AES的加密模式（ECB、CBC）以及填充向量（iv）的关系，在这个例子里我们用官方库cryto解密是解不出来的（当时纠结的要死），后来发现一个API描述更加详细的第三方模块crypto-js，成功解决这个问题。

北纬给的java解密代码片段

	Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
	
由此可见，采用AES加密，加密模式为ECB，采用PKCS5Padding填充方式。了解这些细节后，使用crypto-js解密过程：
	
	var CryptoJS = require('crypto-js');
	var text = 'hDhid85RVclRIUmM6v2IyAnRfO9GARNN8EAS0unm0hONCfpFmwxAdQRBlegNTLYpJPLdO7KOgdsPc4CCw2FKsPppo5sLzWBJAIbOuF1mRcsoHF43Z904NbhKJcZ0AdjR/Cqr9lZxCuhG+DTI1j2wPm89t6ktJkwDe/BHj09puodVWXRlvJql8ARLgK5DOpqcNNK/hXjWy9jZHnsYqDdem7E8Va4h0s/i6wB0jxzrfj3NFphVC/wnVymqKchKrisA6V1BAzTwPyclTeicb7AY7A==';
	
	var key = "xdgvsa3264ndkmdm";

	var a = CryptoJS.AES.decrypt(text, CryptoJS.enc.Utf8.parse(key), {mode: CryptoJS.mode.ECB});

	console.log('a: ', a.toString(CryptoJS.enc.Utf8));
	
解密结果为：
	
	{
   		"state": "1",
    	"extraData": "13802507341711000",
    	"gameServerZone": "sg_server000",
    	"consumeId": "712381",
    	"gameId": "30",
    	"userId": "hanjietest",
    	"consumeValue": "1",
    	"userNumId": "5011451",
    	"reqtime": 1380251090000
	}			

## 相关知识&模块

在做支付过程中，会处理url信息，获取GET或者POST数据，计算散列（MD5），对称加密（AES加密），非对称加密（RAS），有几个比较好的模块可以帮助我们更加有效的进行这些操作处理。

* url

* querystring

* crpto

* crpto-js
 


