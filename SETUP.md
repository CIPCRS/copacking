# Copacking – Jurnal verificări · Ghid de instalare

Aplicația este un singur fișier `index.html` (fără build, fără server propriu) care folosește **Firebase** (planul gratuit *Spark*): **Authentication** (email/parolă) și **Firestore** (bază de date + cache offline). Nu se folosește Storage și nu sunt necesare Cloud Functions.

Fișiere livrate:

| Fișier | Rol |
|---|---|
| `index.html` | aplicația (operator, supervizor, arhivă, utilizatori) |
| `firestore.rules` | regulile de securitate Firestore |
| `SETUP.md` | acest ghid |

Timp estimat: 20–30 de minute.

---

## 1. Creează proiectul Firebase

1. Intră pe <https://console.firebase.google.com> cu un cont Google și apasă **Add project / Creează un proiect**.
2. Nume proiect: de ex. `copacking`. Google Analytics poate fi **dezactivat** (nu e necesar).
3. Apasă **Create project** și așteaptă să se termine.

## 2. Adaugă aplicația web și copiază configurarea

1. În pagina proiectului, apasă pictograma **`</>`** (Web) → nume aplicație `Copacking` → **Register app**. Nu bifa Firebase Hosting aici (îl adaugi la pasul 6 dacă vrei).
2. În ecranul „Add Firebase SDK” apare un obiect `firebaseConfig`. Copiază valorile `apiKey`, `authDomain`, `projectId`, `appId`.
3. Deschide `index.html` într-un editor de text și completează blocul de la începutul fișierului:

```js
window.COPACKING_CONFIG = {
  firebase: {
    apiKey:     "AIza...",
    authDomain: "copacking-xxxxx.firebaseapp.com",
    projectId:  "copacking-xxxxx",
    appId:      "1:1234567890:web:abcdef"
  },
  ...
};
```

Tot acolo poți schimba:
- `logWindowDays` (implicit 60) – câte zile de jurnal se încarcă pe ecranele Operator/Supervizor;
- `photoMaxSide` / `photoQuality` – mărimea și calitatea pozelor salvate.

## 3. Activează autentificarea cu Email/Parolă

1. Meniul din stânga → **Build → Authentication** → **Get started**.
2. Tab **Sign-in method** → **Email/Password** → activează **Enable** (doar prima opțiune; „Email link” rămâne oprit) → **Save**.

> Aplicația transformă „nume + PIN” în cont de email intern: `nume@copacking.app` + PIN ca parolă. Adresele nu trebuie să existe real. Firebase cere parole de **minim 6 caractere**, de aceea PIN-ul are minim 6 cifre.

## 4. Creează baza de date Firestore și lipește regulile

1. **Build → Firestore Database** → **Create database**.
2. Locație: alege una din Europa (de ex. `eur3 (europe-west)` sau `europe-west1`). Locația nu se mai poate schimba ulterior.
3. Mod: **Start in production mode** (regulile le punem imediat) → **Create**.
4. Tab **Rules** → șterge tot conținutul și lipește conținutul fișierului `firestore.rules` → **Publish**.

Nu sunt necesari indecși compuși. Dacă totuși în consola browserului apare o eroare Firestore cu un link „create index”, deschide link-ul și apasă **Create** – e o operațiune de un click.

## 5. Creează primul administrator (manual)

Auto-înregistrarea nu dă acces: regulile cer un document `users/{uid}` cu `active: true`, iar documentele `users` pot fi scrise doar de administratori. Primul admin se face de mână:

**a) Contul de autentificare**

1. **Authentication → Users → Add user**.
2. Email: `admin@copacking.app` (sau `ion.popescu@copacking.app` – partea dinaintea `@` este numele de login, doar litere mici, cifre și puncte, fără diacritice).
3. Password: PIN-ul de 6+ cifre, de ex. `123456`.
4. **Add user** → copiază **User UID**-ul care apare în listă (un șir de ~28 de caractere).

**b) Documentul de profil**

1. **Firestore Database → Data → Start collection**.
2. Collection ID: `users` → Next.
3. Document ID: lipește **UID-ul** copiat (nu lăsa Auto-ID!).
4. Câmpuri:

