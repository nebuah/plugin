# Nebuah — contrato del kernel

El contrato vive en `nebuah/CLAUDE.md`, **adentro del paquete del plugin**, y
se importa acá para que valga también cuando se trabaja dentro de este clon.

@nebuah/CLAUDE.md

---

Se movió el 2026-09-02. Estaba sólo en la raíz del repo, así que quien
instalaba el plugin desde el marketplace obtenía `/nebuah` y los tres
subagentes **sin el contrato**: sin la convención de niveles, sin el formato de
traces, sin las reglas de memoria. Ver `docs/UNIFICACION.md`.
