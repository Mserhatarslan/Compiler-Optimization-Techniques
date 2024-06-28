# Compiler-Optimization-Techniques

Kod Optimizasyonu - Oğuz Karan

Biraz teknik bir konu. Detaylı bir konu. Programlamayı ilerlettikçe daha da fazla ihtiyaç duyacaksınız. 

Kod optimizasyonu nedir? 
Derleyiciler arasındaki en önemli kalite göstergelerinden biri kod optimizasyonudur. Kod optimizasyonu “kodun işlevini değiştirmeden onu daha etkin biçimde çalıştırmak için yeniden düzenleme süreci”dir. Aslında optimizasyon terimi yanlış uydurulmuştur. Optimizasyon eniyileme demektir. Halbuki derleyicilerin böyle bir iddiası yoktur. 

Derleyiciler kodları onların işlevleri değişmemek üzere yeniden düzenleyerek daha iyi hale getirirler. Bu süreç bir optimizasyon süreci değil iyileştirme (improvement) sürecidir. 

Fakat optimizasyon terimi bu yanlışlıkla bilinmesine rağmen yaygın olarak kabul görmüştür ve kullanılmaktadır. 

Kod Optimizasyonu İle İlgili Önemli Noktalar
Optimizasyon kaynak kod üzerinde değil kod üzerinde yapılan bir faaliyettir. Yani optimizasyon ile bizim kaynak kod dosyamızda bir değişiklik olmaz. 

Optimizasyon sırasından kodda hiçbir anlam değişikliği olmaması gerekir. Yani optimize edilmemiş kod ile edilmiş kod tamamen aynı davranışı göstermelidir. Bu konuda standardın kullandığı terim “observable behavior’dur. Derleyici, kodun gözlenebilir davranışın bir değişiklik meydana getirmediği sürece istediği değişikliği/optimizasyonu yapabilir. Ayrıca bu as if rule olarak da geçer. 

Optimizasyon genel olarak “hız” ya da “kod büyüklüğü” temel alınarak yapılabilir. Fakat baskın ölçüt “hız” optimizasyonudur. 

Derleyiciler kodun daha hızlı çalışmasını sağlarken kodu büyütebilirler. 
Baskın ölçüt hızdır, genellikle hız optimizasyonu hedeflenir. 
Derleyiciler default olarak kodun daha hızlı çalışmasını hedeflerler. 

Pek çok derleyicide optimizasyon çeşitli derleme seçenekleri (switch) ile kontrol altına alınabilmektedir. 
Yani biz programcı olarak derleyicilere hangi tarz optimizasyonu yapması gerektiğini, hangilerini yapmaması gerektiğini söyleyebiliriz.
Hatta programcı isterse optimizasyonu tamamen de kapatabilir. Bu durumda derleyici kodu optimize etmeye çalışmaz. 

Kod optimizasyonu C/C++ standartlarının ele aldığı bir konu değildir. 
Ancak standartlarda “derleyicinin programdaki semantik işlevin değişmemesi şartıyla kodu yeniden düzenleyerek derleyebileceği” belirtilmiştir. 

Bir dilin dil yapan şey 2 özelliktir. Birincisi sentaks, semantik. Semantik anlam demek.
Derleyicilerin kod optimizasyonları halen tek akışlı ya da tek thread’li sistemlere göre yapılmaktadır. Yani derleyici kodu optimize ederken programda tek bir akış varmış gibi düşünür. 

Multi-thread sistemlerde derleyici optimizasyon yapamaz mı ? Yapılmıyor dedi hoca. 
Örneğin derleyici global bir değişken kullanırken onun başka bir thread’de kullanılıp kullanılmadığını düşünmez. 

Dolayısıyla çok thread’li uygulamalarda programcı bu durumu göz önüne almalıdır. Çok thread’li uygulamalarda yalnızca global değişkenlerin kullanıldığı durumda böyle bir potansiyel tehlike oluşmaktadır. Bunlara da volatile gibi anahtar sözcüklerle ya da bazı diğer yöntemlerle önlem alınabilmektedir. 

# Derleyicilerin Yaptığı Tipik Optimizasyonlar; 

Derleyicilerin yaptığı tipik optimizasyonların programcısı tarafından bilinmesi önemlidir. Çünkü böylece derleyicilerin hangi optimizasyonları bizim için yapabildiği ve hangi maliyetlerle yapabildiğini anlayabilliriz. Gerektiğinde bu optimizasyon sürecine katkıda bulunabilir ya da süreci daha iyi kontrol altına alabiliriz. 

