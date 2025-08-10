filemanagement_storage_service
====================================

EL1 ~ EL5 逻辑在 base_key.cpp。

FBE class key:
- EL1: DE
- EL2: CE
- EL3: SECE
- EL4: ECE，即AE
- EL5: UECE

EL3 & EL5 与 apple的BE 有差异，根源在于是否用了非对称密钥对。

加解密逻辑在 huks_master.cpp、openssl_crypto.cpp。

huks_master.cpp : device_key 对 fbe class key 的加密。

openssl_crypto.cpp : 基于PIN码认证所获得的secret 进一步hash派生密钥，对fbe class key的密文进行二次加密。



