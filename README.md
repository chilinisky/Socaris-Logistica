# Socaris-Logistica

Repositorio público que publica el **listado de licencias** que lee el programa
SOCARIS Logística para habilitar o bloquear la app.

El programa descarga este archivo al abrir:

```
https://raw.githubusercontent.com/chilinisky/Socaris-Logistica/main/licencias/catalog.json
```

## Qué hay aquí

| Archivo | Qué es |
| --- | --- |
| `licencias/catalog.json` | El listado **ofuscado** que descarga la app. Es lo único que se publica. |
| `licencias/licencias.json` | La lista **maestra en claro** donde se editan las licencias. **No se publica** (está en `.gitignore`). |

## Formato de `catalog.json`

No es JSON a la vista: es un contenedor binario con cabecera para que las claves
no queden públicas de un vistazo.

```
SOCLIC1 | hash32 (4 bytes) | cuerpo ofuscado
```

- `SOCLIC1`: cabecera y versión del formato.
- `hash32`: FNV-1a del texto en claro; si no coincide, el programa rechaza el archivo.
- cuerpo: el JSON en UTF-8 mezclado byte a byte con un flujo determinista.

Es **ofuscación, no cifrado**: sirve para que el listado no esté a la vista, no
para esconder datos de alguien decidido. La garantía real es la fecha de inicio
de cada licencia y el bloqueo del programa.

## Cómo dar de alta, renovar o suspender una licencia

1. Edita `licencias/licencias.json` (la lista maestra, en claro):

```json
{
  "clave": "SOC-2026-0003",
  "cliente": "Bodegas del Sur SpA",
  "plan": "ANUAL",
  "inicio": "2026-11-01",
  "dias": 365,
  "suspendida": false,
  "nota": null
}
```

2. Desde el proyecto de logística, regenera el archivo ofuscado:

```bash
npm run licencias:ofuscar -- --entrada "E:/Proyecto app/Socaris-Logistica/licencias/licencias.json" --salida "E:/Proyecto app/Socaris-Logistica/licencias/catalog.json"
```

3. Sube `licencias/catalog.json`:

```bash
git add licencias/catalog.json
git commit -m "Alta de licencia SOC-2026-0003"
git push
```

La app toma el cambio en la próxima verificación: al abrirse o al pulsar
**Verificar licencia**.

## Reglas del programa

- Clave publicada y dentro del plazo: funciona.
- Clave no publicada, suspendida o con el plazo cumplido: la app se bloquea con el
  aviso **"Su licencia o prueba ya está vencida. Contacte con soporte de SOCARIS."**
- Sin internet se usa el listado guardado en el PC la última vez, y el vencimiento
  igual corre porque sale de la fecha de inicio.
