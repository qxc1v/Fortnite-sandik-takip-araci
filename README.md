# Fortnite-Sand-k-Sayac-Dashboard-Canl-Takip-Arac-

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


"
import base64
import json
import os
import sys
import threading
import time
import subprocess
import keyboard
import shutil

APP_NAME = "Sayaç Temel"  # Kurulum klasörünün adı
SHORTCUT_NAME = "Fortnite Sandik Sayaci.lnk"  # Masaüstü kısayolunun adı
FROZEN = getattr(sys, "frozen", False)
CREATE_NO_WINDOW = 0x08000000


def get_desktop_path():
  """Masaüstünü doğru bulur (OneDrive'a taşınmış olsa bile)."""
  try:
    import winreg

    with winreg.OpenKey(
        winreg.HKEY_CURRENT_USER,
        r"Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders",
    ) as k:
      value, _ = winreg.QueryValueEx(k, "Desktop")
      return os.path.expandvars(value)
  except Exception:
    return os.path.join(os.path.expanduser("~"), "Desktop")


def create_desktop_shortcut(target_exe, work_dir):
  """Masaüstüne, kurulum klasöründeki exe'yi açan bir kısayol oluşturur."""
  shortcut_path = os.path.join(get_desktop_path(), SHORTCUT_NAME)

  def q(s):
    return s.replace("'", "''")

  ps_script = f"""
$ws = New-Object -ComObject WScript.Shell
$s = $ws.CreateShortcut('{q(shortcut_path)}')
$s.TargetPath = '{q(target_exe)}'
$s.WorkingDirectory = '{q(work_dir)}'
$s.IconLocation = '{q(target_exe)},0'
$s.Save()
"""
  # EncodedCommand: Türkçe karakterli yollarda sorun çıkmasın diye
  encoded = base64.b64encode(ps_script.encode("utf-16-le")).decode()
  try:
    subprocess.run(
        ["powershell", "-NoProfile", "-NonInteractive", "-EncodedCommand", encoded],
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL,
        creationflags=CREATE_NO_WINDOW,
        timeout=20,
    )
  except Exception:
    pass
  return shortcut_path


if FROZEN:
  # Klasör masaüstünde DEĞİL: C:\Users\<kullanıcı>\AppData\Local\Sayaç Temel
  base = os.environ.get("LOCALAPPDATA") or os.path.expanduser("~")
  target_folder = os.path.join(base, APP_NAME)
  os.makedirs(target_folder, exist_ok=True)

  current_exe = os.path.abspath(sys.executable)
  target_exe = os.path.join(target_folder, os.path.basename(current_exe))
  DATA_FILE = os.path.join(target_folder, "sandik_gecmisi.json")

  if current_exe.lower() != os.path.abspath(target_exe).lower():
    # İlk çalıştırma: exe'yi klasöre kopyala, kısayol aç, oradan başlat
    try:
      try:
        shutil.copy2(current_exe, target_exe)
      except PermissionError:
        pass  # Hedef zaten çalışıyor olabilir, mevcut kopyayı kullan

      if os.path.exists(target_exe):
        create_desktop_shortcut(target_exe, target_folder)

        env = os.environ.copy()
        env["PYINSTALLER_RESET_ENVIRONMENT"] = "1"  # onefile yeniden başlatma hatasını önler
        subprocess.Popen([target_exe], cwd=target_folder, env=env)

        sys.exit(0)
    except Exception:
      pass  # Kurulum başarısız olursa program bulunduğu yerden çalışmaya devam eder
  else:
    # Zaten kurulum klasöründeyiz; kısayol silindiyse yeniden oluştur
    if not os.path.exists(os.path.join(get_desktop_path(), SHORTCUT_NAME)):
      create_desktop_shortcut(target_exe, target_folder)
else:
  # Normal .py olarak test ederken scriptin yanında çalışsın
  BASE_DIR = os.path.dirname(os.path.abspath(__file__))
  DATA_FILE = os.path.join(BASE_DIR, "sandik_gecmisi.json")

# Windows konsol buffer temizliği için
if os.name == "nt":
  import msvcrt

  os.system("")


