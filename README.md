### secp256k1-php
Readme for signature verification and sign function for PHP (secp256k1)

开发环境：Linux

### 0 安装 secp256k1 php extension
参考：https://github.com/Bit-Wasp/secp256k1-php 注意PHP5请用v0.0分支，PHP7请用v0.1分支

### 1 安装 sha3 php extension
参考：https://github.com/tml/php-sha3
Please use 'git clone https://github.com/tml/php-sha3'   !!!!!!

### 2 验签demo代码

```php
$context = secp256k1_context_create(SECP256K1_CONTEXT_SIGN | SECP256K1_CONTEXT_VERIFY);
$msg32 = sha3('this is a message!', 256);//这个消息是服务器返回的消息
$sigFromServer = "3a6f21e17b981d8d08677e0d3010f3aa9c2b8844f0b583eb0f0d992592601c1c6698980277a4b401541250a192a316cb681e7571aa883d2974626f662c83fcc31c";//这个签名是服务器返回的签名
$pubKeyFromServer = "743352f77078a12f30d37d01783706d5b6dff809";//这个公钥是服务器返回的公钥
$msgByte = hex2bin($msg32);
$recId = hexdec(substr($sigFromServer, 128, 2)) - 27;
$siginput = hex2bin(substr($sigFromServer, 0, 128));
$signature = '';
secp256k1_ecdsa_recoverable_signature_parse_compact($context, $signature, $siginput, $recId);
$pubKey = '';
secp256k1_ecdsa_recover($context, $pubKey, $signature, $msgByte);
$serialized = '';
$compress = false;
secp256k1_ec_pubkey_serialize($context, $serialized, $pubKey, $compress);
$A = bin2hex($serialized);
$B = substr($A, 2, 128);
$pubkeyH = sha3(hex2bin($B), 256);
$pubkeyNative = substr($pubkeyH, 24, 40);//这个公钥是本地根据服务器签名和服务器返回的消息计算出来的
if (strcmp($pubKeyFromServer, $pubkeyNative) == 0) {//如果本地计算的公钥和服务器返回的公钥一致就说明签名正确
    echo "signature is verified!";
} else {
    echo "signature is wrong!";
}
```
### 3 签名demo代码
```php
$context = secp256k1_context_create(SECP256K1_CONTEXT_SIGN | SECP256K1_CONTEXT_VERIFY);
$msg32 = sha3('this is a message!', 256);
$privateKey = pack("H*", "e4ce35c1ccf7f5d79a838bd527a0888fefb1523ce2fca52abd681d0e493bd5ad");//与服务器采用同样的私钥
$sigFromServer = "3a6f21e17b981d8d08677e0d3010f3aa9c2b8844f0b583eb0f0d992592601c1c6698980277a4b401541250a192a316cb681e7571aa883d2974626f662c83fcc31c";//服务器签名结果
$signature = '';
$msgByte = hex2bin($msg32);
if (secp256k1_ecdsa_sign_recoverable($context, $signature, $msgByte, $privateKey) != 1) {
    throw new \Exception("Failed to create recoverable signature");
}
$recId = 0;
$output = '';
secp256k1_ecdsa_recoverable_signature_serialize_compact($context, $signature, $output, $recId);
$signatureNative = bin2hex($output).dechex($recId + 27);//本地签名结果
if (strcmp($sigFromServer, $signatureNative) == 0) {//服务器签名和本地签名对比
    echo "signature is right!";
} else {
    echo "signature is wrong!";
}
```

