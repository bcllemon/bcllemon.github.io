layout: post
title: "Java 生成 AES 秘钥"
date: "2020-09-04 17:34"
categories: 技术
---
# AES工作模式
## 电子密码本：Electronic Code Book Mode (ECB)
ECB模式只是将明文按分组大小切分，然后用同样的密钥正常加密切分好的明文分组。ECB的理想应用场景是短数据（如加密密钥）的加密。此模式的问题是无法隐藏原明文数据的模式，因为同样的明文分组加密得到的密文也是一样的。
## 密码分组链接：Cipher Block Chaining Mode (CBC)
此模式是1976年由IBM所发明，引入了IV（初始化向量：Initialization Vector）的概念。IV是长度为分组大小的一组随机，通常情况下不用保密，不过在大多数情况下，针对同一密钥不应多次使用同一组IV。
CBC要求第一个分组的明文在加密运算前先与IV进行异或；从第二组开始，所有的明文先与前一分组加密后的密文进行异或。[区块链(blockchain)的鼻祖！]
CBC模式相比ECB实现了更好的模式隐藏，但因为其将密文引入运算，加解密操作无法并行操作。同时引入的IV向量，还需要加、解密双方共同知晓方可。
## 密文反馈：Cipher Feedback Mode (CFB)
与CBC模式类似，但不同的地方在于，CFB模式先生成密码流字典，然后用密码字典与明文进行异或操作并最终生成密文。后一分组的密码字典的生成需要前一分组的密文参与运算。
CFB模式是用分组算法实现流算法，明文数据不需要按分组大小对齐。
## 输出反馈：Output Feedback Mode (OFB)
OFB模式与CFB模式不同的地方是：生成字典的时候会采用明文参与运算，CFB采用的是密文。
CTR模式同样会产生流密码字典，但同是会引入一个计数，以保证任意长时间均不会产生重复输出。
## 计数器模式：Counter Mode (CTR)
# 生成 AES 秘钥
```
KeyGenerator kgen = KeyGenerator.getInstance("AES");
SecureRandom secureRandom = new SecureRandom();
kgen.init(256, secureRandom);
SecretKey key = kgen.generateKey();
System.out.println(Base64.encodeBase64String(key.getEncoded()));
```
# AES/CBC/PKCS5Padding 生成 IV
```
SecureRandom secureRandom = new SecureRandom();
kgen.init(256, secureRandom);
byte[] iv = new byte[16];
secureRandom.nextBytes(iv);
System.out.println(Base64.encodeBase64String(iv));
```
# AES/CBC/PKCS5Padding 加解密
```
public class RijndaelCrypt {

    public static final String TAG = "RijndaelCrypt";

    private static final String TRANSFORMATION = "AES/CBC/PKCS5Padding";
    private static final String ALGORITHM = "AES";

    private SecretKey _password;
    private IvParameterSpec _IVParamSpec;


    private Cipher cipher;

    private static Logger log = LoggerFactory.getLogger(RijndaelCrypt.class);

    /**
     * Constructor
     */
    public RijndaelCrypt(byte[] key, byte[] iv) {

        try {
            _password = new SecretKeySpec(key, ALGORITHM);

            //Initialize objects
            cipher = Cipher.getInstance(TRANSFORMATION);

            _IVParamSpec = new IvParameterSpec(iv);

        } catch (NoSuchAlgorithmException e) {
            log.error(TAG, "No such algorithm " + ALGORITHM, e);
        } catch (NoSuchPaddingException e) {
            log.error(TAG, "No such padding PKCS7", e);
        }

    }


    public String decrypt(byte[] cipherText) {
        try {
            cipher.init(Cipher.DECRYPT_MODE, _password, _IVParamSpec);
            byte[] decryptedVal = cipher.doFinal(cipherText);
            return new String(decryptedVal);

        } catch (IllegalBlockSizeException e) {

        } catch (BadPaddingException e) {

        } catch (InvalidAlgorithmParameterException e) {

        } catch (InvalidKeyException e) {

        }
        return null;
    }


    /**
     * Encryptor.
     *
     * @return Base64 encrypted text
     * @text String to be encrypted
     */
    public String encrypt(byte[] text) {

        try {
            cipher.init(Cipher.ENCRYPT_MODE, _password, _IVParamSpec);
            byte[] encryptedData = cipher.doFinal(text);
            return Base64.encodeBase64String(encryptedData);


        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        return null;

    }

    public String encrypt(String text) {
        byte[] data = StringUtils.getBytesUtf8(text);
        return encrypt(data);
    }


}
```
