## DHCP Snooping Áttekintés

A DHCP Snooping egy Layer 2 biztonsági funkció, amely védelmet nyújt a hálózatban megjelenő illetéktelen (Rogue) DHCP szerverek és a rosszindulatú DHCP támadások (pl. DHCP starvation) ellen. A switch portjait két kategóriára osztja: megbízható (Trusted) és nem megbízható (Untrusted) portok.

### Port Típusok és Működés

| Port Típus | Jellemző csatlakozás | Engedélyezett DHCP üzenetek |
| --- | --- | --- |
| **Trusted (Megbízható)** | DHCP Szerverek, Uplink portok (más switchek felé). | Minden DHCP üzenet (Kérések és Válaszok is: Discovery, Request, Offer, ACK). |
| **Untrusted (Nem megbízható)** | Végponti eszközök (PC-k, IP telefonok). | **Csak DHCP kérések** (Discovery, Request). Ha erről a portról DHCP Offer vagy ACK érkezik, a switch lezárja a portot. |

---

## DHCP Snooping Binding Database

A switch egy dinamikus táblázatot épít fel az `untrusted` portokon átmenő, sikeres DHCP folyamatokból.
A tábla tartalma: **MAC cím, IP cím, Lease time, VLAN ID, Port száma**.
*Ezt a táblázatot használja később a DAI (Dynamic ARP Inspection) és az IP Source Guard is.*

---

## Konfigurációs Példa

**Topológia:**

* A DHCP szerver a `GigabitEthernet0/1` porton van (Trusted).
* A felhasználók a `FastEthernet0/1 - 24` portokon vannak (Untrusted).
* A védendő hálózat a **VLAN 10**.

### 1. Globális engedélyezés és VLAN hozzárendelés

```text
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10

```

### 2. Megbízható (Trusted) port beállítása

Alapértelmezés szerint minden port `untrusted`. A szerver felé néző portot manuálisan kell átállítani.

```text
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# ip dhcp snooping trust

```

### 3. Sebességkorlátozás (Rate Limit) az Untrusted portokon

A DHCP starvation (szerver kimerítés) támadások ellen kötelező korlátozni a másodpercenként fogadható DHCP csomagok számát a kliens portokon.

```text
Switch(config)# interface range FastEthernet0/1 - 24
Switch(config-if-range)# ip dhcp snooping limit rate 10

```

---

## Kritikus vizsga-specifikus opció: Option 82

Cisco switcheken a DHCP Snooping bekapcsolásakor a switch automatikusan beszúrja az **Option 82** (Relay Agent Information) mezőt a DHCP kérésekbe.

* **A probléma:** Ha a DHCP szerver közvetlenül ugyanabban a VLAN-ban van (nem relay-en keresztül), és nem ismeri fel az Option 82-t, a switch eldobhatja a válaszként érkező csomagokat.
* **Megoldás (ha a vizsgafeladat kéri az Option 82 kikapcsolását):**

```text
Switch(config)# no ip dhcp snooping information option

```

---

## Ellenőrző és Hibakereső Parancsok

* `show ip dhcp snooping` – Megmutatja, hogy a funkció aktív-e, mely VLAN-okra érvényes, és listázza a Trusted interfészeket.
* `show ip dhcp snooping binding` – Megjeleníti az adatbázist (kliens MAC, kapott IP, VLAN, interfész).
* `clear ip dhcp snooping binding *` – Törli a dinamikusan felépített binding táblát.