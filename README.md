# VPN Site-to-Site IPSec IKEv1 — Policy-Based
**Autor:** Yeury Lopez | **Matrícula:** 20250780 | **Repositorio:** YeuryLopez_20250780_IPSecIKEv1-Policy-Based

---

## Video de demostración

[![Ver en YouTube](https://img.shields.io/badge/YouTube-Ver%20Video-red?logo=youtube)](https://youtu.be/B8s__-cEcVk)

**Enlace directo:** https://youtu.be/B8s__-cEcVk

---

## 1. Objetivo
Configurar una VPN Site-to-Site punto a punto basada en políticas utilizando IPSec IKEv1, permitiendo la comunicación cifrada entre dos LANs remotas. El tráfico interesante se define mediante listas de acceso (ACL) que especifican qué redes deben ser cifradas.

---

## 2. Topología


<img width="940" height="589" alt="image" src="https://github.com/user-attachments/assets/33e4a6d6-8c70-4070-903e-f996efdcc2ee" />


---

## 3. Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Rol |
|---|---|---|---|
| R1 | Fa0/0 | 10.7.80.1/30 | WAN → ISP |
| R1 | Fa1/0 | 192.168.7.1/24 | LAN |
| ISP | Fa0/1 | 10.7.80.2/30 | WAN → R1 |
| ISP | Fa0/0 | 10.7.81.1/30 | WAN → R2 |
| R2 | Fa0/0 | 10.7.81.2/30 | WAN → ISP |
| R2 | Fa1/0 | 192.168.80.1/24 | LAN |
| Linux1 | ens3 | 192.168.7.2/24 | Host LAN R1 |
| Linux2 | ens3 | 192.168.80.2/24 | Host LAN R2 |

---

## 4. Parámetros VPN

| Parámetro | Valor |
|---|---|
| Tipo de VPN | IPSec IKEv1 Policy-Based |
| Fase 1 — Cifrado | AES-128 |
| Fase 1 — Hash | SHA-1 |
| Fase 1 — Autenticación | Pre-Shared Key (PSK) |
| Fase 1 — Grupo DH | Group 2 (1024-bit) |
| Fase 1 — Lifetime | 86400 segundos |
| Fase 2 — Protocolo | ESP |
| Fase 2 — Cifrado | AES-128 |
| Fase 2 — Hash | SHA-1 HMAC |
| Fase 2 — Modo | Tunnel |
| Pre-Shared Key | Yeury0780 |
| Tráfico interesante | ACL extendida |

---

## 5. Configuración

### 5.1 Configuración R1


```
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
crypto isakmp key Yeury0780 address 10.7.81.2
crypto ipsec transform-set TS esp-aes esp-sha-hmac
 mode tunnel
crypto map CMAP 10 ipsec-isakmp
 set peer 10.7.81.2
 set transform-set TS
 match address VPN-TRAFFIC
ip access-list extended VPN-TRAFFIC
 permit ip 192.168.7.0 0.0.0.255 192.168.80.0 0.0.0.255
interface FastEthernet0/0
 crypto map CMAP
```

### 5.2 Configuración R2


```
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
crypto isakmp key Yeury0780 address 10.7.80.1
crypto ipsec transform-set TS esp-aes esp-sha-hmac
 mode tunnel
crypto map CMAP 10 ipsec-isakmp
 set peer 10.7.80.1
 set transform-set TS
 match address VPN-TRAFFIC
ip access-list extended VPN-TRAFFIC
 permit ip 192.168.80.0 0.0.0.255 192.168.7.0 0.0.0.255
interface FastEthernet0/0
 crypto map CMAP
```

---

## 6. Verificación y Funcionamiento

### 6.1 Estado ISAKMP SA

Ejecutar en R1:
```
show crypto isakmp sa
```
> <img width="940" height="181" alt="image" src="https://github.com/user-attachments/assets/f9249367-6d31-4599-89c4-177868844bd6" />


### 6.2 Estado IPSec SA

Ejecutar en R1:
```
show crypto ipsec sa
```
> <img width="940" height="514" alt="image" src="https://github.com/user-attachments/assets/9cfa31b1-2a2d-499f-9cb2-03d51458d5b9" />


### 6.3 Demostración de conectividad

Ejecutar en Linux1:
```
ping -c 4 192.168.80.2
```
> <img width="940" height="228" alt="image" src="https://github.com/user-attachments/assets/3280f037-d368-4802-961c-830d8a0f3a3c" />


---

## 7. Herramientas utilizadas
- **EVE-NG** — Emulador de red
- **Cisco IOS 3725** — c3725-adventerprisek9-mz.124-15.T14
- **Ubuntu Server 22.04** — Hosts de las LANs

