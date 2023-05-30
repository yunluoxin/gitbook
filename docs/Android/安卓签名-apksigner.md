# 利用 apksigner 进行签名



## 通用签名方式（不指定版本）

```shell
apksigner sign --ks my-release-key.jks [--ks-key-alias alias_name] "$APK_NAME"
# --ks-key-alias 为可选参数。对于apksigner，默认使用keystore中的第一个证书，但是可以在这里明确指定。（一般用不到）
```



## 指定签名的版本

如果要指定签名的版本可以这样：
这里禁用v1，允许v2

```shell
apksigner sign --ks my-release-key.jks --v1-signing-enabled false --v2-signing-enabled true "$APK_NAME"
```

> todo: 当我禁用v2，使用v1时候失败了？？？ 不知道是不是原有v2签名惹得，还是我用的apk有最低的下限。 但是v2被禁用应该是生效了。

> 实测进入apk文件夹里发现，v1的签名文件夹里 `META-INFO` 确实多出了签名信息，只不过打印不出来。到底算不算生效呢？

```shell
apksigner sign --ks my-release-key.jks --v1-signing-enabled true --v2-signing-enabled false "$APK_NAME"
```

> enable v1 和 v2 时候情况一样。也是显示v1为false，但实际上是有 `META-INFO` 里的签名的。



## 输出apk的签名信息（签名的证书名、版本号）

```shell
apksigner verify --verbose --print-certs "$APK_NAME"
```

输出结果如下：
```shell
Verifies
Verified using v1 scheme (JAR signing): false
Verified using v2 scheme (APK Signature Scheme v2): false
Verified using v3 scheme (APK Signature Scheme v3): true
Verified using v4 scheme (APK Signature Scheme v4): false
Verified for SourceStamp: false
Number of signers: 1
Signer #1 certificate DN: CN=East, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
Signer #1 certificate SHA-256 digest: fd0f580ebb4e3b4f6367c392a6fbb297c3c86dae17168d1dda02ee1526a7e8f5
Signer #1 certificate SHA-1 digest: e6e34e9aca1b2d0305c3d7063db6333637f61b29
Signer #1 certificate MD5 digest: b6c4afdd12efdaa97d1563b26f6f0635
Signer #1 key algorithm: RSA
Signer #1 key size (bits): 2048
Signer #1 public key SHA-256 digest: feb31e713cc80f30e1341dc1e7004b76b4901a799f78f72ff6fcfd3a12a003d3
Signer #1 public key SHA-1 digest: 2428efadd623c4fc2836449d9ed59c011e7650df
Signer #1 public key MD5 digest: 91eda31b5f7d4596d00e075c8c1e6d7e
```