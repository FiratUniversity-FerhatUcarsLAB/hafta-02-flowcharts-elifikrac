BAŞLA

// --- Değişkenler ---
hasta_kimlik = ""
islem_secim = 0
poliklinik_secim = ""
doktor_secim = ""
saat_secim = ""
tahlil_var = false
sonuc_hazir = false
devam = "E"

// --- Hasta Kimlik Doğrulama ---
YAZ "TC No giriniz:"
OKU hasta_kimlik

EĞER hasta_kimlik doğrulandı mı? İSE
    YAZ "Kimlik doğrulama başarılı."
DEĞİLSE
    YAZ "Kimlik doğrulama başarısız. Sistemden çıkılıyor."
    DUR
BİTİŞEĞER

// --- İşlem Döngüsü ---
DÖNGÜ devam = "E" VEYA devam = "e" İKEN

    YAZ "Yapmak istediğiniz işlemi seçin: 1-Randevu Al, 2-Tahlil Sonucu Gör"
    OKU islem_secim

    EĞER islem_secim = 1 İSE
        // --- Randevu Modülü ---
        YAZ "Poliklinik seçiniz:"
        YAZ poliklinik listesini göster
        OKU poliklinik_secim

        DÖNGÜ true
            YAZ "Seçilen poliklinikteki doktorlar:"
            YAZ doktor listesini göster(poliklinik_secim)
            YAZ "Doktor seçiniz veya 'geri' yazın:"
            OKU doktor_secim

            EĞER doktor_secim = "geri" İSE
                ÇIK
            BİTİŞEĞER

            YAZ "Uygun saatleri gösteriliyor:"
            YAZ uygun saatleri göster(doktor_secim)

            YAZ "Randevu saatini giriniz:"
            OKU saat_secim

            EĞER saat uygun mu? İSE
                YAZ "Randevu onaylandı."
                YAZ "SMS gönderildi."
                ÇIK
            DEĞİLSE
                YAZ "Seçilen saat dolu, lütfen başka saat seçin."
            BİTİŞEĞER
        BİTİŞDÖNGÜ

    DEĞİLSE EĞER islem_secim = 2 İSE
        // --- Tahlil Sonucu Modülü ---
        EĞER tahlil_var = true İSE
            EĞER sonuc_hazir = true İSE
                YAZ "Tahlil sonucu görüntüleniyor."
                YAZ "PDF indir seçeneği sunuluyor."
            DEĞİLSE
                YAZ "Sonuç henüz hazır değil, lütfen daha sonra kontrol edin."
            BİTİŞEĞER
        DEĞİLSE
            YAZ "Henüz tahlil bulunmamaktadır."
        BİTİŞEĞER
    DEĞİLSE
        YAZ "Geçersiz seçim."
    BİTİŞEĞER

    // --- Başka işlem yapmak ister misiniz? ---
    YAZ "Başka işlem yapmak ister misiniz? (E/H)"
    OKU devam

BİTİŞDÖNGÜ

BİTİR
