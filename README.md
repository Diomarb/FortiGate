<div align="center">

<img src="https://img.shields.io/badge/ITLA-Instituto%20Tecnol%C3%B3gico%20de%20las%20Am%C3%A9ricas-0D1F4E?style=for-the-badge" />

# 🛡️ Laboratorio FortiGate — Seguridad Perimetral y Filtrado de Contenido

<img src="https://img.shields.io/badge/Estudiante-Eunice-1C3D8F?style=flat-square" />
<img src="https://img.shields.io/badge/Matr%C3%ADcula-2024--1185-1C3D8F?style=flat-square" />
<img src="https://img.shields.io/badge/Plataforma-GNS3-1C3D8F?style=flat-square" />
<img src="https://img.shields.io/badge/Firewall-FortiGate--VM-0D1F4E?style=flat-square" />
<img src="https://img.shields.io/badge/Link video:-1C3D8F?style=flat-square "/> 

</div>

---

## 📋 Descripción

Topología de seguridad perimetral construida en **GNS3** con **FortiGate**, configurada al 100% por GUI, que implementa:

- Segmentación de red: LAN de usuarios (`/25`) y LAN de servidores (`/28`)
- Acceso a Internet con NAT
- DHCP en la LAN de usuarios
- Restricción de tráfico entre segmentos a **solo HTTP**
- Bloqueo de redes sociales (Facebook, Instagram)
- Bloqueo de llamadas de voz/video de WhatsApp
- Bloqueo de `itla.edu.do` y todos sus subdominios
- Detección y bloqueo de escáneres de red (IPS)
- **WAF** aplicado al servidor web

---

## 🗺️ Topología

```
                    ┌────────────┐
                    │    NAT1    │  (salida real a Internet)
                    └─────┬──────┘
                          │ port1 (WAN · DHCP)
                    ┌─────┴──────┐
                    │  FortiGate │
                    └──┬──────┬──┘
              port2 ───┘      └─── port3
         (LAN-CLIENTE)         (LAN-SERVER)
        10.11.85.0/25        10.11.85.128/28
              │                     │
          ┌───┴───┐             ┌───┴───┐
          │ Switch │             │ Switch │
          └─┬────┬─┘             └───┬────┘
      Cliente   PC1              Servidor Web
     (webterm) (VPCS)           (Kali + Apache)
                                 10.11.85.130/28
```
<img width="268" height="277" alt="Captura de pantalla 2026-07-04 004211" src="https://github.com/user-attachments/assets/7a6e01f1-06e7-467f-88c8-9c53c515c7e8" />

---

## 🔢 Direccionamiento IP

| Segmento / Interfaz     | Red o IP                     | Rol                              |
|--------------------------|-------------------------------|-----------------------------------|
| Port1 (WAN)              | DHCP (asignado por NAT1)      | Salida a Internet                 |
| Port2 (LAN-CLIENTE)       | `10.11.85.1/25`                | Gateway + DHCP server             |
| Port3 (LAN-SERVER)        | `10.11.85.129/28`              | Gateway LAN de servidores         |
| Rango DHCP usuarios      | `10.11.85.2` – `10.11.85.126`  | Clientes (webterm, PC1)           |
| Servidor Web (Kali)       | `10.11.85.130/28`              | IP estática, Apache puerto 80     |

---

## ⚙️ Configuración — resumen

### Interfaces (`Network > Interfaces`)
| Interfaz | Modo | Detalle |
|---|---|---|
| Port1 | DHCP | Role: WAN |
| Port2 | Manual | `10.11.85.1/255.255.255.128` + DHCP Server |
| Port3 | Manual | `10.11.85.129/255.255.255.240` |

### Perfiles de seguridad (`Security Profiles`)
| Perfil | Función |
|---|---|
| `Block-Social-ITLA` | Web Filter — Static URL Filter para redes sociales e `itla.edu.do` |
| `Block-WhatsApp-Call` | Application Control — bloqueo de firmas de voz/video de WhatsApp + QUIC |
| `Block-Scanners` | IPS — firmas de reconnaissance / escaneo de puertos |
| `WAF-WebServer` | WAF — XSS, SQL Injection, HTTP Protocol Constraints |

