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
            font-family: 'Impact', 'Arial Black', sans-serif; /* Sıkı ve kalın font */
            color: #333;
            text-align: center;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
        }

        /* ANTALYA 07 Başlık Stili - SIK ve GÖSTERİŞLİ YAZI */
        #main-title {
            font-size: 5.5em;
            font-weight: 900; 
            margin-bottom: 50px;
            padding: 15px 30px;
            background-color: rgba(255, 255, 255, 0.98);
            border-radius: 20px;
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.6);
            letter-spacing: 10px;
            text-transform: uppercase;
            border: 5px solid #FF0000;
        }

        /* Başlıktaki Harflerin Kırmızı/Beyaz Sıralı Stili */
        .red-char {
            color: #FF0000;
            text-shadow: 
                 2px  2px 0 #FFFFFF,
                -2px -2px 0 #FFFFFF;
        }
        .white-char {
            color: #FFFFFF;
            text-shadow: 
                 2px  2px 0 #FF0000,
                -2px -2px 0 #FF0000,
                 5px  5px 10px rgba(0, 0, 0, 0.8);
        }

        /* Hesaplama Sonuçları Alanı */
        #results-container {
            background-color: rgba(255, 255, 255, 0.98);
            padding: 40px 50px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.7);
            max-width: 700px;
            width: 90%;
            text-align: left;
            border: 5px solid #FF0000;
        }

        h2 {
            color: #FF0000;
            border-bottom: 5px solid #FF0000;
            padding-bottom: 20px;
            margin-top: 0;
            font-size: 2.2em;
            font-weight: 900;
            text-transform: uppercase;
        }

        .time-data {
            font-size: 2em;
            margin: 25px 0;
            font-weight: 900;
            color: #333;
            letter-spacing: 1px;
            text-shadow: 1px 1px 2px #ccc;
        }

        .label {
            color: #FF0000;
            font-weight: 900;
            margin-right: 15px;
            display: inline-block;
            min-width: 170px;
        }
        
        /* Doğum Günü Sayacı İçin Özel Stil */
        #birthday-countdown-container {
            margin-top: 30px;
            border-top: 3px dashed #FF0000;
            padding-top: 30px;
            text-align: center;
        }

        #birthday-countdown-container .time-data {
            font-size: 1.5em; /* Kalan süreyi diğerlerinden biraz küçük tuttuk */
            color: #FF0000; /* Kalan süreyi kırmızı yaptık */
        }
        
        #countdown-label {
            color: #333;
            font-weight: 900;
            text-shadow: none;
            display: block;
            margin-bottom: 10px;
            font-size: 1.2em;
        }

        /* CSS Sonu */
    </style>