Optimizasyonlar yerel (local), global (global) ve döngü optimizasyonları olarak da sınıflandırılabilmektedir. 

Yerel optimizasyonlar kodun küçük bir bölümü üzerinde etkili olur. 
Global optimizasyonlar birden fazla fonksiyonu içine alacak biçimde geniş bölgeyi etkileyecek biçimde yapılır.
Döngü optimizasyonları döngü içlerini optimize etmeyi hedefler. 

Derleyiciler eklenti ile de optimizasyon yapabilir. 

Neden döngüler bu kadar önemli? Uygulamanın çeşidi ne olursa olsun bu uygulamalar ömürlerinin büyük bir bölümünü döngülerde geçirirler. 

# Common Subexpression Elimination (Ortak Alt İfadelerin Elimine Edilmesi)

Bir grup işlemde ortak olan bazı alt ifadelerin her defasında yeniden yapılması yerine derleyici bunların sonuçlarını bir yazmaçta ya da geçici değişkende tutup onları kullanabilir. Örneğin;
```C++

a = x + y + 10;
b = x + y + 20;
c = x + y + 30; 
```

Burada derleyici x+y toplamını her defasında tekrardan hesaplamak yerine bu toplamı geçici bir değişkende ya da yazmaçta (register) tutarak oradan alıp kullanabilir. 

# Loop Invariant (Döngü İçerisinde Değişmeyen İfadelerin Döngü Dışına Çıkartılması) 

Bu optimizasyon temasında döngü içerisindeki ifade döngü içerisinde kullanılmıyorsa döngünün dışına alınır. Örneğin; 
```C++
for (i = 0; i < 100; ++i){
	total+=i;
	x = 10;
}
```

Burada derleyici x = 10 işleminin döngünün çalışmasıyla bir ilgisi yoktur. Yanii bu işlemin her yenilemede yeniden yapılmasına gerek yoktur.

Derleyici kodu aşağıdaki gibi ele alabilir; 
```C++

for (i = 0; i < 100; ++i){
	total+=i;	
}
x = 10;
```

Microsoft’un C derleyicisinde yukarıdaki kod analiz edildiğinde optimizasyon seçeneği kapalı olduğunda derleyicinin x = 10 ifadesini döngünün dışına çıkarmadığı açık olduğu durumda çıkardığı görülmüştür. 

Loop Invariant (Döngü içerisinde Değişmeyen İfadelerin Döngü Dışına Çıkartılması)

Fonksiyon çağrıları (hele ki derleyici fonksiyonun içini göremiyorsa) döngü dışına çıkartılamaz. Örneğin;

```C++
for (i = 0; i< strlen(str); ++i){

}
```

Burada programcı derleyicinin optimizasyon yaparak strlen fonksiyonunu bir kez çağırmasını beklememelidir. 
Derleyiciler standard C fonksiyonlarının ne yaptığını bilmediği için onlara normal bir fonksiyon muamelesi yaparlar. Bu nedenle derleyici strlen fonksiyonunu her defasında çağırmak zorundadır. Döngü içerisinde yapılan çağrılan döngü dışına çıkartılamayabileceğine dikkat ediniz. Yukarıdaki kodu programcı şöyle düzenlemeliydi; 

```C++
len = strlen(str); 
for (i = 0; i< len ; ++i){
}
```

Peki derleyici döngü içerisindeki çağrıları hiç mi döngü dışına çıkartamaz ? 
İşte eğer çağrı yapılan fonksiyon kodun içerisindeyse ve bir ya etkiye yol açmıyorsa derleyiciler onu döngü dışına çıkartabilirler. 

Bunu bilmesi gerekiyor. Bunu nasıl bilecek ?

Bazı C derleyicileri de (örneğin Microsoft derleyicileri de ve gcc derleyicileri de) “intrinsic” fonksiyon kavramı vardır. 
intrinsic fonksiyonların ne yaptıkları derleyici tarafından bilinir. Böylece derleyici o çağrıları optimize edebilmektedir. Gerçekten de Microsoft derleyicileri de ve gcc derleyicilerinde bazı standart C fonksiyonları intrinsic fonksiyonlardır. Bu intrinsic fonksiyonlar için prototip bildirimi de gerekmez. 

