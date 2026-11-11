# Casa 3D: recursos y modelo propio

## Recursos realistas incluidos (Poly Haven, CC0)

La casa en código usa texturas PBR reales (roble, porcelánico, mármol, estuco, hormigón, césped),
un cielo HDRI de tarde y muebles escaneados (sofá, sillas, mesas, lámparas, plantas…) de
[polyhaven.com](https://polyhaven.com), con licencia **CC0** (uso comercial sin atribución).

Ahora se cargan desde el servidor de Poly Haven (~35 MB la primera visita; luego quedan en caché).
**Para producción**: descárgalos y súbelos a tu hosting con la misma estructura
(`Textures/jpg/1k/…`, `HDRIs/hdr/…`, `Models/gltf/1k/…` y `Models/jpg/1k/…`) y cambia
`CONFIG.recursos` en `index.html` a esa carpeta. Así no dependes de un servidor ajeno.

---

# Cómo poner una casa 3D real en la demo

La web ya sabe cargar un modelo `.glb`. Si no hay modelo (o falla la carga), usa la casa construida en código.

## 1. Conseguir el modelo

| Origen | Cómo exportar |
|---|---|
| Arquitecto (Revit, ArchiCAD) | Exportar a IFC/FBX → abrir en Blender → exportar `.glb` |
| SketchUp | Extensión gratuita *glTF Export* → `.glb` |
| Blender | Archivo → Exportar → glTF 2.0 (`.glb`), marcar **Compression (Draco)** |
| Escaneo (Polycam, Matterport, Luma) | Descargar como `.glb`. Ojo: muy pesados, conviene reducirlos |
| Sketchfab / CGTrader | Descargar en formato glTF/GLB. **Revisa la licencia** (uso comercial) |

**Peso recomendado:** menos de 15 MB (ideal < 8 MB para móvil). Reducir con
[gltf.report](https://gltf.report) o con `npx @gltf-transform/cli optimize casa.glb casa-opt.glb --compress draco --texture-compress webp`.

## 2. Preparar el modelo (Blender, 10 minutos)

1. **Escala en metros** y la casa apoyada en el suelo (Z = 0 en Blender).
2. **Fachada principal mirando a la cámara** (eje −Y en Blender = +Z en la web). Si no, usa `rotacionY` en la configuración.
3. **Nombra un objeto por estancia** (puede ser el suelo de la estancia o un grupo/Empty con sus muebles):
   `cocina`, `salon`, `bano`, `dormitorio`, `terraza`. La web calcula la caja de cada uno y la hace clicable.
   Nombra también la escalera como `escalera`: al tocarla se sube (desde planta baja) o se baja (desde planta alta).
4. Sin luces ni cámaras dentro del archivo (la web pone las suyas).

## 3. Configurarlo

Copia el archivo a esta carpeta (`Modelo Reformas/modelos/casa.glb`) y en `index.html`, dentro de `CONFIG.modelo`:

```js
url: 'modelos/casa.glb',
cortes: { pb: 1.6, pa: 4.9 },   // altura del corte en "Planta baja" y "Planta alta"
```

- **`cortes`**: la web secciona la casa a esa altura para ver el interior desde arriba.
  Usa ~1,5 m sobre el suelo de cada planta (p. ej. si la planta alta empieza a 3,2 m → `pa: 4.7`).
- **Si una estancia no tiene nombre en el modelo**, define su caja a mano:
  `bano: { planta: 'pa', x: [-6, -2], y: [3.2, 6], z: [-4.5, 0] }`
  Para sacar coordenadas abre `index.html?debug`, haz clic en el modelo y mira la consola (F12):
  se imprime el punto exacto y el nombre del objeto pulsado.
- **`jardin: false`** si el modelo trae su propia parcela.
- **`puerta: [x, y, z]`** para colocar el pin "Entrar en la casa" en la puerta real.

La consola también avisa de las zonas que no encuentra; esas estancias siguen funcionando desde los botones.

## 4. Probarlo

El modelo **no carga abriendo el archivo con doble clic** (`file://`); hace falta servirlo:

```
npx serve "Modelo Reformas"
```

y abrir la dirección que indique. En Vercel/Hostinger funciona directamente.
