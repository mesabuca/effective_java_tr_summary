## Nesneleri olusturup Silme
### Item 1 - Yapıcı Metot (Constructor) Yerine Statik Fabrika Metotlarını Kullanın
Normalde bir sınıfın nesnesını oluşturmak için constructor kullanırız. Diğer bir yöntemde dönüş değeri kendi nesnesi olan statik bir fabrika metodu (static factory method) tanımlarız ve bu bu metodu kullanarak da obje oluşturabiliriz.

```java
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```

#### Statik metodların avantajları
- Statik metodları isimlendirebilirsiniz
- Her çağırıldıklarında bir obje oluşturmak zorunda değillerdir. Bu size immutable sınıflara (değerleri değiştirilemez sınıflar) önceden tanımlanmış değişkenleri kullanmasını sağlar ya da sınıf, daha önceden yaratılmış nesneleri cacheden döndürmenize olanak sağlar ve böylece aynı nesneden tekrar tekrar yaratmak zorunda kalmazsınız.
- İstediğiniz herhangi bir tipte obje döndürebilirsiniz.
- Methodun return ettiğiobjenin önceden var olması şart değildir

### Item 2 - Çok Sayıda Parametre Kullanılacaksa Builder Kullanın

```java
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
  private final int servingSize; // (mL)
  required
  private final int servings;
  // (per container) required
  private final int calories;
  // (per serving)
  optional
  private final int fat;
  // (g/serving)
  optional
  private final int sodium;
  // (mg/serving)
  optional
  private final int carbohydrate; // (g/serving)
  optional
  public NutritionFacts(int servingSize, int servings) {
    this(servingSize, servings, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories) {
    this(servingSize, servings, calories, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories, int fat) {
    this(servingSize, servings, calories, fat, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories, int fat, int sodium) {
    this(servingSize, servings, calories, fat, sodium, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories, int fat, int sodium, int carbohydrate) {
    this.servingSize  = servingSize;
    this.servings     = servings;
    this.calories     = calories;
    this.fat          = fat;
    this.sodium       = sodium;
    this.carbohydrate = carbohydrate;
  }
}
```
gibi bir objede zorunlu olmayan alanları girmek istemediğiniz zaman şu şekilde tanımlamanız gerekiyor.

`NutritionFacts nf = new NutritionFacts(120, 8, 0, 12, 26, 3);`

Aradaki atamak istemediğiniz değerleri 0 olarak veya etkisiz eleman olarak atamanız gerekmektedir bunun içinde JavaBeans yöntemini kullanırız.

```java
public class NutritionFacts {
  // Parameters initialized to default values (if any)
  private int servingSize   = -1; // Required; no default value
  private int servings      = -1; // Required; no default value
  private int calories      =  0;
  private int fat           =  0;
  private int sodium        =  0;
  private int carbohydrate  =  0;

  public NutritionFacts() { }
  // Setters
  public void setServingSize(int val)   { servingSize = val; }
  public void setServings(int val)      { servings = val; }
  public void setCalories(int val)      { calories = val; }
  public void setFat(int val)           { fat = val; }
  public void setCarbohydrate(int val)  { sodium = val; }
  public void setSodium(int val)        { carbohydrate = val; }
}
```

Şeklinde tanımlayarak tek tek atama yapılabilir örneğin

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

gibi fakat parametre sayısı çok uzamadığı durumlar için geçerliç Eğer çok fazla parametre kullanacaksanız hem iç içe geçmiş yapıcıların hem de JavaBeans yönteminde öne çıkan okunabilirlik özelliklerini beraber kullanabileceğimiz başka bir yöntem daha var bunuda

