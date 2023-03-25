# Create SSL certificates and store in Java Key Store / use with Webpack dev server

## Prerequisites
Java is installed along with keytool



example common name = *.williamsdev.co.uk

## 1. create certificate authority key store

`keytool -genkeypair -alias rootCA -dname "CN=rootCA, OU=Example Org, O=Example Company, L=Swindon, ST=Wiltshire, C=UK" -keyalg RSA -keystore keystore-ca.jks -keysize 2048`

## 2. export ca cert from ca keystore

`keytool -export -v -alias rootCA -file rootCA.crt -keystore keystore-ca.jks -rfc`

## 3. create certificate for your application

`keytool -genkeypair -alias local.new-project.com -dname "CN=local.new-project.com, OU=Example Org, O=Example Company, L=Swindon, ST=Wiltshire, C=UK -keyalg RSA -keystore keystore.jks -keysize 2048`

## 4. create signing request for your cert

`keytool -certreq -alias local.new-project.com -keystore keystore.jks -file new-project.csr`

## 5. Sign your certificate with you CA cert

`keytool -gencert -v -alias rootCA -keystore keystore-ca.jks -infile new-project.csr -outfile new-project.crt -ext KeyUsage:critical="digitalSignature,keyEncipherment" -ext EKU="serverAuth" -ext SAN="DNS:local.new-project.com" -rfc`

## 6. tell you application keystore it can trust your CA as a signer
```
keytool -import -v -alias rootCA -file rootCA.crt -keystore keystore.jks -storetype JKS -storepass Reggie2425 << EOF

Yes

EOF
```

## 7. Import your signed cert back into you application keystore

`keytool -import -v -alias *.new-project.com -file new-project.crt -keystore keystore.jks`

## 8. get private key

`keytool -importkeystore -v -srcalias *.new-project.com -srckeystore keystore.jks -srcstoretype jks -destkeystore new-project.p12 -deststoretype PKCS12`

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
