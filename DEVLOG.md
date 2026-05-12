# DEVLOG — MARMO E ANIMA
### Museo 3D Virtuale · Antonio Canova
*Ultimo aggiornamento: 29 Aprile 2026*

---

## Panoramica

Questo documento descrive in dettaglio tutto ciò che è stato costruito, modificato e deciso durante lo sviluppo del museo virtuale 3D **"Marmo e Anima"** — una mostra interattiva in prima persona dedicata ai capolavori neoclassici di Antonio Canova.

L'intera applicazione è contenuta in **un singolo file `index.html`** (≈1200 righe), senza framework, senza bundler runtime, senza dipendenze npm a runtime. Vite viene usato solo come dev server.

---

## Stack Tecnico

| Layer | Tecnologia | Versione |
|---|---|---|
| Motore 3D | Three.js | r128 (CDN) |
| Loader modelli | GLTFLoader | r128 (CDN jsDelivr) |
| Dev server | Vite | ^5.4.0 (npm) |
| Linguaggio | Vanilla JavaScript (ES6+) | — |
| Stile UI | Vanilla CSS | — |
| Font | Georgia (serif, browser nativo) | — |

---

## Come Avviare

### ✅ Metodo raccomandato — npm (Vite)
```bash
# Dalla cartella del progetto:
npm install      # una volta sola
npm run dev      # avvia il server e apre il browser
```
Vite serve i file su **`http://localhost:8080`** e apre automaticamente il browser.

### ⚠️ Perché non aprire `index.html` direttamente
Il `GLTFLoader` di Three.js carica i file `.glb` tramite fetch HTTP.
Aprire il file con il protocollo `file://` fa scattare i blocchi CORS del browser,
impedendo il caricamento dei modelli 3D.
Il file `.html` contiene un **guard** che intercetta `file://` e mostra
un messaggio di errore chiaro invece di silenziare il problema:
```javascript
if (window.location.protocol === 'file:') {
  // mostra schermata di avviso con istruzioni
  throw new Error('file:// not supported');
}
```

---

## Struttura del Progetto

```
art meseum/
├── index.html          ← Tutta l'applicazione (JS + CSS + HTML)
├── package.json        ← Vite come devDependency
├── node_modules/       ← Vite (non toccare)
├── README.md           ← Come avviare + controlli + modelli attesi
├── DEVLOG.md           ← Questo file
└── models/
    ├── 3d_printable_psyche_revived_by_cupid.glb     (≈38 MB)
    ├── the_three_graces.glb                          (≈50 MB)
    ├── 10-smk-venus-italica-kas-dep1.glb             (≈307 MB) ⚠️ pesante
    ├── Statue of Paolina Borghese como Venere
    │   vincitrice by Antonio Canova,
    │   Galleria Borghese, Roma .glb                  (≈31 MB)
    ├── ebe.glb                                       ← MANCANTE (placeholder)
    └── teseo.glb                                     ← MANCANTE (placeholder)
```

---

## Cosa è Stato Costruito

### 1. Geometria della Sala (`buildRoom`)

La sala è un ambiente neoclassico con le seguenti dimensioni:
- **50m larghezza (X) × 30m profondità (Z) × 8m altezza (Y)**
- Il giocatore nasce a `(0, 1.72, 10)` guardando nord (verso `-Z`)

**Elementi costruiti con Three.js geometry:**