```java
// Builder Pattern
public class NutritionFacts {
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;
  public static class Builder {
    // Required parameters
    private final int servingSize;
    private final int servings;
    // Optional parameters - initialized to default values
    private int calories      = 0;
    private int fat           = 0;
    private int sodium        = 0;
    private int carbohydrate  = 0;

    public Builder(int servingSize, int servings) {
      this.servingSize  = servingSize;
      this.servings     = servings;
    }

    public Builder calories(int val)
    { calories = val; return this; }
    public Builder fat(int val)
    { fat = val; return this; }
    public Builder sodium(int val)
    { sodium = val; return this;}
    public Builder carbohydrate(int val)
    { carbohydrate = val; return this;}
    public NutritionFacts build() {
      return new NutritionFacts(this);}
    }
  }
  private NutritionFacts(Builder builder) {
    servingSize   = builder.servingSize;
    servings      = builder.servings;
    calories      = builder.calories;
    fat           = builder.fat;
    sodium        = builder.sodium;
    carbohydrate  = builder.carbohydrate;
  }
}

NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```

seklindede kullanabiliriz.

### Item 3 - Singleton Sınıfları Private Yapıcı Metot veya Enum Türüyle Güçlendirin
Singleton sadece bir kere oluşturulan (instantiate) sınıf anlamına gelir.

Bunun için 2 yöntem vardır. Bunların ikisinde de yapıcı metotları private tanımlayıp singleton nesneyi public static tanımlayarak yapılır. Bunlardan birtanesi

```java
// Singleton with public final field
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public void leaveTheBuilding() { ... }
}
```

Diğer bir yol ise

```java
// Singleton with static factory
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public static Elvis getInstance() { return INSTANCE; }
  public void leaveTheBuilding() { ... }
}
```

Elvis.getInstance() her kullanıldığında aynı obje çağırılacaktır ve yeni bir obje oluşturulmayacaktır.

İlk yöntemin avantajı basit olmasıdır.

İkinci yöntemin avantajı ise API değiştirmeden objenin singleton özelliğini değiştirme esnekliğini vermesidir.

Diğer bir avantaj olarak, istenildiği taktirde generic bir singleton fabrika metodu yazılabiliyor olmamızdır..

Son avantajı ise metot referansı bir Supplier olarak kullanılabilir. Örneğin Elvis::getInstance, Supplier<Elvis> olarak kullanılabilir. Eğer Bu avantajlardan birisini kullanmıyacaksak ilk yöntem tercih edilebilr.

Bu iki yöntemden birini kullanan bir singleton sınıfı serializable yapmak için `implements Serializable` eklemek yeterli olmayacaktır. Sınıfın singleton özelliğini korumak için, sınıfın bütün alanlarını transient olarak tanımlamak ve readResolve metodunu gerçekleştirmek gerekir yoksa serileştirilen nesne yeniden okunduğunda (deserialization), singleton nesneyi döndürmek yerine yeni bir nesne yaratılacaktır. Bunu engellemek için şu şekilde bir readResolve metodu yazılabilir

```java
// readResolve method to preserve singleton property
private Object readResolve() {
  // Return the one true Elvis and let the garbage collector
  // take care of the Elvis impersonator.
  return INSTANCE;
}
```
Singleton nesne yaratmanın üçüncü bir yolu ise tek elemanlı bir Enum kullanmaktır.
```java
// Enum singleton - the preferred approach
public enum Elvis {
  INSTANCE;
  public void leaveTheBuilding() { ... }
}
```
Tek elemanlı Enum yöntemi çoğu zaman singleton nesne yaratmak için kullanılabilecek en iyi yöntemdir.

### Item 4 - Nesne Yaratılmasını İstemediğiniz Sınıfları Private Yapıcı Metot İle Güçlendirin

İlkel türler veya diziler üzerinde işlem yapan, birbiriyle alakalı metotları gruplamak için kullanılabilirler

Yardımcı sınıflardan (utility class) nesne oluşturulmamalı. Fakat herhangi bir yapıcı metot tanımlanmadığınızda Java derleyicisinin kendisi parametre almayan public bir default constructor metot tanımlar.

Bunu engellemek için bir sınıfı abstract tanımlamak işe yaramaz. Bu sınıf katılılarak çocuk sınıftan bir nesne yaratılabilir. Bundan kaçınmak için şu şekilde private bir yapıcı metot tanımlayabilirsiniz

```java
// Noninstantiable utility class
public class UtilityClass {
  // Suppress default constructor for noninstantiability
  private UtilityClass() {
    throw new AssertionError();
  }
  ... // Remainder omitted
}
```