Tabii intrinsic fonksiyon kavramı C standartlarında yoktur. Bu neden taşınabilirlik bakımından bunların intrinsic olarak kullanılması taşınabilirliği bozabilir. İntrinsic fonksiyonlar için derleyiciler doğrudan call komutunu kaldırarak sanki onlar inline fonksiyonlarmış gibi kod da yerleştirilmektedir. 

# Constant Folding (Sabit İfadelerinin Derleme Aşamasında Ele Alınması) 

Bilindiği gibi sabit ifadelerinin sayısal değerleri derleme aşamasında hesaplanabilmektedir. Örneğin; 

a = 100 + 2 * 5 + b; 
Böyle bir ifadedeki 100 + 2* 5 alt ifadesinin programın çalışma zamanına bırakılmasının bir anlamı yoktur. 

Bu ifade derleme aşamasında bir kez yapılıp kod ona göre üretilebilir. Örneğin; 

a = 110 + b; 

Bu kod yukarıdakiyle aynı anlamdadır. Sabit ifadelerinin derleme aşamasında ele alınması çok temel bir optimizasyondur. Derleyicilerin optimizasyon seçenekleri kapalı olsa bile derleyiciler bu optimizasyonu default olarak yapabilmektedir. 



# Dead Code Elimination (Etkisiz Kodların Elimine Edilmesi)

Derleyiciler etkisi olmayan kod parçalarını optimizasyon sırasında tamamen koddan çıkartabilmektedir. 

Döngüye girersem boşu boşuna iş var. 

Burada aslında döngünün de total değişkenin de result değişkenin de koda bir etkisi yoktur. Yani başka bir deyişle program şöyle olsaydı da yazan için değişen bir şey olmazdı; 

Çünkü her ne kadar total değişkeni üzerinde bir toplama yapılmışsa da bu toplam başka bir yerde kullanılmamıştır. Benzer biçimde result değişkeni de bildirilmiştir fakat kullanılmamıştır. 

Tabii bir kodun etkisinin olmadığı iyi bir biçimde analiz edilmelidir. Bazı kodlar dolaylı yan etkilere sahip olabilmektedir. Şimdi aklınıza aşağıdaki gibi bir kodun etkisiz olup olmadığı sorusu gelebilir. 
```C++
for (i = 0; i < 100; ++i){
	foo();
}
```
Bu kod optimizasyon seçenekleri açılmışsa derleyiciler tarafından etkisiz kod oldukları gerekçesiyle elimine edilebilirler. Halbuki programcı bu kodu zaman kaybı oluşturmak için yazmış olabilir. Etkisi olup olmadığı fonksiyonun kodları bilinmeden söylenemez. 

# Loop Unswitching (Koşul İçeren Döngülerin Yeniden Düzenlenmesi) 

Bir döngü içerisinde döngüye bağlı olmayan bir koşul altında farklı işlemler yapılmak istenebilir. 
Bu durumda döngünün içerisinde her defasında bu koşula bakmak yerine koşulun içerisinde döngü kullanmak daha hızlı çalışmaya yol açabilir. 
Kodu büyütebilmektedir. 

Örneğin flag değişkeni 1 ise döngü içerisinde bir işlem 0 ise başka bir işlem yapılıyor olsun; 

```C++
for (i = 1; i < 100000 ; ++i){
	ifade1;
	if(flag)
		ifade2;
	else 
		ifade3;
}
```
Flag değişkeni döngü içerisinde bir yerde değişmemiş. 
Döngü şöyle düzenlenirse daha hızlı çalışacak hale getirilebilir. 

```C++
if (flag)
		for (i = 1; i < 100000 ; ++i){
			ifade1;
			ifade2;
	}
else
		for (i = 1; i < 100000 ; ++i){
	ifade1;
	ifade3;
}
```

Hızdan taviz vermemek için bu sefer code size’dan feragat ettik. 
Şüphesiz optimizasyonlar gerçek anlamda değecek bir kazanç sağlanıyorsa uygulanmalıdır. 
Örneğin burada optimize edilmiş biçim kodun daha hızlı çalışmasını sağlamasına karşın hem kodu daha karmaşık hale getirmekte hem de kodu büyütmektedir. 
Buradaki döngü eğer çok az dönüyorsa derleyicinin böyle bir düzenleme yapması toplamda önemli bir hız kazancı sağlamayacaktır. 


# Loop Unrolling (Döngü Açımı)

Döngü açımı döngü içerisindeki deyimleri çoklayarak döngünün daha az dönmesini sağlayan ve böylece döngü karşılaştırmasını azaltmayı hedefleyen bir tekniktir. Örneğin; 

