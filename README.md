# VPN-site-to-site-punto-a-punto-con-t-nel-GRE-IPSec-IKEv1

# 🔐 VPN Site-to-Site — Túnel GRE sobre IPsec (IKEv1)

**Juan Francisco Burgos Hiciano – 2023-1981**

📹 Video demostración: https://youtu.be/QIDH8X94Xzw  

---

## 📌 Información General

| Campo | Detalle |
|------|--------|
| Protocolo de túnel | GRE over IPsec |
| Gestión de claves | IKEv1 — Pre-Shared Key |
| Plataforma | Cisco IOS (GNS3 / IOU) |
| Clasificación | Confidencial / Uso interno |
| Tipo de solución | VPN Site-to-Site punto a punto |

---

## 🎯 1. Objetivo

Implementar una **VPN Site-to-Site** utilizando un **túnel GRE encapsulado dentro de IPsec**, permitiendo la comunicación segura entre dos redes LAN remotas (HQ y Branch) a través de un ISP simulado.

La solución garantiza:
- Confidencialidad
- Integridad
- Autenticación del tráfico
- Conectividad transparente entre LANs

---

## 🧱 2. Topología de Red

- **IOU1 (ISP):** Router de tránsito (sin VPN)
- **IOU2 (HQ):** Sede central
- **IOU3 (Branch):** Sucursal remota
- **VPCS:** Hosts finales para pruebas

📌 El túnel GRE permite transportar tráfico entre redes internas sobre una conexión cifrada IPsec.

---

## 🗺️ 3. Esquema de Direccionamiento IP

| Dispositivo | Interfaz | IP | Función |
|------------|----------|----|--------|
| IOU1 | e0/0 | 12.0.0.1/30 | WAN hacia HQ |
| IOU1 | e0/1 | 23.0.0.1/30 | WAN hacia Branch |
| IOU2 | e0/0 | 12.0.0.2/30 | WAN |
| IOU2 | e0/1 | 192.168.1.1/24 | LAN HQ |
| IOU2 | Tunnel0 | 10.0.0.1/30 | GRE Tunnel |
| IOU3 | e0/1 | 23.0.0.2/30 | WAN |
| IOU3 | e0/2 | 192.168.2.1/24 | LAN Branch |
| IOU3 | Tunnel0 | 10.0.0.2/30 | GRE Tunnel |

---

## ⚙️ 4. Arquitectura GRE over IPsec

La solución combina dos tecnologías:

### 🔹 GRE (Generic Routing Encapsulation)
- Crea un túnel lógico punto a punto
- Permite transportar tráfico de capa 3 (rutas dinámicas o estáticas)

### 🔹 IPsec
- Protege el tráfico GRE mediante cifrado
- Usa:
  - AES-256 (cifrado)
  - SHA-256 (integridad)

📌 Flujo:
```
LAN → GRE Encapsulation → IPsec Encryption → Red Pública → IPsec Decryption → GRE Decapsulation → LAN
```

---

## 🔐 5. Parámetros de Seguridad (IKEv1)

### Fase 1 — ISAKMP
- Cifrado: AES 256
- Hash: SHA-256
- Autenticación: Pre-Shared Key
- DH Group: 14
- Lifetime: 86400

### Fase 2 — IPsec
- ESP AES-256 + SHA-256-HMAC
- Modo: Transport (para proteger GRE)

---

## 💻 6. Configuración

### 🔹 IOU1 (ISP)

```bash
hostname IOU1

interface Ethernet0/0
 ip address 12.0.0.1 255.255.255.252
 no shutdown

interface Ethernet0/1
 ip address 23.0.0.1 255.255.255.252
 no shutdown
```

---

### 🔹 IOU2 (HQ)

```bash
hostname IOU2

interface Ethernet0/0
 ip address 12.0.0.2 255.255.255.252
 no shutdown

interface Ethernet0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown

crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400

crypto isakmp key CLAVE_VPN address 23.0.0.2

crypto ipsec transform-set TS_GRE esp-aes 256 esp-sha256-hmac
 mode transport

ip access-list extended ACL_GRE_HQ
 permit gre host 12.0.0.2 host 23.0.0.2

crypto map CMAP 10 ipsec-isakmp
 set peer 23.0.0.2
 set transform-set TS_GRE
 match address ACL_GRE_HQ

interface Ethernet0/0
 crypto map CMAP

interface Tunnel0
 ip address 10.0.0.1 255.255.255.252
 tunnel source Ethernet0/0
 tunnel destination 23.0.0.2

ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

---

### 🔹 IOU3 (Branch)

```bash
hostname IOU3

interface Ethernet0/1
 ip address 23.0.0.2 255.255.255.252
 no shutdown

interface Ethernet0/2
 ip address 192.168.2.1 255.255.255.0
 no shutdown

crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400

crypto isakmp key CLAVE_VPN address 12.0.0.2

crypto ipsec transform-set TS_GRE esp-aes 256 esp-sha256-hmac
 mode transport

ip access-list extended ACL_GRE_BRANCH
 permit gre host 23.0.0.2 host 12.0.0.2

crypto map CMAP 10 ipsec-isakmp
 set peer 12.0.0.2
 set transform-set TS_GRE
 match address ACL_GRE_BRANCH

interface Ethernet0/1
 crypto map CMAP

interface Tunnel0
 ip address 10.0.0.2 255.255.255.252
 tunnel source Ethernet0/1
 tunnel destination 12.0.0.2

ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

---

## 🔍 7. Verificación

```bash
show crypto isakmp sa
show crypto ipsec sa
show interface tunnel0
```

### Pruebas

```bash
ping 10.0.0.2 source 10.0.0.1
ping 192.168.2.10 source 192.168.1.10
```

---

## 🧠 8. Conclusión

- GRE permite flexibilidad en el transporte de tráfico entre sitios
- IPsec garantiza seguridad en la capa de transporte
- IKEv1 establece negociación segura de claves
- La solución es escalable y compatible con Cisco IOS

---
