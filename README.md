# 🏆 Gold Digger Credit Card - Altın Taksit Hesaplama

Altını taksitle mi almalı, yoksa peşin mi almalı? Bu uygulama, kredi kartı ile taksitli altın alımında fon getirisi hesaplayarak en karlı seçeneği gösterir.

## 🎯 Proje Hakkında

Bu uygulama, altını taksitle almanın peşine göre karlı olup olmadığını hesaplar. Temel mantık:

1. Altının peşin fiyatını fona yatırıyoruz
2. Her ay fon getirisi kazanıyoruz
3. Taksit ödemelerini fondan yapıyoruz
4. Kredi kartı bonusu varsa, o da fonda çalışıyor
5. Sonunda toplam kazancı, taksit farkı ile karşılaştırıyoruz

**Eğer toplam kazanç > taksit farkı ise** → Taksitli almak karlı ✅  
**Eğer toplam kazanç < taksit farkı ise** → Peşin almak daha mantıklı ❌

## ✨ Özellikler

- ⚡ **Anlık Hesaplama**: Değerleri girdikçe otomatik hesaplama
- 💳 **Kart Profilleri**: Ziraat, Akbank, Garanti kartları için hazır tarih ayarları
- 📊 **Detaylı Analiz**: Dönem dönem fon getirisi tablosu
- 🎯 **Maksimum Fiyat**: Kârlı olmak için maksimum taksitli fiyat hesabı
- 🌐 **Türkçe Arayüz**: Tamamen Türkçe kullanıcı deneyimi
- 📱 **Responsive Tasarım**: Mobil ve masaüstü uyumlu

## 🛠️ Teknolojiler

- **Vue 3** - Modern JavaScript framework
- **Vuetify 3** - Material Design komponent kütüphanesi
- **TypeScript** - Tip güvenli geliştirme
- **Moment.js** - Tarih hesaplamaları
- **Pinia** - State management
- **Vite** - Hızlı build tool

## 💿 Kurulum

Projeyi klonlayın ve bağımlılıkları yükleyin:

```bash
# Projeyi klonla
git clone https://github.com/ahmetubeydullah/gold-digger-credit-card.git
cd gold-digger-credit-card

# Bağımlılıkları yükle
npm install

# Geliştirme sunucusunu başlat
npm run dev
```

Uygulama [http://localhost:3000](http://localhost:3000) adresinde çalışacak.

## 🚀 Canlı Demo

Uygulamayı canlı olarak deneyin: [https://ahmetubeydullah.github.io/gold-digger-credit-card/](https://ahmetubeydullah.github.io/gold-digger-credit-card/)

## 📖 Kullanım

1. **Kredi Kartı Seç**: Sağ taraftaki kartlardan birini seçin (tarihler otomatik ayarlanır)
2. **Fiyatları Gir**: Peşin fiyat, taksitli fiyat ve taksit sayısını girin
3. **Fon Bilgileri**: Günlük getiri oranını ve kredi kartı bonusunu girin
4. **Tarihleri Ayarla**: Satın alma, ekstre kesim ve ödeme tarihlerini kontrol edin
5. **Hesapla**: Otomatik hesaplama ile sonuçları görün

## 📊 Hesaplama Mantığı

### Bonus Aktifleşme

Kredi kartı bonusu, satın alma tarihinden sonraki ayın 1'inde ekstreden kesilir, ödeme tarihinde ödenir ve bir sonraki dönemde fonda çalışmaya başlar.

### Gün Hesabı

Fon getirisi hesaplanırken başlangıç tarihi dahil, bitiş tarihi (ödeme günü) hariç tutulur.

### Ekstre Kesim Mantığı

Eğer satın alma tarihi, ekstre kesim tarihinde veya sonrasında ise, ilk ödeme 1 ay sonraya kayar.

## 🏗️ Production Build

Production için build almak:

```bash
npm run build
```

Build dosyaları `dist` klasöründe oluşturulur.

## 🌐 Deployment

GitHub Pages'e deploy etmek için:

```bash
npm run deploy
```

## 📝 Lisans

Bu proje MIT lisansı altında lisanslanmıştır.

## 👨‍💻 Geliştirici

**Ahmet Ubeydullah**

- GitHub: [@ahmetubeydullah](https://github.com/ahmetubeydullah)

## 🤝 Katkıda Bulunma

Katkılarınızı bekliyoruz! Pull request göndermekten çekinmeyin.

---

⭐ Bu projeyi beğendiyseniz yıldız vermeyi unutmayın!
