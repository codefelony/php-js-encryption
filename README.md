# Matching PHP and JS Encryption

##Problem

You'd think that encryption is in encryption- AES is AES, DES is DES, etc. Encrypt with scheme X, decrypt with scheme X, and it comes out clean. Turns out, though, that it's not that easy. Or rather, it is, but the relevant libraries on different platforms like to do things different. Consider that to encrypt/decrypt something with AES you need:

 * To use the same mode (CBC, EBC, etc)
 * The same block type for block cipher modes
 * The same key size (128 bit, 256 bit, etc)
 * The same key (of course)
 * The same initialization vector
 * To know whether the library pads/truncates your key
 * To know whether/how the library hashes your key
 * To know whether the library includes necessary extra information (plaintext size and IV) in its output or whether you need to keep track of that yourself
 * To know whether your ciphertext output it hex, base64 encoded, UTF8, or whatever.

In further fun, the libraries you're looking at may only expose half of these variables and just pick their own numbers for the other half.

So, when you need to encrypt something client-side and decrypt it in, say, PHP, it's not as simple as you'd like.

##Quick Aside

Why would you want to encrypt client-side! That's insane! Well, the short version is that I'm not hiding data from bad guys- I'm hiding it from my own system. What's happening is that the user is entering credential information for an external system. That includes a username + password, plus some variable other stuff. The variable other stuff (VOS for now) is what needs to get encrypted. The username and password are already treated with kid gloves in our system- they're not saved, they're blocked out if a dev tries to log them, etc. The VOS, though, is new, and we're passing it back as part of a complex data object that isn't treated carefully- it could end up in analytics, logs, dump files, emails, etc. As such, we're encrypting it client-side with the user's password and decrypting it just before we need to send it to the external system (a point at which we also have the user's password). As such, if it accidentally gets logged or something, it's not stored in the clear.

##Solution time

Not that complex, just putting it here because there didn't seem to be a good ready-made recommendation anywhere I google.

I ended up using [SlowAES](https://github.com/codefelony/slowaes), which is fast enough for many purposes. It comes with parallel implementations in PHP, JS, Python and Ruby. They each take the same parameters and use the same utilities, so it's easy to get them to line up. To save you some time, here's a simple use example for PHP and JS (not thoroughly tested because we decided to go for another solution before this made it to that point, but anyway):

###encrypt.php
```php
/**
 * Some simple testing code
 **/
<?php
require_once './encrypt.php';

$plaintext = "Testing the php encryption/decryption. Unicode LOD: ಠ_ಠ!";
$key = "multipass!";
$cipherText = encrypt($plaintext,$key);
$result = decrypt($cipherText,$key);
?>
<!doctype html>
<html lang="en"><head><meta charset="UTF-8"></head><body>
    <h1>Encrypting</h1>
    <b>plaintext:</b> <?= $plaintext ?> <br>
    <b>key:</b> <?= $key ?> <br>
    <b>raw cipherText:</b> <?= $cipherText ?> <br>
    <h1>Decrypting</h1>
    <b>result:</b><?= $result ?>
</body></html>
```

###encrypt.js.html
```js
/**
 * Some simple testing code
 **/
$(function(){
    var key = "multipass!";
    var plaintext = "Testing the php encryption/decryption. Unicode LOD: ಠ_ಠ!";
    var output = "";
    var cipherText = encrypt(plaintext,key);
    var newPlaintext = decrypt(cipherText,key);
    output += ("<br>plaintext=" + plaintext);
    output += ("<br>cipherText=" + cipherText);
    output += ("<br>newPlaintext=" + newPlaintext);

    $('#output').html(output);
});
```

##Final
Remember that this is probably neither efficient nor elegant. It hasn't been tested extensively, so use at your own risk. Should do the job, though.
