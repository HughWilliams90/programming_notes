# Create SSL certificates and store in Java Key Store / use with Webpack dev server

## Prerequisites
Java is installed along with keytool

example common name = *.williamsdev.co.uk

## 1. create certificate authority key store

`keytool -genkeypair -alias williamsdevCA -dname "CN=williamsdevCA, OU=Example Org, O=Example Company, L=Swindon, ST=Wiltshire, C=UK" -keyalg RSA -keystore williams-ca.jks -keysize 2048 -validity 99999`

## 2. export ca cert from ca keystore

`keytool -export -v -alias williamsdevCA -file williamsdevCA.crt -keystore williams-ca.jks -rfc -validity 99999`

## 3. create certificate for your application

`keytool -genkeypair -alias local.williamsdev.co.uk -dname "CN=local.williamsdev.co.uk, OU=Example Org, O=Example Company, L=Swindon, ST=Wiltshire, C=UK" -keyalg RSA -keystore williams.jks -keysize 2048 -validity 99999`

## 4. create signing request for your cert

`keytool -certreq -alias local.williamsdev.co.uk -keystore williams.jks -file williamsdev.csr -validity 99999`

## 5. Sign your certificate with you CA cert

`keytool -gencert -v -alias williamsdevCA -keystore williams-ca.jks -infile williamsdev.csr -outfile williamsdev.crt -ext KeyUsage:critical="digitalSignature,keyEncipherment" -ext EKU="serverAuth" -ext SAN="DNS:local.williamsdev.co.uk" -rfc -validity 999999`

## 6. tell you application keystore it can trust your CA as a signer
```
keytool -import -v -alias williamsdevCA -file williamsdevCA.crt -keystore williams.jks -storetype JKS -storepass Reggie2425 << EOF

Yes

EOF
```

## 7. Import your signed cert back into you application keystore

`keytool -import -v -alias local.williamsdev.co.uk -file williamsdev.crt -keystore williams.jks`

## 8. get private key

`keytool -importkeystore -v -srcalias local.williamsdev.co.uk -srckeystore williams.jks -srcstoretype jks -destkeystore williamsdev.p12 -deststoretype PKCS12`

then

`openssl pkcs12 -nocerts -nodes -in new-project.p12 -out new-project.key`

## 9. Make localhost trust certs
add both the CA cert and the application cert to trust store i.e mac keychain and trust

## For webpack dev server

add the private key and cert like so

```
https: {
  key: fs.readFileSync(path.resolve(".certificates", "williamsdev.key")),
  cert: fs.readFileSync(path.resolve(".certificates", "williamsdev.crt"))
}
```

## For play application

add keystore path to dev settings in build sbt

```
val devSettings = Seq(
  PlayKeys.playDefaultAddress := "local.williamsdev.co.uk",
  PlayKeys.devSettings ++= Seq(
    "play.server.http.port" -> "8000",
    "play.server.https.port" -> "4430",
    "play.server.https.keyStore.path" -> "./.certificates/williams.jks",
    "play.server.https.keyStore.type" -> "JKS",
    "play.server.https.keyStore.password" -> "Reggie2425"
  )
)
```
