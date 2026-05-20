**Ubuntu DNS Szerver (BIND9) Telepítési és Konfigurációs Guideline**

Ez a guide a legelterjedtebb és vizsgákon is leggyakrabban használt **BIND9** nevű DNS szerver telepítését és alapvető konfigurációját mutatja be Ubuntu 22.04/24.04 LTS rendszeren.

### 1. Rendszer előkészítése

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install bind9 bind9-utils bind9-doc -y
```

**Vizsgatipp:** Mindig frissítsd a csomaglistát telepítés előtt! (`apt update`)

### 2. Alapvető konfiguráció

#### A fő konfigurációs fájl: `/etc/bind/named.conf.options`

```bash
sudo nano /etc/bind/named.conf.options
```

Példa tartalom (forwarder + ACL ajánlott):

```conf
options {
    directory "/var/cache/bind";

    recursion yes;                    # Engedélyezzük a rekurzív kéréseket
    allow-recursion { 192.168.1.0/24; localhost; };  # Csak a saját hálózatból

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation auto;

    listen-on-v6 { any; };
    listen-on { any; };
};
```

**Vizsgatipp:** `listen-on` és `allow-recursion` beállítása nagyon gyakori vizsgakérdés!

### 3. Zónák definiálása (`named.conf.local`)

```bash
sudo nano /etc/bind/named.conf.local
```

Példa forward és reverse zóna:

```conf
zone "pelda.hu" {
    type master;
    file "/etc/bind/db.pelda.hu";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```

### 4. Forward zóna fájl létrehozása

```bash
sudo cp /etc/bind/db.local /etc/bind/db.pelda.hu
sudo nano /etc/bind/db.pelda.hu
```

Példa tartalom:

```zone
$TTL    604800
@       IN      SOA     ns1.pelda.hu. admin.pelda.hu. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      ns1.pelda.hu.
@       IN      A       192.168.1.10
ns1     IN      A       192.168.1.10
www     IN      A       192.168.1.20
mail    IN      A       192.168.1.30
```

**Vizsgatipp:** A **Serial** számot mindig növeld (pl. YYYYMMDDxx formátum) módosítás után!

### 5. Reverse (PTR) zóna fájl

```bash
sudo cp /etc/bind/db.127 /etc/bind/db.192
sudo nano /etc/bind/db.192
```

Példa:

```zone
$TTL    604800
@       IN      SOA     ns1.pelda.hu. admin.pelda.hu. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      ns1.pelda.hu.
10      IN      PTR     ns1.pelda.hu.
20      IN      PTR     www.pelda.hu.
```

### 6. Szintaxis ellenőrzés (KÖTELEZŐ VIZSGA LÉPÉS!)

```bash
# Fő konfiguráció ellenőrzése
sudo named-checkconf /etc/bind/named.conf
sudo named-checkconf /etc/bind/named.conf.local
sudo named-checkconf /etc/bind/named.conf.options

# Zóna fájlok ellenőrzése
sudo named-checkzone pelda.hu /etc/bind/db.pelda.hu
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192
```

**Vizsgatipp:** Ha `named-checkzone` hibát jelez, az nagyon sok pontot ér a vizsgán, ha kijavítod!

### 7. Szolgáltatás újraindítása és engedélyezése

```bash
sudo systemctl restart bind9
sudo systemctl enable bind9
sudo systemctl status bind9
```

Logok ellenőrzése:
```bash
sudo journalctl -u bind9 -xe
sudo tail -f /var/log/syslog | grep named
```

### 8. Tesztelés

```bash
# Lokális tesztek
dig @localhost www.pelda.hu
dig @localhost -x 192.168.1.20

# Külső gépről
nslookup www.pelda.hu <DNS-szerver-IP>
dig @<DNS-szerver-IP> pelda.hu
```

### Extra Vizsgatippek és Gyakori Parancsok

| Feladat                        | Parancs                                              |
|--------------------------------|------------------------------------------------------|
| Konfiguráció teljes ellenőrzés | `named-checkconf`                                    |
| Zóna ellenőrzés                | `named-checkzone zóna.nev /útvonal/fájl`             |
| Zóna újratöltése manuálisan    | `rndc reload` vagy `rndc reload pelda.hu`            |
| Cache ürítése                  | `rndc flush`                                         |
| Statisztikák                   | `rndc stats`                                         |
| BIND9 verzió                   | `named -v`                                           |
| Port ellenőrzés                | `ss -ltnp \| grep :53`                               |
| Teljes újraindítás             | `systemctl restart bind9`                            |

**Biztonsági tippek (vizsgán plusz pont):**
- Ne használd a `any;` beállítást éles környezetben recursion-hez!
- Használj ACL-t (Access Control List) a `named.conf.options`-ben.
- Engedélyezd a DNSSEC-et, ahol lehetséges.
- Firewall: csak 53/tcp és 53/udp portot nyiss meg a szükséges gépek felé.

```bash
sudo ufw allow from 192.168.1.0/24 to any port 53 proto udp
sudo ufw allow from 192.168.1.0/24 to any port 53 proto tcp
```

**Gyakorlati tanács:** Mindig másold le az eredeti fájlokat (`cp fájl fájl.bak`) konfiguráció módosítás előtt!

Ha master-slave (secondary) DNS-t is kell konfigurálni, vagy views-t (split DNS), jelezd, és azt is bővítem a guide-hoz.