# Tema Tanımlamaları (ANSI Renk Kodları)
THEMES = {
    "eva": {
        "name": "EVA-01 (Mor & Yeşil - Klasik)",
        "p": "\033[95m",
        "g": "\033[92m",
        "c": "\033[96m",
        "y": "\033[93m",
        "r": "\033[91m",
        "w": "\033[97m",
        "gray": "\033[90m",
    },
    "cyberpunk": {
        "name": "Cyberpunk (Neon Cyan & Sarı)",
        "p": "\033[96m",
        "g": "\033[93m",
        "c": "\033[95m",
        "y": "\033[93m",
        "r": "\033[91m",
        "w": "\033[97m",
        "gray": "\033[90m",
    },
    "matrix": {
        "name": "Matrix (Hacker Yeşili)",
        "p": "\033[32m",
        "g": "\033[92m",
        "c": "\033[36m",
        "y": "\033[32m",
        "r": "\033[31m",
        "w": "\033[37m",
        "gray": "\033[90m",
    },
    "blood": {
        "name": "Blood Moon (Koyu Kırmızı)",
        "p": "\033[91m",
        "g": "\033[93m",
        "c": "\033[31m",
        "y": "\033[93m",
        "r": "\033[91m",
        "w": "\033[97m",
        "gray": "\033[90m",
    },
}

RESET = "\033[0m"


def takip_sayisi(sess):
  """Sadece tuşla takip edilen sandık sayısı (manuel ekleme/çıkarma hariç)."""
  return sess["sandik"] - sess.get("manuel", 0)


def load_data():
  if os.path.exists(DATA_FILE):
    try:
      with open(DATA_FILE, "r", encoding="utf-8") as f:
        data = json.load(f)
        if isinstance(data, list):
          sessions = data
          for s in sessions:
            if "sure_sn" not in s:
              s["sure_sn"] = 0
            if "isim" not in s:
              s["isim"] = f"Oturum {s['oturum']}"
          return "eva", "e", sessions
        elif isinstance(data, dict):
          theme = data.get("theme", "eva")
          key = data.get("key", "e")
          sessions = data.get("sessions", [])
          for s in sessions:
            if "sure_sn" not in s:
              s["sure_sn"] = 0
            if "isim" not in s:
              s["isim"] = f"Oturum {s['oturum']}"
          return theme, key, sessions
    except Exception:
      pass
  return "eva", "e", [
      {
          "oturum": 1,
          "isim": "İlk Oturum",
          "tarih": time.strftime("%d.%m.%Y %H:%M"),
          "sandik": 0,
          "sure_sn": 0,
      }
  ]


def save_data(theme, key, sessions):
  data = {"theme": theme, "key": key, "sessions": sessions}
  try:
    with open(DATA_FILE, "w", encoding="utf-8") as f:
      json.dump(data, f, ensure_ascii=False, indent=4)
  except PermissionError:
    pass


def clear_screen():
  os.system("cls" if os.name == "nt" else "clear")


def flush_input():
  """Konsolda biriken sahipsiz tuşları tamamen temizler"""
  if os.name == "nt":
    while msvcrt.kbhit():
      msvcrt.getch()


# Verileri ve ayarları yükle
current_theme_key, current_key, sessions = load_data()
if not sessions:
  sessions.append(
      {
          "oturum": 1,
          "isim": "İlk Oturum",
          "tarih": time.strftime("%d.%m.%Y %H:%M"),
          "sandik": 0,
          "sure_sn": 0,
      }
  )
  save_data(current_theme_key, current_key, sessions)

current_session = sessions[-1]
data_lock = threading.Lock()
is_running = True


# Arka planda dinamik tuşu dinleyen ve süreyi artıran döngü
def background_worker():
  global current_session
  is_pressed = False
  press_start = 0
  last_time = time.time()
  save_counter = 0

  while is_running:
    now = time.time()
    delta = now - last_time

    if delta >= 1.0:
      with data_lock:
        current_session["sure_sn"] += int(delta)
        save_counter += 1
        if save_counter >= 5:
          save_data(current_theme_key, current_key, sessions)
          save_counter = 0
      last_time = now

    try:
      with data_lock:
        target_key = current_key

      if keyboard.is_pressed(target_key):
        if not is_pressed:
          is_pressed = True
          press_start = time.time()
      else:
        if is_pressed:
          duration = time.time() - press_start
          is_pressed = False
          if 0.5 <= duration <= 1.5:
            with data_lock:
              current_session["sandik"] += 1
              save_data(current_theme_key, current_key, sessions)
    except Exception:
      pass

    time.sleep(0.04)


# Arka plan işçisini başlat
t = threading.Thread(target=background_worker, daemon=True)
t.start()


def format_sure(saniye):
  dakika = saniye // 60
  sn = saniye % 60
  return f"{dakika}dk {sn}sn"


