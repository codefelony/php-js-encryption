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
