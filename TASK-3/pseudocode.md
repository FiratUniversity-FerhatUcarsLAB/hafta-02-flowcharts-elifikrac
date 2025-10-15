BASLA
  // Kimlik doğrulama
  YAZ "TC Kimlik Numaranızı Giriniz: "
  TC = GİRİŞ_AL()

  EGER TC GEÇERLİ İSE
      YAZ "Hoşgeldiniz, lütfen yapmak istediğiniz işlemi seçin:"
      YAZ "1 - Randevu Al"
      YAZ "2 - Tahlil Sonuçlarını Gör"

      ISLEM = GİRİŞ_AL()

      // Randevu modülü
      EGER ISLEM == 1 İSE
          YAZ "Poliklinik Listesi Gösteriliyor..."
          
          DONGU (Poliklinikler arasında gezin)
              YAZ "Poliklinik Adı:"
              POLIKLINIK = SEÇİM_YAP()
          DONGU SONU
          
          YAZ "Seçilen Poliklinikteki Doktorlar Gösteriliyor..."
          
          DONGU (Doktor listesi içinde gezin)
              YAZ "Doktor Adı:"
              DOKTOR = SEÇİM_YAP()
          DONGU SONU
          
          YAZ "Uygun Randevu Saatleri Gösteriliyor..."
          
          DONGU (Uygun saat listesi boyunca)
              YAZ "Saat:"
              SAAT = SEÇİM_YAP()
          DONGU SONU

          YAZ "Randevu Onaylanıyor..."
          RANDEVU_OLUSTUR(TC, POLIKLINIK, DOKTOR, SAAT)

          YAZ "Randevu Onaylandı!"
          SMS_GONDER(TC, "Randevunuz onaylandı.")

      // Tahlil sonuçları modülü
      EĞER ISLEM == 2 İSE
          YAZ "Tahlil kayıtları kontrol ediliyor..."

          EGER TAHLIL_VAR_MI(TC) İSE
              YAZ "Tahlil kaydı bulundu, sonuç durumu kontrol ediliyor..."

              EGER SONUC_HAZIR_MI(TC) İSE
                  YAZ "Tahlil Sonuçlarınız Hazır:"
                  SONUCU_GOSTER(TC)

                  YAZ "PDF olarak indirmek ister misiniz? (E/H)"
                  CEVAP = GİRİŞ_AL()
                  
                  EGER CEVAP == "E" İSE
                      PDF_INDIR(TC)
                  DEGILSE
                      YAZ "İndirme iptal edildi."
                  İSE SONU
              DEGILSE
                  YAZ "Sonuçlar henüz hazır değil. Lütfen daha sonra tekrar deneyin."
              İSE SONU
          DEGILSE
              YAZ "Tahlil kaydı bulunamadı."
          İSE SONU
      DEGILSE
          YAZ "Geçersiz işlem seçtiniz."
      İSE SONU

      // Tekrar işlem yapmak isteği
      DONGU
          YAZ "Başka bir işlem yapmak ister misiniz? (E/H)"
          CEVAP2 = GİRİŞ_AL()
          
          EGER CEVAP2 == "E" İSE
              TEKRAR BASA_DON()
          DEGILSE
              CIKIS
          İSE SONU
      DONGU SONU
  DEGILSE
      YAZ "Kimlik doğrulama başarısız! Giriş reddedildi."
  İSE SONU

BITIR
