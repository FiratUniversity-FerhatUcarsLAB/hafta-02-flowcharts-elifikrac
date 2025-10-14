BAŞLA

//--- Değişkenler ---
bakiye ← 5000
gunluk_limit ← 2000
pin_hak ← 3
dogru_pin ← 1234
cekilen_toplam ← 0

//--- PIN Doğrulama ---
DÖNGÜ pin_hak > 0 İKEN
    YAZ "Lütfen PIN kodunuzu giriniz:"
    OKU girilen_pin
    
    EĞER girilen_pin = dogru_pin İSE
        YAZ "PIN doğrulandı."
        ÇIK // Döngüden çık
    DEĞİLSE
        pin_hak ← pin_hak - 1
        YAZ "Hatalı PIN! Kalan hakkınız: ", pin_hak
    BİTİŞEĞER
BİTİŞDÖNGÜ

EĞER pin_hak = 0 İSE
    YAZ "Kartınız bloke olmuştur. Lütfen banka ile iletişime geçiniz."
    DUR
BİTİŞEĞER

//--- İşlem Döngüsü ---
DÖNGÜ sonsuz
    YAZ "Bakiyeniz: ", bakiye, " TL"
    YAZ "Günlük limitiniz: ", gunluk_limit - cekilen_toplam, " TL"
    
    YAZ "Çekmek istediğiniz tutarı giriniz:"
    OKU tutar
    
    //--- Tutar kontrolü ---
    EĞER tutar % 20 ≠ 0 İSE
        YAZ "Hata: Çekilecek tutar 20 TL'nin katı olmalıdır!"
        DEVAM ET // Döngü başına dön
    BİTİŞEĞER
    
    //--- Günlük limit kontrolü ---
    EĞER cekilen_toplam + tutar > gunluk_limit İSE
        YAZ "Günlük limit aşıldı! En fazla ", gunluk_limit - cekilen_toplam, " TL çekebilirsiniz."
        DEVAM ET
    BİTİŞEĞER
    
    //--- Bakiye kontrolü ---
    EĞER tutar > bakiye İSE
        YAZ "Yetersiz bakiye! Bakiyeniz: ", bakiye, " TL"
        DEVAM ET
    BİTİŞEĞER
    
    //--- İşlem Onay ---
    bakiye ← bakiye - tutar
    cekilen_toplam ← cekilen_toplam + tutar
    
    YAZ "Lütfen paranızı ve fişinizi alınız."
    YAZ "Yeni bakiyeniz: ", bakiye, " TL"
    
    //--- Tekrar işlem seçeneği ---
    YAZ "Başka bir işlem yapmak ister misiniz? (E/H)"
    OKU cevap
    
    EĞER cevap = "H" VEYA cevap = "h" İSE
        YAZ "Kartınızı almayı unutmayınız. İyi günler!"
        DUR
    DEĞİLSE
        DEVAM ET // yeni işlem için döngü tekrar başa döner
    BİTİŞEĞER

BİTİŞDÖNGÜ

BİTİR