def main_menu():
  global current_session, current_theme_key, current_key, sessions, is_running
  while True:
    flush_input()
    clear_screen()

    th = THEMES.get(current_theme_key, THEMES["eva"])
    p, g, c, y, r, w, gray = (
        th["p"],
        th["g"],
        th["c"],
        th["y"],
        th["r"],
        th["w"],
        th["gray"],
    )

    with data_lock:
      grand_total = sum(s["sandik"] for s in sessions)
      active_count = current_session["sandik"]
      active_oturum = current_session["oturum"]
      active_sure = current_session["sure_sn"]
      active_isim = current_session["isim"]
      active_key = current_key.upper()

    print(p + "========================================" + RESET)
    print(p + "  FORTNITE SANDIK SAYACI // DASHBOARD" + RESET)
    print(p + "========================================" + RESET)

    print("\n" + c + "[ GEÇMİŞ OTURUMLAR ]" + RESET)
    for s in sessions:
      marker = g + " (Aktif)" + RESET if s["oturum"] == active_oturum else ""
      sure_str = format_sure(s.get("sure_sn", 0))
      s_isim = s.get("isim", f"Oturum {s['oturum']}")
      print(
          gray
          + f"> [{s_isim}] ({s['tarih']}): "
          + w
          + f"{s['sandik']} sandık "
          + gray
          + f"({sure_str})"
          + marker
          + RESET
      )

    print(c + "----------------------------------------" + RESET)
    print(
        "🔥 "
        + y
        + f"AKTİF OTURUM: {active_isim} (Oturum {active_oturum})"
        + RESET
    )
    print("📦 " + g + f"Bu Oturumdaki Sandık : {active_count}" + RESET)
    print("⏱️  " + c + f"Oturum Süresi        : {format_sure(active_sure)}" + RESET)
    print("💎 " + p + f"GENEL TOPLAM         : {grand_total} sandık" + RESET)
    print(
        gray
        + f"⌨️  Sayım Tuşu          : [{active_key}] | Tema: {th['name']}"
        + RESET
    )
    print(p + "========================================" + RESET)
    print(w + "[1] Canlı Takip Modu (Oyuna Geç & Anlık Gör)" + RESET)
    print(w + "[2] Manuel Sandık Ekle (Miktar Belirle)" + RESET)
    print(w + "[3] Manuel Sandık Çıkar (Miktar Belirle)" + RESET)
    print(w + "[4] Yeni Oturum Başlat (İsim Ver & Sıfırla)" + RESET)
    print(w + "[5] Aktif Oturumun Adını Düzenle" + RESET)
    print(w + "[6] Sayım Tuşunu Değiştir (Şu an: " + active_key + ")" + RESET)
    print(w + "[7] Tema Değiştir" + RESET)
    print(c + "[8] İstatistik & Özet Paneli (All-Time Stats)" + RESET)
    print(r + "[9] Tüm Geçmişi Sıfırla (Sıfırdan Başla)" + RESET)
    print(r + "[10] Çıkış" + RESET)
    print(p + "========================================" + RESET)

    flush_input()
    choice = input(y + "Seçiminiz (1-10): " + RESET).strip()

    if choice == "1":
      clear_screen()
      print(p + "========================================" + RESET)
      print(p + "       CANLI TAKİP MODU AKTİF" + RESET)
      print(p + "========================================" + RESET)
      print(
          f" Oyun oynarken [{active_key}] tuşuna basarak sandık sayacaksın.\n"
      )
      print(" Menüye dönmek için klavyeden " + y + "'Q'" + RESET + " tuşuna bas.\n")
      print(c + "----------------------------------------" + RESET)

      flush_input()
      last_printed_count = -1
      last_printed_time = -1

      while True:
        with data_lock:
          curr_c = current_session["sandik"]
          curr_t = takip_sayisi(current_session)
          curr_s = current_session["sure_sn"]

        dakika = curr_s / 60.0
        dks = curr_t / dakika if dakika > 0 else 0.0

        if curr_c != last_printed_count or curr_s != last_printed_time:
          sure_str = format_sure(curr_s)
          print(
              f"\r{g}📦 Sandık: {w}{curr_c} {c}| ⏱️ Süre: {w}{sure_str} {y}| ⚡ DKS: {w}{dks:.2f} sandık/dk{RESET}   ",
              end="",
              flush=True,
          )
          last_printed_count = curr_c
          last_printed_time = curr_s

        # Çıkış tuşları SADECE bu konsol penceresi odaktayken algılanır
        # (msvcrt yalnızca aktif konsolun klavye girdisini okur, oyundaki Q'yu görmez)
        quit_requested = False
        while msvcrt.kbhit():
          ch = msvcrt.getch()
          if ch in (b"\x00", b"\xe0"):  # F tuşu / ok tuşu gibi özel tuşların 2. baytını at
            msvcrt.getch()
          elif ch.lower() == b"q" or ch == b"\x1b":  # Q veya ESC
            quit_requested = True
        if quit_requested:
          flush_input()
          break
        time.sleep(0.1)

    elif choice == "2":
      flush_input()
      try:
        val = int(input(g + "Eklenecek sandık miktarını girin: " + RESET).strip())
        if val > 0:
          with data_lock:
            current_session["sandik"] += val
            current_session["manuel"] = current_session.get("manuel", 0) + val
            save_data(current_theme_key, current_key, sessions)
          print(g + f"[✔] {val} sandık başarıyla eklendi!" + RESET)
        else:
          print(r + "[✖] 0'dan büyük bir sayı girmelisin!" + RESET)
      except ValueError:
        print(r + "[✖] Geçersiz değer girdin!" + RESET)
      time.sleep(1.2)

    elif choice == "3":
      flush_input()
      try:
        val = int(
            input(r + "Çıkarılacak sandık miktarını girin: " + RESET).strip()
        )
        if val > 0:
          with data_lock:
            cikarilan = min(val, current_session["sandik"])
            current_session["sandik"] -= cikarilan
            current_session["manuel"] = current_session.get("manuel", 0) - cikarilan
            save_data(current_theme_key, current_key, sessions)
          print(g + f"[✔] {val} sandık çıkarıldı!" + RESET)
        else:
          print(r + "[✖] 0'dan büyük bir sayı girmelisin!" + RESET)
      except ValueError:
        print(r + "[✖] Geçersiz değer girdin!" + RESET)
      time.sleep(1.2)

    elif choice == "4":
      flush_input()
      isim_input = (
          input(
              y
              + "Oturum için bir isim gir (Örn: Solo Grind, Boş bırakırsan 'Oturum X' olur): "
              + RESET
          )
          .strip()
      )
      with data_lock:
        new_id = sessions[-1]["oturum"] + 1
        s_isim = (
            isim_input if isim_input else f"Oturum {new_id}"
        )
        new_sess = {
            "oturum": new_id,
            "isim": s_isim,
            "tarih": time.strftime("%d.%m.%Y %H:%M"),
            "sandik": 0,
            "sure_sn": 0,
        }
        sessions.append(new_sess)
        current_session = new_sess
        save_data(current_theme_key, current_key, sessions)
      print(f"\n{g}[✔] Yeni Oturum '{s_isim}' başlatıldı!{RESET}")
      time.sleep(1.2)

    elif choice == "5":
      flush_input()
      yeni_isim = input(
          y + f"Aktif oturum için yeni isim gir ({active_isim}): " + RESET
      ).strip()
      if yeni_isim:
        with data_lock:
          current_session["isim"] = yeni_isim
          save_data(current_theme_key, current_key, sessions)
        print(g + f"[✔] Oturum adı '{yeni_isim}' olarak güncellendi!" + RESET)
      else:
        print(r + "[✖] İsim boş olamaz!" + RESET)
      time.sleep(1.2)

    elif choice == "6":
      clear_screen()
      print(p + "========================================" + RESET)
      print(p + "          SAYIM TUŞUNU DEĞİŞTİR" + RESET)
      print(p + "========================================" + RESET)
      print(f"Mevcut tuş: {active_key}")
      print("Örnek tuşlar: e, f, r, space, shift, ctrl vb.\n")
      flush_input()
      yeni_tus = (
          input(y + "Yeni tuşu yazın (örn: f): " + RESET).strip().lower()
      )
      if yeni_tus:
        with data_lock:
          current_key = yeni_tus
          save_data(current_theme_key, current_key, sessions)
        print(
            g
            + f"[✔] Sayım tuşu başarıyla [{yeni_tus.upper()}] olarak değiştirildi!"
            + RESET
        )
      else:
        print(r + "[✖] Geçersiz tuş!" + RESET)
      time.sleep(1.2)

    elif choice == "7":
      clear_screen()
      print(p + "========================================" + RESET)
      print(p + "           TEMA SEÇİMİ" + RESET)
      print(p + "========================================" + RESET)
      keys = list(THEMES.keys())
      for idx, k in enumerate(keys, 1):
        print(f" [{idx}] {THEMES[k]['name']}")
      print(p + "========================================" + RESET)
      flush_input()
      t_choice = input(
          y + f"Seçtiğiniz Tema (1-{len(keys)}): " + RESET
      ).strip()
      try:
        idx_val = int(t_choice) - 1
        if 0 <= idx_val < len(keys):
          current_theme_key = keys[idx_val]
          save_data(current_theme_key, current_key, sessions)
          print(g + "[✔] Tema başarıyla değiştirildi!" + RESET)
        else:
          print(r + "[✖] Geçersiz seçim!" + RESET)
      except ValueError:
        print(r + "[✖] Sayı girmelisin!" + RESET)
      time.sleep(1.2)

    elif choice == "8":
      clear_screen()
      print(p + "========================================" + RESET)
      print(p + "        TÜM ZAMANLAR İSTATİSTİKLERİ" + RESET)
      print(p + "========================================" + RESET)
      with data_lock:
        total_oturum = len(sessions)
        total_sandik = sum(takip_sayisi(s) for s in sessions)
        total_manuel = sum(s.get("manuel", 0) for s in sessions)
        total_sure = sum(s.get("sure_sn", 0) for s in sessions)
        
        best_sandik_sess = max(sessions, key=takip_sayisi) if sessions else None
        
        max_dks = 0
        best_dks_sess_isim = "-"
        for s in sessions:
          s_sure_dk = s.get("sure_sn", 0) / 60.0
          s_dks = takip_sayisi(s) / s_sure_dk if s_sure_dk > 0 else 0.0
          if s_dks > max_dks:
            max_dks = s_dks
            best_dks_sess_isim = s.get("isim", f"Oturum {s['oturum']}")

      toplam_saat = total_sure // 3600
      toplam_dakika = (total_sure % 3600) // 60
      toplam_saniye = total_sure % 60
      
      if toplam_saat > 0:
        sure_str = f"{toplam_saat} saat {toplam_dakika} dk"
      else:
        sure_str = f"{toplam_dakika} dk {toplam_saniye} sn"
      
      genel_dakika = total_sure / 60.0
      genel_dks = total_sandik / genel_dakika if genel_dakika > 0 else 0.0

      print(f"📁 Toplam Oturum Sayısı : {w}{total_oturum}{RESET}")
      print(f"📦 Toplam Açılan Sandık : {g}{total_sandik}{RESET} {gray}(sadece takip edilenler){RESET}")
      if total_manuel != 0:
        print(f"✍️  Manuel Ayarlama      : {gray}{total_manuel:+d} (istatistiklere dahil değil){RESET}")
      print(f"⏱️ Toplam Oynama Süresi : {c}{sure_str}{RESET}")
      print(f"⚡ Genel Ortalama DKS   : {y}{genel_dks:.2f} sandık/dk{RESET}")
      print(c + "----------------------------------------" + RESET)
      if best_sandik_sess:
        print(f"🏆 En Çok Sandık (Oturum): {w}{best_sandik_sess.get('isim')} ({takip_sayisi(best_sandik_sess)} sandık){RESET}")
      print(f"🚀 En Yüksek Verimlilik : {w}{best_dks_sess_isim} ({max_dks:.2f} DKS){RESET}")
      print(p + "========================================" + RESET)
      flush_input()
      input(gray + "Ana menüye dönmek için Enter'a bas..." + RESET)

    elif choice == "9":
      clear_screen()
      print(r + "========================================" + RESET)
      print(r + "        TÜM GEÇMİŞİ SIFIRLAMA" + RESET)
      print(r + "========================================" + RESET)
      flush_input()
      confirm = (
          input(
              y + "Tüm geçmiş ve oturumlar silinecek. Emin misin? (E/H): " + RESET
          )
          .strip()
          .lower()
      )
      if confirm == "e":
        with data_lock:
          sessions = [
              {
                  "oturum": 1,
                  "isim": "İlk Oturum",
                  "tarih": time.strftime("%d.%m.%Y %H:%M"),
                  "sandik": 0,
                  "sure_sn": 0,
              }
          ]
          current_session = sessions[-1]
          save_data(current_theme_key, current_key, sessions)
        print(g + "[✔] Tüm geçmiş sıfırlandı, 0'dan başlandı!" + RESET)
      else:
        print(c + "[i] Sıfırlama işlemi iptal edildi." + RESET)
      time.sleep(1.5)

    elif choice == "10":
      is_running = False
      with data_lock:
        save_data(current_theme_key, current_key, sessions)
      print(f"\n{p}Sistem kapatılıyor. İyi oyunlar kanka!{RESET}")
      time.sleep(1)
      break


if __name__ == "__main__":
  main_menu()"


  kodlar burda isterseniz deneyebilirsiniz kendiniz. iyi günler dilerim




