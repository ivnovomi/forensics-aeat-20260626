# 🎣 Anatomía de un phishing que suplantó a la Agencia Tributaria (AEAT)

> Análisis forense de un kit de *phishing* en tiempo real distribuido por SMS, suplantando a Hacienda para robar datos bancarios. Desde el SMS hasta el **takedown**.

![status](https://img.shields.io/badge/site-DOWN%20%E2%9C%85-success)
![type](https://img.shields.io/badge/threat-AitM%20phishing-red)
![purpose](https://img.shields.io/badge/purpose-defensive%20%2F%20educational-blue)
![lang](https://img.shields.io/badge/report-Espa%C3%B1ol-yellow)

---

## TL;DR

Me llegó un SMS de *"Hacienda"* diciendo que tenía una **devolución pendiente**. En lugar de borrarlo, lo analicé. No era un formulario tonto que roba una tarjeta: era un **panel de phishing operado por humanos en tiempo real** capaz de **saltarse la verificación en dos pasos (2FA)** del banco.

Preservé las pruebas de forma forense (**recolección 100 % pasiva**, sin tocar el servidor), documenté los indicadores y reporté al proveedor de *hosting*. **El sitio cayó en horas.** ✅

Este repositorio contiene el informe completo y el desglose técnico, **con fines exclusivamente defensivos y educativos**.

---

## 🧠 Por qué importa: *Adversary-in-the-Middle*

El truco no es robar la contraseña. Es esto:

1. Tú metes tus datos en la web falsa.
2. Un **operador humano** los usa, en directo, en la web **REAL** de tu banco.
3. El banco le pide a *él* un código por SMS → él **te empuja la pantalla** que te pide ese código a *ti*.
4. Tú lo escribes. Él entra. **2FA superado.**

El clásico *"no compartas nunca tu OTP"* se queda corto cuando **la propia web te lo pide en el segundo exacto** en que el banco lo espera.

```
[VÍCTIMA] --SMS--> [WEB FALSA AEAT] --datos--> [OPERADOR] --datos--> [BANCO REAL]
                                                    |                      |
                                                    |<------- pide OTP -----|
                          te reenvía la pantalla <--|
                                que pide el OTP
```

---

## 🔍 Indicadores de Compromiso (IOC)

| Tipo | Valor |
|---|---|
| URL phishing | `https://agenciatributaria.gob-at.eu.cc/es` |
| Dominio base | `gob-at.eu.cc` (subdominio gratuito bajo `eu.cc`) |
| IP | `43.130.56.91` |
| ASN / Hosting | `AS132203` — Tencent Cloud |
| DNS | `a.share-dns.com` / `b.share-dns.net` (DNSPod) |
| Servidor | Caddy |
| C2 (WebSocket) | `wss://<host>/ws?token=<UUID>` |
| C2 (HTTP) | `/JccxhIaBaX/api` · `/JccxhIaBaX/api/input` |
| SHA-256 del JS | `f0832bcd33a0fa2100c17d8366a217b93abe0413172efea51ccb0ce012c00e4b` |
| Slug de campaña | `we125_es_Taxrefund_agenciatributaria` |

> 🧭 **Pivotes para *threat hunting*:** el slug `we125_…` y la ruta `JccxhIaBaX` son cadenas muy distintivas; sirven para encontrar infraestructura y campañas hermanas.

---

## 🧩 Cómo funciona (en una imagen)

| Pantalla | Qué te roba |
|---|---|
| `/home` | DNI / NIE |
| `/address` | Nombre, domicilio, teléfono, email |
| `/card` | **Tarjeta completa + CVV** |
| `/otpValid`, `/bbvaotp` | Código **OTP** del SMS |
| `/OtpPin` | **PIN** de la tarjeta |
| `/bbvalogin` | Usuario y clave de **banca online** |
| *(todas)* | UUID de rastreo en cookie + localStorage + IndexedDB |

Y exfiltra **cada pulsación de teclado**, sin esperar a que pulses *"Enviar"*.

---

## 📂 Contenido del repositorio

```
.
├── README.md
├── informe_forense_aeat_public.pdf      # Informe completo (versión pública, sin PII)
├── informe_forense_aeat_public.pdf.sha256
├── iocs/                                # IOCs en texto plano para bloqueo
├── evidence/                            # WHOIS, DNS, certificado, cabeceras (sanitizado)
└── analysis/                            # Código comentado (fragmentos didácticos)
```

El **informe PDF** incluye: resumen ejecutivo, cronología, análisis técnico, tabla de IOCs, cadena de custodia, contactos de abuso, tipificación legal orientativa, diagrama del ataque, **código comentado paso a paso** y glosario.

---

## 🛡️ 3 cosas que deberías saber (mándaselas a tu familia)

1. **Hacienda y tu banco NUNCA te mandan enlaces por SMS.** Si dudas, escribe la dirección a mano.
2. **Lee el dominio entero, de derecha a izquierda.** Lo que importa está justo antes del `.com`/`.es`, no el `agenciatributaria` de la izquierda.
3. **Si caes, corre:** llama al banco, bloquea la tarjeta y **denuncia**. Tu denuncia protege al siguiente.

---

## ⚖️ Aviso legal y ético

- Todo el análisis se realizó mediante **recolección pasiva**, sin acceso no autorizado, escaneo agresivo ni explotación de sistemas de terceros.
- Este material se publica con **fines defensivos, educativos y de concienciación**. **No** contiene el kit funcional listo para desplegar ni instrucciones de reproducción.
- Los fragmentos de código se incluyen **comentados y parciales** para explicar el funcionamiento del fraude y ayudar a detectarlo.
- Se ha reportado a las autoridades competentes y a los contactos de *abuse* del proveedor.

> Si eres víctima en España: **Policía Nacional – GDT**, **Guardia Civil – Equipo de Delitos Telemáticos** e **INCIBE (017)**.

---

## 🤝 Contribuir

¿Has visto el mismo SMS o una campaña hermana (`weXXX`)? Abre un *issue* con los IOCs. Cuantos más ejemplos circulen, antes lo reconoce la gente.

⭐ Si te ha sido útil, deja una estrella — ayuda a que llegue a más gente.

---

*Mantente a salvo. La mejor defensa no es el miedo, es entender cómo te la quieren colar.* 🙌
