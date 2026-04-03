<b><h1>Giriş</h1></b>

Son dönemde JA4 fingerprint konusunu anlamaya çalışırken önce TLS el sıkışmasının mantığını, özellikle de ClientHello mesajının ne taşıdığını daha detaylı anlamam gerektiğini fark ettim. Çünkü JA4 doğrudan bu katmandaki bazı özelliklerden üretilen bir fingerprint yapısına dayanıyor. <br>

Ağ güvenliği tarafında bir istemciyi yalnızca IP adresiyle ya da görünen uygulama bilgileriyle tanımak çoğu zaman yeterli olmuyor. Bu nedenle istemcinin bağlantı sırasında bıraktığı teknik izler önem kazanıyor. TLS ClientHello mesajı da bu izlerin en dikkat çekici örneklerinden biri. <br>

Bu yazıda JA4’ün ne olduğunu, neden ortaya çıktığını, ClientHello ile nasıl ilişkilendiğini ve JA3’e göre neden daha farklı bir yaklaşım sunduğunu kendi anladığım çerçevede toparlamaya çalışacağım.


<b><h1>Neden Fingerprinting Gerekli ? </h1></b>

Ağ güvenliğinde bir istemciyi tanımak, çoğu zaman görünen kimlik bilgilerinden daha fazlasını gerektirir. Çünkü görünen bilgiler değiştirilebilir, eksik olabilir ya da yeterince ayırt edici olmayabilir.Bu yüzden istemcinin bağlantı sırasında bıraktığı teknik izlerden yararlanmak gerekir. Fingerprinting tam da burada devreye girer. <br>

Fingerprinting, bir istemcinin ağ üzerindeki teknik davranışını ayırt etmeye yardımcı olur. Böylece benzer istemciler gruplanabilir, alışılmadık bağlantılar tespit edilebilir ve görünürlük artırılabilir. Tek başına kesin hüküm vermese de, fingerprinting özellikle tehdit avcılığı, olay analizi ve anomali tespiti açısından önemli bir veri noktasıdır.<br>

Özellikle TLS ClientHello mesajı, fingerprinting açısından dikkat çekici bir veri kaynağıdır. <br>


<b><h1>TLS ve ClientHello Nedir? </h1></b>

TLS (Transport Layer Security), istemci ile sunucu arasındaki dinlemeyi, verinin değiştirilmesini ve sahteciliği zorlaştıracak şekilde güvenli iletişim kurmak için kullanılan prokoldür. Uygulama verisi aktarılmadan önce iki tarafın "nasıl güvenli konuşacağız?" sorusunu çözdüğü bir hazırlık sürecini içermektedir. Bu hazırlık süreci genel olarak TLS handshake olarak adlandırılır. Handshake sırasında istemci ve sunucu; hangi TLS sürümününün kullanılacağını, hangi şifreleme seçeneklerinin uygun olduğunu ve bağlantının hangi güvenlik parametreleriyle devam edeceğini kararlaştırır. 

Başlangıç akışı kabaca istemcinin ClientHello göndermesiyle başlamaktadır. Ardından sunucu ServerHello ile yanıt verir ve devamında gerekli uzantılar, sertifika bilgileri ve finished mesajlarıyla güvenli oturum tamamlanır. Uygulama verisi ise bu aşamadan sonra aktarılır. Bu nedenle ClientHello mesajı fingerprinting açısından oldukça değerlidir. Çünkü istemci daha güvenli oturum tam kurulmadan önce hangi sürümleri desteklediğini, hangi şifreleme seçeneklerini tercih ettiğini ve hangi uzantılarla bağlantı kurmak istediğini göstermiş olur. Başka bir ifadeyle ClientHello, istemcinin “kim olduğunu” değilse bile “nasıl davrandığını” gösteren teknik bir iz bırakır. JA4 gibi yaklaşımlar da tam olarak bu izlerden yararlanır.

