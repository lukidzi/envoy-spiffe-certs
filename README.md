## Notes

1. CA in `certs` directory is one that kuma generates by the default
2. Client and Server parameters should be similar to one kuma uses to generate
3. CA in `spiffe` directory is a different one with more valid CA
4. Client parameters should have just one spiffe ID

SPIRE sets
```yaml
                  custom_validator_config:
                    name: envoy.tls.cert_validator.spiffe
                    typed_config:
                      "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.SPIFFECertValidatorConfig
                      trust_domains:
                      - name: demo
                        trust_bundle:
                          filename: "./certs/ca.crt"
                      - name: example.org
                        trust_bundle:
                          filename: "./spiffe/ca.crt"
```
so on the server side I've set it

By default Envoy doesn't validate SAN/hash when only trustedBundle is set, and only this takes place, unless we set `SPIFFECertValidatorConfig`
```yaml
                  match_subject_alt_names:
                    - prefix: "spiffe://demo"
                    - prefix: "spiffe://example.org"
```


### Setup

1. Enter certs
2. Generate server cert
```bash
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -config server-san.cnf
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256 -extfile server-san.cnf -extensions v3_req
```

3. Generate client cert
```bash
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr -config client-san.cnf
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 -sha256 -extfile client-san.cnf -extensions v3_req
```

4. Generate client with different identities
```bash
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr -config client-san.cnf
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 -sha256 -extfile client-san.cnf -extensions v3_req
```
5. Start server
```bash
./envoy -c server.yaml -l debug
```

6. Start client
```bash
./envoy -c client.yaml -l debug
```

7. Start client-new-identity
```bash
./envoy -c client-new-identity.yaml -l debug
```

8. Do requests to 10000 and 10001 and request should succeed