| Field | Type | Value |
|---|---|---|
| `name` | string | `Admin` (numele afișat) |
| `login` | string | `admin` (partea dinaintea `@`) |
| `role` | string | `admin` |
| `active` | boolean | `true` |

5. **Save**.

Ceilalți utilizatori îi creezi din aplicație (pasul 7).

## 6. Publică aplicația (hosting)

`index.html` trebuie servit prin **https** (camera telefonului și cache-ul offline funcționează doar pe https sau pe `localhost`). Două variante gratuite:

### Varianta A – Netlify Drop (cea mai rapidă, fără instalare)

1. Pune `index.html` singur într-un folder (de ex. `copacking-site`).
2. Intră pe <https://app.netlify.com/drop> și trage folderul în pagină.
3. Primești o adresă de forma `https://nume-aleator.netlify.app`. Din **Site settings → Change site name** o poți face mai prietenoasă, de ex. `https://copacking-dsv.netlify.app`.
4. La fiecare modificare a fișierului, retrage folderul în **Deploys → drag & drop**.

### Varianta B – Firebase Hosting (din linia de comandă)

```bash
npm install -g firebase-tools
firebase login
cd folderul-cu-index.html
firebase init hosting     # alege proiectul, public directory: .  (punct), single-page app: No, nu suprascrie index.html
firebase deploy --only hosting
```

Adresa va fi `https://PROJECT_ID.web.app`.

### Autorizează domeniul (obligatoriu, altfel login-ul dă eroare `auth/unauthorized-domain`)

**Authentication → Settings → Authorized domains → Add domain** → adaugă domeniul de hosting (ex. `copacking-dsv.netlify.app`). `localhost` și `PROJECT_ID.web.app`/`firebaseapp.com` sunt deja în listă.

## 7. Adaugă utilizatori

1. Deschide adresa aplicației pe laptop, loghează-te cu **Nume** = `admin` (partea dinaintea `@`) și **PIN**-ul setat.
2. Tab **Utilizatori** → completează Nume (poate avea diacritice și spații – aplicația afișează sub câmp numele de login rezultat, ex. `ion.popescu`), PIN (minim 6 cifre), Rol → **Creează utilizator**.
3. Din listă poți **activa / dezactiva** un utilizator sau schimba rolul. Un admin nu se poate dezactiva singur.

Roluri:

| Rol | Ecrane |
|---|---|
| `operator` | Operator |
| `supervisor` | Operator + Supervizor + Arhivă |
| `admin` | toate + Utilizatori |

Ecranele au și adrese directe: `…/#supervizor`, `…/#arhiva`, `…/#utilizatori` (de ex. pentru laptopul supervizorului).

## 8. Instalează aplicația pe telefon

Aplicația este o PWA (fișierele `manifest.webmanifest`, `sw.js` și iconițele `icon-*.png` trebuie să fie în repo lângă `index.html`).

- **Android (Chrome):** deschide adresa, loghează-te și apasă butonul **⬇ Instalează** din bara albastră de sus (apare după câteva secunde). Alternativ: meniul ⋮ → **Instalează aplicația** / **Adaugă la ecranul de pornire**. Aplicația se deschide apoi pe tot ecranul, fără bara de adresă.
- **iPhone (Safari):** butonul **Partajare** (pătratul cu săgeată) → **Adaugă la ecranul principal**. iOS nu oferă un buton de instalare în pagină; după adăugare aplicația rulează pe tot ecranul.
- Dacă aplicația a fost adăugată pe ecranul de start înainte de a exista aceste fișiere, șterge iconița veche și adaug-o din nou.

Service worker-ul folosește strategia *network-first*: când există internet, pagina se ia mereu de pe server (actualizările apar imediat, fără cache vechi); offline se folosește copia locală.

Pe fiecare telefon: loghează-te o dată (sesiunea rămâne), setează **Tura** și **Intervalul de poze** (se rețin pe telefon).

---

## Note și limite

**PIN uitat.** Nu există „resetare parolă” (adresele de email nu sunt reale). Un admin **dezactivează** vechiul utilizator din tab-ul Utilizatori și **creează unul nou** cu alt nume de login (ex. `ion.popescu2`) sau șterge contul vechi din Authentication → Users și îl recreează cu același nume.

**Limitele planului gratuit Spark (Firestore):**

