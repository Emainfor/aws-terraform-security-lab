# 🔐 Laboratorio AWS + Terraform: exposición de SSH

> Laboratorio personal de práctica en seguridad cloud. **No es trabajo de cliente ni un entorno productivo.**

![Terraform](https://img.shields.io/badge/IaC-Terraform-7B42BC)
![AWS](https://img.shields.io/badge/Cloud-AWS-FF9900)
![Estado](https://img.shields.io/badge/Uso-solo%20laboratorio-red)

## 🎯 Objetivo
Demostrar con evidencia por qué **no se debe dejar el puerto de administración (SSH, 22/tcp) abierto a todo internet**, y cómo restringirlo con un security group definido como código.

## ⚠️ Riesgo que ilustra
Un SSH expuesto a `0.0.0.0/0` es descubierto por escaneos automáticos y recibe intentos de fuerza bruta. Es un hallazgo típico en cualquier assessment de infraestructura.

| Marco | Referencia |
|---|---|
| NIST CSF 2.0 | PR.AA (acceso) y PR.IR (protección de la infraestructura) |
| Buena práctica | Mínimo privilegio y reducción de superficie de ataque |

## 🧱 Arquitectura

```
   Internet ──► Security Group "lab-ssh" ──► EC2 t3.micro (Ubuntu)
                 regla: 22/tcp desde ssh_cidr        │
                                                     └─ IMDSv2 obligatorio
```

Región: São Paulo (`sa-east-1`). VPC por defecto (simplificación de laboratorio).

## 🧪 Pruebas realizadas
Misma red de origen, cambiando solo la regla de entrada:

| # | Regla de entrada (22/tcp) | Esperado | Resultado |
|---|---|---|---|
| 1 | Abierta a todos (`0.0.0.0/0`) | Puerto accesible | Accesible [verificar cómo se probó] |
| 2 | Otra dirección distinta | Bloqueado | Falla (sin respuesta) |
| 3 | Mi dirección (`/32`) | Puerto accesible | Accesible |

📎 Evidencia: carpeta `evidencia/` *(pendiente de agregar, sin IPs reales)*.

## 🛡️ Controles implementados

| Control | Qué protege | Dónde se ve |
|---|---|---|
| Regla SSH limitada por `ssh_cidr` | Reduce quién puede llegar al puerto | `aws_security_group.lab` |
| **IMDSv2 obligatorio** (`http_tokens = "required"`) | Dificulta el robo de credenciales vía SSRF | `metadata_options` |
| Infraestructura como código | Cambios revisables y repetibles | `main.tf` |

## 📚 Aprendizajes
- Mi IP pública cambia: una regla `/32` es frágil en la práctica.
- En un entorno real conviene **no abrir el 22**: usar **AWS Systems Manager (Session Manager)** o VPN.
- Una alerta de presupuesto de AWS me permitió rastrear el gasto del laboratorio (≈ USD 0,05) hasta el servicio y la región.

## 🚧 Limitaciones conocidas (honestas)
- Salida (egress) abierta a todo.
- Sin cifrado explícito del volumen ni monitoreo detallado.
- Usa la VPC por defecto con IP pública.
- Sin pipeline de escaneo automático todavía.

## 🗺️ Próximos pasos
- [ ] Agregar par de claves SSH para autenticar de verdad
- [ ] Que el código bloquee `0.0.0.0/0` salvo pedido explícito
- [ ] Reemplazar SSH por SSM Session Manager
- [ ] Escaneo con Checkov/Trivy y GitHub Actions

## ▶️ Cómo repetirlo
```bash
terraform init
terraform plan  -var 'ssh_cidr=TU.IP.AQUI/32'
terraform apply -var 'ssh_cidr=TU.IP.AQUI/32'
# ...practicar...
terraform destroy   # siempre, para no generar costos
```
Requisitos: Terraform, AWS CLI y un perfil con permisos mínimos.

## 💰 Costo
Unas 2 horas de `t3.micro` + disco + IPv4 pública ≈ **USD 0,05** (verificado en AWS Cost Explorer).

> ⚠️ Solo para laboratorio. No usar en producción.
