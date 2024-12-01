https://www.youtube.com/watch?v=nsofIV11xXY


## intro

Starting off with a small reminder of what spring security is capable of

AuthenticationManager -> major building block of the authentication abstraction
UserDetailsService -> get user details

Keep in mind Spring Security does a **lot**. Also the things you **forget** about (cleaning up sessions, comparing passwords, setting them, ...)


## password history

increasing complexity:

1. plaintext -> bad bcs hackers
2. hashing w/ oneway function (no way to reverse it) -> lookup via rainbow tables
3. salts to combat rainbow tables
4. GPUs are able to calculate billions of hashes -> very easy to crack
5. 1 way adaptive functions now (parametrize how long it takes to calculate, trade-off)


## passwords demo

Spring securitys _defaultPasswordEncoder_ uses bcrypt and has support for you to seamlessly upgrade passwords from weaker (sha256) to stronger (bcrypt) (via UserDetailsPasswordService)


## webauthn or passkeys

- work in sb security not in main branch yet (webauthn)
- passkeys are cross device
- build upon asymmetric crypto (private key to produce a signature (kept by you), whilst a public key is used to establish proof it was you (available to everyone))
  - confidentiality (sign with other persons pubk, only he can decrypt with his pk)
  - authenticity (encrypt with my private key he can verify msg is yours by using my pub key)
- private keys are stored in cloud (keychain,1pass, ...): auth once and anywhere 
- can use a OTT (one time token)