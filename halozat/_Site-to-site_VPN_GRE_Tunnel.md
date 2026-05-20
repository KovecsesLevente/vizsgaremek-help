## GRE Tunnel Áttekintés

Titkosítatlan IP-feletti VPN alagutak létrehozására Cisco környezetben a **GRE (Generic Routing Encapsulation)** protokollt használjuk. Egy virtuális, pont-pont (Point-to-Point) linket hoz létre két távoli router között IP hálózaton keresztül.

### Működési feltételek

| Paraméter | Leírás | Szerepe a konfigurációban |
| --- | --- | --- |
| **Tunnel Source** | A helyi router fizikai IP címe vagy interfésze. | Az alagút kiindulópontja. |
| **Tunnel Destination** | A távoli router nyilvános/fizikai IP címe. | Az alagút végpontja. |
| **Tunnel IP Address** | Egy dedikált, belső alhálózatból származó IP cím. | Ezen keresztül kommunikál a két router egymással (ezen fut a routing). |

---

## Konfigurációs Példa

**Topológia:**

* Router-A fizikai IP: `1.1.1.1` | Tunnel IP: `10.0.0.1/30`
* Router-B fizikai IP: `2.2.2.2` | Tunnel IP: `10.0.0.2/30`

### 1. Router-A Konfiguráció

```text
Router-A(config)# interface Tunnel0
Router-A(config-if)# ip address 10.0.0.1 255.255.255.252
Router-A(config-if)# tunnel source GigabitEthernet0/0
Router-A(config-if)# tunnel destination 2.2.2.2
Router-A(config-if)# tunnel mode gre ip

```

### 2. Router-B Konfiguráció

```text
Router-B(config)# interface Tunnel0
Router-B(config-if)# ip address 10.0.0.2 255.255.255.252
Router-B(config-if)# tunnel source GigabitEthernet0/0
Router-B(config-if)# tunnel destination 1.1.1.1
Router-B(config-if)# tunnel mode gre ip

```

> 💡 **Megjegyzés:** A `tunnel mode gre ip` az alapértelmezett beállítás Cisco IOS-en, de a vizsgákon a transzparencia miatt kötelezően kiírathatják.

### 3. Forgalom irányítása a Tunnelen keresztül (Routing)

Ahhoz, hogy a belső hálózatok használják a tölcsért, statikus vagy dinamikus routing szükséges.

```text
Router-A(config)# ip route 192.168.20.0 255.255.255.0 Tunnel0

```

---

## Vizsga-specifikus kritikus beállítások (MTU / MSS)

A GRE enkapszuláció plusz **24 bájt** overheadet ad az IP csomaghoz (20 bájt IP header + 4 bájt GRE header). Ha a fizikai interfész MTU-ja 1500, a tunnelben keletkező fragmentation (darabolás) elkerülése érdekében kötelező az alábbi korrekció a tunnel interfészek alatt:

```text
Router-A(config-if)# ip mtu 1476
Router-A(config-if)# ip tcp adjust-mss 1436

```

---

## Ellenőrző és Hibakereső Parancsok

* `show interfaces tunnel 0` – Interfész állapotának ellenőrzése.
* **Kívánt státusz:** `Tunnel0 is up, line protocol is up`
* Ellenőrzi a Source, Destination és MTU értékeket.


* `show ip interface brief | include Tunnel` – Gyors ellenőrzés, hogy az IP cím jó-e és az interfész UP állapotú-e.
* `ping 10.0.0.2` – A túloldali tunnel IP cím pingelése a kapcsolat tesztelésére.