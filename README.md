# ECDSA Nonce Recovery Write up

# 問題文  
今回は、id0-rsaの[ECDSA Nonce Recovery](https://id0-rsa.pub/problem/17/)について紹介します。
以下問題文になります。

As part of signing something using DSA (digital signature algorithm) one must select a secret, cryptographically secure random number k to be used as a nonce. k must never be reused. Why you ask? Well you could [ask Sony](https://www.schneier.com/blog/archives/2011/01/sony_ps3_securi.html), or I could just tell you that you can recover k given two signature / message pairs that used the same k and signing key, which can lead to the signing key being compromised. I've signed two messages (z1,z2) with the same k (using the NIST curve P-192), resulting in the signatures (s1,r1) and (s2,r2). Your job is to recover k (submit your answer in hex). [Some reading to get started (of most relevance is section 2.3).](http://eprint.iacr.org/2015/839.pdf)

	z1 = 78963682628359021178354263774457319969002651313568557216154777320971976772376
	s1 = 5416854926380100427833180746305766840425542218870878667299
	r1 = 5568285309948811794296918647045908208072077338037998537885

	z2 = 62159883521253885305257821420764054581335542629545274203255594975380151338879
	s2 = 1063435989394679868923901244364688588218477569545628548100
	r2 = 5568285309948811794296918647045908208072077338037998537885

	n = 6277101735386680763835789423176059013767194773182842284081

## 問題文の理解
デジタル署名アルゴリズムは暗号学的に安全で、かつ秘密の乱数kを１度だけ選択して使用しています。
２度使えない理由として、二つの別々のメッセージに同じ乱数kで署名を行った場合、kが復元可能であるという脆弱性が存在するからです。

この問題では、z1とz2に同じ乱数kで暗号化して、シグネチャ(s1,r1)とシグネチャ(s2,r2)を生成しています。
問題文の趣旨としては、シグネチャ(s1,r1)とシグネチャ(s2,r2)を用いて乱数kを復元せよというものだと考えました。