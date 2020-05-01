##Vault 

### Glossary 
- Storage Backend - A storage backend is responsible for durable storage of encrypted data. Backends are not trusted by Vault and are only expected to provide durability. The storage backend is configured when starting the Vault server.
- Barrier - The barrier is cryptographic steel and concrete around the Vault. All data that flows between Vault and the storage backend passes through the barrier. The barrier ensures that only encrypted data is written out, and that data is verified and decrypted on the way in. Much like a bank vault, the barrier must be "unsealed" before anything inside can be accessed.
- Secrets Engine - A secrets engine is responsible for managing secrets. Simple secrets engines like the "kv" secrets engine simply return the same secret when queried. Some secrets engines support using policies to dynamically generate a secret each time they are queried. This allows for unique secrets to be used which allows Vault to do fine-grained revocation and policy updates. As an example, a MySQL secrets engine could be configured with a "web" policy. When the "web" secret is read, a new MySQL user/password pair will be generated with a limited set of privileges for the web server.
- Audit Device - An audit device is responsible for managing audit logs. Every request to Vault and response from Vault goes through the configured audit devices. This provides a simple way to integrate Vault with multiple audit logging destinations of different types.
- Auth Method - An auth method is used to authenticate users or applications which are connecting to Vault. Once authenticated, the auth method returns the list of applicable policies which should be applied. Vault takes an authenticated user and returns a client token that can be used for future requests. As an example, the userpass auth method uses a username and password to authenticate the user. Alternatively, the github auth method allows users to authenticate via GitHub.
- Client Token - A client token (aka "Vault Token") is conceptually similar to a session cookie on a web site. Once a user authenticates, Vault returns a client token which is used for future requests. The token is used by Vault to verify the identity of the client and to enforce the applicable ACL policies. This token is passed via HTTP headers.
- Secret - A secret is the term for anything returned by Vault which contains confidential or cryptographic material. Not everything returned by Vault is a secret, for example system configuration, status information, or policies are not considered secrets. Secrets always have an associated lease. This means clients cannot assume that the secret contents can be used indefinitely. Vault will revoke a secret at the end of the lease, and an operator may intervene to revoke the secret before the lease is over. This contract between Vault and its clients is critical, as it allows for changes in keys and policies without manual intervention.
- Server - Vault depends on a long-running instance which operates as a server. The Vault server provides an API which clients interact with and manages the interaction between all the secrets engines, ACL enforcement, and secret lease revocation. Having a server based architecture decouples clients from the security keys and policies, enables centralized audit logging and simplifies administration for operators.

### Enable a Secrets Engine 
- To get started, enable the kv secrets engine. 
    - Each path is completely isolated and cannot talk to other paths. 
    - For example, a kv secrets engine enabled at foo has no ability to communicate with a kv secrets engine enabled at bar.
        - **vault secrets enable -path=kv kv**
    - The path where the secrets engine is enabled defaults to the name of the secrets engine. 
        - Thus, the following command is equivalent to executing the above command.    
        - **vault secrets enable kv**
- To verify our success and get more information about the secrets engine, use the vault secrets list command:        
    - **vault secrets list**
    - The sys/ path corresponds to the system backend. These paths interact with Vault's core system and are not required for beginners.
- To create secrets, use the kv put command.
    - vault kv put kv/hello target=world
- To read the secrets stored in the kv/hello path, use the kv get command.    
    - vault kv get kv/hello
- Create secrets at the kv/my-secret path.
    - vault kv put kv/my-secret value="s3c(eT"
- Read the secrets at kv/my-secret.    
    - vault kv get kv/my-secret
- Delete the secrets at kv/my-secret.
    - vault kv delete kv/my-secret

### What is a Secrets Engine
- Vault behaves similarly to a virtual filesystem. The read/write/delete/list operations are forwarded to the corresponding secrets engine, and the secrets engine decides how to react to those operations.
- This abstraction is incredibly powerful. It enables Vault to interface directly with physical systems, databases, HSMs, etc. But in addition to these physical systems, Vault can interact with more unique environments like AWS IAM, dynamic SQL user creation, etc. all while using the same read/write interface.