```C++
for( i = 0; i < 100; ++i)
	foo();
```
Burada 100 kez foo fonksiyonu çağrılmıştır. 

Döngü aşağıdaki gibi düzenlenirse daha hızlı çalışacak hale getirilebilir. 

```C++
for (i = 0; i < 100 ; i+=4)
{
foo();
foo();
foo();
foo();
}
```
Bu  döngü 25 kez döndürülecek. Döngünün dönme sayısı azaltılır. 
Tabii döngü şöyle de açılabilirdi. 

```C++
foo();  //1
foo();  //2
foo();  //3
….
foo();  //99
foo();  //100
```
Ancak bu açık kodu çok büyütürdü. İşte derleyiciler bunun iyi bir noktasını bulmaya çalışmaktadır. 


# Loop Inversion (Döngülerin Ters Çevrilmesi) 

Bu optimizasyon tekniğinde while döngüsü ters yüz edilerek do-while döngüsü haline getirilir. Örneğin; 

``` C++
while (i < 100){
	foo();
	++i;
}
```
Yukarıda verilen while döngüsünün aşağıdaki ters yüz edilmiş hali daha hızlıdır; 

```C++
if (i < 100)
	do {
		foo();
		++i;
	     } while( i < 100);
```
Bu biçimde döngü içerisinde tek bir jump işlemi yapılmaktadır. jump komutları sıçrama komutları olduğu için işlemciler şöyle çalışıyorlar. pipeline diye bir mekanizma var işlemcilerin çalışmasında. İşlemci bir komutu işlerken bir sonraki komutu da almaya başlıyor. Hızlı çalışabilmesi için. Pipelining deniyor. Modern işlemciler bunu yapıyor. Jump komutu çok fazla ise bunu yapamayabilir. Jump komutundan sonra akışın nereye gideceği bilinmiyor. Bilemeyeceği için de çok jump komutu yerine daha az jump komutu üretilsin. Do-while döngüsünde daha az jump komutu üretiliyor. 

Jump işlemi pek çok işlemci için pipeline mekanizması üzerinde olumsuz etkileri olan bir işlemdir. İşlemci jump işlemini gördüğünde sonraki komutların çalışıp çalışmayacağını bilmediğinden onların için hazırlık işlemleri yapmayabilir. Bu konuda modern işlemciler arasında çeşitli farklılıklar bulunduğunu belirtelim. Her işlemci de aynı yaklaşım olmayabilir. 

#  Loop Fusion (Döngü Birleştirmesi) 

Birden fazla peş peşe döngü tek döngü olarak birleştirilebilir. Örneğin; 

```c++
int a[100], i; 
for (i = 0; i < 100; ++i)
	a[i] = i;
for (i = 0; i < 100; ++i)
	foo(i);
```
Aşağıdaki döngü daha hızlı çalışabilir; 
```C++
for (i = 0; i < 100; ++i){
a[i] = i;
	foo(i);
}
```
Cache (ön bellek) kullanan bazı işlemcilerde döngü birleştirmesi kar değil zarar oluşturabilir. Bunun nedeni işlemcinin bir bölgeye eriştiğinde oradaki belli miktarda byte’ı cache almasıdır. Bu cache’i alıp bırakma cache tazeleme sorununa yol açabilir ve toplam verimi düşürebilir. 


# Loop Interchange (Döngü Yer Değiştirmesi) 

Özellikle çok boyutlu dizinlerinde indekslenmesinde kullanılan bir optimizasyon tekniğidir. BU teknikte bellekteki atlamaların kısaltılarak cache etkinliğinin artırılması hedeflenir. Örneğin;

```C++
for (k = 0; k < colSize; ++k) 
	for (i = 0; i < colSize; ++i) 
		a[i][k] = val; 
```
C/C++ gibi programlama dillerinde iki boyutlu diziler satır dizilerinin art arda getirilmesiyle tek boyutlu olarak oluşturulmaktadır. 
Yani bu dillerde a[i][k] elemanı tek boyutlu dizinin i*colSize +k indeksinde bulunur. 
Yani bu organizasyon dikkate alındığında yukarıdaki örnekte iç döngüde uzun atlamaların olduğu söylenebilir. 

Dikkat ediniz, iç döngünün her yinelenmesinde bellekte bir sütun uzunluğu kadar atlama yapılmıştır. Bu atlama modern işlemcilerde fazlaca cache miss oluşmasına yol açabilir. İşte bu optimizasyon tekniğinde derleyici döngüleri ters sırada dizerek bu sorunu gidermeye çalışır. 

