# Guía de Usuario: Ecomo Report

## 📋 Introducción

Este sistema te permite crear plantillas Word que se rellenan automáticamente con imágenes y texto extraído de ecografías u otras imágenes médicas mediante OCR (Reconocimiento Óptico de Caracteres).

Solo necesitas insertar **etiquetas especiales** en tu documento Word, y el programa se encargará de buscar las imágenes correspondientes y extraer el texto necesario.

---

## 📥 Descarga e Instalación

### Descargar

Descarga la última versión del instalador:

➡️ **[Descargar ECOMO Report](https://aturing.github.io/ecomo-report-docs/ECOMO_Report_latest.pkg)**

### Instalación en macOS

1. Descarga el archivo `.pkg` desde el enlace anterior
2. Haz doble clic en el archivo descargado
3. Sigue las instrucciones del instalador
4. Una vez instalado, podrás acceder a ECOMO Report desde el menú de Servicios del Finder

### Uso

1. Selecciona la plantilla del informe que deseas generar. La plantilla deberá estar en la misma carpeta donde se encuentre una carpeta imagenes con las ecografías.
2. Haz clic derecho y selecciona **Servicios > Crear informe**
4. El informe se generará automáticamente

---

## 🏷️ Sintaxis de Etiquetas

Las etiquetas siguen el formato de Jinja2: `{{ nombre_variable }}`

### Tipos de etiquetas:

1. **`IMG_`** → Para insertar imágenes completas
2. **`TXT_`** → Para extraer y insertar texto específico

---

## 🖼️ Insertar Imágenes

### Funcionamiento:
- Las variables `IMG_` son **listas de imágenes** que contienen todas las imágenes que coinciden con el texto especificado
- El sistema buscará entre todas las imágenes aquellas que contengan el texto especificado
- El texto a buscar debe escribirse sin tildes ni ñs, independientemente de que el texto de la imagen contenga tildes o no.

### Reglas:
- Usa **guiones bajos `_`** para representar espacios
- El texto debe aparecer en la imagen (será detectado por OCR)

---

### Insertar una única imagen

Cuando solo existe **una imagen** con esa etiqueta, o cuando quieres insertar **solo la primera** imagen de la lista, utiliza el filtro `first`:

#### Sintaxis:
```
{{ IMG_texto_a_buscar | first }}
```

#### Ejemplos:

```
{{ IMG_ecografia_renal | first }}
```
→ Inserta la primera imagen que contenga el texto "ecografia renal"

```
{{ IMG_4_camaras | first }}
```
→ Inserta la primera imagen que contenga el texto "4 camaras"

```
{{ IMG_doppler_color | first }}
```
→ Inserta la primera imagen que contenga el texto "doppler color"

---

### Insertar múltiples imágenes

Cuando existen **varias imágenes** con la misma etiqueta y quieres insertarlas todas, utiliza un bucle `for`:

#### Sintaxis:
```
{%tr for img in IMG_texto_a_buscar %}  {{ img }}  {%tr endfor %}
```

> **Nota:** `{%tr ... %}` indica que el bucle se aplica a nivel de fila de tabla (table row). Esto es útil para insertar cada imagen en una fila separada.

#### Ejemplos:

```
{%tr for img in IMG_pulmon %}  {{ img }}  {%tr endfor %}
```
→ Inserta todas las imágenes que contengan el texto "pulmon", cada una en una fila

```
{%tr for img in IMG_ecografia_renal %}  {{ img }}  {%tr endfor %}
```
→ Inserta todas las imágenes que contengan el texto "ecografia renal"

```
{%tr for img in IMG_doppler %}  {{ img }}  {%tr endfor %}
```
→ Inserta todas las imágenes que contengan el texto "doppler"

---

## 📝 Extraer Texto

### Sintaxis:
```
{{ TXT_texto_antes__texto_despues }}
```

### Funcionamiento:
- Los **dos guiones bajos `__`** representan el **wildcard** (el valor a extraer)
- El sistema busca el patrón en las imágenes y extrae el valor entre las dos partes

### Reglas:
- Un guión bajo `_` = espacio en blanco
- Dos guiones bajos `__` = wildcard (valor a extraer)

### Ejemplos:

#### Ejemplo 1: Extraer un valor numérico simple
```
{{ TXT_volumen__ml }}
```
Si en la imagen aparece: **"volumen 350 ml"**  
→ Extrae: `350`

#### Ejemplo 2: Extraer con espacios en el patrón
```
{{ TXT_frecuencia_cardiaca__bpm }}
```
Si en la imagen aparece: **"frecuencia cardiaca 145 bpm"**  
→ Extrae: `145`

#### Ejemplo 3: Extraer valores con decimales
```
{{ TXT_tapse__cm }}
```
Si en la imagen aparece: **"tapse 2.3 cm"**  
→ Extrae: `2.3`

#### Ejemplo 4: Extraer valores complejos
```
{{ TXT_presion_arterial__mmHg }}
```
Si en la imagen aparece: **"presion arterial 120/80 mmHg"**  
→ Extrae: `120/80`

---

## 🔧 Filtros de Transformación

Puedes aplicar transformaciones al texto extraído usando el símbolo **pipe `|`**

### Sintaxis:
```
{{ TXT_patron__texto | filtro(parametro) }}
```

### Filtros Numéricos:

#### `multiply(n)` - Multiplicar
```
{{ TXT_tapse__cm | multiply(10) }}
```
Si extrae `2.3` → Resultado: `23`

#### `divide(n)` - Dividir
```
{{ TXT_volumen__ml | divide(1000) }}
```
Si extrae `1500` → Resultado: `1.5`

#### `add(n)` - Sumar
```
{{ TXT_valor__unidad | add(5) }}
```
Si extrae `10` → Resultado: `15`

#### `subtract(n)` - Restar
```
{{ TXT_medida__mm | subtract(2) }}
```
Si extrae `25` → Resultado: `23`

#### `round(n)` - Redondear a n decimales
```
{{ TXT_tapse__cm | multiply(10) | round(1) }}
```
Si extrae `2.34` → Multiplica por 10 → `23.4` → Resultado: `23.4`

### Filtros de Texto:

#### `first` - Obtener el primer elemento de una lista
```
{{ IMG_ecografia | first }}
```
Obtiene la primera imagen de la lista → útil cuando `IMG_` devuelve múltiples imágenes pero solo necesitas una

#### `upper` - Convertir a MAYÚSCULAS
```
{{ TXT_codigo__fin | upper }}
```
Si extrae `abc123` → Resultado: `ABC123`

#### `lower` - Convertir a minúsculas
```
{{ TXT_nombre__apellido | lower }}
```
Si extrae `JUAN` → Resultado: `juan`

#### `replace(viejo, nuevo)` - Reemplazar texto
```
{{ TXT_valor__x | replace(., ,) }}
```
Si extrae `1.5` → Resultado: `1,5`

### Combinar Múltiples Filtros:

Puedes encadenar varios filtros usando múltiples pipes:

```
{{ TXT_tapse__cm | multiply(10) | round(1) }}
```
1. Extrae: `2.34`
2. Multiplica por 10: `23.4`
3. Redondea a 1 decimal: `23.4`

```
{{ TXT_volumen__ml | divide(1000) | round(2) }}
```
1. Extrae: `1567`
2. Divide por 1000: `1.567`
3. Redondea a 2 decimales: `1.57`

---

## 📄 Ejemplo de Plantilla Completa

```word
═══════════════════════════════════════
        INFORME ECOCARDIOGRÁFICO
═══════════════════════════════════════

PACIENTE: Dr. García López
FECHA: 15/12/2024

---

IMÁGENES PRINCIPALES:

Vista 4 Cámaras:
{{ IMG_4_camaras | first }}

Doppler Tisular:
{{ IMG_doppler_tisular | first }}

---

MEDICIONES:

• TAPSE: {{ TXT_tapse__cm | multiply(10) | round(1) }} mm
• Volumen telediastólico: {{ TXT_volumen_td__ml }} ml
• Volumen telesistólico: {{ TXT_volumen_ts__ml }} ml
• Fracción de eyección: {{ TXT_fevi__porcentaje }}%
• Frecuencia cardíaca: {{ TXT_fc__bpm }} bpm
• Presión arterial: {{ TXT_presion__mmHg }} mmHg

---

OBSERVACIONES:

Técnica utilizada: {{ TXT_tecnica__observaciones | upper }}
Calidad de imagen: {{ TXT_calidad__ventana }}

---

CONCLUSIONES:

Contractilidad: {{ TXT_contractilidad__segmentos }}
```

---

## ✅ Consejos y Buenas Prácticas

### 1. Nomenclatura Clara
- Usa nombres descriptivos que coincidan con el texto real en las imágenes
- Ejemplo: Si la imagen dice "eco renal derecho", usa `{{ IMG_eco_renal_derecho }}`

### 2. Patrones Específicos
- Sé lo más específico posible en los patrones de búsqueda
- Malo: `{{ TXT_valor__x }}`
- Bueno: `{{ TXT_tapse__cm }}`

### 3. Orden de las Etiquetas
- No importa el orden en que coloques las etiquetas en el documento
- El sistema procesará todas automáticamente

### 4. Espacios y Guiones Bajos
- **Un guión bajo `_`** = un espacio
- **Dos guiones bajos `__`** = wildcard
- Ejemplo: `TXT_frecuencia_cardiaca__bpm` busca "frecuencia cardiaca X bpm"

### 5. Manejo de Errores
- Si no se encuentra una imagen: aparecerá `[Imagen 'texto' no encontrada]`
- Si no se encuentra texto: aparecerá `[No encontrado]`
- Revisa el archivo de log para más detalles

---

## 🔍 Resolución de Problemas

### La imagen no se inserta
✅ Verifica que el texto especificado aparezca exactamente en la imagen  
✅ Revisa que estés usando guiones bajos para los espacios  
✅ Comprueba que la imagen esté en formato compatible (PNG, JPG, TIFF, BMP)

### El texto extraído es incorrecto
✅ Asegúrate de que el patrón `antes__despues` sea único en la imagen  
✅ Verifica que el OCR pueda leer claramente el texto de la imagen  
✅ Usa patrones más específicos para evitar coincidencias falsas

### Los filtros no funcionan
✅ Verifica la sintaxis: `| filtro(parametro)`  
✅ Asegúrate de que el valor extraído sea numérico si usas filtros numéricos  
✅ Revisa el log para ver mensajes de error específicos

---

## 📊 Casos de Uso Comunes

### Ecocardiografía
```
TAPSE: {{ TXT_tapse__cm | multiply(10) | round(1) }} mm
E/A: {{ TXT_ratio_e_a__adimensional | round(2) }}
FE: {{ TXT_fevi__porcentaje }}%
```

### Ecografía Obstétrica
```
Edad gestacional: {{ TXT_eg__semanas }} semanas
Peso fetal estimado: {{ TXT_pfe__g | divide(1000) | round(2) }} kg
Líquido amniótico: {{ TXT_ila__cm }} cm
```

### Ecografía Abdominal
```
Tamaño hepático: {{ TXT_higado__cm }} cm
Vesícula biliar: {{ TXT_vesicula__descripcion }}
Bazo: {{ TXT_bazo__cm }} cm
```

---

## 📞 Soporte

Si encuentras problemas o necesitas ayuda:

1. Revisa el archivo `ecomo_report.log` para detalles de errores
2. Verifica que las imágenes sean legibles y tengan buena calidad
3. Asegúrate de que el texto en las imágenes esté claramente visible

---

**Versión:** 0.1  
**Última actualización:** Diciembre 2025