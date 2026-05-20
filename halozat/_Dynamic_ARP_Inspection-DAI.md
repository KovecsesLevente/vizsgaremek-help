## Dynamic ARP Inspection (DAI) Áttekintés

A Dynamic ARP Inspection (DAI) egy Layer 2 biztonsági funkció, amely megvédi a hálózatot az **ARP spoofing / ARP poisoning** (Man-in-the-Middle) támadásoktól. A DAI ellenőrzi az összes hálózaton áthaladó ARP csomag érvényességét, és eldobja a hamisított MAC-IP párosítást tartalmazó kéréseket/válaszokat.

### Működési mechanizmus és függőség

| Biztonsági elem | Leírás | Szerepe a DAI működésében |
| --- | --- | --- |
| **DHCP Snooping** | Kötelező előfeltétel. | A DAI a DHCP Snooping által felépített **Binding Database** (MAC-IP tábla) alapján ellenőrzi az ARP csomagokat. |
| **ARP ACL** | Manuális konfigurációs tábla. | Statikus IP-címmel rendelkező eszközök (szerverek, nyomtatók) MAC-IP párosításának kézi engedélyezésére szolgál. |

---

## Port Típusok

| Port Típus | Elhelyezkedés | DAI Működés |
| --- | --- | --- |
| **Trusted (Megbízható)** | Uplink portok, más switchek/routerek felé. | **Nincs ellenőrzés.** Az innen érkező ARP csomagokat a switch vizsgálat nélkül átengedi. |
| **Untrusted (Nem megbízható)** | Végponti portok (kliensek, PC-k). | **Szigorú ellenőrzés.** Minden bejövő ARP csomag MAC-IP párosítását veti össze a DHCP Snooping táblával / ARP ACL-lel. |

---

## Konfigurációs Példa

**Topológia:**

* A router/uplink a `GigabitEthernet0/1` porton van (Trusted).
* A kliensek a `FastEthernet0/1 - 24` portokon vannak (Untrusted).
* A védendő hálózat a **VLAN 10**.

### 1. Globális engedélyezés a kívánt VLAN-ra

```text
Switch(config)# ip arp inspection vlan 10

```

### 2. Megbízható (Trusted) interfész kijelölése

Alapértelmezés szerint minden port `untrusted`. Az uplink portot kötelező átállítani, különben a hálózat működése leáll.

```text
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# ip arp inspection trust

```

### 3. Statikus IP-s eszközök kezelése (ARP ACL)

Ha a VLAN-ban van fix IP-s eszköz (pl. Szerver: `10.0.10.50` a `0011.2233.4455` MAC címmel), akkor azt külön engedélyezni kell, mivel nem szerepel a DHCP táblában.

```text
Switch(config)# arp access-list STATIC-DEVICES
Switch(config-arp-acl)# permit ip host 10.0.10.50 mac host 0011.2233.4455
Switch(config-arp-acl)# exit
Switch(config)# ip arp inspection filter STATIC-DEVICES vlan 10

```

---

## Extra Ellenőrzések (Vizsga-specifikus validációk)

A DAI alapértelmezetten csak az ARP csomag törzsében lévő IP- és MAC-címeket veti össze a DHCP táblával. Szigorúbb vizsgafeladatok megkövetelhetik a fejléc és a törzs egyezésének ellenőrzését is:

```text
Switch(config)# ip arp inspection validate src-mac dst-mac ip

```

* `src-mac`: Ellenőrzi az Ethernet fejlécben lévő forrás MAC címet az ARP törzsben lévő forrás MAC címmel.
* `dst-mac`: ARP válaszoknál ellenőrzi a cél MAC címet az Ethernet fejlécben lévővel.
* `ip`: Ellenőrzi az ARP törzsben lévő IP címet (pl. eldobja a 0.0.0.0-s vagy multicast IP-t tartalmazó érvénytelen ARP-ket).

---

## Ellenőrző és Hibakereső Parancsok

* `show ip arp inspection` – Általános státusz, engedélyezett VLAN-ok és a blokkolt/átengedett ARP csomagok számlálói.
* `show ip arp inspection interfaces` – Kilistázza az interfészek DAI bizalmi státuszát (`Trust state`) és az érvényben lévő sebességkorlátokat.
* `show ip arp inspection vlan 10` – Specifikusan a megadott VLAN DAI konfigurációját és statisztikáit mutatja meg.