```C++
for (i = 0; i < colSize; ++i) 
	for (k = 0; k < colSize; ++k) 
		a[i][k] = val; 
```
Daha az atlama meydana gelir. 
Şimdi artık içteki döngünün her yinelemesinde uzun atlamalar oluşmuyor. Dolayısıyla cache miss oranının düşürüleceğini söyleyebiliriz. Ayrıca, döngülerin bu biçimde yer değiştirilmesi vektörel işlem yapan işlemcilerde hız kazancı sağlayabilecek potansiyel bir durumu da oluşturduğunu belirtelim. 

# Pointer Aliasing (Göstericilere İlişkin Optimizasyonlar) 

Bir göstericinin gösterdiği yerdeki bilgi geçici bir değişkende ya da yazmaçta tutulursa tekrar tekrar o göstericinin gösterdiği yere erişme işlemi elimine edilebilir. 

Çünkü örneğin C’de *p gibi ya da p[i] gibi işlemler aslında birden fazla makine komutuyla yapılmaktadır. Örneğin; 

```C++
a = *p +1; 
b = *p + 2;
c = *p + 3; 
```
Burada her defasında *p işlemi yapmak yerine kod aşağıdaki gibi düzenlenirse hız kazancı sağlanabilir. 

```C++
temp =  *p;             // temp bir yazmaç da olabilir // 
a = temp + 1;
b = temp + 2;
c = temp + 3;
```
Tabii derleyicinin bu optimizasyonu yapabilmesi için kesinlikle o sırada bu göstericinin gösterdiği yerdeki değerin değişmeyeceğini garanti altına alması gerekir. Örneğin; 

```C++
a = *p +1; 
foo();
b = *p + 2;
c = *p + 3; 
```
Burada foo p’nin gösterdiği yerde değişiklik yapıyor olabilir. Bu durumda derleyici foo’nun içini de incelemeden böyle bir kararı veremez. 
Örneğin p bir global nesneyi gösteriyor olabilir. foo da o nesneyi değiştiriyor olabilir. Bu durumda foo çağrısından sonra *p değişmiş olacaktır. 

Böylece derleyici *p değerini yazmaçlarda tutarak onu foo çağrısından sonra kullanamaz. 

Eğer bir göstericinin gösterdiği yere yalnızca o gösteriri tarafından erişiliyorsa yani o gösterici dışında oraya erişen başka bir kod yoksa bu yukarıda belirtilen tarzda optimzasyonlara yol açabilir. 

Örneğin, bir sistemde belleğin bir bölgesinden b byte’ı tek hamlede çekerek diğerine tek hamlede aktaran makine komutunun var olduğunu düşünelim. Bu durumda derleyici aşağıdaki gibi bir memcpy fonksiyonunu bu komutu kullanarak optimize edebilir mi ?  

Evet, optimize edebilir. 
Eğer bloklar çakışık ise derleyici bu makine komutundan faydalanamaz. İşte mymemcpy’yi çağıran kişi böyle bir duruma yol açabileceğinden derleyici bu olasılığı göz ardı edemez. 

Çakışık olabilme durumu için derleyici bu kodu optimize edemez. 

C99 ile birlikte derleyicilerin gösterici optimizasyonlarını iyileştirmek için restrict gösterici kavramı dile sokulmuştur. Restrict anahtar sözcüğü göstericinin kendisi için kullanılabilir, gösterdiği yer için kullanılamaz. 

Bir gösterici restrict anahtar sözcüğü ile bildirilmişse artık derleyici o gösterinin yaşamı boyunca o göstericinin gösterdiği yere yalnızca bu gösterici yoluyla erişildiği garantisini alır. Başka bir pointer yardımı ile erişmeyeceğim bilgisini veriyorum. Yani çakışık blok olmayacağının garantisini veriyoruz derleyiciye. Derleyici de bu garanti ile kodu optimize edebilir. 

restrict pointer. 
Derleyiciye şunun garantisini veriyoruz. “Derleyici ben sana iki adres vereceğim. Bu göstericilerin faaliyet alanı bitene kadar bu adreslerdeki bilgilere ben başka yolla erişmeyeceğim. Dolayısıyla source ve dest de çakışık blokları göstermeyecek. Sen bu bilgilerden hareketle gösterici optimizasyonlarını yapabilirsin. Ben bu kurala uymazsam başıma geleceklere razıyım.”

