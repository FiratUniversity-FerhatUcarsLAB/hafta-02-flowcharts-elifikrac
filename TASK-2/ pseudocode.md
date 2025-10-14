BAŞLA

// --- Değişkenler ---
kullanici_giris = false
kullanici_ad = ""
kullanici_sifre = ""
katalog = [kategori_listesi ve ürünler]    // her ürün: {id, isim, fiyat, stok, kategori}
sepet = []                                 // sepet elemanları: {urun_id, isim, fiyat, adet}
toplam = 0
indirim_kodu = ""
indirim_tutar = 0
kargo_ucreti = 0
odeme_yontemi = ""
MINIMUM_SIPARIS = 50
UCRETSIZ_KARGO_LIMIT = 200
STANDART_KARGO = 20

// --- Kullanıcı Girişi ---
YAZ "Kullanıcı adı giriniz:"
OKU kullanici_ad
YAZ "Şifre giriniz:"
OKU kullanici_sifre

EĞER kullanici_ad ve kullanici_sifre doğrulanıyor mu? İSE
    kullanici_giris = true
    YAZ "Giriş başarılı. Hoşgeldiniz, ", kullanici_ad
DEĞİLSE
    YAZ "Giriş başarısız. Lütfen giriş yapınız veya kaydolunuz."
    DUR
BİTİŞEĞER

// --- Kategori gezinti ve ürün seçimi (döngü) ---
DÖNGÜ true
    YAZ "Ürün kategorileri:"
    YAZ kategori_listesini göster
    YAZ "Gezinmek istediğiniz kategori id'sini girin veya 'checkout' yazın:"
    OKU secim
    
    EĞER secim = "checkout" İSE
        ÇIK // kategori döngüsünden çık ve ödeme sürecine geç
    BİTİŞEĞER

    // Kullanıcı seçtiği kategorideki ürünleri listeler
    YAZ "Seçilen kategorideki ürünler:"
    YAZ ürün_listesini göster(secim)

    YAZ "Sepete eklemek için ürün id'si girin veya 'geri' yazın:"
    OKU urun_id_giris

    EĞER urun_id_giris = "geri" İSE
        DEVAM ET // kategori listesine dön
    BİTİŞEĞER

    YAZ "Adet giriniz:"
    OKU adet

    // --- Stok kontrolü (koşul) ---
    EĞER katalog[urun_id_giris].stok >= adet İSE
        // Sepete ekle veya mevcutsa adeti arttır
        EĞER sepetde_ayni_urun_var mı? İSE
            sepet.adet += adet
        DEĞİLSE
            sepet'e ekle {urun_id, isim, fiyat, adet}
        BİTİŞEĞER
        katalog[urun_id_giris].stok -= 0   // stok gerçek azaltma ödemede yapılır; bunu opsiyonel belirt
        YAZ "Ürün sepete eklendi."
    DEĞİLSE
        YAZ "Stok yetersiz! Mevcut stok: ", katalog[urun_id_giris].stok
    BİTİŞEĞER

    YAZ "Başka kategori gezmek ister misiniz? (E/H)"
    OKU devam_kategori
    EĞER devam_kategori = "H" VEYA devam_kategori = "h" İSE
        ÇIK
    DEĞİTSE
        DEVAM ET
    BİTİŞEĞER
BİTİŞDÖNGÜ

// --- Sepeti görüntüleme ve düzenleme (döngü) ---
DÖNGÜ true
    // Sepeti listele ve toplam hesapla
    toplam = 0
    YAZ "---- SEPET ----"
    DÖNGÜ her sepet öğesi İÇİN
        YAZ öğe.isim, " x", öğe.adet, " -> ", öğe.fiyat * öğe.adet, "TL"
        toplam = toplam + (öğe.fiyat * öğe.adet)
    BİTİŞDÖNGÜ
    YAZ "Ara toplam: ", toplam, " TL"

    YAZ "Sepeti düzenlemek ister misiniz? (E/H)"
    OKU duzenle
    EĞER duzenle = "E" VEYA duzenle = "e" İSE
        YAZ "Silmek için ürün id girin veya adet güncellemek için 'id,adet' girin, bitirmek için 'tamam' yazın:"
        OKU duzenle_islem

        EĞER duzenle_islem = "tamam" İSE
            ÇIK // düzenleme döngüsünden çık
        DEĞİTSE
            EĞER duzenle_islem içinde ',' var mı? İSE
                // adet güncelle
                OKU parçalara ayır -> id, yeni_adet
                EĞER yeni_adet = 0 İSE
                    sepetten sil id
                DEĞİLSE
                    sepet[id].adet = yeni_adet
                BİTİŞEĞER
            DEĞİLSE
                // silme işlemi
                sepetten sil duzenle_islem
            BİTİŞEĞER
        BİTİŞEĞER
    DEĞİLSE
        ÇIK
    BİTİŞEĞER