<b><h1>JA3 Fingerprint Nedir?</h1></b>
JA3, istemcinin gönderdiği TLS ClientHello içinden belirli alanları alır ve bunları tek bir metin dizisine dönüştürür. Salesforce’un açıklamasına göre bu alanlar şunlardır: TLS/SSL sürümü, cipher listesi, extension listesi, elliptic curve listesi ve elliptic curve point format listesi. 

<div align="center">
<pre>
JA3 = SSLVersion, Cipher, SSLExtension, EllipticCurve, EllipticCurvePointFormat
</pre>
</div>

Bu beş alan, aralarında virgül olacak şekilde birleştirilir; her alanın içindeki çoklu değerler ise tire ile ayrılır. 

Örneğin;

<div align="center">
<pre>
  769,47-53-5-10-49161-49162-49171-49172-50-56-19-4,0-10-11,23-24-25,0
</pre>
</div>

İstemci ClientHello mesajında ​​SSL uzantıları yoksa, alanlar boş bırakılır.
<div align="center">
<pre>
  769,4-5-10-9-100-98-3-6-19-18-99,,,
</pre>
</div>

Son aşamada ortaya çıkan bu metin MD5 ile hash’lenir ve 32 karakterlik JA3 değeri elde edilir.  JA3, istemcinin gönderdiği TLS ClientHello içinden belirli alanları alır ve bunları tek bir metin dizisine dönüştürür. 

<div align="center">
<pre>
769,47-53-5-10-49161-49162-49171-49172-50-56-19-4,0-10-11,23-24-25,0 --> ada70206e40642a3e4461f35503241d5
769,4-5-10-9-100-98-3-6-19-18-99,,, --> de350869b8c85de67a350c8d186f11e6
</pre>
</div>

<b><h1>JA4 Fingerprint Nedir?</h1></b>

<div align="center"><img width="958" height="541" alt="image" src="https://github.com/user-attachments/assets/3dca0607-95ae-40d3-bce3-74ddf272f141" />
<br><b>Görsel kaynağı:Github_FoxIO-LLC/ja4</b></div><br>


JA4, TLS istemcisinin gönderdiği <b>ClientHello</b> mesajına bakarak istemciye ait bir fingerprint üretir bu sayede şifrelemeyi bozmaya gerek kalmadan ağda nelerin konuşulduğunu anlamanıza olanak tanır. Temel amaç, istemcinin TLS bağlantısını nasıl başlattığını daha okunabilir, parçalanabilir ve analizde kullanılabilir bir formata dönüştürmektir. FoxIO'nun tanımına göre JA4,TLS <b>ClientHello</b> içindeki belirli öznitelikleri kullanır ve sonucu <b>a_b_c</b> biçiminde üç parçalı bir fingerprint olarak üretir. 

<b>JA4 Algoritması</b>

<pre>
(protokol) (TLS sürümü) (SNI var/yok) (cipher sayısı) (extension sayısı) (ALPN özeti)_(cipher listesinin kısaltılmış SHA-256 özeti)_(extension listesi+signature algorithms bilgisinin kısaltılmış SHA-256 özeti)
</pre>

