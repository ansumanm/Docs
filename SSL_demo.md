## Summary of OpenSSL Asymmetric Key Encryption Conversation

### Generating Asymmetric Keys with OpenSSL

1. **Generate the Private Key**
   ```shell
   openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
   ```
   This command creates an RSA private key of 2048 bits and saves it to `private_key.pem`.

2. **Generate the Public Key**
   ```shell
   openssl pkey -in private_key.pem -out public_key.pem -pubout
   ```
   This command extracts the public key from the private key file `private_key.pem` and writes it to `public_key.pem`.

### Encrypting a Message Using the Public Key

- **Encrypt Command**
  ```shell
  openssl rsautl -encrypt -inkey public_key.pem -pubin -in message.txt -out message.enc
  ```
  This command encrypts `message.txt` using the public key in `public_key.pem`, resulting in the encrypted file `message.enc`.

### Decrypting a Message Using the Private Key

- **Decrypt Command**
  ```shell
  openssl rsautl -decrypt -inkey private_key.pem -in message.enc -out decrypted_message.txt
  ```
  This command decrypts `message.enc` using the private key in `private_key.pem`, resulting in the decrypted file `decrypted_message.txt`.
