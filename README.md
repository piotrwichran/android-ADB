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
adb shell cat /proc/cpuinfo                              > 08_cpuinfo.txt
adb shell cat /proc/meminfo                              > 09_meminfo.txt
```
### === 4. Struktura pamięci wewnętrznej ===
```bash
adb shell ls -R /sdcard                                   > drzewo_sdcard.txt
adb shell du -sh /sdcard/* | sort -h                      > rozmiary_katalogow.txt
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
adb shell ps -A -o pid,ppid,name,rss,vsz,%cpu,%mem       > procesy_szczegolowe.txt
adb shell dumpsys wifi                                   > wifi_info.txt
adb shell ls /system/bin                                 > binarki.txt
adb shell dumpsys telephony.registry                     > sim_i_siec.txt
```
---
```bash
adb shell settings get global private_dns_mode
adb shell settings get global private_dns_specifier
adb shell cat /system/etc/hosts
adb shell ping google.com
```