**JA4_a Bölümü**
Bu bölüm sırayla FoxIO dökümanında şu şekilde tanımlanır;
- Prokotol tipi **>>>** TCP için "t", QUIC için "q", DTLS için "d" kullanılır. 
- TLS sürümü **>>>** TLS sürümü iki karakterle gösterilir. Örneğin; TLS 1.3 için 13, TLS 1.2 için 12 kullanılır. 
- SNI (Server Name Indication) (istemcinin hangi alan adına gitmek istediğini belirtir.) **>>>** Eğer SNI uzantısı varsa fingerprint'te d, yoksa i yer alır. Bazı istemciler alan adı üzerinden bağlanır, bazıları doğrudan IP ye gider ya da SNI göndermez.
- Cipher suite sayısı **>>>** Şifreleme paketlerinin sayısı iki karakterli olmalıdır. Yani ClientHello paketinde 6 şifreleme paketi varsa, değer "06" olmalıdır. 
- Extension sayısı **>>>** Şifreleme algoritmalarını saymakla aynıdır. 
- ALPN (Application-Layer Protocol Negotiation) değerinden türetilen iki karakter **>>>** Bu, TLS anlaşması tamamlandıktan sonra uygulamanın iletişim kurmak istediği protokolü temsil eder.İlk ALPN değerinin ilk ve son karakteri alınır. Mesela h2, HTTP/2 yi işaret eder. (Wireshark'ta bu alan tls.handshake.extensions_alpn_str altında yer almaktadır.) "00", ALPN'nin olmadığını gösterir. 

QUIC, TLS 1.3 iletişimini UDP paketleri içinde taşıyan ve HTTP/3 tarafından kullanılan bir protokol olarak öne çıkar. Google kökenli olması sebebiyle, Google servislerinin yaygın kullanıldığı kurumsal yapılarda ağ trafiğinin büyük bir kısmı QUIC üzerinden gerçekleşebilir; bu nedenle bu trafiğin izlenmesi kritik kabul edilir.QUIC, TLS 1.3 iletişimini UDP paketleri içinde taşıyan ve HTTP/3 tarafından kullanılan bir protokol olarak öne çıkar. Google kökenli olması sebebiyle, Google servislerinin yaygın kullanıldığı kurumsal yapılarda ağ trafiğinin büyük bir kısmı QUIC üzerinden gerçekleşebilir; bu nedenle bu trafiğin izlenmesi kritik kabul edilir.


**JA4_b Bölümü** <br>

Bu bölüm sırayla FoxIO dökümanında şu şekilde tanımlanır;
İstemcinin sunduğu cipher suite listesinin kısa özetidir. FoxIO'ya göre burada önce cipher hex kodları alınır, GREASE değerleri çıkarılır, kalan liste hex sıraya göre sıralanır ve ardından bu listenin SHA-256 özeti alınır. Sonuç, 12 karaktere kısaltılarak JA4_b bölümüne yazılır. 

İki istemcinin aynı ya da benzer cipher davranışı gösterip göstermediği hızlıca karşılaştırılabilir. 

**JA4_c Bölümü** <br>

Extension listesi ile signature algorithms bilgisinin kıs aözetidir. Teknik dökümantasyona göre burada extension hex kodları GREASE değerleri çıkarılarak ele alınır ve extension listesi sıralanır. Ardından bu veri, signature algorithms bilgisi ile birlikte işlenir ve sonuç tekrar kısaltılmış SHA-256 biçiminde üretilir. 

JA4_C, istemcinin extension tarafındaki davranışını daha çok temsil eder. JA4_B data çok cipher tarafını tanımlarken, JA4_C extension ve imza algoritmaları tarafındaki profili yansıtır. 

Özetle JA4, TLS ClientHello mesajındaki belirli özellikleri kullanarak istemcinin bağlantı davranışını üç parçalı bir fingerprint biçiminde temsil eder. İlk bölüm bağlantının temel karakteristiğini okunabilir şekilde sunarken, ikinci ve üçüncü bölümler cipher ve extension/signature algoritmaları tarafındaki davranışı özetler. Bu yapı sayesinde JA4 yalnızca teknik bir tanımlayıcı değil, aynı zamanda operasyonel analizde kullanılabilecek anlamlı bir veri noktası hâline gelir.

**JA4 Neden Önemlidir?** <br>

JA4’ün önemli olmasının nedeni, yalnızca TLS istemcileri için yeni bir fingerprint üretmesi değildir. Asıl önemli nokta, istemcinin bağlantı davranışını daha okunabilir ve daha karşılaştırılabilir bir formata dönüştürmesidir. Bu da ağ görünürlüğünü artırma, benzer istemcileri gruplama, anomali tespiti yapma ve şüpheli istemci davranışlarını ayırt etme açısından güvenlik ekiplerine fayda sağlar.
Özellikle TLS trafiğinin yoğun olduğu ortamlarda JA4, yalnızca IP adresi, domain veya user-agent gibi kolay değişebilen göstergelere bağlı kalmadan istemcinin teknik karakteristiğine dair ek bağlam sunabilir.


<b><h1>JA4 Fingerprint Nedir?</h1></b>

**CTI Ekipleri :** JA4, IP veya domain gibi kolay değiştirilebilen göstergelerin ötesinde istemci davranışını ilişkilendirmek için kullanılabilir. Benzer JA4 değerleri, aynı TLS istemci profilinin farklı zamanlarda veya farklı altyapılar üzerinden tekrar ettiğini gösterebilir. Bu nedenle JA4, IOC’den çok davranışsal pivot olarak değerlidir.


**SOC Ekipleri :** JA4, bir olay sırasında görülen şüpheli TLS istemcilerini ayırmak ve aynı davranışı gösteren başka hostları bulmak için kullanılabilir. Analist, şüpheli bir JA4 değerini geriye dönük olarak aratarak aynı fingerprint’in başka hangi sistemlerde, hangi süreçlerle ve hangi hedeflere karşı tekrar ettiğini inceleyebilir. Böylece olayın kapsamı daha net anlaşılabilir.

**Detection Ekipleri :** JA4, yeni analitikler ve korelasyon kuralları geliştirmek açısından anlamlıdır. Örneğin kurumda beklenen tarayıcı profillerinden sapmalar, SNI içermeyen alışılmadık TLS istemcileri veya aynı JA4'ün şüpheli süreçlerle birlikte görülmesi gibi örüntüler kural mantığına dönüşebilir.

<b><h1>Analiz Örneği</h1></b>

<div align="center"> <img width="829" height="518" alt="Ekran görüntüsü 2026-04-03 044621" src="https://github.com/user-attachments/assets/18d4169d-2784-4977-8876-6ae9ce7d814e" />
 <br><b>Görsel kaynağı:blog.cloudflare.com/ja4-signals</b></div><br> </div>

<ul>
  <li><b>Protokol :</b></li> TLS 1.3 sürümünü temsil etmektedir. Bu istemcinin güvenli protokolü kullandığını doğrulamaktadır. 
  <li><b>SNI Bilgisi :</b></li> Domain barındırmaktadır. (IP adresi, SNI'nın yokluğunu gösterir.)
  <li><b>Extension Sayısı : </b></li> Bu örnekte 16 olarak görünmektedir. <br>
 
 **Not :** Extension sayısı, istemci ile ilgili desteklenen işlevsellik özelliğini belirlemeye yardımcı olur. Extension sayısı ne kadar fazlaysa istemci daha modern ve daha özelliklidir diyebiliriz. Tarayıcı trafiğinde extension sayısı çoğu zaman daha zengin görünür. Düşük extension sayısı, sade ve daha dar amaçlı istemci profili yansıtmaktadır. Bu değer tek başına zararlı veya meşru ayrımı yapmak için yeterli değildir. Düşük extension sayısı bazı özel kurumsal uygulamalarda, ajanlarda veya sadece TLS kullanan istemcilerde de görülebilir. 

 <li><b>ALPN Değeri : </b></li> HTTP/2'dir. (Web performansı için istemcinin protokol tercihlerini gösterir.)

 <li><b>Cipher Hash : </b></li> 8daaf6152771 (Onaltılık düzende sıralanmış şifreleme paketleri listesinin kısaltılmış bir SHA256 özetidir.)

 <li><b>Extension Hash : </b></li> 02713d6af862 (Bu özet değerlerden ham listeyi geri çıkaramayız. Aynı cipher veya extension davranışını paylaşıp paylaşmadığını karşılaştırmak için kullanılır.)
</ul>
<br>

 **Örneği Değerlendirme** <br>
Bu örnekte dikkat çeken asıl nokta, JA4 değerinin tek başına varlığı değil; aynı JA4 fingerprint’inin birden fazla hostta, benzer zaman aralıklarında ve alışılmış tarayıcı süreçleri dışında kalan süreçlerle birlikte görülmesidir. Özellikle <code>powershell.exe</code>, <code>rundll32.exe</code>, <code>mshta.exe</code> ve <code>curl.exe</code> gibi süreçlerin aynı JA4 ile tekrar etmesi, analist açısından bu davranışın araştırılmaya değer olduğunu düşündürebilir. <br>
Ayrıca loglarda hedefin doğrudan IP adresi olması, SNI alanının boş görünmesi ve ALPN bilgisinin bulunmaması, bu trafiğin tipik bir modern browser HTTPS davranışına benzemediğini düşündürebilir. Bu durum tek başına zararlı anlamına gelmez; ancak ek bağlamla birlikte değerlendirildiğinde anlamlı bir inceleme başlangıcı oluşturur.<br>

Bu nedenle JA4 her zaman süreç bilgisi, hedef, kullanıcı, sertifika, DNS, proxy ve EDR logları ile birlikte değerlendirilmelidir. 

**Sorulabilecek Sorular**
- Aynı JA4 değeri başka hangi gostlarda görülüyor?
- Bu fingerprint hangi süreçlerle birlikte ortaya çıkmaktadır?
- Hep aynı hedef IP veya benzer hedeflere mi gidiyor?
- Bu hostlarda yakın zamanda şüpheli komut çalıştırılmış mı?
- Aynı hostlarda DNS, proxy veya EDR telemetrisi tarafında ek bir anomali var mı?


JA4, daha geniş bir fingerprint ailesinin yalnızca TLS istemci tarafına odaklanan parçasıdır. Farklı protokol ve katmanlar için geliştirilen JA4 türleri de bulunmaktadır. Bu yazının odak noktası TLS istemci fingerprinting olduğu için, diğer JA4 türlerine kısa bir çerçeve sunmak daha doğru olacaktır. 


| Tür | Kapsadığı konu |
|---|---|
| **JA4** | TLS istemci fingerprinting | 
| **JA4S** | TLS sunucu yanıtı / oturum fingerprinting |
| **JA4H** | HTTP istemci fingerprinting | 
| **JA4L** | İstemciden sunucuya gecikme / light distance ölçümü | 
| **JA4LS** | Sunucudan istemciye gecikme / light distance ölçümü |
| **JA4X** | X.509 TLS sertifika fingerprinting |
| **JA4SSH** | SSH trafik fingerprinting |
| **JA4T** | TCP istemci fingerprinting |
| **JA4TS** | TCP sunucu yanıtı fingerprinting | 
| **JA4TScan** | Aktif TCP fingerprint taraması |
| **JA4D** | DHCP fingerprinting | 
| **JA4D6** | DHCPv6 fingerprinting |


<b><h1>REFERANSLAR</h1></b>

- FoxIO — [JA4 Technical Details](https://github.com/FoxIO-LLC/ja4/blob/main/technical_details/JA4.md)
- FoxIO — [JA4+ Technical Details](https://github.com/FoxIO-LLC/ja4/blob/main/technical_details/README.md)
- FoxIO — [JA4+ for Zeek](https://github.com/FoxIO-LLC/ja4/blob/main/zeek/README.md)
- FoxIO — [JA4+ for Wireshark](https://github.com/FoxIO-LLC/ja4/blob/main/wireshark/README.md)
- FoxIO - [JA4+ Network Fingerprinting] (https://blog.foxio.io/ja4%2B-network-fingerprinting)
- FoxIO - [JA4T: TCP Fingerprinting] (https://blog.foxio.io/ja4t-tcp-fingerprinting)
- AWS WAF — [JA4Fingerprint](https://docs.aws.amazon.com/waf/latest/APIReference/API_JA4Fingerprint.html)
- Cloudflare — [JA4 Signals](https://blog.cloudflare.com/ja4-signals/)
- RFC 8446 — [The Transport Layer Security (TLS) Protocol Version 1.3](https://www.rfc-editor.org/rfc/rfc8446.html)
- RFC 8701 — [Applying GREASE to TLS Extensibility](https://www.rfc-editor.org/rfc/rfc8701.html)
- Salesforce - [JA3 - A method for profiling SSL/TLS Clients] (https://github.com/salesforce/ja3/blob/master/README.md)























