BASLA
  // Sistem başlangıcı ve aktiflik kontrolü
  BASLA
    YAZ "Sistem başlatılıyor..."
    SISTEM_AKTIF = TRUE
  BITIR

  // Sürekli ana döngü: sensör okuma ve tehdit tespiti
  DONGU  // sürekli çalışır
    BASLA
      // 1) Sistem aktif mi?
      BASLA
        EGER-ISE SISTEM_AKTIF == FALSE
          YAZ "Sistem pasif. Bekleniyor..."
          DONGU  // döngü içinde bekle ve tekrar kontrol et
            BASLA
              SLEEP(1 dakika)
              EGER-ISE SISTEM_AKTIF == TRUE
                BREAK  // sisteme tekrar geçiş
              İSE SONU
            DONGU SONU
        DEGILSE
          // Sistem aktifse sensör okumalarına devam et
        İSE SONU
      BITIR

      // 2) Sensör okumaları (hareket, kapı/pencere, kamera durumu vb.)
      BASLA
        HAREKET_VERI = SENSOR_OKU("hareket")
        KAPI_VERI = SENSOR_OKU("kapi_pencere")
        CAM_ONLINE = KAMERA_DURUMU()
        BATTERY_STATUS = SENSOR_BATTERY_STATUS()
        ZAMAN_STAMP = SIMDI()
      BITIR

      // 3) Tehdit tespiti: hangi sensörler tetiklendi?
      BASLA
        TEHDITLER = []
        EGER-ISE HAREKET_VERI == "TETIK" İSE
          TEHDITLER'E_EKLE("Hareket")
        İSE SONU

        EGER-ISE KAPI_VERI == "ACILDI" İSE
          TEHDITLER'E_EKLE("Kapı/Pencere")
        İSE SONU

        // Diğer sensörler eklenebilir (cam kırılma, duman, gaz vb.)
      BITIR

      // 4) Eğer tehdit yoksa kısa bekle ve döngüye geri dön
      BASLA
        EGER-ISE UZUNLUK(TEHDITLER) == 0 İSE
          SLEEP(5 saniye)
          CONTINUE  // döngü başa dönsün
        İSE SONU
      BITIR

      // 5) Kamera aktivasyonu - varsa kayıt ve canlı yayın başlat
      BASLA
        EGER-ISE CAM_ONLINE == TRUE İSE
          KAMERA_BASLAT("kayit")
          KAMERA_CANLI_YAYIN(BASLAT = TRUE)
        DEGILSE
          YAZ "Kamera çevrimiçi değil, sadece sensör verisi var."
        İSE SONU
      BITIR

      // 6) Yanlış alarm kontrolü: ev sahibi evde mi? (geofencing / kullanıcı onayı)
      BASLA
        EV_SAHIBI_EVINDE = KULLANICI_DURUMU("evde_mi")  // TRUE/FALSE/UNKNOWN
        EGER-ISE EV_SAHIBI_EVINDE == TRUE İSE
          // Ev sahibinin evde olduğu durumda yanlış alarm olma olasılığı yüksek
          YAZ "Ev sahibi evde: Ön alarm (doğrulama gerekli)."
          // Ev sahibine otomatik bildirim / doğrulama sorusu gönder
          BILDIRIM_GONDER("owner", "Önemli: Evde hareket tespit edildi. Alarm onaylıyor musunuz? (E/H)")
          // Kısa süre bekle doğrulama için
          DONGU
            BASLA
              SLEEP(20 saniye)
              OWNER_RESPONSE = BILDIRI_CEVAP_AL("owner")
              EGER-ISE OWNER_RESPONSE == "E" İSE
                // Ev sahibi alarmı onayladı -> tam alarm
                BREAK
              EGER-ISE OWNER_RESPONSE == "H" İSE
                // Ev sahibi yanlış alarm olduğunu onayladı -> alarmı sıfırla
                YAZ "Ev sahibi yanlış alarm bildirdi. Alarm sıfırlanıyor."
                ALARM_DURUM = "SIFIRLANMIS"
                ALARM_RESET()
                CONTINUE  // ana döngüye dön
              DEGILSE
                // cevap yoksa tekrar kontrol et veya zaman aşımı
                EGER-ISE SURE > 60 saniye İSE
                  BREAK
                İSE SONU
              İSE SONU
            BITIR
          DONGU SONU
        DEGILSE
          YAZ "Ev sahibi evde değil veya durum bilinmiyor: alarm prosedürü devam ediyor."
        İSE SONU
      BITIR

      // 7) Alarm seviyesi belirleme (1: düşük, 2: orta, 3: yüksek)
      BASLA
        ALARM_SEVIYESI = 1  // varsayılan düşük
        EGER-ISE "Kapı/Pencere" IN TEHDITLER VE "Hareket" IN TEHDITLER İSE
          ALARM_SEVIYESI = 3
        DEGILSE EGER "Kapı/Pencere" IN TEHDITLER VE "Hareket" NOT IN TEHDITLER İSE
          ALARM_SEVIYESI = 2
        DEGILSE EGER "Hareket" IN TEHDITLER VE EK_SENSOR_GUVENIYORSA İSE
          ALARM_SEVIYESI = 2
        DEGILSE
          ALARM_SEVIYESI = 1
        İSE SONU
        YAZ "Belirlenen alarm seviyesi: ", ALARM_SEVIYESI
      BITIR

      // 8) Bildirim oluşturma ve gönderme (SMS, App, Email)
      BASLA
        MESAJ = FORMAT("Alarm Seviyesi: %d. Tespit: %s. Zaman: %s", ALARM_SEVIYESI, TEHDITLER, ZAMAN_STAMP)
        BILDIRIM_GONDER("sms", MESAJ)
        BILDIRIM_GONDER("app", MESAJ)
        BILDIRIM_GONDER("email", MESAJ)
        LOG_YAZ("ALARM", MESAJ)
      BITIR

      // 9) Bekle ve tekrar kontrol et (escalation / tekrar doğrulama)
      BASLA
        EGER-ISE ALARM_SEVIYESI == 3 İSE
          // Yüksek seviye -> acil durum servisleri ile paylaşma opsiyonu
          YAZ "Yüksek alarm: Acil durum servisine bildirim gönderilsin mi? (E/H)"
          RESP = KULLANICI_ONAYI_AL()  // sistem ayarlarına göre otomatik olabilir
          EGER-ISE RESP == "E" İSE
            SERVIS_BILDIR("112/Polis", MESAJ)
            LOG_YAZ("ESKALASYON", "Acil servise bildirildi.")
          DEGILSE
            YAZ "Acil servis bildirimi iptal edildi."
          İSE SONU
        DEGILSE
          // Orta/düşük seviyede, kısa süreli tekrar izle
          YAZ "Kısa süreli izleme başlıyor..."
          FOR i FROM 1 TO 6 DONGU  // örn. 6 kere 10 saniye = 60 sn izle
            BASLA
              SLEEP(10 saniye)
              YENI_HAREKET = SENSOR_OKU("hareket")
              YENI_KAPI = SENSOR_OKU("kapi_pencere")
              EGER-ISE YENI_HAREKET == "TETIK" VE YENI_KAPI == "ACILDI" İSE
                YAZ "İzleme sırasında artan tehdit tespit edildi. Yeniden değerlendiriliyor..."
                BREAK
              İSE
                CONTINUE
              İSE SONU
            BITIR
          DONGU SONU
        İSE SONU
      BITIR

      // 10) Alarm sıfırlama veya devam ettirme
      BASLA
        // Kullanıcı müdahalesi veya otomatik sıfırlama kontrolleri
        EGER-ISE KULLANICI_MUDAHALE_EDDI_MI() İSE
          YAZ "Kullanıcı müdahalesi tespit edildi. Alarm sıfırlanıyor."
          ALARM_RESET()
          CONTINUE  // ana döngüye dön
        DEGILSE
          EGER-ISE TEHDITLER devam ediyor İSE
            YAZ "Tehdit devam ediyor. Alarm aktif kalıyor."
            // gerekirse tekrar bildirim gönder veya ısrarla servis çağır
            CONTINUE
          DEGILSE
            YAZ "Tehdit sona erdi. Alarm otomatik sıfırlanıyor."
            ALARM_RESET()
            CONTINUE
          İSE SONU
        İSE SONU
      BITIR

    BITIR
  DONGU SONU

BITIR
