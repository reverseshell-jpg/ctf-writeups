# dream-hack-MemoVault
## Exploit chain: JWT Attacks -> SQL Injection -> GET Flag
## Source
+ Some notable functions:
  + `_get_token_from_request`: retrieves the value of the `token` field from the cookie
  + `_verify_and_decode_eddsa`: decodes the JWT
+ config.py:
  <img width="582" height="58" alt="image" src="https://github.com/user-attachments/assets/8cdd4fd5-2a49-492d-b0db-c455d6557c97" />

  + Trying to access the endpoints in the source -> obtained the public key
## Exploit
+ In the `_verify_and_decode_eddsa` function, the challenge intends to use `EdDSA` to decode the JWT. However, the source code uses `return jwt.decode(token, key=pubkey, algorithms=jwt.algorithms.get_default_algorithms())`, which means the server only verifies and decodes the JWT according to the algorithm type sent by the client, instead of fixing it to `EdDSA`
-> JWT Attacks: EdDSA is a form of Asymmetric Encryption (an encryption scheme that uses a private key to sign and a public key to verify), and since the server doesn't fix the algorithm type the client uses, we can instead use a Symmetric Encryption scheme (an encryption scheme that uses a single secret key for both signing and verification) (you can learn more [here](https://github.com/nakov/practical-cryptography-for-developers-book/blob/master/encryption-symmetric-and-asymmetric.md)). Here we use `HS256`
+ At the `/profile` endpoint, the server takes the `uid` field from the token in the cookie and passes it directly into the SQL query without validation
  <img width="773" height="302" alt="image" src="https://github.com/user-attachments/assets/2af9d73d-2a0f-49a7-bfce-8aedb54ea668" />

-> SQL Injection UNION attack: we can pass a payload into `uid`, test payload: `-1 UNION SELECT 1,2-- -` (you can forge a new JWT [here](https://www.jwt.io/))
  <img width="1912" height="740" alt="image" src="https://github.com/user-attachments/assets/3ec46265-c33e-437a-a5e3-f451a1717d24" />

## Get flag
<img width="1910" height="733" alt="image" src="https://github.com/user-attachments/assets/e96a83fc-82f0-47c0-9466-3a5aad10b70f" />
