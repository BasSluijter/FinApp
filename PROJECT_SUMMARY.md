# Financieel Overzicht - Project Samenvatting

## 📋 Projectoverzicht

Een Progressive Web App (PWA) voor het bijhouden van persoonlijke financiën, gebouwd met React en Supabase. Multi-user support met household-gebaseerde data isolatie.

**Laatste Update:** 23 februari 2026

---

## 🗂️ Bestandsstructuur

```
project/
├── index.html              # Hoofdapplicatie (React + Babel)
├── config.js              # Supabase credentials (NIET committen!)
├── config.example.js      # Template voor config.js
├── manifest.json          # PWA manifest
├── service-worker.js      # Service worker voor offline functionaliteit
├── .gitignore            # Git ignore (config.js staat hierin)
├── QUICK_SETUP.md        # Setup instructies
├── SUPABASE_SETUP.md     # Database setup
└── EXPORT_IMPORT_GUIDE.md # Data migratie guide
```

---

## 🚀 Tech Stack

**Frontend:**
- React 18 (CDN-based, geen build step)
- TailwindCSS (CDN)
- Babel Standalone (JSX transpiler)
- HTML5 Drag & Drop API

**Backend:**
- Supabase (PostgreSQL database)
- Row Level Security (RLS)
- Real-time subscriptions mogelijk

**PWA:**
- Service Worker voor offline support
- Manifest voor installatie op mobiel
- Cache-first strategie

---

## 🎯 Features

### ✅ Geïmplementeerd

**Financiële Tracking:**
- Inkomsten en uitgaven bijhouden
- 3 frequenties: eenmalig, maandelijks, specifieke maanden
- Maandoverzicht met navigatie (vorige/volgende maand)
- Items handmatig afvinken per maand
- Drag & drop om volgorde aan te passen (desktop)
- Prognose met aanpasbare datumbereik

**Multi-User & Households:**
- Authenticatie (email + wachtwoord)
- Household systeem voor gedeelde data
- Invite codes voor anderen toevoegen
- Bestaande gebruikers kunnen joinen via code
- Bij registratie direct invite code invoeren

**Account Management:**
- Hamburger menu (mobiel-vriendelijk)
- Account instellingen scherm
- Display naam instellen
- Household naam wijzigen
- Invite code genereren/kopiëren
- Overzicht van household leden
- Uitloggen

**UI/UX:**
- Light/dark mode (blijft bewaard na refresh)
- Responsive design
- Mobiel-geoptimaliseerd
- Smooth animaties
- Kleurcodering (groen=inkomsten, rood=uitgaven)

**Data Management:**
- Export naar JSON bestand
- Import van JSON bestand
- Automatische localStorage → Supabase migratie
- Real-time database sync

**PWA:**
- Installeerbaar op mobiel (Add to Home Screen)
- Offline functionaliteit
- App-achtige ervaring

---

## 🗄️ Database Schema

### Tables

**households**
```sql
id: UUID (PK)
name: TEXT
invite_code: TEXT (UNIQUE)
created_at: TIMESTAMP
```

**profiles**
```sql
id: UUID (PK, FK → auth.users)
household_id: UUID (FK → households)
email: TEXT
full_name: TEXT
display_name: TEXT
created_at: TIMESTAMP
```

**items**
```sql
id: BIGSERIAL (PK)
household_id: UUID (FK)
name: TEXT
amount: DECIMAL(10,2)
type: TEXT ('inkomsten' | 'uitgaven')
frequency: TEXT ('eenmalig' | 'maandelijks' | 'specifiek')
month: INTEGER (0-11)
year: INTEGER
selected_months: INTEGER[]
created_at: TIMESTAMP
```

**checked_items**
```sql
id: BIGSERIAL (PK)
household_id: UUID (FK)
item_id: BIGINT (FK → items)
year: INTEGER
month: INTEGER (0-11)
checked: BOOLEAN
created_at: TIMESTAMP
UNIQUE(household_id, item_id, year, month)
```

**sort_orders**
```sql
id: BIGSERIAL (PK)
household_id: UUID (FK)
year: INTEGER (NULL voor default)
month: INTEGER (NULL voor default)
item_order: BIGINT[]
created_at: TIMESTAMP
UNIQUE(household_id, year, month)
```

**user_settings**
```sql
id: UUID (PK, FK → auth.users)
dark_mode: BOOLEAN
created_at: TIMESTAMP
```

### Row Level Security (RLS)

