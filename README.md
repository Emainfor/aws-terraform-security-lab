\# Laboratorio de seguridad en AWS con Terraform



Creo una instancia en AWS con el puerto de administración (SSH, puerto 22)

abierto a todo internet, y lo limito a una sola dirección.



Probé tres casos desde la misma red:

1\. Abierto a todos: la conexión funciona.

2\. Permitida otra dirección: la conexión falla.

3\. Permitida mi dirección: la conexión funciona.



Aprendizaje: mi dirección de internet cambia, así que en un trabajo real

conviene usar una conexión privada (VPN o AWS Systems Manager).



Aviso: la configuración por defecto abre el puerto 22 a propósito, solo

para el laboratorio. No usar en producción.

