# Copacking – Jurnal verificări

Aplicație web mobile-first (un singur `index.html`, fără build) pentru jurnalul activității de copacking din depozit: PO-uri active, poze de verificare la interval, ecran de supervizare live, arhivă și administrare utilizatori. Backend: Firebase (Authentication + Firestore, plan gratuit Spark).

| Fișier | Rol |
|---|---|
| `index.html` | aplicația (operator, supervizor, arhivă, utilizatori) |
| `firestore.rules` | regulile de securitate Firestore – se lipesc în consolă |
| `SETUP.md` | ghid de instalare pas cu pas (Firebase, hosting, utilizatori) |
| `.nojekyll` | spune GitHub Pages să publice fișierele ca atare |

## Publicare pe GitHub Pages (pe scurt)

1. Completează configurarea Firebase în blocul `window.COPACKING_CONFIG` din `index.html`.
2. Urcă fișierele în repo (branch `main`).
3. **Settings → Pages → Build and deployment → Source: Deploy from a branch → `main` / `(root)` → Save.**
4. Adresa va fi `https://<utilizator>.github.io/<repo>/` – adaug-o în Firebase **Authentication → Settings → Authorized domains** (`<utilizator>.github.io`).

Detalii complete, inclusiv crearea primului admin și limitele planului gratuit: vezi `SETUP.md`.
