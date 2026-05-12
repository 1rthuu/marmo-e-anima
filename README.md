# MARMO E ANIMA — Museo Virtuale 3D
### Una Mostra su Antonio Canova · Cinque Capolavori Neoclassici

---

## Come avviare il museo

GLTFLoader richiede un server HTTP — aprire `index.html` direttamente con `file://` bloccherà il caricamento dei modelli 3D per via delle restrizioni CORS.

**1. Apri una finestra del terminale nella cartella del progetto:**
```
c:\Users\chair\Desktop\art meseum
```

**2. Avvia il server Python:**
```
python -m http.server 8080
```

**3. Apri nel browser:**
```
http://localhost:8080
```

**Browser consigliato:** Chrome o Edge (desktop) — richiedono l'API Pointer Lock.

---

## Controlli

| Tasto | Azione |
|-------|--------|
| `W A S D` | Muoversi |
| `Mouse` | Guardare intorno |
| `Shift` (tenuto) | Correre |
| `E` | Aprire la scheda dell'opera vicina |
| `ESC` | Chiudere la scheda / sbloccare il mouse |
| `Click` sulla scena | Bloccare il mouse (pointer lock) |

---

## File modelli nella cartella `models/`

| Opera | File atteso |
|-------|-------------|
| Psiche ravvivata dal bacio di Amore | `3d_printable_psyche_revived_by_cupid.glb` |
| Le Tre Grazie | `the_three_graces.glb` |
| Venere Italica | `10-smk-venus-italica-kas-dep1.glb` |
| Paolina Borghese come Venere vincitrice | `Statue of Paolina Borghese como Venere vincitrice by Antonio Canova, Galleria Borghese, Roma .glb` |
| Ebe | `ebe.glb` *(mancante — viene mostrato un segnaposto cilindrico)* |

> **Nota:** Se un modello non viene trovato, il museo non va in crash — mostra automaticamente un cilindro bianco al posto della scultura.

---

## Come aggiungere `ebe.glb`

Quando il file sarà disponibile:

1. Genera il modello 3D da una foto della scultura usando uno di questi servizi:
   - [tripo3d.ai](https://tripo3d.ai)
   - [Hugging Face — TRELLIS](https://huggingface.co/spaces/JeffreyXiang/TRELLIS)
2. Esporta come **GLB**
3. Rinomina il file in `ebe.glb`
4. Copialo nella cartella `models/`
5. Ricarica `http://localhost:8080`

---

## Struttura del progetto

```
art meseum/
├── index.html                    ← L'intera applicazione museo
├── README.md                     ← Questo file
└── models/
    ├── 3d_printable_psyche_revived_by_cupid.glb
    ├── the_three_graces.glb
    ├── 10-smk-venus-italica-kas-dep1.glb
    ├── Statue of Paolina Borghese como Venere vincitrice by Antonio Canova, Galleria Borghese, Roma .glb
    └── ebe.glb  (da aggiungere)
```

---

## Stack tecnico

- **Three.js r128** — motore 3D WebGL
- **GLTFLoader** — caricamento modelli sculture
- **Pointer Lock API** — controllo mouse in first-person
- **Canvas 2D API** — texture procedurali (pavimento a diamanti, soffitto a cassettoni, minimappa)
- Zero dipendenze npm — tutto vanilla HTML + JavaScript

---

*Tema: La Bellezza · Antonio Canova (1757–1822)*