BİTİŞDÖNGÜ

// --- İndirim kodu uygulanabilir (koşul) ---
YAZ "İndirim kodu var mı? (Y/N)"
OKU var_mi
EĞER var_mi = "Y" VEYA var_mi = "y" İSE
    YAZ "İndirim kodunu giriniz:"
    OKU indirim_kodu
    EĞER indirim_kodu geçerli mi? İSE
        indirim_tutar = hesapla_indirim(toplam, indirim_kodu)   // örn. %10 veya sabit TL
        YAZ "İndirim uygulandı: ", indirim_tutar, " TL"
    DEĞİLSE
        indirim_tutar = 0
        YAZ "Geçersiz indirim kodu."
    BİTİŞEĞER
DEĞİLSE
    indirim_tutar = 0
BİTİŞEĞER

ara_toplam = toplam - indirim_tutar

// --- Minimum 50 TL kontrolü (koşul) ---
EĞER ara_toplam < MINIMUM_SIPARIS İSE
    YAZ "Siparişler için minimum tutar ", MINIMUM_SIPARIS, " TL'dir. Mevcut tutar: ", ara_toplam, " TL"
    YAZ "Lütfen ürün ekleyin veya indirimleri kaldırın."
    // Kullanıcıyı ürün ekleme akışına yönlendir veya işlemi sonlandır
    DUR
BİTİŞEĞER

// --- Kargo ücreti hesaplama (koşul) ---
EĞER ara_toplam >= UCRETSIZ_KARGO_LIMIT İSE
    kargo_ucreti = 0
    YAZ "Ücretsiz kargo uygulanıyor."
DEĞİLSE
    kargo_ucreti = STANDART_KARGO
    YAZ "Kargo ücreti: ", kargo_ucreti, " TL"
BİTİŞEĞER

genel_toplam = ara_toplam + kargo_ucreti
YAZ "Ödenecek toplam: ", genel_toplam, " TL"

// --- Ödeme yöntemi seçimi (koşul) ---
YAZ "Ödeme yöntemi seçin: 1) Kredi Kartı 2) Kapıda Ödeme 3) Havale/EFT"
OKU odeme_secim
EĞER odeme_secim = "1" İSE
    odeme_yontemi = "Kredi Kartı"
    // kredi kartı bilgileri alınır
    YAZ "Kart numarası giriniz:"
    OKU kart_no
    YAZ "Son kullanma tarihi giriniz:"
    OKU skt
    YAZ "CVV giriniz:"
    OKU cvv
    // Kart ile ödeme doğrulama
    EĞER kart doğrulandı mı? İSE
        YAZ "Ödeme başarılı."
    DEĞİLSE
        YAZ "Ödeme başarısız. Lütfen başka yöntem seçin."
        // İsteğe bağlı: tekrar ödeme seçeneğine dön
        DUR
    BİTİŞEĞER
DEĞİLSE EĞER odeme_secim = "2" İSE
    odeme_yontemi = "Kapıda Ödeme"
    YAZ "Kapıda ödeme seçildi."
DEĞİLSE EĞER odeme_secim = "3" İSE
    odeme_yontemi = "Havale/EFT"
    YAZ "Banka bilgileri gösteriliyor. İşlem tamamlandığında lütfen bildiriniz."
DEĞİLSE
    YAZ "Geçersiz seçim. İşlem iptal."
    DUR
BİTİŞEĞER

// --- Sipariş Onayı ---
YAZ "Siparişi onaylıyor musunuz? (E/H)"
OKU onay
EĞER onay = "E" VEYA onay = "e" İSE
    // Stok güncellemesi ve sipariş kaydı
    DÖNGÜ her sepet öğesi İÇİN
        EĞER katalog[öğe.urun_id].stok >= öğe.adet İSE
            katalog[öğe.urun_id].stok = katalog[öğe.urun_id].stok - öğe.adet
        DEĞİLSE
            YAZ "Sipariş sırasında stok problemi: ", öğe.isim
            YAZ "İşlem iptal ediliyor."
            DUR
        BİTİŞEĞER
    BİTİŞDÖNGÜ

    siparis_id = siparis_olustur(kullanici_ad, sepet, genel_toplam, odeme_yontemi)
    YAZ "Siparişiniz oluşturuldu. Sipariş No: ", siparis_id
    YAZ "Teşekkürler!"
DEĞİLSE
    YAZ "Sipariş onaylanmadı. Sepete dönülüyor."
    // Opsiyonel: sepete dön veya çık
    DUR
BİTİŞEĞER

BİTİR
