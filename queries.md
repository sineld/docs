
# Sorgu Oluşturucu

- [Giriş](#introduction)
- [Seçme](#selects)
- [Joins](#joins)
- [Gelişmiş Where](#advanced-wheres)
- [Toplamak](#aggregates)
- [Ham İfadeler(Raw Expressions)](#raw-expressions)
- [Ekleme](#inserts)
- [Güncelleme](#updates)
- [Silme](#deletes)
- [Unions](#unions)
- [Önbellek Sorguları](#caching-queries)

<a name="introduction"></a>
## Giriş

Veritabanı sorgu oluşturucu, veritaban sorguları oluşturmak ve çalıştırmak için uygun, akıcı bir arayüz sağlar. Uygulamanızda çoğu veritabanı işlemlerinde kullanıp desteklenen tüm veritaban sistemlerinde çalıştırabilirsiniz.

> **Not:** Laravel sorgu oluşturucu SQL enjeksiyon saldırılarına karşı uygulamayı korumak için bağlayıcı PDO parametresini kullanır. String değerleri temizlemenize gerek yoktur.

<a name="selects"></a>
## Seçmek

**Tablodan Tüm Satırları Almak**

	$kullanicilar = DB::table('kullanicilar')->get();

	foreach ($kullanicilar as $kullanici)
	{
		var_dump($kullanici->isim);
	}

**Tablodan Tek Satır Almak**

	$kullanici = DB::table('kullanicilar')->where('isim', 'Can')->first();

	var_dump($kullanici->isim);

**Satırdaki Sütunun Verilerini Almak**

	$isim = DB::table('kullanicilar')->where('isim', 'Can')->pluck('isim');

**Kolon Değerlerinin Listesini Almak**

	$roller = DB::table('roller')->lists('baslik');

Bu metod dizi tipinde roller'deki baslik değerlerini döndürecektir. Ayrıca verilen dizi için özel bir key sütunu belirtebilirsiniz:

	$roller = DB::table('roller')->lists('baslik', 'isim');

**Belirtilen Maddeyi Seçmek**

	$kullanicilar = DB::table('kullanicilar')->select('isim', 'email')->get();

	$kullanicilar = DB::table('kullanicilar')->distinct()->get();

	$kullanicilar = DB::table('kullanicilar')->select('isim as kullanici_isim')->get();

**Varolan Sorguya Seçilmiş Madde Eklemek**

	$sorgu = DB::table('kullanicilar')->select('isim');

	$kullanicilar = $query->addSelect('yas')->get();

**Where İşlemlerini Kullanma**

	$kullanicilar = DB::table('kullanicilar')->where('oylar', '>', 100)->get();

**Or Durumları**

	$kullanicilar = DB::table('kullanicilar')
	                    ->where('oylar', '>', 100)
	                    ->orWhere('isim', 'Can')
	                    ->get();

**Where Between Kullanmak**

	$kullanicilar = DB::table('kullanicilar')
	                    ->whereBetween('oylar', array(1, 100))->get();

**Dizi ile Birlikte Where Kullanmak**

	$kullanicilar = DB::table('kullanicilar')
	                    ->whereIn('id', array(1, 2, 3))->get();

	$kullanicilar = DB::table('kullanicilar')
	                    ->whereNotIn('id', array(1, 2, 3))->get();

**Where Null Kullanarak Değer Atanmamış Değerleri Bulmak**

	$users = DB::table('kullanicilar')
	                    ->whereNull('guncellenme')->get();

**Order By, Group By, ve Having**

	$kullanicilar = DB::table('kullanicilar')
	                    ->orderBy('isim', 'desc')
	                    ->groupBy('count')
	                    ->having('count', '>', 100)
	                    ->get();

**Offset & Limit**

	$kullanicilar = DB::table('kullanicilar')->skip(10)->take(5)->get();

<a name="joins"></a>
## Joins

Sorgu oluşturucu join(birleştirme) durumları yazmak içinde kullanılabilir. Aşağıdaki örneklere bir göz atın:

**Basit Join Demeci**

	DB::table('kullanicilar')
	            ->join('kisiler', 'kullanicilar.id', '=', 'kisiler.kullanici_id')
	            ->join('siparisler', 'kullanicilar.id', '=', 'siparisler.kullanici_id')
	            ->select('kullanicilar.id', 'kisiler.tel', 'siparisler.fiyat');

Aynı zamanda daha gelişmiş join durumları tanımlayabilirsiniz:

	DB::table('kullanicilar')
	        ->join('kisiler', function($join)
	        {
	        	$join->on('kullanicilar.id', '=', 'kisiler.kullanici_id')->orOn(...);
	        })
	        ->get();

<a name="advanced-wheres"></a>
## Gelişmiş Where

Bazen "where" cümlecikleri "where exists" ya da iç içe parametre gruplaması gibi daha gelişmişini oluşturmanız gerekebilir.

**Parametre Gruplama**

	DB::table('kullanicilar')
	            ->where('isim', '=', 'Can')
	            ->orWhere(function($query)
	            {
	            	$query->where('oylar', '>', 100)
	            	      ->where('baslik', '<>', 'Admin');
	            })
	            ->get();

Yukarıdaki sorgu böyle bir SQL kodu oluşturacaktır:

	select * from kullanicilar where isim = 'Can' or (oylar > 100 and baslik <> 'Admin')

**Mevcut Demeçler**

	DB::table('kullanicilar')
	            ->whereExists(function($query)
	            {
	            	$query->select(DB::raw(1))
	            	      ->from('siparisler')
	            	      ->whereRaw('siparisler.kullanici_id = kullanicilar.id');
	            })
	            ->get();

Yukarıdaki sorgu böyle bir SQL kodu oluşturacaktır:

	select * from kullanicilar
	where exists (
		select 1 from siparisler where siparisler.kullanici_id = kullanicilar.id
	)

<a name="aggregates"></a>
## Toplama

Sorgu oluşturucu ayni zamanda farklı toplama metodları sağlar.Bunlar; `count`, `max`, `min`, `avg`, ve `sum`.

**Toplama Metodlarını Kullanma**

	$kullanicilar = DB::table('kullanicilar')->count();

	$ucret = DB::table('siparisler')->max('ucret');

	$ucret = DB::table('siparisler')->min('ucret');

	$ucret = DB::table('siparisler')->avg('ucret');

	$toplam = DB::table('kullanicilar')->sum('oylar');

<a name="raw-expressions"></a>
## Ham İfadeler(Raw Expressions)

Bazen sorguda ham ifade(raw expression) kullanma ihtiyacı duyabilirsiniz. Bu ifadelere string olarak sorgu enjekte olacaktır.Bu yüzden herhangi SQL injection noktaları oluşturmamaya dikkat edin! Ham ifade oluşturmak için, `DB::raw` methodu kullanılır:

**Ham İfade Kullanımı**

	$kullanicilar = DB::table('kullanicilar')
	                     ->select(DB::raw('count(*) as kullanici_count, status'))
	                     ->where('status', '<>', 1)
	                     ->groupBy('status')
	                     ->get();

**Kolonda artan yada azalan değer**

	DB::table('users')->increment('oylar');

	DB::table('users')->decrement('oylar');

<a name="inserts"></a>
## Ekleme

**Tabloya Kayıtlar Ekleme**

	DB::table('kullanici')->insert(
		array('email' => 'can@ornek.com', 'oylar' => 0)
	);

Eğer tablo otomatik artan(auto-incrementing) id'ye sahipse, `insertGetId` kullanarak id'yi ekleyip , alabilirsiniz:

**Tabloya ID'si Otomatik Artan(Auto-Incrementing) Kayıtlar Ekleme**

	$id = DB::table('kullanicilar')->insertGetId(
		array('email' => 'can@ornek.com', 'oylar' => 0)
	);

> **Not:** PostgreSQL kullanırken insertGetId metodu otomatik artması(auto-incrementing) için kolon adı "id" olmalıdır.

**Tabloya Çoklu Kayıtlar Ekleme**

	DB::table('kullanicilar')->insert(array(
		array('email' => 'taylor@ornek.com', 'oylar' => 0),
		array('email' => 'dayle@ornek.com', 'oylar' => 0),
	));

<a name="updates"></a>
## Güncelleme

**Tablodaki Kayıtları Güncelleme**

	DB::table('kullanicilar')
	            ->where('id', 1)
	            ->update(array('oylar' => 1));

<a name="deletes"></a>
## Silme

**Tablodaki Kayıtları Silme**

	DB::table('kullanicilar')->where('oylar', '<', 100)->delete();

**Tablodaki Tüm Kayıtları Silme**

	DB::table('kullanicilar')->delete();

**Tabloyu Budamak(Truncate)**

	DB::table('kullanicilar')->truncate();

<a name="unions"></a>
## Unions

Sorgu oluşturucu bunun yanında çabuk bir yol olarak iki sorguyla birlikte "union" sorgusunu sağlar:

**Union Sorgu Uygulama**

	$ilk = DB::table('kullanicilar')->whereNull('ilk_isim');

	$kullanicilar = DB::table('kullanicilar')->whereNull('son_isim')->union($first)->get();

Aynı zamanda `unionAll` metodu da mevcuttur ve `union` metodu ile aynı kullanımdır.

<a name="caching-queries"></a>
## Önbellek Sorguları

Sorgu sonuçlarını `remember` metodu ile kolayca önbellekleyebilirsiniz:

**Sorgu Sonucunu Önbelleklemek**

	$kullanicilar = DB::table('kullanicilar')->remember(10)->get();

Bu örnekte, sorgu sonuçları 10 dakika içinde belleklenecektir. Önbelleklenme süresi boyunca, sorgu veritabanda çalışmayacak ve sonuçlar uygulamanız için belirtilen varsayılan önbellek sürücüsünden yüklenir.