Undefined behavior durumu meydana gelir. Bu kurala uyulmadığı takdirde. 

Burada eğer p restrict bir göstericiyse derleyici artık aşağıdaki gibi bir optimizasyonu yapabilir. 

```C++
temp = *p; 
a = temp + 1; 
foo();
b = temp + 2; 
c = temp + 3; 
```
Çünkü artık derleyici p’nin gösterdiği yere foo tarafından da erişilmeyeceğinin sözünü almıştır. 


# Yazmaç Tahsisatları (Register Allocation) 

İşlemciler içerisinde kullanılan bellek alanlarına register denir. İşlemcilerde belli sayıda yazmaç vardır. İşlemciler registarlarla çok hızlı işlem yapıyorlar. 
Değişkenler yazmaçlara çekilerek işleme sokulurlar. Bu durumda derleyiciler hangi değişkenlerin hangi yazmaçlara çekileceği kararını vermek zorundadır. 
Bu karar verilirken çok kullanılan değişkenlerin mümkün olduğu kadar fazla süre yazmaçta kalmasına çalışılır. Böylece derleyici aynı değişkeni tekrar bellekten yazmaca çekmek zorunda kalmaz. İşte hangi değişkenlerin hangi yazmaçlara çekileceği önemli bir optimizasyon konusudur. 

# Instruction Scheduling and Reordering (Makine Komutlarının Düzenlenmesi ve Yer Değiştirilmesi) 

Bazı makine komutlarının sırasının değiştirilmesi hız kazancı sağlayabilmektedir. İşte eğer makine komutları arasında girdi çıktı ilişkisi yoksa derleyici bunları yeniden düzenleyerek kodun daha hızlı çalışmasını sağlayabilir. Bu optimizasyon sandığı kadar basit değildir ve polinomsal olmayan (NP) tarzda karmaşıklıklara sahiptir. 
Dolayısıyla derleme zamanını bu tür optimizasyonlar uzatabilmektedir. 


Bu söylenen arasında önemli olarak şunu belirtmek gerekir. 
Derleyicilerin optimizasyonları tek thread’li akışa göre yapılmaktadır. 
Yani eğer çok thread’li uygulama yapılıyorsa derleyicinin bu optimizasyonları bizim kodumuzun anlamını değiştirebilir. 
Çok threadli uygulamalarda bu duruma programcının kendisinin dikkat etmesi gerekmektedir. Örneğin; 

```C++
a = x + y + 10;
—----> Bu noktada x,y değişirse ne olur ? 
b = x + y + 20; 
c = x + y + 30; 
```
Burada x ve y’nin global değişkenler olduğunu düşünelim ve ok ile belirtilen noktada başka bir thread’in x ve y’yi değiştirdiğini varsayalım. Derleyici burada başka bir thread’in x ve y’yi değiştirebileceğini dikkate almaz. Ve optimizasyonunu aşağıdaki gibi yapabilir. 

```C++
temp = x + y; 
—----> Bu noktada x,y değişirse ne olur ? 
b =temp + 20; 
c = temp+ 30; 
```
Görüldüğü gibi burada artık x ve y’nin thread tarafından değiştirilen yeni değeri değil eski değeri sonraki işlemlere sokulmuştur. Biz bu tür optimizasyonlara dikkat etmeliyiz. Burada volatile anahtar sözcüğü kurtarıcı olabilir. 
volatile x,y;
Artık volatile bir nesneye derleyici erişirken her zaman onun o anki değerini bellekten alarak kullanır. 

volatile kimyada uçucu demek. Sen bunu registerda saklama bunu bellekten kullan. Bellekten okuyacağı için derleyici bunu optimize edemez. 

```C++
while (g_flag){

}
```
Burada başka bir thread g_flag değişkenini sıfıra çektiğinde döngü sonlanır. Programcı böyle bir sonlanmayı istemiş olabilir. Fakat derleyici g_flag değerini yazmaçta tutarak bir optimizasyon yapabilir. Yani derleyici sürekli olarak g_flag’in değerini yeniden bellekten almak zorunda değildir. Bu nedenle bu koddaki g_flag değişkeni volatile yapılmalıdır.

Derleyiciler eğer mult thread bir ortamı dikkate alarak optimizasyon yapsaydı yukarıda gördüğümüz optimizasyon temalarının çoğu devre dışı kalırdı.  