| Elemento | Geometry | Note |
|---|---|---|
| Pavimento | `PlaneGeometry 50×30` | `MeshStandardMaterial` crema `#e8e2d4` |
| Sovrapposizione diamante | Secondo `PlaneGeometry` | Texture canvas con linee diagonali a 45° |
| Soffitto | `PlaneGeometry 50×30` a y=8 | Texture canvas con pannelli a cassettone + bordi oro |
| 4 Pareti | `BoxGeometry` | Colore `#ede8dc`, ombre |
| Dado rail | `BoxGeometry` sottile a y=1.0 | Colore `#d8d0c0` su ogni parete |
| Cornicione oro | `BoxGeometry` a y=7.9 | `color #c9a96e`, `metalness 0.7` |
| 4 Colonne | `CylinderGeometry` r 0.65→0.75, h=8 | Con basamento quadrato `BoxGeometry 2×0.4×2` |
| Arco ingresso | `TorusGeometry` + 2 pilastri | `color #c9a96e`, `metalness 0.6` |
| Targa titolo | `BoxGeometry 6×0.8×0.15` | Oro sopra l'arco |
| 6 Piedistalli | `BoxGeometry 1.5×1.2×1.5` | Colore `#2a2520` (quasi nero) |
| Top piedistallo | `BoxGeometry 1.5×0.05×1.5` | Colore `#3a3530` |
| Targhetta nome | `BoxGeometry 0.9×0.15×0.06` | Oro, orientata verso lo spettatore |

Le texture del pavimento (diamanti) e del soffitto (cassettoni) sono generate
proceduralmente con la **Canvas 2D API** — nessuna immagine esterna.

---

### 2. Illuminazione (`buildLighting`)

Il sistema di illuminazione è stato progettato per essere **teatrale**:
luce concentrata sulle sculture, penombra nel resto della sala.

| Luce | Tipo | Intensità | Scopo |
|---|---|---|---|
| Ambient | `AmbientLight #1a1208` | 0.22 | Elimina il nero assoluto |
| Hemisphere | `HemisphereLight` | 0.18 | Calore dal soffitto, terra scura |
| 4× Ceiling fill | `PointLight #fff0d8` range 18 | 0.55 | Illuminazione di base della sala |
| 6× Main spot | `SpotLight #fff5d0` cono stretto | 6.0 | Fascio teatrale sulle sculture |
| 6× Fill spot | `SpotLight #ffd090` cono largo | 1.2 | Lato frontale scultura |
| 6× Rim | `PointLight #8090b0` | 0.6 | Luce fredda/blu da dietro = profondità |
| 4× Column uplight | `PointLight #c08040` | 0.5 | Uplight caldo sulla base delle colonne |
| 4× Corner fill | `PointLight #ffeedd` range 14 | 0.18 | Evita angoli completamente neri |

**Tone mapping:** `ACESFilmicToneMapping`, exposure `0.82` (scuro/cinematografico).
**Shadow map:** `PCFSoftShadowMap`, 1024×1024 per ogni spot sulle sculture.

**Direzione spot adattiva per riga:**
- Riga nord (sculture che guardano sud): spot arriva da `az + 1.8`
- Riga sud (sculture che guardano nord): spot arriva da `az - 1.8`

---

### 3. Layout Sculture — 2 Righe di 3 (Back to Back)

```
PARETE NORD (z = -15)
══════════════════════════════════════════════════════
  [Psiche]         [Tre Grazie]      [Venere Italica]
  x=-9, z=-8       x=0, z=-8        x=9, z=-8
  ↓ guardano SUD (verso l'ingresso) ↓

- - - - - - - - CORRIDOIO CENTRALE - - - - - - - -

  ↑ guardano NORD (spalle all'ingresso) ↑
  [Paolina]        [Ebe]             [Teseo]
  x=-9, z=+7       x=0, z=+7        x=9, z=+7
══════════════════════════════════════════════════════
INGRESSO / ARCO (z = +15)
```

Le posizioni x=±9 sono state scelte deliberatamente per evitare
le 4 colonne posizionate a `x=±12, z=±5`.

Le targhette dei piedistalli sono orientate verso lo spettatore
(riga nord: `z + 0.78`, riga sud: `z - 0.78` con `rotation.y = Math.PI`).

---

### 4. Caricamento Modelli GLB (`GLTFLoader`)

Tutti i modelli vengono ridimensionati automaticamente a **1.8m di altezza**
(bounding box) per uniformità:
```javascript
const scale = 1.8 / size.y;
model.scale.setScalar(scale);
```

