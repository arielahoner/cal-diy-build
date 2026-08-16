# patches/

Parches aplicados con `git apply` sobre el checkout de `calcom/cal.diy@<ref>` **antes** de construir la
imagen. Si hay alguno, la imagen se etiqueta con el sufijo `-patched` para que nunca se confunda con la
versión limpia del mismo SHA.

**Hoy esta carpeta está vacía a propósito.** El único parche contemplado (`sendUpdates:"all"`, para que sea
Google quien envíe la invitación nativa desde el calendario destino) está **pospuesto** hasta decidirlo con
evidencia en la Fase 4 / prueba P9 (D-010).

## Reglas

1. Un parche por cambio lógico, con nombre `NNN-descripcion-corta.patch` (p. ej. `010-google-send-updates.patch`).
2. Cabecera obligatoria en el propio archivo, como comentarios `#` antes del diff:
   - **Propósito** (qué cambia y por qué),
   - **SHA de upstream** sobre el que se generó y probó,
   - **Prueba** de `docs/pruebas.md` que lo justifica,
   - **Fecha** y **decisión** asociada en `docs/decisiones.md`.
3. Cada parche exige revalidación en **cada** actualización de versión: `git apply --check` puede pasar y
   aun así cambiar el comportamiento si el código de alrededor se movió.
4. Un parche no es el sitio para configuración: si algo se puede resolver con una variable de entorno o
   desde la interfaz, se resuelve así.

## Cómo generar uno

```bash
git clone https://github.com/calcom/cal.diy /tmp/cal.diy
cd /tmp/cal.diy && git checkout <sha>
# ...editar...
git diff > /ruta/al/repo/build/patches/010-descripcion.patch
# añadir a mano la cabecera con propósito / SHA / prueba / fecha
```