Alle tables hebben RLS enabled met policies:
- Users kunnen alleen data van hun eigen household zien
- Users kunnen alleen hun eigen settings zien/bewerken
- Automatisch household aanmaken bij nieuwe user

---

## 🔧 Configuratie

### Supabase Setup

1. **Project aanmaken** op supabase.com
2. **Database schema** uitvoeren (zie SUPABASE_SETUP.md)
3. **Credentials ophalen:**
   - Project URL: `https://xxxxx.supabase.co`
   - Anon key: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

### Config.js Setup

```javascript
// config.js
window.SUPABASE_CONFIG = {
    url: 'https://jouwproject.supabase.co',
    anonKey: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
};
```

**Belangrijk:** 
- `config.js` staat in `.gitignore`
- Gebruik `config.example.js` als template
- Elke developer maakt eigen `config.js`

---

## 📱 Deployment

### Netlify (Aanbevolen)

1. Upload bestanden:
   - `index.html`
   - `config.js` (met jouw credentials!)
   - `manifest.json`
   - `service-worker.js`

2. Netlify detecteert automatisch `index.html`

3. Site is live op: `https://jouw-site.netlify.app`

### Alternatieve Hosting

- **Vercel:** Zelfde proces als Netlify
- **GitHub Pages:** Zet bestanden in repo root
- **Firebase Hosting:** `firebase deploy`

---

## 👥 Multi-User Workflow

### Optie 1: Gedeeld Account
1. Eén persoon registreert
2. Beide gebruikers loggen in met zelfde email/wachtwoord
3. Data wordt automatisch gedeeld

### Optie 2: Aparte Accounts met Invite Code
1. **Persoon A registreert:**
   - Krijgt automatisch household met invite code
2. **Persoon B registreert:**
   - Voert invite code in bij registratie
   - OF voert code later in via Account Settings
3. Beide personen zien dezelfde data

### Invite Code Genereren
- Menu → Account Instellingen → "Genereer Invite Code"
- Code kopiëren en delen
- Anderen kunnen code invoeren bij registratie of in Account Settings

---

## 🐛 Bekende Issues & Beperkingen

### ⚠️ Huidige Beperkingen

1. **Drag & Drop alleen op Desktop**
   - HTML5 drag & drop werkt niet op touch devices
   - Op mobiel: volgorde niet aanpasbaar
   - *Mogelijke oplossing:* SortableJS library toevoegen

2. **localStorage gebruikt voor dark mode cache**
   - Nodig voor instant load zonder flicker
   - Extra dependency naast Supabase

3. **Geen real-time updates**
   - Wijzigingen van andere gebruikers niet live zichtbaar
   - Refresh nodig om updates te zien
   - *Mogelijk:* Supabase subscriptions toevoegen

4. **Config.js moet handmatig worden ingevuld**
   - Kan niet via environment variables in pure HTML
   - *Mogelijk:* Build step toevoegen

### ✅ Opgeloste Issues

- ~~Dark mode reset na refresh~~ → localStorage cache toegevoegd
- ~~Drag & drop werkt niet~~ → Werkt nu op desktop
- ~~Sort order blijft niet bewaard~~ → Database save gefixt
- ~~Wit scherm bij verkeerde config~~ → Duidelijke foutmelding
- ~~PWA installatie failed~~ → manifest.json start_url gefixt

---

## 🔮 Toekomstige Features (Ideeën)

### Prioriteit Hoog
- [ ] Mobiele drag & drop support (SortableJS)
- [ ] Real-time sync tussen gebruikers
- [ ] Notificaties voor onbetaalde items
- [ ] Zoek/filter functionaliteit

### Prioriteit Middel
- [ ] Categorieën/tags voor items
- [ ] Grafieken en statistieken
- [ ] Budget doelen per categorie
- [ ] Recurring items auto-afvinken
- [ ] Export naar CSV/Excel
- [ ] Meerdere households per user

### Prioriteit Laag
- [ ] Email notificaties
- [ ] Herinneringen voor betalingen
- [ ] Rapportages (maandelijks/jaarlijks)
- [ ] API voor externe integraties
- [ ] Admin dashboard
- [ ] Audit log (wie wijzigde wat)

---

## 🛠️ Development Notes

### Code Structuur

**Hoofdcomponenten:**
- `FinanceTracker` - Main app component
- `AuthScreen` - Login/registratie
- `AccountScreen` - Account instellingen
- `ItemRow` - Individueel transaction item
- `ItemModal` - Add/edit item form
- `MigrationModal` - localStorage import