### Item 5 - Bağımlılıkları Kendiniz Yaratmak Yerine Dependency Injection Kullanın

Dependency, sınıfların işlevlerini yerine getirebilmek için çeşitli kaynaklara ihtiyaç duymalarına denir.

```java
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
  private static final Lexicon dictionary = ...;
  private SpellChecker() {} // Noninstantiable
  public static boolean isValid(String word) { ... }
  public static List<String> suggestions(String typo) { ... }
}
```

```java
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
  private final Lexicon dictionary = ...;
  private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
  }
```

Yorum satırında da yazdığı gibi bu yöntemler tadmin edici değildirler. Çünkü birden fazla dil içi birden fazla sözlük gerektği zaman bu iki kod parçası kullanışsız kalmakta.

Bu sebeten dolayı SpellChecker sınıfını, birden fazla nesnesini oluşturacak şekilde tasarlayıp ve her nesne için kullanılacak sözlüğün istemci tarafından belirlenebilir bir şekilde yazmalıyız.

```java
// Dependency injection provides flexibility and testability
public class SpellChecker {
  private final Lexicon dictionary;
  public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }
  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```
### Item 6 - Gereksiz Nesneler Oluşturmayın

Bunu direk kodlarla açıklayacağım

```java
String s = new String("merhaba")   // YANLIS KULLANIM!!

String s = "merhaba"   // DOGRU KULLANIM!


String a = "abc";
String b = "abc";
System.out.println(a == b);  // True (a ve b referansları aynı nesneye işaret etmektedir)

String c = new String("abc");
String d = new String("abc");
System.out.println(c == d);  // False (c ve d referansları farklı nesnelere işaret etmektedir)
```

Ayrıca kullandığınız sınıf metodlarına da dikkat etmelsiniz. Mesela `String.matches()`. Bu metod kendi içerisinde bir **Pattern** nesnesi yaratıp sadece bir kez kullanılır ve bu nesneler çöp toplayıcısı (garbage collector) tarafından temizlenesiye kadar hafızada yer kaplarlar. Bunun yerine daha basit bir regex kullanılabilir.

Daha detaylı bilgi için **Effective Java** ktabına bakınız.

### Item 7 - Erişilmeyen Nesnelerin Referanslarından Kurtulun

Çöp toplayıcı eğer bir nesneye işaret eden referans varsa, o nesnenin hala erişilebilir olduğunu düşünecek ve nesneyi bellekte bırakacaktır.

Örnek bir **Memory Leak (Bellek Sızıntısı)** kodu

```java
// Can you spot the "memory leak"?
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  public Object pop() {
    if (size == 0)
    throw new EmptyStackException();
    return elements[--size];
  }
  /**
  * Ensure space for at least one more element, roughly
  * doubling the capacity each time the array needs to grow.
  */
  private void ensureCapacity() {
    if (elements.length == size)
    elements = Arrays.copyOf(elements, 2 * size + 1);
  }
}
```
Burada yığıtın boyutu önce artıp daha sonra azalırsa, yığıttan çıkartılan elemanlar çöp toplayıcı tarafından temizlenemeyecektir. Çünkü bu kod yığıttan çıkarılan nesnelerin referanslarını hala saklamaktadır.

Bu tarz problemleri çözmek için kullanılmayan referanslara null değerini atamak yeterli olacaktır. Yukarıdaki örneği göz önüne alırsak, pop() metodunun aşağıdaki gibi değiştirilmesi sorunu çözecektir.

```java
public Object pop() {
  if (size == 0)
  throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null; // Eliminate obsolete reference
  return result;
}
```

### Item 8 - Sonlandırıcı ve Temizleyicilerden Kaçının

Sonlandırıcılar tahmin edilemez, genellikle tehlikeli ve gereksizdirler. Bu alt başlığın özeti çıkarılamayacağı için **Effective Java** kitabından bölümü okuyunuz.

### Item 9 - try-finally Yerine try-with-resources Tercih Edin