### Políticas de Firewall (`Policy & Objects > Firewall Policy`)
| # | Nombre | In → Out | Servicio | NAT | Perfiles |
|---|---|---|---|---|---|
| 1 | `Block-QUIC` | port2 → port1 | UDP/443 | — (Deny) | — |
| 2 | `Usuarios-a-Internet` | port2 → port1 | ALL | ✔ SNAT | Web Filter, App Control, IPS |
| 3 | `Usuarios-a-Servidor-HTTP` | port2 → port3 | HTTP | Disable | IPS, WAF |

> El *implicit deny* del FortiGate bloquea automáticamente cualquier tráfico no contemplado entre segmentos — no se necesita una política de deny explícita.

### Servidor (Kali)
```bash
sudo nano /etc/network/interfaces
# auto eth0
# iface eth0 inet static
#     address 10.11.85.130
#     netmask 255.255.255.240
#     gateway 10.11.85.129
#     dns-nameservers 8.8.8.8
sudo systemctl restart networking
sudo apt update && sudo apt install apache2 -y
sudo systemctl enable apache2 --now
```
Código
---
```
sudo nano /var/www/html/index.html
<!DOCTYPE html>
<html>
<body style="text-align:center; font-family: Arial; background-color: #f0f0f0;">
    <h1 style="color: #d35400;">Práctica de Seguridad en Redes</h1>
    <h2 style="color: #2c3e50;">Estudiante: [TU NOMBRE AQUÍ]</h2>
    <br>
    <div style="font-size: 100px;">🌻</div>
    <p>Servidor Web Seguro (WAF Activo)</p>
</body>
</html>
```
## ✅ Pruebas realizadas

- [x] `ping 8.8.8.8` y navegación a Internet desde la LAN de usuarios
<img width="224" height="109" alt="Captura de pantalla 2026-07-08 193251" src="https://github.com/user-attachments/assets/4d45bfb0-04c3-4bfc-a0ae-9506a008cb89" />




- [x] `http://10.11.85.130` carga la página de Apache — tráfico HTTP permitido entre segmentos
      
<img width="442" height="367" alt="Captura de pantalla 2026-07-07 223633" src="https://github.com/user-attachments/assets/dcc26985-79ab-4d71-921f-62aba131b7cd" />

- [x] `ping` / SSH / Nmap hacia el servidor desde la LAN de usuarios — bloqueado (implicit deny)
      
<img width="160" height="63" alt="image" src="https://github.com/user-attachments/assets/b74518a5-7d20-4826-8d05-64a25f451769" />

- [x] Acceso a `facebook.com` / `instagram.com` — conexión rechazada (Web Filter + SSL Inspection)

<img width="533" height="347" alt="Captura de pantalla 2026-07-07 225416" src="https://github.com/user-attachments/assets/704c2dd9-bd5a-4ef0-b702-1785e3ec9247" />

- [x] Acceso a `itla.edu.do` — página de bloqueo de FortiGuard
      
<img width="431" height="338" alt="Captura de pantalla 2026-07-07 230900" src="https://github.com/user-attachments/assets/f70a8062-3994-4db3-a159-631b1d70b2ad" />

- [x] Logs verificados en `Log & Report > Web Filter` / `Forward Traffic`
      
<img width="422" height="59" alt="Captura de pantalla 2026-07-08 010808" src="https://github.com/user-attachments/assets/2a324a9d-a294-4f27-89b8-f1ca702d3410" />

<img width="281" height="107" alt="Captura de pantalla 2026-07-08 004026" src="https://github.com/user-attachments/assets/3d122a0e-861c-4f53-bc6d-0c22d5844f6e" />

<img width="488" height="290" alt="Captura de pantalla 2026-07-07 225723" src="https://github.com/user-attachments/assets/cd0d1126-f830-4e0c-92ba-01992a096436" />

<img width="452" height="326" alt="image" src="https://github.com/user-attachments/assets/4bff9fd8-f18b-4406-abd1-a0fa139d7374" />

---

## 🧩 Notas técnicas relevantes

- El **SSL/SSH Inspection** (`certificate-inspection`, modo Proxy-based) es obligatorio para que el Web Filter tenga visibilidad del dominio (SNI) en tráfico HTTPS.
- Facebook e Instagram pueden evadir la inspección TCP normal mediante **QUIC (HTTP/3 sobre UDP)** — se bloqueó la firma `QUIC` en Application Control para forzar el fallback a TLS estándar.
- El **Static URL Filter** se usó en lugar del filtrado por categoría de FortiGuard para no depender de una licencia activa en el entorno de laboratorio.
---

<div align="center">

**ITLA** — Instituto Tecnológico de las Américas

</div>
