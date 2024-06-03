This repository references https://github.com/sweetdrake/certificate-verification.git to demonstrate how a certificate can be verified against an upstream public key.

### 1.Gen keys
	openssl genrsa -out rootCA.key 2048
	openssl genrsa -out intermediate.key 2048
	openssl genrsa -out leaf.key 2048

### 2.Self sign (CA)
	openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem

### 3.Gen intermediate CSR
	openssl req -new -sha256 -key intermediate.key -out intermediate.csr

### 4.CSR sign with CA key (with CA:True mark)
	openssl x509 -req -in intermediate.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out intermediate.pem -days 365 -sha256 -extfile <(echo "basicConstraints=critical,CA:TRUE")

### 5.Gen leaf CSR
	openssl req -new -sha256 -key leaf.key -out leaf.csr

### 6.CSR sign with intermediate key
	openssl x509 -req -in leaf.csr -CA intermediate.pem -CAkey intermediate.key -CAcreateserial -out leaf.pem -days 365 -sha256

### 7.Concatanate trust chain
	cat rootCA.pem intermediate.pem > trustChain.pem

### 8.Verify chain
	openssl verify -CAfile rootCA.pem intermediate.pem
	openssl verify -CAfile trustChain.pem leaf.pem
	openssl verify -CAfile intermediate.pem leaf.pem (Note: openssl verify trust chain until CA so intermediate cert cant use to verify trust chain

### 9.RSA signature verification (How leaf certificate has signature in the end)
	openssl asn1parse -inform PEM -in leaf.pem -strparse 4 -out leef.val.der
	openssl dgst -sha256 -sign intermediate.key -out leaf.signature leaf.val.der	

### 10.Print leaf.pem
	openssl x509 -in leaf.pem -text -noout

### 11. Compare both signature between No.9 and No.10