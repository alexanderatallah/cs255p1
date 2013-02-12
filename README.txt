cs255p1
=======

CS 255 Project 1

Thomas Davids - tdavids
Alexander Atallah - aatallah

All instructions are given in prompt dialogues or error messages. Keys should be created in Settings.

To encrypt, we implemented a CBC with a random IV. This random IV allowed us to use the same key more than once, which was important for practical reasons. We elected to use this over the Random Counter Mode, despite the fact that Random Counter Mode is slightly more secure, because we expect Facebook group posts to be relatively short, which makes the extra factor of the length of the message less significant. We realize that this would have some security issues if the nonce could be predicted -- however, using the GetRandomValues() function allowed us to use a completely random nonce, which removes that security hole. Finally, we added padding to make sure the message would work for CBC.

To store the keys, we kept an encrypted stringified JSON of the keys for the user's group. The key for the decryption of this database was the hash of the user's password concatenated with a salt, which was stored in local storage (i.e. in the clear). We checked to see if the user had entered the correct password based on whether the first characters of the decryption were "{}" or "{\"", as well as checking for consistency (no unreadable characters). Once the user entered the correct password and successfully decrypted the database, they were able to access all of the keys and see posts in any of their groups. This was not completely secure, for reasons described below, and in the end, much of the security at this level of the implementation depended on the strength of the user's password. However, for a reasonably secure password, our method was secure for all but the most sophisticated attackers.

If an attacker were able to break one part of our cryptography scheme, he/she would possibly compromise aspects we assume to be secure. For example, if an attacker brute-forced anyone's database password, everything else is pointless, since the keys for their groups are exposed.

For our MAC system, we implemented a CBC-MAC, encrypting with a derived key on the last step.

Discussion questions:

1. Browser cryptography suffers from several issues. Since all of the SJCL code isn't compiled or signed, an attacker can inspect the code easily and find ways to attack it. But more importantly, JavaScript is easy to inject into a page: if an attacker took control of a user's browser, he/she could inject a <script> tag that intercepts "Encrypt" button presses and takes the message before the user actually encrypts it.

Additionally, in this specific implementation, cryptography in the browser can be insecure because we give our key and other information to someone we might not entirely trust (in this case, the browser). This has a couple of negative consequences. For one, we have to store the salt unsecurely, so someone using our salt and a table of common passwords could potentially match the hashed password (and encryption key) of a specific user to one of these passwords and hack into their account. We also have to store the encryption keys of each group with Facebook (albeit encrypted), so if an attacker was able to hack a single user, they would be able to look at all the keys for their groups.

However, there are also benefits to doing cryptography in the browser. It makes crypto available to everyone, so the benefits are widespread and would reduce attacks on the whole. And in server-based cryptography, if the server goes down or the php interpreter fails, code that is expected to be secret isn't accidentally exposed. Browser-based crypto forces programmers to know that the algorthms must be secure even when exposed to the attacker.

2. There are a couple ways someone could circumvent our security. As mentioned above, someone could use our salt with a variety of common passwords to generate the hashes of those passwords. Since we use the hashed password as the key, they could try to use this key to break into our database of keys. Even worse, once they were able to decrypt that database, they would not only be able to read our messages, but the messages of anyone who had posted in any of the groups we were in.

Other, less clever attacks such as phishing could also circumvent our security. If another Facebook app was able to trick the user into entering their password into a similar-looking box, they could store that password and use it to access their key database.

Finally, injecting another script tag into the page could allow an attacker to intercept messages before they're encrypted. They would only need to hijack the click event for the Encrypt button.

