# ECDSA Nonce Recovery Write up

# 問題文  
[ECDSA Nonce Recovery](https://id0-rsa.pub/problem/17/)

`
As part of signing something using DSA (digital signature algorithm) one must select a secret, cryptographically secure random number k to be used as a nonce. k must never be reused. Why you ask? Well you could [ask Sony](https://www.schneier.com/blog/archives/2011/01/sony_ps3_securi.html), or I could just tell you that you can recover k given two signature / message pairs that used the same k and signing key, which can lead to the signing key being compromised. I've signed two messages (z1,z2) with the same k (using the NIST curve P-192), resulting in the signatures (s1,r1) and (s2,r2). Your job is to recover k (submit your answer in hex). [Some reading to get started (of most relevance is section 2.3).](http://eprint.iacr.org/2015/839.pdf)
`
