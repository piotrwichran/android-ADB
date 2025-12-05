# Kompletny backup i diagnostyka Androida przez ADB (bez roota) — 2025

Najpełniejszy zestaw poleceń ADB, jaki zrobisz na telefonie bez roota.

## Przygotowanie (zrób to raz)

### 1. Sprawdź, czy telefon jest widoczny
```bash
adb devices

# Powinno zwrócić coś takiego:
# List of devices attached
# 1234567890abcdef    device
# Jeśli widzisz unauthorized → odblokuj telefon i zaakceptuj klucz RSA
# Jeśli widzisz offline → wyciągnij i włóż kabel albo zmień port USB
# Jeśli nic nie widać → zainstaluj sterowniki telefonu / inny kabel
```
### 2. (Opcjonalnie) Wejdź do trybu tylko ADB (bez ładowania)
```bash
adb usb
```

### 3. Włącznie wyłącz weryfikację przy każdym połączeniu (wygodne przy skryptach)
```bash
adb kill-server && adb start-server
```


# Pełny backup telefonu z Androidem za pomocą ADB (bez roota)

### Tworzymy główny folder
```bash
mkdir telefon_backup
cd telefon_backup
```
### === 1. Kopiujemy najważniejsze foldery użytkownika ===
```bash
adb pull /sdcard/DCIM                 DCIM
adb pull /sdcard/Pictures              Pictures
adb pull /sdcard/Download              Download
adb pull /sdcard/Documents             Documents
adb pull /sdcard/Movies                Movies
adb pull /sdcard/Music                 Music
adb pull /sdcard/Android/data          Android_data          # dane aplikacji (częściowo)
adb pull /sdcard/Android/obb           Android_obb           # duże pliki gier
adb pull /sdcard/SMSBackupRestore      SMS                   # jeśli używasz tej aplikacji
adb pull /sdcard/WhatsApp              WhatsApp              # cały WhatsApp!
adb pull /sdcard/Telegram              Telegram
```
### === 2. Listy aplikacji ===
```bash
adb shell pm list packages -3                    > apps_user.txt
adb shell pm list packages -s                    > apps_system.txt
adb shell pm list packages                       > apps_all.txt
adb shell pm list packages -3 | sed 's/package://g' > apps_user_clean.txt
```
### === 3. Informacje o urządzeniu i systemie ===
```bash
adb shell getprop                                        > 01_system_info.txt
adb shell cat /system/build.prop                         > 02_build_prop.txt
adb shell wm size                                        > 03_screen_resolution.txt
adb shell wm density                                     > 04_screen_density.txt
adb shell dumpsys battery                                > 05_battery.txt
adb shell dumpsys meminfo                                > 06_memory.txt
adb shell df -h                                          > 07_storage_usage.txt
adb shell cat /proc/cpuinfo                                 > 08_cpuinfo.txt
adb shell cat /proc/meminfo                              > 09_meminfo.txt
```
### === 4. Struktura pamięci wewnętrznej ===
```bash
adb shell ls -lR /sdcard                                 > drzewo_sdcard.txt
adb shell du -sh /sdcard/* | sort -h                     > rozmiary_katalogow.txt
```
### === 5. Logi i diagnostyka ===
```bash
adb logcat -d                                            > logcat_full.txt
adb logcat -d -b crash                                   > logcat_crash.txt
adb bugreport                                            > bugreport.zip
```
### === 6. Zrzuty ekranu i nagrywanie ===
```bash
adb exec-out screencap -p                                > screen_$(date +%Y%m%d_%H%M%S).png
adb shell screenrecord /sdcard/screenrecord.mp4 --time-limit 180
adb pull /sdcard/screenrecord.mp4                      screenrecord_$(date +%Y%m%d_%H%M%S).mp4
adb shell rm /sdcard/screenrecord.mp4
```
### === 7. Dodatkowe przydatne rzeczy ===
```bash
adb shell dumpsys package                                > pakiety_szczegoly.txt
adb shell dumpsys activity activities                    > aktualne_aktywnosci.txt
adb shell top -n 1                                       > top.txt
adb shell ps -A -o pid,ppid,name,rss,vsz,%cpu,%mem        > procesy_szczegolowe.txt
adb shell dumpsys wifi                                   > wifi_info.txt
adb shell dumpsys telephony.registry                     > sim_i_siec.txt
```




