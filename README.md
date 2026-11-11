# MARMO E ANIMA — Museo Virtuale 3D
### Una Mostra su Antonio Canova · Cinque opere neoclassiche + un'opera moderna

---

## Come avviare il museo

GLTFLoader richiede un server HTTP — aprire `index.html` direttamente con `file://` bloccherà il caricamento dei modelli 3D per via delle restrizioni CORS.

**1. Apri una finestra del terminale nella cartella del progetto**

**2. Installa le dipendenze (una volta sola) e avvia il dev server Vite:**
```
npm install
npm run dev
```

**3. Apri nel browser:**
```
http://localhost:8080
```

In alternativa, senza npm: `python -m http.server 8080`, poi apri `http://localhost:8080`.

**Browser consigliato:** Chrome o Edge (desktop) — richiedono l'API Pointer Lock.

---

## Controlli

| Tasto | Azione |
|-------|--------|
| `W A S D` | Muoversi |
| `Mouse` | Guardare intorno |
| `Shift` (tenuto) | Correre |
| `E` | Aprire la scheda dell'opera vicina (o il pannello d'uscita, vicino alla porta) |
| `T` | Ispezione 3D dell'opera vicina |
| `R` | Esci dall'ispezione 3D |
| `F` | Schermo intero |
| `ESC` | Chiudere la scheda / sbloccare il mouse |
| `Click` sulla scena | Bloccare il mouse (pointer lock) |

---

## File modelli nella cartella `models/models_opt/`

| Opera | File |
|-------|------|
| Psiche ravvivata dal bacio di Amore | `psiche.glb` |
| Le Tre Grazie | `le_tre_grazie.glb` |
| Venere Italica | `venere_italica.glb` |
| Paolina Borghese come Venere vincitrice | `paolina.glb` |
| Ebe | `ebe.glb` |
| Reclining Figure (Henry Moore, 1951) | `Reclining_Figure.glb` |

Tutti e 6 i modelli sono inclusi nel repository. Se un file non viene trovato, il museo non va in crash — mostra automaticamente un cilindro bianco al posto della scultura.

---

## Rigenerare un modello 3D

Per rigenerare una scultura da una foto (es. sostituire un modello esistente):

1. Usa [tripo3d.ai](https://tripo3d.ai) o [Hugging Face — TRELLIS](https://huggingface.co/spaces/JeffreyXiang/TRELLIS)
2. Esporta come **GLB** (compressione Draco consigliata)
3. Sostituisci il file corrispondente in `models/models_opt/`
4. Ricarica `http://localhost:8080`

---

## Struttura del progetto

```
marmo-e-anima/
├── index.html                          ← Tutta l'applicazione (JS + CSS + HTML)
├── package.json                        ← Vite come dev server (`npm run dev`)
├── README.md / DEVLOG.md
├── Antonio-Canova-biografia-e-opere.jpg ← Retratto nella schermata d'ingresso
├── statue_pics/                        ← Foto delle opere (tele dei pannelli)
└── models/models_opt/                  ← I 6 modelli GLB delle sculture
```

---

## Stack tecnico

- **Three.js r128** — motore 3D WebGL
- **GLTFLoader** — caricamento modelli sculture
- **Pointer Lock API** — controllo mouse in first-person
- **Canvas 2D API** — texture procedurali (pavimento a diamanti, soffitto a cassettoni, minimappa)
- **Vite 5** — solo dev server; a runtime zero dipendenze npm (vanilla HTML + JS)

---

*Tema: La Bellezza · Antonio Canova (1757–1822)*