Material override: ogni mesh del modello riceve un `MeshStandardMaterial`
marmo bianco (`#f8f5ef`, roughness 0.25, metalness 0.0).

**Fallback:** se un modello non si trova (404), viene mostrato un cilindro
bianco (`CylinderGeometry 0.25→0.35, h=1.6`) senza crash.

**Loading overlay:** barra dorata che si riempie man mano che i modelli
arrivano (20% per modello). Svanisce quando tutti e 6 sono pronti.

---

### 5. Controllo Giocatore (First-Person)

| Parametro | Valore |
|---|---|
| FOV base | 72° |
| FOV sprint | 76° (lerp 0.1) |
| Altezza | y = 1.72m (costante) |
| Velocità camminata | 5 m/s |
| Velocità corsa (Shift) | 9 m/s |
| Sensibilità mouse | 0.0016 |
| Pitch clamp | ±50° |

**Pointer Lock API:** click sul canvas → blocco mouse → `isPointerLocked = true`.
ESC chiude il pannello info se aperto, altrimenti sblocca il mouse.

**Collisione:**
- Pareti: AABB con margine 0.5m
- Colonne: distanza dal centro < raggio + 0.6m (push-out radiale)
- Bounds globali: `x ∈ [-24, 24]`, `z ∈ [-14.5, 14.5]`

---

### 6. HUD — Elementi dell'Interfaccia

| Elemento | Posizione | Descrizione |
|---|---|---|
| `#title-watermark` | Top-left | "MARMO E ANIMA" — piccolo, colore scuro |
| `#controls-hint` | Bottom-left | Guida controlli, opacità 35% |
| `#artwork-prompt` | Bottom-center | Tasto **[E]** + "LEGGI SCHEDA OPERA" — grande, pulsante |
| `#crosshair` | Centrato | Mirino dorato a croce, visibile solo con pointer lock |
| `#minimap` | Top-right | Canvas 110×110px con mappa sala, punto giocatore e direzione |

L'`#artwork-prompt` è composto da due layer sovrapposti:
1. `#artwork-prompt-bg`: pillola scura con bordo oro (sfondo)
2. `#artwork-prompt`: tasto `[E]` bordo dorato + testo cream

Entrambi pulsano tra opacità 55%→100% quando attivi.

---

### 7. Pannello Info Opera (`#info-panel`)

Si apre dal lato destro con transizione `cubic-bezier` (0.45s).
Premere `E` vicino a una scultura (< 5m) popola e apre il pannello.