| Resursă | Limită gratuită | Ce înseamnă pentru noi |
|---|---|---|
| Stocare | 1 GiB total | o poză redimensionată ≈ 150–350 KB → **~3.000 de poze** în total |
| Scrieri | 20.000 / zi | o poză = 2 scrieri; mult peste nevoie |
| Citiri | 50.000 / zi | fiecare telefon citește la prima deschidere jurnalul ultimelor `logWindowDays` zile, apoi doar modificările |
| Mărime document | 1 MiB | aplicația reduce calitatea JPEG automat ca poza să încapă |

Recomandări:
- Urmărește **Firestore → Usage** în consolă. Când stocarea se apropie de 1 GiB, exportă arhiva (CSV) și șterge documentele vechi din colecțiile `photos` și `logs` (ștergerea e permisă doar adminilor; se poate face din consolă sau cu un script). Rezumatele din `archive` (cu miniaturi de 128 px, ~5 KB fiecare) pot rămâne.
- Alternativ, treci pe planul **Blaze** (pay-as-you-go, aceleași cote gratuite incluse, plătești doar ce depășești – pentru volumele de aici, de ordinul câțiva euro/lună sau zero).
- Miniaturile (128 px) sunt salvate direct în evenimentul din jurnal, deci listele și galeriile nu descarcă pozele complete; poza completă se descarcă doar la deschiderea ei pe ecran întreg.

**Comportament offline.**
- Aplicația păstrează un cache local (IndexedDB) și funcționează fără internet după ce a fost deschisă cel puțin o dată și utilizatorul s-a logat o dată cu conexiune.
- Pozele și verificările făcute offline apar imediat pe telefonul respectiv și se trimit automat când revine conexiunea; celelalte telefoane / laptopul supervizorului le văd abia atunci.
- Ștergerea datelor de navigare (cache / site data) din browser golește cache-ul și cere o nouă logare.
- Cronometrele „ultima poză” se calculează pe ora telefonului; verificați ca telefoanele să aibă ora setată automat.

**Jurnalul și arhiva.**
- Ecranele Operator și Supervizor încarcă doar ultimele `logWindowDays` zile (implicit 60). PO-urile închise mai vechi se văd în **Arhivă**, unde rămân definitiv.
- La închiderea unui PO, aplicația scrie automat rezumatul în `archive`. Butonul **„Importă PO-urile închise din jurnal”** reconstruiește rezumatele pentru PO-uri închise înainte de a exista arhiva sau dacă scrierea a eșuat (le sare pe cele deja existente).

**Planificare PO-uri (supervizor/admin).** Tab-ul **Planificare** permite introducerea PO-urilor în avans (număr, articol, cantitate, %, dată, tură, observații) sau importul lor dintr-un fișier Excel (`.xlsx`/`.xls`/`.csv`) cu un rând de antet; coloanele recunoscute sunt **PO, Articol, Cantitate** (obligatorii) și **Procent, Data, Tura, Observatii** (opționale) – butonul „Model Excel" descarcă un șablon. Înainte de import, aplicația arată o previzualizare și marchează rândurile invalide sau cu PO deja folosit. Operatorii văd PO-urile planificate în secțiunea „PO nou" și le pornesc cu un singur buton; rămâne și opțiunea „PO nou neplanificat". Biblioteca de citire Excel (SheetJS) se încarcă de pe cdnjs doar când e nevoie, deci dispozitivul care importă are nevoie de internet.

**Roluri – rezumat drepturi.** Operator: pornește PO-uri (planificate sau nu), face verificări/poze, închide PO la țintă, își schimbă PIN-ul. Supervizor: în plus vede Supervizor/Arhivă/Planificare, trimite mesaje operatorilor, planifică PO-uri. Admin: în plus gestionează utilizatorii, poate închide forțat sau șterge PO-uri active și poate șterge PO-uri închise din arhivă (împreună cu pozele lor).

**Export CSV.** Fișierele folosesc separatorul `;` și BOM UTF‑8, deci se deschid corect cu dublu-click în Excel cu setări regionale românești (diacriticele se păstrează).

**Cod de bare din poză.** Completarea automată a câmpului „Articol verificat” din poză folosește API-ul `BarcodeDetector` al browserului – disponibil pe Android (Chrome) și pe iOS/Safari 17+. Pe alte browsere câmpul se completează manual; poza se salvează oricum.
