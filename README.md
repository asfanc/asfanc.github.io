<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ANTALYA 07 - Süre Hesaplayıcı</title>
    <style>
        /* CSS Başlangıcı */
        body {
            /* Kırmızı-Beyaz Dikey Çizgili Arka Plan */
            background: repeating-linear-gradient(
                to right,
                #FF0000, /* Kırmızı */
                #FF0000 100px, /* Kırmızı genişliği */
                #FFFFFF 100px, /* Beyaz başlangıcı */
                #FFFFFF 200px /* Beyaz genişliği (Kırmızı ile aynı) */
            );
            font-family: 'Impact', 'Arial Black', sans-serif; /* Daha sıkı ve kalın bir font */
            color: #333; /* Genel metin rengi */
            text-align: center;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            min-height: 100vh;
        }

        /* Antalyaspor Logosu Stil Tanımı */
        .logo-container {
            width: 180px; /* Logoyu biraz büyüttük */
            height: 180px;
            margin-bottom: 30px;
            background-color: white; 
            border-radius: 50%;
            border: 8px solid red; /* Daha kalın kenarlık */
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.4); /* Daha belirgin gölge */
            position: relative; 
            overflow: hidden; /* Eğer görsel kullanılırsa taşmayı engeller */
        }
        
        .logo-text {
            font-size: 36px;
            font-weight: 900;
            color: red;
            text-shadow: 2px 2px 0px #eee; /* Logo metnine hafif gölge */
            letter-spacing: 2px;
        }

        /* ANTALYA 07 Başlık Stili - EN SIK VE GÖSTERİŞLİ YAZI BURASI */
        #main-title {
            font-size: 5em; /* Çok büyük font */
            font-weight: 900; 
            margin-bottom: 40px;
            padding: 10px 20px;
            background-color: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.5); /* Derin gölge */
            letter-spacing: 8px; /* Harfler arası boşluk */
            text-transform: uppercase; /* Tüm harfler büyük */
        }

        /* Başlıktaki Harflerin Kırmızı/Beyaz Sıralı Stili */
        .red-char {
            color: #FF0000; /* Kırmızı */
            /* Kırmızı harflere beyaz kontür/gölge ekleyerek sık ve belirgin yaptık */
            text-shadow: 
                 1px  1px 0 #FFFFFF,
                -1px -1px 0 #FFFFFF,
                -2px  2px 0 #FFFFFF,
                 2px -2px 0 #FFFFFF;
        }
        .white-char {
            color: #FFFFFF; /* Beyaz */
            /* Beyaz harflere kırmızı kontür/gölge ekleyerek sık ve belirgin yaptık */
            text-shadow: 
                 1px  1px 0 #FF0000,
                -1px -1px 0 #FF0000,
                -2px  2px 0 #FF0000,
                 2px -2px 0 #FF0000,
                 4px  4px 8px rgba(0, 0, 0, 0.7); /* Derin gölge */
        }

        /* Hesaplama Sonuçları Alanı */
        #results-container {
            background-color: rgba(255, 255, 255, 0.95);
            padding: 30px 40px;
            border-radius: 15px;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.5);
            max-width: 650px;
            width: 90%;
            text-align: left;
            border: 3px solid #FF0000; /* Kırmızı çerçeve */
        }

        h2 {
            color: #FF0000;
            border-bottom: 4px solid #FF0000; /* Kalın alt çizgi */
            padding-bottom: 15px;
            margin-top: 0;
            font-size: 2em;
            font-weight: 900;
            text-transform: uppercase;
        }

        .time-data {
            font-size: 1.8em; /* Sonuç metinleri büyük */
            margin: 20px 0;
            font-weight: 900; /* Çok kalın */
            color: #333;
            letter-spacing: 1px;
            text-shadow: 1px 1px 1px #ccc; /* Metinlere hafif gölge */
        }

        .label {
            color: #FF0000; /* Etiketler kırmızı ve kalın */
            font-weight: 900;
            margin-right: 10px;
            display: inline-block;
            min-width: 150px;
        }

        /* CSS Sonu */
    </style>
</head>
<body>

    <div class="logo-container">
        <span class="logo-text">A S</span> 
    </div>

    <h1 id="main-title"></h1>

    <div id="results-container">
        <h2>⏱️ DOĞUMDAN BUGÜNE GEÇEN SÜRE</h2>
        <div class="time-data"><span class="label">DOĞUM TARİHİ:</span> 28 Kasım 2011, 10:07:00</div>
        <div class="time-data" id="days-passed"></div>
        <div class="time-data" id="hours-passed"></div>
        <div class="time-data" id="seconds-passed"></div>
        <div class="time-data" id="update-info"><span class="label">SON GÜNCELLEME:</span> <span id="current-time"></span></div>
    </div>


    <script>
        // JavaScript Başlangıcı

        // 1. Doğum Tarihini Tanımlama
        // Not: Zamanı 28 Ekim 2025 14:49:45'ten hesaplayacaktır
        const birthDate = new Date('2011-11-28T10:07:00'); 

        // 2. ANTALYA 07 Başlığını Oluşturma (Kırmızı-Beyaz Sıralı)
        function createTitle() {
            const titleText = "ANTALYA 07";
            const titleElement = document.getElementById('main-title');
            let html = '';

            for (let i = 0; i < titleText.length; i++) {
                const char = titleText[i];
                // Harf/Sayı tek (0, 2, 4...) ise kırmızı, çift (1, 3, 5...) ise beyaz
                // Başlık harflerine kırmızı-beyaz sırasını uyguluyoruz
                const className = (i % 2 === 0) ? 'red-char' : 'white-char'; 
                html += `<span class="${className}">${char}</span>`;
            }
            titleElement.innerHTML = html;
        }

        // 3. Süreyi Hesaplama ve Güncelleme Fonksiyonu
        function updateTime() {
            const now = new Date();
            const timeDifferenceMs = now - birthDate; // Milisaniye cinsinden fark

            // Hesaplamalar
            const totalSeconds = Math.floor(timeDifferenceMs / 1000);
            const totalHours = Math.floor(timeDifferenceMs / (1000 * 60 * 60));
            const totalDays = Math.floor(totalHours / 24);

            // Sonuçları HTML'e Yazma
            document.getElementById('days-passed').innerHTML = 
                `<span class="label">GEÇEN GÜN:</span> ${totalDays.toLocaleString('tr-TR')} GÜN`; 
            
            document.getElementById('hours-passed').innerHTML = 
                `<span class="label">GEÇEN SAAT:</span> ${totalHours.toLocaleString('tr-TR')} SAAT`;

            // Saniye sürekli güncelleneceği için, saniyeyi her zaman yeniden hesaplayıp yazıyoruz.
            document.getElementById('seconds-passed').innerHTML = 
                `<span class="label">GEÇEN SANİYE:</span> ${totalSeconds.toLocaleString('tr-TR')} SANİYE`;

            // Son Güncelleme Zamanını Yazma
            document.getElementById('current-time').textContent = now.toLocaleString('tr-TR');
        }

        // Başlığı oluştur
        createTitle();

        // Sayfa yüklendiğinde bir kez çalıştır
        updateTime();

        // Her saniye tekrar çalıştır (Saniyeyi sürekli güncellemek için)
        setInterval(updateTime, 1000);

        // JavaScript Sonu
    </script>
</body>
</html>