**State Management:**
- React useState voor lokale state
- useEffect voor data loading
- useRef voor drag & drop tracking

**Data Flow:**
1. User login → Fetch household_id
2. Load items, checked_items, sort_orders voor household
3. Wijzigingen → Direct naar Supabase
4. Reload → Data komt uit Supabase

### Performance Optimizations

- Dark mode cached in localStorage voor instant load
- Service worker cachet CDN resources
- Minimale re-renders door specifieke state updates
- Database queries gefilterd op household_id

### Security

- Row Level Security (RLS) op alle tables
- Household isolation op database niveau
- Anon key is veilig voor frontend gebruik
- Service role key NOOIT in frontend

---

## 📝 Maintenance Checklist

### Bij Updates

- [ ] Test op desktop (Chrome, Firefox, Safari)
- [ ] Test op mobiel (Android Chrome, iOS Safari)
- [ ] Check dark mode functionaliteit
- [ ] Test drag & drop (desktop)
- [ ] Test account switching
- [ ] Verifieer data persistence na refresh
- [ ] Check PWA installatie

### Bij Database Changes

- [ ] Update SUPABASE_SETUP.md
- [ ] Test migratie van oude data
- [ ] Update RLS policies indien nodig
- [ ] Test met meerdere households

### Bij Deploy

- [ ] Update service-worker.js versie
- [ ] Clear browser cache
- [ ] Test op schone browser (incognito)
- [ ] Verifieer manifest.json werkt

---

## 🆘 Troubleshooting

### "Page not found" bij PWA installatie
**Oplossing:** Check `manifest.json` start_url → moet `./index.html` zijn

### Wit scherm bij laden
**Oplossing:** 
1. Check of `config.js` bestaat
2. Verifieer Supabase credentials
3. Open console (F12) voor errors

### Dark mode reset na refresh
**Oplossing:** Oude versie - gebruik nieuwste `index.html` met localStorage cache

### Drag & drop werkt niet
**Desktop:** Moet werken - check console voor errors
**Mobiel:** Niet ondersteund (bekende limitatie)

### Data synchroniseert niet tussen gebruikers
**Check:**
1. Beide users in zelfde household?
2. Refresh pagina (geen real-time sync)
3. Check Supabase Dashboard → data komt wel aan?

### Import/Export werkt niet
**Check:**
1. Bestand is geldig JSON
2. Heeft `items` array
3. Browser ondersteunt File API

---

## 📞 Support & Contact

**Voor vragen tijdens development:**
- Check deze documentatie eerst
- Console errors (F12) geven vaak hints
- Supabase Dashboard voor database issues

**Handige Links:**
- Supabase Docs: https://supabase.com/docs
- React Docs: https://react.dev
- TailwindCSS: https://tailwindcss.com

---

## 📜 Changelog

### v2.0 (23 Feb 2026) - Current
- ✅ Multi-user support met households
- ✅ Account scherm met invite codes
- ✅ Hamburger menu
- ✅ Dark mode persistence (localStorage cache)
- ✅ Config.js systeem
- ✅ Database save voor drag & drop
- ✅ PWA manifest fix voor installatie
- ✅ Betere error handling

### v1.0 (Feb 2026) - Initial
- ✅ Basic finance tracking
- ✅ localStorage data storage
- ✅ Dark/light mode
- ✅ Drag & drop sorting (desktop)
- ✅ PWA functionality
- ✅ Import/export features

---

## 🎓 Lessons Learned

### Wat Werkt Goed
- CDN-based React = geen build complexity
- Supabase RLS = veilige multi-tenancy
- localStorage cache = betere UX
- Config.js = makkelijk te onderhouden

### Wat Beter Kan
- HTML5 drag & drop ≠ mobiel-vriendelijk
- Babel transpiler = tragere load time
- Geen TypeScript = runtime errors
- Geen build step = moeilijker optimaliseren

### Als We Opnieuw Beginnen
- Overweeg: Next.js of Vite voor build
- Gebruik: SortableJS vanaf begin
- Toevoegen: TypeScript voor type safety
- Implementeer: Real-time sync vanaf v1

---

## 📄 License

Privé project - geen publieke licentie

---

**Laatste update:** 23 februari 2026
**Versie:** 2.0
**Status:** ✅ Productie-ready voor persoonlijk gebruik
