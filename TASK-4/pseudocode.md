BASLA
  YAZ "Öğrenci Numarası Giriniz:"
  OGR_NO = GİRİŞ_AL()
  YAZ "Şifre Giriniz:"
  SIFRE = GİRİŞ_AL()

  EGER OGRENCI_GIRIS(OGR_NO, SIFRE) DOĞRU İSE
      YAZ "Hoşgeldiniz!"
      TOPLAM_KREDI = 0
      SEÇİLEN_DERSLER = []

      DONGU (Ders_Listesi içinde HER DERS için)
          DERS = SIRADAKI_DERS
          YAZ "Ders: ", DERS.ADI, " (", DERS.KREDI, " kredi)"

          YAZ "Bu dersi seçmek ister misiniz? (E/H)"
          CEVAP = GİRİŞ_AL()

          EGER CEVAP == "E" İSE
              
              // 1. Kontenjan Kontrolü
              EGER DERS.KONTENJAN > 0 İSE

                  // 2. Önkoşul Kontrolü
                  EGER ONKOSUL_GECILDI_MI(OGR_NO, DERS.ONKOSUL) İSE
                      
                      // 3. Zaman Çakışması Kontrolü
                      EGER ZAMAN_CAKISMASI_YOK_MU(SEÇİLEN_DERSLER, DERS) İSE

                          // 4. Kredi Limiti Kontrolü
                          EGER TOPLAM_KREDI + DERS.KREDI <= 35 İSE
                              SEÇİLEN_DERSLER'E EKLE(DERS)
                              TOPLAM_KREDI = TOPLAM_KREDI + DERS.KREDI
                              DERS.KONTENJAN = DERS.KONTENJAN - 1
                              YAZ "Ders başarıyla eklendi!"
                          DEGILSE
                              YAZ "Kredi limiti aşıldı! Bu dersi alamazsınız."
                          İSE SONU

                      DEGILSE
                          YAZ "Zaman çakışması tespit edildi!"
                      İSE SONU

                  DEGILSE
                      YAZ "Bu dersin önkoşulunu tamamlamadınız!"
                  İSE SONU

              DEGILSE
                  YAZ "Kontenjan dolu! Bu dersi seçemezsiniz."
              İSE SONU
          DEGILSE
              YAZ "Ders atlandı."
          İSE SONU

      DONGU SONU

      // Danışman Onayı Gerekliliği
      EGER GPA(OGR_NO) < 2.5 İSE
          YAZ "Danışman onayı gerekmektedir. Onay bekleniyor..."
          ONAY = DANSISMAN_ONAYI_AL(OGR_NO)
          EGER ONAY == "ONAYLANDI" İSE
              DEVAM_ET = DOĞRU
          DEGILSE
              DEVAM_ET = YANLIŞ
          İSE SONU
      DEGILSE
          DEVAM_ET = DOĞRU
      İSE SONU

      // Kayıt Onaylama
      EGER DEVAM_ET == DOĞRU İSE
          YAZ "Kayıt Özeti:"
          DONGU (SEÇİLEN_DERSLER içinde)
              YAZ DERS.ADI, "-", DERS.KREDI, " kredi"
          DONGU SONU

          YAZ "Toplam Kredi: ", TOPLAM_KREDI
          YAZ "Kayıt Onaylıyor musunuz? (E/H)"
          ONAY = GİRİŞ_AL()

          EGER ONAY == "E" İSE
              KAYIT_TAMAMLA(OGR_NO, SEÇİLEN_DERSLER)
              YAZ "Kayıt başarıyla tamamlandı!"
          DEGILSE
              YAZ "Kayıt iptal edildi."
          İSE SONU
      DEGILSE
          YAZ "Danışman onayı alınamadı. Kayıt tamamlanamadı."
      İSE SONU

  DEGILSE
      YAZ "Giriş başarısız! Öğrenci numarası veya şifre hatalı."
  İSE SONU

BITIR
