# crypto-js

## 安装

	npm install crypto-js
	
项目地址：[https://github.com/evanvosberg/crypto-js](https://github.com/evanvosberg/crypto-js)	

## 调用

	var CryptoJS = require('crypto-js');

## Hashers

### The Hasher Input

哈希算法的输入数据可以是字符串，也可以是CryptoJS.lib.WordArray（crypto-js自定义的一种数据类型）的实例。当输入数据是个字符串的时候，程序将自动把这个字符串以UTF-8编码格式转化成CryptoJS.lib.WordArray实例。

### The Hasher Output

经过处理后，你获取的哈希值还不是一个字符串，它是CryptoJS.lib.WordArray实例。当你使用toString方法转化时，将默认返回hex格式字符串。

#### MD5
	
	var hash = CryptoJS.MD5("Message");
	console.log('hash hex: ', hash.toString());
	console.log('hash base64: ', hash.toString(CryptoJS.enc.Base64));
	
	//打印信息如下
	hash hex:  4c2a8fe7eaf24721cc7a9f0175115bd4
	hash base64:  TCqP5+ryRyHMep8BdRFb1A==

#### SHA-1

	var hash = CryptoJS.SHA1("Message");
	console.log('hash hex: ', hash.toString());
	console.log('hash base64: ', hash.toString(CryptoJS.enc.Base64));
	
	//打印信息如下
	hash hex:  68f4145fee7dde76afceb910165924ad14cf0d00
	hash base64:  aPQUX+593navzrkQFlkkrRTPDQA=

#### SHA-2
SHA-256

	var hash = CryptoJS.SHA256("Message");
	
SHA-512

	var hash = CryptoJS.SHA512("Message");
	
CryptoJS也支持SHA-224和SHA-384

#### SHA-3

SHA-3可以指定输出的哈希长度，可选的有224、256、384、512，默认是512

	var hash = CryptoJS.SHA3("Message");
	var hash = CryptoJS.SHA3("Message", { outputLength: 512 });

#### RIPEMD-160
	
	 var hash = CryptoJS.RIPEMD160("Message");

## cipher 

加密算法支持AES、DES和Triple DES、Rabbit、RC4和RC4Drop。

### The Cipher Input

对于明文，加密算法接收字符串或者CryptoJS.lib.WordArray的实例。

对于密钥，当你传递的是一个字符串时，程序会自动拿这个字符串生成一个实际的加密密钥以及填充向量。当你传递的是一个CryptoJS.lib.WordArray实例，那你传递的这个值就是实际的加密密钥，此时，你还必须传递填充向量。

对于密文，算法接收字符串或者CryptoJS.lib.CipherParams实例。CryptoJS.lib.CipherParams实例就是密钥、填充向量以及密文的集合，当你传递一个字符串时，系统会自动将其转化成CryptoJS.lib.CipherParams实例。

### The Cipher Output

加密后返回的并不是一个字符串，而是一个CryptoJS.lib.CipherParams实例，通过这个实例可以获得解密时所需要的实际密钥、填充向量等，当你对这个实例使用toString方法时，将默认获得base64格式的字符串。

解密后返回的也不是一个字符串，而是一个CryptoJS.lib.WordArray实例，对它使用toString方法将默认获得hex的字符串。

### Block Modes and Padding

CryptoJS支持以下模式：

* CBC（默认）

* CFB

* CTR

* OFB

* ECB 

CryptoJS支持以下填充方式：

* Pkcs7 (默认) 

* Iso97971 

* AnsiX923

* Iso10126

* ZeroPadding

* NoPadding   

### AES

#### test_1

	function test_1 () {
		var encrypted = CryptoJS.AES.encrypt("Message", "Secret Passphrase");
		console.log('encrypted: ', encrypted.toString());
		var decrypted = CryptoJS.AES.decrypt(encrypted, "Secret Passphrase");
		console.log('decrypted: ', decrypted.toString());
		console.log('decrypted: ', decrypted.toString(CryptoJS.enc.Utf8));
	}

	test_1();
	
	//打印信息如下
	encrypted:  U2FsdGVkX1+dx6qHUXo4Lnn8QoeXmGag2zGqgOM45D0=
	decrypted:  4d657373616765
	decrypted:  Message
	
#### test_2

	function test_2 () {
		var encrypted = CryptoJS.AES.encrypt("Message", "Secret Passphrase");
		console.log('encrypted.key: ', encrypted.key.toString());
		console.log('encrypted.iv: ', encrypted.iv.toString());
		console.log('encrypted.salt: ', encrypted.salt.toString());
		console.log('encrypted.ciphertext: ', encrypted.ciphertext.toString());
		console.log('encrypted: ', encrypted.toString());
		var decrypted = CryptoJS.AES.decrypt(encrypted, "Secret Passphrase");
		console.log('decrypted: ', decrypted.toString());
		console.log('decrypted: ', decrypted.toString(CryptoJS.enc.Utf8));
	}

	test_2();	
	
	//打印信息如下
	encrypted.key:  973490604cf4553a51dc77363d1ca889e02b62cd42d78403313ed82c24163ce4
	encrypted.iv:  7d5d8143a731a4240361f346ebe785f5
	encrypted.salt:  5732a58b66aced07
	encrypted.ciphertext:  0e74d6f3c4075d27939570860806f651
	encrypted:  U2FsdGVkX19XMqWLZqztBw501vPEB10nk5VwhggG9lE=
	decrypted:  4d657373616765
	decrypted:  Message
	
#### test_3

	function test_3 () {
		var encrypted ="U2FsdGVkX1+dovNkl13BFqnnCae8WXHXEuoLEXqIDc4=";
		var decrypted = CryptoJS.AES.decrypt(encrypted, "Secret Passphrase");
		console.log('decrypted: ', decrypted.toString());
		console.log('decrypted: ', decrypted.toString(CryptoJS.enc.Utf8));
	}

	test_3();	
	
	//打印信息如下
	decrypted:  4d657373616765
	decrypted:  Message
	
#### test_4

	function test_4 () {
		var key = CryptoJS.lib.WordArray.random(16);
		var iv = CryptoJS.lib.WordArray.random(16);
		var encrypted = CryptoJS.AES.encrypt("Message", key, {mode: CryptoJS.mode.ECB, iv: iv});
		console.log('encrypted: ', encrypted.toString());
		// var decrypted = CryptoJS.AES.decrypt(encrypted.toString(), key, {mode: CryptoJS.mode.ECB, iv: iv});
		var decrypted = CryptoJS.AES.decrypt(encrypted, encrypted.key, {mode: CryptoJS.mode.ECB, iv: encrypted.iv});
		var decrypted = CryptoJS.AES.decrypt(encrypted, key, {mode: CryptoJS.mode.ECB, iv: iv});
		console.log('decrypted: ', decrypted.toString(CryptoJS.enc.Utf8));
	}	

	test_4();	
	
	//打印信息如下
	encrypted:  hKx9GlL8T836IgUdyE0qmg==
	decrypted:  Message
	
#### test_5

特别说明的是，下面需要被解密的数据是java使用Cipher.getInstance("AES/ECB/PKCS5Padding")加密得来的，我们使用下面的方式进行解密

	function test_5 () {
		var text = "j1RLRyw2/gQWh5xjuU2DOk9+CfUx13ePlqMoeLNZRBUk0M1jHHVtKDN/XFkvT2N3izI3Astm3sLX0urF/Kmqh/cv3YXy8PosPngxMTmeQtzIxP2yZjzpuDGhY9eXHn0ihhfMJ9w4XkV87STG2p2YJOG278xI6MABMqrSYTcODM1mCtP/sw/Cn9V2PM40IIGJRLF/jLlOhCrhy0XRXy/nAw==";
		var key = "gameserverurlkey";
		var a = CryptoJS.AES.decrypt(text, CryptoJS.enc.Utf8.parse(key), {mode: CryptoJS.mode.ECB});
		console.log('a: ', a.toString(CryptoJS.enc.Utf8));
	}

	test_5();	
	
	//打印信息如下
	a:  {"userId":"cacawang","gameId":"1","reqtime":1346742112296 ,"state":"1","consumeValue":"200.00","extraData":"wasjdaosiasd","gameServerZone":"Mzone"}

## 说明

由于作者没有太多时间更新，假如不能满足需求的话，推荐使用库 [forge](https://github.com/digitalbazaar/forge).

crypto-js官方API说明：[https://code.google.com/p/crypto-js/](https://code.google.com/p/crypto-js/)
