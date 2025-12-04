Oto gotowy, czytelny tutorial po polsku w formacie Markdown, idealny do wrzucenia na GitHub (np. jako `README.md` lub `BACKUP_ANDROID.md`):

```markdown
# Pełny backup telefonu z Androidem za pomocą ADB (bez roota)

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

