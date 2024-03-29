# ECDSA Nonce Recovery Write up

# 問題文  
今回は、id0-rsaの[ECDSA Nonce Recovery](https://id0-rsa.pub/problem/17/)について紹介します。
中身として、DSAの脆弱性を突く問題になります。
以下問題文になります。

As part of signing something using DSA (digital signature algorithm) one must select a secret, cryptographically secure random number k to be used as a nonce. k must never be reused. Why you ask? Well you could [ask Sony](https://www.schneier.com/blog/archives/2011/01/sony_ps3_securi.html), or I could just tell you that you can recover k given two signature / message pairs that used the same k and signing key, which can lead to the signing key being compromised. I've signed two messages (z1,z2) with the same k (using the NIST curve P-192), resulting in the signatures (s1,r1) and (s2,r2). Your job is to recover k (submit your answer in hex). [Some reading to get started (of most relevance is section 2.3).](http://eprint.iacr.org/2015/839.pdf)

	z1 = 78963682628359021178354263774457319969002651313568557216154777320971976772376
	s1 = 5416854926380100427833180746305766840425542218870878667299
	r1 = 5568285309948811794296918647045908208072077338037998537885

	z2 = 62159883521253885305257821420764054581335542629545274203255594975380151338879
	s2 = 1063435989394679868923901244364688588218477569545628548100
	r2 = 5568285309948811794296918647045908208072077338037998537885

	n = 6277101735386680763835789423176059013767194773182842284081
# 問題文の理解
デジタル署名アルゴリズムは暗号学的に安全で、かつ秘密の乱数kを１度だけ選択して使用しています。
２度使えない理由として、二つの別々のメッセージに同じ乱数kで署名を行った場合、kが復元可能であるという脆弱性が存在するからです。

この問題では、ある2つのメッセージを暗号化して、暗号化した結果がそれぞれz1とz2、シグネチャ(s1,r1)とシグネチャ(s2,r2)を生成しています。
問題文の趣旨としては、シグネチャ(s1,r1)とシグネチャ(s2,r2)を用いて乱数kを計算せよというものです。


#️ ECDSAとは
ECDSA(Elliptic Curve Digital Signature Algorithm)とは楕円曲線デジタル署名アルゴリズムのことです。
詳しくは<https://esac.jipdec.or.jp/intro/publicKey.html>、<https://www.jssec.org/dl/20170710_03_urushima.pdf>、[Wikipedia](https://ja.wikipedia.org/wiki/楕円曲線DSA)等を参照してみてください。
ざっくりいうと、署名における公開鍵やシステムのパラメータの決定に楕円曲線を利用するというものです。
結果から言えば、この問題ではDSA方式の内容をサーベイできれば十分だったのでもしもし気になる方がいましたらご確認ください。

# 回答方針
本問題では、アルゴリズムの署名部分に着目します。
r1とr2が同じ値であることから、問題のシグネチャは同じ乱数kを用いて生成されたシグネチャであることがわかります。
wikipediaより、乱数kは、

<img src="https://latex.codecogs.com/gif.latex?k&space;=&space;\frac{H(m_1)-H(m_2)}{s_1-s_2}\bmod&space;n" />
で計算できます。
ここではH(m1)=z1,H(m2)=z2です。
しかし、そのまま計算したらエラーを吐きます。(TypeError: 'float' object cannot be interpreted as an integer)

そこで、式の中における
<img src="https://latex.codecogs.com/gif.latex?(s_1-s_2)^{-1}\bmod&space;n" />
部分を工夫して計算しなくてはいけません。そこで、<https://www.pebblewind.com/entry/2017/07/04/215544>を参考にすると、<img src="https://latex.codecogs.com/gif.latex?(s_1-s_2)^{-1}\bmod&space;n" />は<img src="https://latex.codecogs.com/gif.latex?(s_1-s_2)^{n-2}\bmod&space;n" />というべき乗の計算式で計算できます。

以上の方針から、プログラムsame_k_attack.pyを作成しました。

	z1 = 78963682628359021178354263774457319969002651313568557216154777320971976772376
	s1 = 5416854926380100427833180746305766840425542218870878667299
	r1 = 5568285309948811794296918647045908208072077338037998537885

	z2 = 62159883521253885305257821420764054581335542629545274203255594975380151338879
	s2 = 1063435989394679868923901244364688588218477569545628548100
	r2 = 5568285309948811794296918647045908208072077338037998537885

	n = 6277101735386680763835789423176059013767194773182842284081

	def mod_invese(x):
		return pow(x, n - 2, n) #x^(n - 2) % n

	try:
		k = (z1 - z2)*mod_invese(s1 - s2) % n
		print(hex(k)[2:])#hex(k)の２文字目まで飛ばす.hex(k):kを１６進数で表記
	except:
		print("Error")

hex(k)(=0xabad1dea)をそのまま入れるとAcceptされず、最初の0xを除いた数値(=abad1dea)が正解になります。


# 感想
DSA暗号の脆弱性を知ることができました。

この問題では、r1とr2が同じ値であることに気づいて回答の方針を立てる力が必要と思われます。

Cryptは知識の積み重ねが大事と感じました。

参考にしたWrite upにはsame k attackと書いてあったのですが、ググってもあまりヒットしなかったので攻撃の具体的な名前はわかりませんでした。

# 参考にしたWriteup
<http://kent056-n.hatenablog.com/entry/2018/10/26/011953>

