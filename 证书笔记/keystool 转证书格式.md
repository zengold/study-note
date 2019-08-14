keytool -import -v -alias test -file test.cer -keystore test.jks -storepass 123456 -noprompt



keytool -importkeystore -srckeystore  mycert.pfx -srcstoretype pkcs12 -destkeystore mycert.jks -deststoretype JKS

