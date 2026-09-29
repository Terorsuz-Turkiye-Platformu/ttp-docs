<p align="center">
  <img src="https://files.catbox.moe/a3drda.png" alt="Terörsüz Türkiye Platformu" width="280" />
</p>

<h1 align="center">Terörsüz Türkiye Platformu</h1>

<p align="center">
  <strong>Hepimiz Kardeşiz</strong> · 2025
</p>

<p align="center">
  Organizasyon dokümantasyon deposu. Platformun amacı, faaliyetleri ve yazılım kuralları burada durur.
</p>

---

## Platformun Amacı ve Faaliyetleri

Terörsüz Türkiye Platformu, teröre karşı ortak bir duruş, huzur, güven, yatırım ve kardeşlik ortamını ülke genelinde anlatmak için kurulmuş bir sivil toplum oluşumudur.

- **Toplumsal Dayanışma.** Doğu ve Güneydoğu Anadolu bölgelerindeki aşiret temsilcileri, kanaat önderleri, vakıf ve konfederasyon başkanlarını bir araya getirerek teröre karşı ortak bir duruş sergiler.
- **Süreç Bilgilendirmesi.** Ülke genelinde "Terörsüz Türkiye" vizyonunu, huzur, güven, yatırım ve kardeşlik ortamını halka anlatmaya yönelik buluşmalar ve programlar düzenler.
- **Kurucu ve Yönetim.** Platformun Kurucu Genel Başkanlığını Turgut Lenk yürütmektedir. Oluşum, yedi bölgede çeşitli sivil toplum çalışmaları ve istişareler gerçekleştirmektedir.

## Organizasyon Kuralları

Bu org **AI-first** ve **async-first** çalışır. Aşağıdaki kurallar her depo, her agent ve her değişiklik için geçerlidir.

### 1. AI-first

Her geliştirme bir AI katkısı ile yapılır. Üretim kodunu insan yazmaz.

- İnsan yönlendirir, kapsamı netleştirir, sonucu gözden geçirir ve kararı verir.
- Kodu agent yazar. Elle yazılmış üretim kodu org standardının dışındadır.
- Bir değişiklik sohbet özeti ile kapanmaz. Kod, doküman ve inceleme birlikte tamamlanır.

### 2. Async-first

Organizasyon ve kod async-first'tir. Bekleyen iş, isteği ve agent'ı kilitlemez.

- I/O yapan her yol gerçekten async olur: `async` / `await`, `Task` / `Task<T>`.
- `*Async` adlı bir metot `async` olmadan içteki `Task`'i geri döndüremez. Bu sahte async'tir.
- Beklenecek doğal bir çağrı yoksa bloklayan iş `Task.Run(...)` ile ayrılır ve `await` edilir.
- Controller, servis ve repository içinde senkron veritabanı veya dosya çağrısı yapılmaz.

### 3. Arc's DDD

Agentlar [Arc's DDD](./arc-ddd.md) stiline uyar. Kısa kural seti [`AGENTS.md`](./AGENTS.md) içindedir. Kod yazmadan önce ikisi de okunur.

- Bounded context bir klasördür. Tablolar paylaşılabilir. Entity'ler paylaşılmaz.
- Başka bir bağlamın satırına ihtiyaç varsa tam aggregate çekilmez; yerel bir projection yazılır ve aynı fiziksel tabloya map edilir.
- Core saf kalır: EF yok, çapraz bounded-context `using` yok.
- HTTP cevabı yalnızca API DTO'su döner. Veritabanından gelen tip dışarı çıkmaz.
- Katman sırası `API → Domain → Core` şeklindedir.

Tek satır:

> Klasörler bounded context'tir; tablolar paylaşılabilir; entity'ler paylaşılmaz — ihtiyacı olan bağlam kendi projection'ını yazar.

## Güncel Gelişmeler

Platformun buluşmaları, bölgesel istişareleri ve kamuoyu programları bu bölüme işlenir. Yeni bir gelişme olduğunda tarih, yer ve kısa özet ile buraya eklenir.
