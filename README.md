# Philips TV Kanal Listesi Düzenleyici

Philips Android TV'lerin USB üzerinden yedeklenen kanal listesini (`tv.db`) tarayıcıda açıp; kanalları **sıralamak, yeniden numaralamak, gizlemek/göstermek ve silmek** için geliştirilmiş, tek dosyalık bir web aracı. Kurulum gerektirmez, hiçbir sunucuya veri göndermez — tamamen tarayıcınızda (SQLite'ı WebAssembly ile çalıştıran [sql.js](https://github.com/sql-js/sql.js) kullanarak) çalışır.

## Neden bu araç var?

Philips'in "Kanal Listesi Kopyala" özelliği, kanal listesini bir USB belleğe SQLite veritabanı dosyaları olarak yedekler (`tv.db`, `list.db`, `chanLst.bin` vb.). Bu format resmi olarak belgelenmemiştir; TV'nin arayüzünde kanalları toplu halde sıralamak veya düzenlemek oldukça zahmetlidir. Bu araç, o dosyaları doğrudan düzenleyip TV'ye geri yüklemenizi sağlar.

## Özellikler

- **Listeleme:** `tv.db` içindeki tüm kanalları (numara, ad, tür — TV/Radyo, gizli/görünür, kilit durumu) bir tabloda gösterir.
- **Hızlı sıralama:** Bir kanalın "No" kutusuna hedef sırayı yazıp Enter'a basmanız yeterli — kanal o sıraya taşınır, aradaki kanallar otomatik kayar ve liste 1'den yeniden numaralanır. Sürükle-bırak ve ok tuşlarıyla taşıma da desteklenir.
- **Ada / türe / numaraya göre toplu sıralama** ve tek tuşla "1'den Yeniden Numarala".
- **Gizle / Göster:** Kanalları tek tek veya toplu (**"Seçilenler Dışındakileri Gizle"**) gizleyebilir, `tv.db`'den silmeden sadece görünmez yapabilirsiniz — bu, çok sayıda kanalı silmekten daha güvenlidir (bkz. aşağıdaki uyarı).
- **Silme:** Tek tek (🗑 ikonu) veya toplu (kutucuklarla seçip "Seçilenleri Sil") kanal silme. Silinen kanalların EPG/izlenme kayıtları da otomatik temizlenir.
- **Arama/filtreleme.**
- **`chanLst.bin` sağlama (checksum) güncellemesi:** TV, içe aktarma sırasında `chanLst.bin` içindeki CRC16 sağlamasını gerçek `tv.db` içeriğiyle karşılaştırır; bu araç her indirmede o sağlamayı otomatik yeniden hesaplar. (Bkz. [Teknik notlar](#teknik-notlar).)

## Nasıl kullanılır?

1. USB bellekteki yedek klasöründe (genelde `ChannelMap_xx/ChannelList/`) bulunan **`tv.db`** ve **`chanLst.bin`** dosyalarının PC'deki kopyalarını ayrı bir yere yedekleyin.
2. Aracı bir tarayıcıda açın (`philips-kanal-duzenleyici.html` dosyasına çift tıklamanız yeterli).
3. Her iki dosyayı da seçin: `tv.db` (zorunlu) ve `chanLst.bin` (aynı klasörden — **çok önemli**, aşağıya bakın).
4. Kanalları sıralayın / gizleyin / silin / yeniden adlandırın.
5. **"Değişiklikleri İndir"** ile güncellenmiş `tv.db` ve `chanLst.bin` dosyalarının ikisini de indirin.
6. İndirilen **iki dosyayı da** USB bellekteki aynı `ChannelList` klasörüne, eskilerinin üzerine yazacak şekilde kopyalayın. Diğer dosyalara (`list.db`, `channellib`, `s2channellib` vb.) dokunmayın.
7. USB belleği TV'ye takıp kanal listesi içe aktarma menüsünden (genelde *Ayarlar/Kurulum → Kanallar → Kanal Listesi Kopyalama → TV'ye Kopyala*) yükleyin.

## ⚠️ Önemli uyarılar

- **`chanLst.bin` şart:** Sadece `tv.db`'yi değiştirip üzerine yazarsanız, TV sağlama (checksum) tutmadığı için değişikliği **"kanal listesi değiştirilmedi"** diyerek reddedebilir.
- **Toplu silme riskli olabilir:** Bazı TV modellerinde, kanal sayısını çok büyük oranda azaltan bir silme işlemi (ör. 750 kanaldan sadece birkaçını bırakmak), kalan kanalların bir kısmının TV'de **"kurulmamış"** görünmesine yol açabiliyor. Bunun yerine istemediğiniz kanalları **gizlemeniz** önerilir — aynı görsel sonucu (istenmeyen kanallar listede görünmez) verir ve bu riski taşımaz.
- **Favori listeleri:** Kanal favori listelerine (`list.db`) eklenmişse ve o kanalı silerseniz, favori kaydı karşılıksız kalabilir. Emin değilseniz silmek yerine gizleyin.
- Bu, resmi olmayan, tersine mühendislikle çözülmüş bir formattır. TV'ye yüklemeden önce mutlaka orijinal dosyaların yedeğini ayrı bir yerde tutun.

## Teknik notlar

- `tv.db`, Android TV'nin standart `TvProvider` şemasını kullanan bir SQLite veritabanıdır (`channels`, `programs`, `watched_programs` tabloları). Kanal sırası/numarası `display_number`, görünürlük `browsable` alanında tutulur.
- `version_number` alanı (Android TV'nin satır bazlı "bu güncellendi" işaretçisi), her kayıtta artırılır — aksi halde TV değişikliği fark etmeyebilir.
- `chanLst.bin`, klasördeki her dosyanın **CRC16/MODBUS** sağlamasını saklayan küçük bir ikili manifesto dosyasıdır; format [PredatH0r/ChanSort](https://github.com/PredatH0r/ChanSort) projesinin kaynak koduna bakılarak ve gerçek dosyalar üzerinde doğrulanarak çözülmüştür.
- Silme işlemleri, `programs`/`watched_programs` tablolarındaki ilişkili kayıtları temizlemek için `PRAGMA foreign_keys = ON` ile şemadaki `ON DELETE CASCADE` kısıtlarını kullanır; ayrıca her indirmede olası "yetim" kayıtlar için ek bir temizlik geçişi yapılır.

## Gizlilik

Hiçbir dosya bir sunucuya yüklenmez. Tüm işlemler tarayıcınızda, yerel olarak çalışır (yalnızca sql.js kütüphanesi bir CDN'den indirilir).

## Katkı / geri bildirim

Sorun bildirmek veya öneride bulunmak için bir issue açabilirsiniz.