`InputStream`, `OutputStream` ve `java.sql.Connection` gibi kaynaklar performans sorunu yaşamamak için açıldıklarında kapatılmalarıda lazımdır.

Önceden bunun için **try-finally** kaynağın kapatılması için en iyi yoldu exception fırlatsa bile.

```java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
    return br.readLine();
  } finally {
    br.close();
  }
}
```

Bu size kötü gözükmeyeblir fakat 2ç bir kaynak eklediğiniz zaman işler kötüleşir.

```java
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OutputStream out = new FileOutputStream(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
      out.write(buf, 0, n);
    } finally {
      out.close();
    }
  } finally {
    in.close();
  }
}
```

Hem `try` hem de `finally` bloklarındaki kodların istisna fırlatma ihtimali bulunmaktadır. Örneğin, ilk örnekteki firstLineOfFile metodunda, `try` bloğundaki `readLine` çağrısı fiziksel aygıttaki olası bir problemden dolayı istisna fırlatabilir, `finally` içerisindeki close çağrısı da aynı sebepten dolayı başarısız olabilir. Bu durumda, ikinci fırlatılan istisna birinciyi tamamen görünmez kılacaktır.

Yukarıdaki ilk örneğin `try-with-resources` ile çözümü şu şekildedir

```java
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
  try (BufferedReader br = new BufferedReader(
  new FileReader(path))) {
    return br.readLine();
  }
}
```

İkinci örneğin bu yöntemle çözümüde şu şekildedir.

```java
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
  try (InputStream
  in = new FileInputStream(src);
  OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
    out.write(buf, 0, n);
  }
}
```

`try-with-resources` kullanırken `catch` blokları da eklemek mümkündür.

```java
// try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
  try (BufferedReader br = new BufferedReader(
  new FileReader(path))) {
    return br.readLine();
  } catch (IOException e) {
    return defaultVal;
  }
}
```
## Bütün Nesnelerin Ortak Metodları

### Item 10 - `equals()` Metodunu Geçersiz Kılarken Sözleşmeye Uyun

Eğer aşağıdaki durumlardan herhangi biri geçerli ise `equals()` meroduna dokunmamak çok daha iyidir.

* Bir sınıfın her nesnesi tektir.
* Sınıfın **"mantıksal eşitlik"** testi sağlamak gibi bir ihtiyacı yoktur
* Bir super class equals() metodunu zaten geçersiz kılmıştır ve o davranış sub class için de uygundur.

[Object sınıfı](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#equals(java.lang.Object) içerisinde equals() metodunu geçersiz kılarken uyulması gereken bir “sözleşme” belirtilmiştir. Bu sözleşme aşağıdaki şekildedir:

* Dönüşlülük: Null olmayan bir x referansı için, `x.equals(x)` **true** değerini üretmelidir.

* Simetriklik: Null olmayan x ve y referansları için, `y.equals(x)` **true** değerini üretiyor ise `x.equals(y)` de mutlaka **true** üretmelidir.

* Geçişlilik: Null olmayan x, y ve z referansları için, `x.equals(y)` ve `y.equals(z)` **true** üretiyorsa, `x.equals(z)` de mutlaka **true** üretmelidir.

* Tutarlılık: Null olmayan x ve y referansları için, `equals()` metodu içerisinde kullanılan alanlar değiştirilmediği sürece, `x.equals(y)` tutarlı bir biçimde her zaman **true** veya **false** değerini üretmelidir.

* Null olmayan bir x referansı için, `x.equals(null)` **false** üretmelidir.

Şimdi bu konular hakkında yanlış örnekleri inceleyelim.

```java
// Broken - violates symmetry!
public final class CaseInsensitiveString {
  private final String s;
  public CaseInsensitiveString(String s) {
    this.s = Objects.requireNonNull(s);
  }
  // Broken - violates symmetry!
  @Override public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString)
    return s.equalsIgnoreCase(
    ((CaseInsensitiveString) o).s);
    if (o instanceof String) // One-way interoperability!
    return s.equalsIgnoreCase((String) o);
    return false;
  }
  ... // Remainder omitted
}
```

Bu sınıf `String` sınıfıyla da uyumlu çalışması istenmiş lakin büyük/küçük harflere duyarsız stringleri ele aldığı için şu şekildek bir kullanımda sorun yaşayacaktır

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s) // TRUE
s.equals(cis) // FALSE
```

Bu yüzden `String` objelerle karşılaştırma yapan kısmı çıkarmalıyız.

```java
@Override
public boolean equals(Object o) {
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```
___

```java
public class Point {
  private final int x;
  private final int y;
  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }
  @Override public boolean equals(Object o) {
    if (!(o instanceof Point))
    return false;
    Point p = (Point)o;
    return p.x == x && p.y == y;
  }
  ...
  // Remainder omitted
}

