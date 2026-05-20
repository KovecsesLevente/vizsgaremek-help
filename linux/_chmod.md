**chmod** magyarázata Linux/Ubuntu szerveren (érthetően, lépésről lépésre)

### Mire való a chmod?
A `chmod` (change mode) paranccsal állítod be, hogy ki mit csinálhat egy fájllal vagy könyvtárral:
- **olvasás** (read)
- **írás** (write)
- **futtatás** (execute)

### 1. Jogosultságok három csoportja

Minden fájlnak/könyvtárnak 3 jogosultsági csoportja van:

| Csoport     | Jelentése                          | Rövidítés |
|-------------|------------------------------------|---------|
| **owner**   | a fájl tulajdonosa (user)          | **u**   |
| **group**   | a fájlhoz tartozó csoport tagjai   | **g**   |
| **others**  | mindenki más (a világ)             | **o**   |

### 2. Numerikus (octal) mód – ez a leggyakoribb (pl. 777)

Ez a leggyorsabb és legelterjedtebb módszer szerveren.

Minden csoportnak egy számot adunk **0-7** között:

| Érték | Bináris | Jogosultság | Rövidítés |
|-------|---------|-------------|---------|
| **7** | 111     | rwx         | read + write + execute |
| **6** | 110     | rw-         | read + write           |
| **5** | 101     | r-x         | read + execute         |
| **4** | 100     | r--         | read only              |
| **3** | 011     | -wx         | write + execute        |
| **2** | 010     | -w-         | write only             |
| **1** | 001     | --x         | execute only           |
| **0** | 000     | ---         | semmi                  |

#### Hogyan jön ki a **777**?

```
777 = 7  7  7
      │  │  │
   owner group others
```

- **7** = 4 (read) + 2 (write) + 1 (execute) = **rwx**
- Tehát: **rwxrwxrwx** → mindenki mindent csinálhat (teljes hozzáférés)

**Példák gyakori értékekre:**

| Parancs          | Jelentés                                      | Szimbolikus megfelelő |
|------------------|-----------------------------------------------|-----------------------|
| `chmod 777 file` | Mindenki mindent csinálhat                    | rwxrwxrwx             |
| `chmod 755 file` | Tulajdonos: mindent, mások: csak olvas+futtat | rwxr-xr-x             |
| `chmod 644 file` | Tulajdonos: rw, mások: csak olvas             | rw-r--r--             |
| `chmod 600 file` | Csak a tulajdonos olvashatja és írhatja       | rw-------             |
| `chmod 700 file` | Csak a tulajdonos mindent csinálhat           | rwx------             |

### 3. Szimbolikus mód (alternatíva)

Ha csak egy dolgot akarsz megváltoztatni, ezt szoktad használni:

```bash
chmod u+x file          # ownernek adsz execute jogot
chmod g+w file          # groupnak adsz write jogot
chmod o-r file          # others-tól elveszed a read jogot
chmod a+x file          # mindenki kap execute jogot (a = all)
chmod u=rwx,g=rx,o= file # pontos beállítás
```

### 4. Gyakorlati példák Ubuntu szerveren

```bash
# Normál fájl (weboldalnál ajánlott)
chmod 644 index.html

# PHP fájl (webserver fusson benne)
chmod 644 index.php

# Script, amit futtatni akarsz
chmod 755 script.sh
chmod +x script.sh        # ugyanaz, rövidebben

# Biztonságos konfigurációs fájl (pl. .env)
chmod 600 .env

# Könyvtár (általában 755)
chmod 755 /var/www/html
```

### Extra tippek

- **Rekurzív** (könyvtárra és minden benne lévőre):
  ```bash
  chmod -R 755 /path/to/dir
  ```

- **chmod 777** használata:  
  **Csak teszteléshez** ajánlott! Éles szerveren biztonsági kockázat, mert bárki írhatja/törölheti a fájlt.

- Ellenőrzés:
  ```bash
  ls -l          # mutatja a jogokat (pl. -rwxr-xr-x)
  stat filename  # részletesebb infó
  ```

Szeretnél egy konkrét példát valamilyen helyzetre (pl. weboldal, script, adatbázis backup stb.)?