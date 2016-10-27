#How to initialize and write private keys to memory for further usage

##Note:  Assumes fully set up dev environment.  See https://github.com/LedgerHQ/ledger-dev-doc/blob/master/nanos/setup.rst

For this, we will be using the Samplesign app from ledger's sample apps https://github.com/LedgerHQ/blue-sample-apps


The main.c file contains the following code which we will change to use our own private key

```
            if (N_initialized != 0x01) {
                unsigned char canary;
                cx_ecfp_private_key_t privateKey;
                cx_ecfp_public_key_t publicKey;
                cx_ecfp_generate_pair(CX_CURVE_256K1, &publicKey, &privateKey,
                                      0);
                nvm_write(&N_privateKey, &privateKey, sizeof(privateKey));
                canary = 0x01;
                nvm_write(&N_initialized, &canary, sizeof(canary));
            }
```


As you can see, what this code does is generate a keypair.

In order to use keys in BOLOS, they need to be written to the memory to the variable N_privateKey, which is initialized at the top of the file.

The function `cx_ecfp_generate_pair`accepts the following parameters: `Curve, pointer to cx_ecfp_public_key_t to output to, pointer to the cx_ecfp_private_key_t to output to, keep private`
```
Keep Private is described in cx.h in the sdk files as the following:

 - if the private ecfp key is fully inited, i.e  parameter 'rawkey' of
 *       'cx_ecfp_init_private_key' is NOT null, the private key value is kept
 *       if the 'keep_private' parameter is non zero
```

The cx_ecfp_generate_pair function only generates a privatekey if the value of the cx_ecfp_private_key_t passed is null.  So if we initiate the privatekey with our own, we can generate and store its public key
We can then use nvm_write to put the privatekey in the location in memory of N_privateKey.

```
          unsigned char canary;
                // Insert Private Key Data
                unsigned char privateKeyData[32] = {
                    0xf5, 0xf0, 0xb3, 0x46, [...] private key hex raw format goes here
                };
                // Create Both Key variables 
                cx_ecfp_private_key_t privateKey;
                cx_ecfp_public_key_t publicKey;
                // Init Public Key Null And Private Key with data from privateKeyData
                cx_ecdsa_init_public_key(CX_CURVE_256R1, NULL, 0, &publicKey);
                cx_ecdsa_init_private_key(CX_CURVE_256K1, privateKeyData, 32, &privateKey);
                // Generate pair with keepprivate set to 1
                cx_ecfp_generate_pair(CX_CURVE_256K1, &publicKey, &privateKey, 1);
                nvm_write(&N_privateKey, &privateKey, sizeof(privateKey));
                canary = 0x01;
                nvm_write(&N_initialized, &canary, sizeof(canary));
```