public class ColorPoint extends Point {
  private final Color color;
  public ColorPoint(int x, int y, Color color) {
    super(x, y);
    this.color = color;
  }
  ...
  // Remainder omitted
}
```
Burada `equals()` metodu nasıl yazılmalıdır? Hiç yazmazsanız metod `Point` sınıfından alınacaktır ve renk (color) bilgisi eşitlik karşılaştırmasında gözardı edilecektir.
Bu davranış `equals()` sözleşmesine aykırı olmasa da doğru davranış değildir.

```java
// Adds a value component without violating the equals contract
public class ColorPoint {
  private final Point point;
  private final Color color;
  public ColorPoint(int x, int y, Color color) {
    point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }
  /**
  * Returns the point-view of this color point.
  */
  public Point asPoint() {
    return point;
  }
  @Override public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
    return false;
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
  ...
  // Remainder omitted
}
```

Bu şekilde ata sınıfı karşılaştırmaların dışında tutmak en doğru seçimdir.
___

Bütün bu kuralların nasıl uygulandığını gösteren somut bir örnek aşağıda verilmektedir:

```java
// Class with a typical equals method
public final class PhoneNumber {
  private final short areaCode, prefix, lineNum;
  public PhoneNumber(int areaCode, int prefix, int lineNum) {
    this.areaCode = rangeCheck(areaCode, 999, "area code");
    this.prefix   = rangeCheck(prefix, 999, "prefix");
    this.lineNum  = rangeCheck(lineNum, 9999, "line num");
  }
  private static short rangeCheck(int val, int max, String arg) {
    if (val < 0 || val > max)
    throw new IllegalArgumentException(arg + ": " + val);
    return (short) val;
  }
  @Override public boolean equals(Object o) {
    if (o == this)
    return true;
    if (!(o instanceof PhoneNumber))
    return false;
    PhoneNumber pn = (PhoneNumber)o;
    return pn.lineNum == lineNum && pn.prefix == prefix
    && pn.areaCode == areaCode;
  }
  ... // Remainder omitted
}
```
### Item 11 - `equals()` ile Birlikte Mutlaka `hashCode()` Metodunu da Geçersiz Kılın

`hashCode()` metodunu gerektiği yerde geçersiz kılmamak (override) o sınıfın hash tabanlı `HashMap`, `HashSet`, `HashTable` gibi veri yapılarıyla birlikte kullandığında yanlış çalışmasına yol açar. Bunun sebebi ise Object sınıfından gelen hashCode metodunun hash kodunu hesaplarken nesnenin o anda bulunduğu bellek adresini kullanmasıdır.

Object sınıfı belirtiminde tarif edildiği üzere sözleşme genel olarak şu şekildedirler

* `equals` karşılaştırmasında kullanılan alanlar sabit kaldığı sürece, `hashCode` metodu aynı uygulama içerisinde üst üste çağrıldığında her zaman aynı sonucu üretmelidir.

* Eğer iki nesne `equals` metoduna göre birbirine eşitse, bu iki nesnenin `hashCode` metotları da aynı integer değerini üretmelidir.

* Eğer iki nesne `equals` metoduna göre eşit değilse, `hashCode` metodu bu iki nesne için farklı integer sonuçları üretmek zorunda değildir. Ancak yazılımcı bilmelidir ki eşit olmayan nesneler için farklı `hash` kodları üretmek `hash table` performansını artırabilir.
