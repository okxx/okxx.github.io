---
layout: post
title: PHP RSA Algorithm
categories: [php]
tags: [php]
sidebar: []
---

PHP RSA Algorithm.

```php
$public_key_path = '/data/site_config/open_pem/rsa_private_key.pem';
$private_key_path = '/data/site_config/open_pem/rsa_private_key.pem';

//create pkey
$keys = openssl_pkey_new([
    "private_key_bits" => 1024,// bit 1024
]);
//export private key
openssl_pkey_export($keys,$private_key);

//use public key generate private key.
$public_key = openssl_pkey_get_details($keys);
$public_key = $public_key['key'];

// save
file_put_contents($private_key_path,$private_key);
file_put_contents($public_key_path,$public_key);


$public_key = file_get_contents($public_key_path);
$private_key = file_get_contents($private_key_path);

$data = [
    'openId'    => '855FB184081208CA29C7AE59E7898494N124j7W',
    'nickname'  => 'alan',
    'avatar'    => 'http://xxx.xxx.com/avatar/xxx/xx/xxxx_avatar.jpg?v=xxxx',
    'time'      => 'xxxxxxxxxx',
];
dump($data);

echo "<br /><br />";
$data = json_encode($data);
echo 'encrypt data：'.$data."<br /><br />";


openssl_private_encrypt($data,$encrypted,$private_key);
$encrypted = base64_encode($encrypted);
echo "private encrypt data:".$encrypted."<br /><br />";
echo "length:".mb_strlen($encrypted)."<br /><br />";

//public decrypt
openssl_public_decrypt(base64_decode($encrypted), $decrypted, $public_key);
echo "public decrypt data:".$decrypted,"<br/><br/>";

// public encrypt
openssl_public_encrypt($data, $encrypted, $public_key);
$encrypted = base64_encode($encrypted);
echo "public encrypt data:".$encrypted."<br /><br />";
echo "length:".mb_strlen($encrypted)."<br /><br />";

openssl_private_decrypt(base64_decode($encrypted), $decrypted, $private_key);
echo "private decrypt data :".$decrypted."<br /><br />";
```