</head>
<body>

    <h1 id="main-title"></h1>

    <div id="results-container">
        <h2>⏱️ DOĞUMDAN BUGÜNE GEÇEN SÜRE</h2>
        <div class="time-data"><span class="label">DOĞUM TARİHİ:</span> 28 Kasım 2011, 10:07:00</div>
        <div class="time-data" id="days-passed"></div>
        <div class="time-data" id="hours-passed"></div>
        <div class="time-data" id="seconds-passed"></div>
        <div class="time-data" id="update-info"><span class="label">SON GÜNCELLEME:</span> <span id="current-time"></span></div>
        
        <div id="birthday-countdown-container">
            <span id="countdown-label">BİR SONRAKİ DOĞUM GÜNÜNE KALAN SÜRE:</span>
            <div class="time-data" id="birthday-countdown"></div>
        </div>
    </div>


    <script>
        // JavaScript Başlangıcı

        // 1. Doğum Tarihini Tanımlama
        const birthDate = new Date('2011-11-28T10:07:00'); 

        // 2. ANTALYA 07 Başlığını Oluşturma (Kırmızı-Beyaz Sıralı)
        function createTitle() {
            const titleText = "ANTALYA 07";
            const titleElement = document.getElementById('main-title');
            let html = '';

            for (let i = 0; i < titleText.length; i++) {
                const char = titleText[i];
                const className = (i % 2 === 0) ? 'red-char' : 'white-char'; 
                html += `<span class="${className}">${char}</span>`;
            }
            titleElement.innerHTML = html;
        }

        // 3. Süreyi Hesaplama ve Güncelleme Fonksiyonu
        function updateTime() {
            const now = new Date();
            const timeDifferenceMs = now - birthDate; 

            // Hesaplamalar
            const totalSeconds = Math.floor(timeDifferenceMs / 1000);
            const totalHours = Math.floor(timeDifferenceMs / (1000 * 60 * 60));
            const totalDays = Math.floor(totalHours / 24);

            // Sonuçları HTML'e Yazma
            document.getElementById('days-passed').innerHTML = 
                `<span class="label">GEÇEN GÜN:</span> ${totalDays.toLocaleString('tr-TR')} GÜN`; 
            
            document.getElementById('hours-passed').innerHTML = 
                `<span class="label">GEÇEN SAAT:</span> ${totalHours.toLocaleString('tr-TR')} SAAT`;

            document.getElementById('seconds-passed').innerHTML = 
                `<span class="label">GEÇEN SANİYE:</span> ${totalSeconds.toLocaleString('tr-TR')} SANİYE`;

            // Son Güncelleme Zamanını Yazma
            document.getElementById('current-time').textContent = now.toLocaleString('tr-TR');
            
            // Doğum Günü Sayacını da güncelle
            updateBirthdayCountdown(now);
        }
        
        // 4. Doğum Gününe Kalan Süreyi Hesaplama
        function updateBirthdayCountdown(now) {
            
            // Doğum günü ayı ve günü (28 Kasım)
            const birthMonth = 10; // Kasım (0-indexed: 0=Ocak, 10=Kasım)
            const birthDay = 28;
            const birthHour = 10;
            const birthMinute = 7;
            const birthSecond = 0;

            // Bir sonraki doğum günü tarihini belirle (28 Kasım 10:07)
            let nextBirthday = new Date(now.getFullYear(), birthMonth, birthDay, birthHour, birthMinute, birthSecond);

            // Eğer bir sonraki doğum günü zaten geçmişse, gelecek yılki doğum gününü ayarla
            if (nextBirthday < now) {
                nextBirthday.setFullYear(now.getFullYear() + 1);
            }

            let remainingMs = nextBirthday - now;

            if (remainingMs <= 0) {
                document.getElementById('birthday-countdown').innerHTML = 'İYİ Kİ DOĞDUNUZ! KUTLU OLSUN!';
                return;
            }

            // Milisaniyeyi saniye, dakika, saat, gün, ay ve yıla çevirme
            const seconds = Math.floor((remainingMs / 1000) % 60);
            const minutes = Math.floor((remainingMs / (1000 * 60)) % 60);
            const hours = Math.floor((remainingMs / (1000 * 60 * 60)) % 24);
            const totalDaysRemaining = Math.floor(remainingMs / (1000 * 60 * 60 * 24));
            
            // Günleri Yıl/Ay/Gün olarak ayırma (Yaklaşık hesaplama)
            // Yıl ve ay hesaplamaları, gün sayılarının sabit olmamasından dolayı her zaman *yaklaşık* olacaktır.
            const remainingYears = Math.floor(totalDaysRemaining / 365.25);
            let daysAfterYears = totalDaysRemaining % 365.25;
            
            const remainingMonths = Math.floor(daysAfterYears / 30.44); // Ortalama ay uzunluğu
            const remainingDays = Math.floor(daysAfterYears % 30.44);


            const resultHtml = `
                ${remainingYears} YIL, 
                ${remainingMonths} AY, 
                ${remainingDays} GÜN, 
                ${hours} SAAT, 
                ${minutes} DAKİKA, 
                ${seconds} SANİYE
            `;

            document.getElementById('birthday-countdown').innerHTML = resultHtml;
        }

        // Başlığı oluştur
        createTitle();
        updateTime();

        // Her saniye tekrar çalıştır
        setInterval(updateTime, 1000);

        // JavaScript Sonu
    </script>
</body>
</html>