Contiene (dall'alto in basso):
- Numero opera: `OPERA 01 / 06`
- Titolo, anno, location, materiale
- Committente (storia del commissioning)
- Descrizione (lettura della scultura)
- Emozioni (interpretazione soggettiva)

---

### 8. Minimap

Aggiornata ogni frame. Sistema di coordinate:
```javascript
mapX = ((worldX + 25) / 50) * 102 + 4;
mapZ = ((-worldZ + 15) / 30) * 102 + 4;
```
Mostra: outline sala, 4 colonne (punti), 6 opere (rettangoli oro),
punto giocatore (cerchio dorato), linea direzione sguardo.

---

## Performance — Note Importanti

### ⚠️ Movimento Lento / Lag

Durante i test è stato osservato un certo **rallentamento nel movimento** del giocatore.
La causa principale è quasi certamente **la dimensione dei file GLB**:

| Modello | Dimensione |
|---|---|
| Venere Italica | **307 MB** ← principale responsabile |
| Le Tre Grazie | 50 MB |
| Psiche | 38 MB |
| Paolina Borghese | 31 MB |

Un modello da 307 MB contiene una geometria estremamente densa (milioni di poligoni)
che stressa sia la RAM della GPU che la pipeline di rendering.

### Soluzioni Consigliate

1. **Ottimizza i modelli con Blender (gratuito):**
   - Apri il `.glb` in Blender
   - `Modifier Properties → Decimate` → porta il ratio a 0.1–0.2
   - File → Export → glTF 2.0 (.glb)
   - Target: **< 20 MB** per modello

2. **Usa Draco compression** durante l'export Blender:
   - Spunta "Draco mesh compression" nell'export glTF
   - Richiede l'aggiunta di `DRACOLoader` a `index.html`
   - Riduzione tipica: 60–80% della dimensione

3. **Strumento online:** [gltf.report](https://gltf.report/) o [meshopt](https://meshoptimizer.org/)
   permette di comprimere senza installare nulla.

4. **LoD manuale:** due versioni del modello (alta/bassa risoluzione),
   switch automatico in base alla distanza dalla camera.

### Shadow Map Performance
Sono attive 6 shadow map da 1024×1024 (una per spotlight scultura).
Se la GPU fatica, ridurre a 512×512:
```javascript
spot.shadow.mapSize.set(512, 512); // invece di 1024
```

---

## Iterazioni di Sviluppo

### Versione 1 — Scaffold
- Renderer Three.js base, camera, loop animate
- Verifica 60fps con canvas nero

### Versione 2 — Room + Lighting + Player + UI (parallelo)
- Geometria sala completa con texture procedurali
- Sistema di illuminazione a 3 layer
- Controller first-person con pointer lock
- Pannello info, minimap, HUD

### Versione 3 — Fix Lighting (su feedback visivo)
- Ridotto ambient da `0.6` a `0.22` (troppo piatto)
- Ridotto hemisphere da `0.4` a `0.18`
- Ridotto exposure da `1.1` a `0.82`
- Aggiunto rim light blu-freddo per profondità
- Spotlight: cono più stretto (`PI/10`), intensità aumentata (`6.0`)

### Versione 4 — E Prompt Redesign
- Sostituito testo piccolo con HUD grande (tasto `[E]` + testo cream)
- Aggiunto sfondo pillola scura con bordo oro
- Animazione pulse 55%→100%

### Versione 5 — Layout 3+3 + 6° Opera
- 5 sculture su riga orizzontale → 2 righe di 3 (back to back)
- Aggiunta 6ª opera: **Teseo sul Minotauro** (1781–1783, V&A Londra)
- Targhette ruotate per riga sud (`rotation.y = Math.PI`)
- Spotlight adattivi per direzione riga

### Versione 6 — npm / Vite
- Sostituito `python -m http.server` con Vite
- Aggiunto `package.json` + `npm install`
- `npm run dev` → apre automaticamente `http://localhost:8080`
- Aggiunto guard `file://` con schermata di errore styled

---

## Opere nella Mostra

| # | Titolo | Anno | Location originale | File |
|---|---|---|---|---|
| 1 | Psiche ravvivata dal bacio di Amore | 1787–1793 | Louvre, Parigi | ✅ caricato |
| 2 | Le Tre Grazie | 1812–1817 | Ermitage, San Pietroburgo | ✅ caricato |
| 3 | Venere Italica | 1804–1812 | Palazzo Pitti, Firenze | ✅ caricato |
| 4 | Paolina Borghese come Venere vincitrice | 1804–1808 | Galleria Borghese, Roma | ✅ caricato |
| 5 | Ebe | 1796–1799 | Nationalgalerie, Berlino | ⏳ `ebe.glb` mancante |
| 6 | Teseo sul Minotauro | 1781–1783 | V&A Museum, Londra | ⏳ `teseo.glb` mancante |

Per aggiungere i modelli mancanti: genera da [tripo3d.ai](https://tripo3d.ai)
o [Hugging Face TRELLIS](https://huggingface.co/spaces/JeffreyXiang/TRELLIS),
esporta come `.glb`, rinomina e metti in `models/`.

---

*Museo costruito con Three.js r128 · Vite 5.4 · Vanilla HTML/CSS/JS*