Ten poradnik pokazuje, jak w kilka minut zrobić bardzo szczegółowy backup ważnych danych z telefonu z Androidem – bez rootowania urządzenia. Wszystko robimy z poziomu komputera za pomocą narzędzia `adb` (Android Debug Bridge).

## Wymagania
- Komputer z systemem Windows / Linux / macOS
- Zainstalowane [Android Platform Tools](https://developer.android.com/tools/releases/platform-tools) (zawiera `adb`)
- Telefon z Androidem z włączoną opcją **Debugowanie USB**  
  (Ustawienia → O telefonie → 7× kliknij „Numer kompilacji” → Opcje programistyczne → Debugowanie USB ✓)
- Kabel USB
- (Opcjonalnie) sterowniki USB dla Twojego telefonu (zwykle Windows instaluje je sam)

## Krok po kroku

1. Podłącz telefon do komputera kablem USB
2. Na telefonie zezwól na debugowanie USB (zaznacz „Zawsze zezwalaj z tego komputera”)
3. Otwórz terminal / wiersz poleceń w folderze, w którym masz rozpakowane platform-tools (lub dodaj go do PATH)

### Tworzymy folder na backup i kopiujemy najważniejsze dane użytkownika

```bash
mkdir telefon_backup
adb pull /sdcard/DCIM telefon_backup/DCIM
adb pull /sdcard/Pictures telefon_backup/Pictures
adb pull /sdcard/Download telefon_backup/Download
adb pull /sdcard/Documents telefon_backup/Documents
adb pull /sdcard/Movies telefon_backup/Movies
adb pull /sdcard/Music telefon_backup/Music
adb pull /sdcard/SMSBackupRestore telefon_backup/SMS    # jeśli używasz aplikacji SMS Backup & Restore
```

### Lista zainstalowanych aplikacji (tylko od użytkownika, bez systemowych)

```bash
adb shell pm list packages -3 > telefon_backup/packages.txt
```

### Dodatkowe przydatne informacje diagnostyczne i forensyczne

```bash
# Aktualnie uruchomione procesy
adb shell ps -A > procesy.txt

# Powtórka listy aplikacji użytkownika (dla pewności)
adb shell pm list packages -3 > aplikacje_user.txt

# Struktura katalogów na karcie SD / pamięci wewnętrznej
adb shell ls -l /sdcard > katalogi_sdcard.txt

# Pełne drzewo katalogów (bardzo przydatne!)
adb shell ls -R /sdcard > drzewo_sdcard.txt

# Podstawowe informacje o systemie (model, wersja Androida, build itp.)
adb shell getprop > system_info.txt

# Wszystkie pakiety (systemowe + użytkownika)
adb shell pm list packages > aplikacje_wszystkie.txt

# Szczegółowe informacje o wszystkich pakietów (wersje, ścieżki itp.)
adb shell dumpsys package > pakiety_dane.txt

# Informacje o aktualnie uruchomionych aktywnościach
adb shell dumpsys activity activities > aktywnosci.txt

# Pełny logcat (historia logów systemowych i aplikacji)
adb logcat -d > logcat.txt

# Zrzut aktualnego ekranu
adb exec-out screencap -p > screen.png
```

## Co dokładnie dostajesz w backupie?

| Plik / katalog                  | Co zawiera                                                        |
|---------------------------------|-------------------------------------------------------------------|
| `telefon_backup/DCIM`           | Wszystkie zdjęcia i filmy z aparatu                               |
| `telefon_backup/Pictures`       | Zrzuty ekranu, zdjęcia z WhatsAppa, Messengera itp.               |
| `telefon_backup/Download`       | Wszystkie pobrane pliki                                           |
| `telefon_backup/Documents`      | Dokumenty, PDF-y itp.                                             |
| `telefon_backup/Movies`, `Music` | Filmy i muzyka                                                    |
| `telefon_backup/SMS`            | Backup SMS-ów (jeśli używasz aplikacji SMS Backup & Restore)      |
| `packages.txt`                  | Lista wszystkich aplikacji zainstalowanych przez użytkownika      |
| `system_info.txt`               | Model telefonu, wersja Androida, numer kompilacji, IMEI itp.      |
| `logcat.txt`                    | Pełne logi systemowe – bardzo przydatne przy diagnozie problemów  |
| `screen.png`                    | Aktualny zrzut ekranu                                            |

