
# poc pki
Refer to [pki infrastructure in linux](https://evilshit.wordpress.com/2013/06/19/how-to-create-your-own-pki-with-openssl)


```bash
export ALTNAME="DNS:www.antoniocaccamo.net, DNS:api.antoniocaccamo.net, DNS:www2.antoniocaccamo.net"

mkdri certs

mkdir -p ./RootCA/private ./RootCA/certs ./RootCA/newcerts RootCA/crl
touch ./RootCA/index.txt
echo '01' > ./RootCA/serial

mkdir -p ./SubCA/private ./SubCA/certs ./SubCA/newcerts SubCA/crl
touch ./SubCA/index
echo '01' > ./SubCA/serial

```

## create a self signed root certificate




1. random file

        $ openssl rand -out  RootCA/private/.randRootCA 8192

2. private key

        $ openssl genrsa -aes256 -out  RootCA/private/root.CA.key -rand  RootCA/private/.randRootCA

3. public cert

        $ openssl req -new -x509 -days 3650 -key RootCA/private\root.CA.key            \
             -out RootCA/root.CA.crt -config openssl.conf                              \
             -subj "/C=IT/CN=HeadQuarter/OU=IT/O=antoniocaccamo.net/L=Milano/ST=Milano"


        $ openssl x509 -text -in RootCA/root.CA.crt


## create a sub ca certificate (signed by the root ca)

1. random file

        $ openssl rand -out ./SubCA/private/.randSubCA 8192

2. private key

        $ openssl genrsa -aes256 -out ./SubCA/private/sub.CA.key -rand ./SubCA/private/.randSubCA    

3. public certificate request

        $ openssl req -new -key ./SubCA/private/sub.CA.key -out SubCA/sub.ca.csr \
                -subj "/C=IT/CN=SW/OU=IT/O=antoniocaccamo.net/L=Milano/ST=Milano" \
                -config openssl.conf

        $ openssl req -text -in SubCA/sub.ca.csr
        

4. root ca sign the csr

        openssl ca -name CA_RootCA -in csrs/sub.ca.csr -out SubCA/sub.ca.crt \
            -extensions subca_cert -config openssl.conf

        openssl x509 -text -in SubCA/sub.ca.crt

## create a server certificate (signed by the sub ca)            

        mkdir server

        openssl rand -out ./server/.randServer 8192

        openssl genrsa -aes256 -out ./server/server.key -rand ./server/.randServer

        openssl req -new -key ./server/server.key \
         -subj "/C=IT/CN=www.antoniocaccamo.net/OU=IT/O=antoniocaccamo.net/L=Milano/ST=Milano" \
         -out csrs/server.csr -config openssl.conf 

         openssl ca -name CA_SubCA -in csrs/server.csr -out certs/server.crt  \
                -extensions server_cert -config openssl.cnf# poc-pki
