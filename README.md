# Fortnite-Sand-k-Sayac-Dashboard-Canl-Takip-Arac-

Dosyada imza (Microsoft sertifikası) olmadığı için uyarı veriyor ek bilgi kısmından yine de çalıştır derseniz sorun olmaz.

Oyun oynarken aralıksız sandık açıyoruz ama "Acaba bu oturum kaç sandık açtım, dakikada kaç kasıyorum?" sorusunun cevabını elle tutmak imkansız olduğu için bu aracı geliştirdik. Tamamen hızlı, şık ve yormayan bir terminal aracıdır.

🌟 Özellikleri Nelerdir?
Canlı Takip: Oyundayken seçtiğin tuşa (örneğin E veya F) bastığın an arka planda sandık sayısını, oturum süresini ve DKS (Dakika Başına Sandık) verimliliğini anlık olarak hesaplar.

Görsel Temalar: 4 farklı cyberpunk/anime esintili renk teması sunar (EVA-01 Mor-Yeşil, Cyberpunk Neon, Matrix Yeşili, Blood Moon).

Oturumlu Sistem: Geçmiş maçlarını ve oturumlarını hafızasında tutar, istatistiklerini kaybetmez.

Akıllı Kurulum (Self-Deployment): Programı ilk açtığın an hiçbir kurulumla uğraşmazsın; arka planda AppData\Local\Sayaç Temel klasörünü kendisi kurar, veritabanını oraya sabitler ve masaüstüne özel ikonlu kısayolunu tertemiz atar.

⚙️ Sistem Kaynakları (CPU ve RAM Durumu)
İçeride ağır arayüz motorları veya gereksiz şişkinlikler olmadığı için cihazı yorması imkansızdır:

İşlemci (CPU): Arka plandaki mini işçi (threading) ve time.sleep(0.04) optimizasyonu sayesinde işlemciyi %0.0 ile %0.1 arasında, yani neredeyse hiç harcamaz. Oyunlarında FPS düşüşü yaşatmaz.

Bellek (RAM): Ortalama 15 MB - 25 MB arası RAM tüketir. Arkada açık unutsan bile unuttuğunu fark etmezsin.

🛠️ Hangi Teknolojilerle Geliştirildi?
Python: Altyapı ve veri yönetimi.

keyboard Kütüphanesi: Oyunda alt-tab yapmadan, tuşlara global (sistem genelinde) olarak tepki vermesini sağlar.

threading (Çoklu İş Parçacığı): Süre sayacı ve tuş dinleme ana menüyü dondurmasın diye arka planda bağımsız bir kolda akar.

🚨 Virustotal'de Neden "Şüpheli / Virüs" Çıkıyor? (False-Positive)
Dosyayı VirusTotal'e attığında bazı küçük motorlar uyarı verebilir. Tamamen yanlış alarmdır. Sebebi şudur:

Global Tuş Dinleme: keyboard kütüphanesi bilgisayardaki tuş hareketlerini izlediği için (oyunda sayım yapabilmek adına) antivirüsler bunu "Keylogger" zanneder. Bizim kodumuz ise sadece senin belirlediğin tek bir tuşu dinler, dışarıya asla veri sızdırmaz.

PyInstaller İmzasısız Olması: Python kodunu tek bir .exe haline getiren bu paketleyici, resmi bir Microsoft dijital imza sertifikasına sahip olmadığı için güvenlik yazılımları dosyayı doğrudan "Bilinmeyen" olarak etiketler.

🔍 "Ben Bu Exe'ye Güvenmiyorum, Kodları Nasıl Kontrol Edebilirim?"
İçinde en ufak bir şüphe kalan biri olursa, .exe dosyasını alıp tekrar saf Python koduna dönüştürebilir ve yazdığımız satırların birebir aynı olduğunu şu yöntemle kanıtlayabilir:

İçini Açmak (pyinstxtractor): Ücretsiz bir Python aracı olan pyinstxtractor kullanılarak .exe dosyası terminalde çözülür (python pyinstxtractor.py sayac.exe).

Kodu Okumak: Çıkan sayac.pyc dosyası okunabilir python formatına çevrildiğinde; yukarıdaki tuş dinleme döngülerinin, AppData\Local\Sayaç Temel klasör yolunun ve JSON kayıt sisteminin kelimesi kelimesine aynı olduğunu kendi gözleriyle görür.

İsteyen doğrudan yukarıdaki ham sayac.py kaynak kodunu inceleyebilir veya kendi bilgisayarında Python ile (python sayac.py) çalıştırabilir.

🗑️ Nasıl Tamamen Kaldırılır?
İşin bittiğinde bilgisayarından iz bırakmadan silmek istersen:

Masaüstündeki Fortnite Sandik Sayaci kısayolunu sil.

Windows + R tuşuna bas, %localappdata% yaz ve Enter'a bas.

Açılan pencerede Sayaç Temel klasörünü bul ve komple sil. Bilgisayarında programdan eser kalmaz!
