## SetUp 
- Install vault 
    - https://www.vaultproject.io/downloads


curl --request POST --data '{"secret_shares": 1, "secret_threshold": 1}' http://127.0.0.1:8200/v1/sys/init | jq    