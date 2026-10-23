---
name: deteccion-persistencia
description: Guía la búsqueda forense de mecanismos de persistencia (tareas programadas, servicios, claves de registro/autorun, cuentas nuevas, claves SSH, cron jobs) en un activo con sospecha de compromiso. Úsala cuando una alerta escala a P1/P2 o cuando analisis-log-incidente confirma evidencia directa de compromiso.
---

# Detección de Persistencia

## Objetivo
Verificar sistemáticamente si un atacante dejó un mecanismo para mantener
acceso tras la contención inicial, en vez de asumir que limpiar el vector
original es suficiente.

## Cuándo se activa
- Incidente escalado a P1/P2 con evidencia directa (no hipótesis) de
  compromiso.
- Antes de cerrar un incidente como "contenido" — es un paso de verificación
  obligatorio, no opcional.

## Checklist por plataforma

### Linux
- Cron jobs de usuario y de sistema (`crontab -l`, `/etc/cron.*`).
- Servicios systemd no reconocidos o modificados recientemente.
- Claves SSH autorizadas añadidas (`~/.ssh/authorized_keys`) y nuevas
  configuraciones en `/etc/ssh/sshd_config`.
- Cuentas de usuario nuevas o con UID 0 inesperado.
- Binarios con SUID/SGID añadidos recientemente.
- Entradas en `/etc/rc.local`, `/etc/profile.d/`, o shell rc files
  modificados.

### Windows
- Tareas programadas nuevas o modificadas (Task Scheduler).
- Servicios nuevos o binarios de servicio reemplazados.
- Claves Run/RunOnce en el registro (HKLM/HKCU).
- Cuentas de administrador local nuevas o añadidas a grupos privilegiados.
- WMI event subscriptions (técnica de persistencia avanzada, fácil de pasar
  por alto).

## Flujo de trabajo
1. Define la ventana temporal del compromiso (desde el primer IOC confirmado).
2. Revisa cada ítem del checklist correspondiente a la plataforma, anotando
   evidencia directa (no supongas ausencia sin verificar).
3. Si encuentras un mecanismo de persistencia, documenta: dónde, cuándo se
   creó/modificó, y si sigue activo.
4. Correlaciona con el enriquecimiento-ioc si el mecanismo usa un binario o
   dominio ya conocido.
5. Registra el resultado — positivo o negativo — en la memoria de sesión,
   porque "se verificó y no había persistencia" también es evidencia útil
   para el reporte final.

## Principios
- Ausencia de evidencia no es evidencia de ausencia: si no revisaste un
  ítem del checklist, dilo explícitamente en vez de omitirlo.
- Nunca cierres un incidente P1/P2 sin haber corrido este checklist.
