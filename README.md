# fünftersatz — Technische Dokumentation

> Tischtennis Mental-Coaching App · Live: [rekoproducer.github.io/fuenftersatz](https://rekoproducer.github.io/fuenftersatz/)

---

## 📍 Wo ist was?

| Was | Wo | Link |
|-----|----|------|
| **App (live)** | GitHub Pages | https://rekoproducer.github.io/fuenftersatz/ |
| **Code** | GitHub Repo | https://github.com/rekoproducer/fuenftersatz |
| **Einzige Code-Datei** | `index.html` im Repo | Gesamte App in einer Datei (~75 KB) |
| **KI-Proxy** | Cloudflare Workers | `fuenftersatz-proxy.rekoproducer.workers.dev` |
| **API-Key (sicher)** | Cloudflare Dashboard | Workers & Pages → fuenftersatz-proxy → Settings → Variables |

---

## 🏗️ Architektur

```
Nutzer (Browser / Handy)
         │
         ▼  lädt Seite
GitHub Pages  ←──  index.html  (gesamte App)
         │
         ▼  HTTPS POST (Chat-Nachricht)
Cloudflare Workers Proxy
         │
         ▼  API-Call mit geheimem Key
Anthropic Claude API  (claude-sonnet-4-20250514)
         │
         ▼  Antwort zurück
Nutzer sieht Mentor-Antwort
```

**Warum der Umweg über Cloudflare?**  
Der Anthropic API-Key darf **nie** direkt im öffentlichen GitHub-Code stehen — GitHub scannt das automatisch und deaktiviert sofort exponierte Keys. Der Cloudflare Worker hält den Key sicher als verschlüsseltes Secret und leitet alle Anfragen durch. Kostenlos, dauerhaft.

---

## 📦 Frühere Deployments (alle abgelöst)

| Platform | Status | Warum weg |
|----------|--------|-----------|
| Direkter Browser → Anthropic API | ❌ geht nicht | Browser blockiert das aus Sicherheitsgrønden (CORS) |
| **Netlify** | ❌ aufgegeben | API-Key war kurz im Code sichtbar → Sicherheitspause durch Anthropic |
| **Replit** | ❌ aufgegeben | Schläft ein, unzuverlässig, teuer für dauerhaften Betrieb |
| **GitHub Pages + Cloudflare Proxy** | ✅ **aktuell** | Kostenlos, zuverlässig, sicher |

---

## 🔧 Wie man Änderungen deployed

Die App besteht aus **genau einer Datei**: `index.html`.

### Methode A: Direkt auf GitHub (einfachste Variante)

1. Öffne https://github.com/rekoproducer/fuenftersatz/blob/main/index.html
2. Klick auf das **Stift-Icon** (Edit this file)
3. Änderungen machen
4. Unten auf **"Commit changes"** klicken
5. ✅ Fertig — GitHub Pages deployed automatisch (dauert ~1–2 Minuten)

### Methode B: Über die GitHub API (für Claude / automatisierte Fixes)

Wird genutzt, wenn Claude direkt Änderungen pusht. Ablauf:

**Schritt 1 — Token erstellen**  
→ https://github.com/settings/tokens/new  
- Note: z.B. `fuenftersatz-fix`
- Scope: nur **`repo`** ankreuzen
- Ablauf: 30 Tage reicht

**Schritt 2 — Aktuellen SHA holen** *(muss bei jedem Push neu geholt werden!)*
```
GET https://api.github.com/repos/rekoproducer/fuenftersatz/contents/index.html
→ Feld: sha
```

**Schritt 3 — Datei pushen**
```
PUT https://api.github.com/repos/rekoproducer/fuenftersatz/contents/index.html
Authorization: token ghp_...
Body: { message: "...", content: "<base64>", sha: "<aktueller-sha>" }
```

**Schritt 4 — Token sofort löschen**  
→ https://github.com/settings/tokens

> ⚠️ **Wichtig — UTF-8/Umlaute korrekt kodieren!**
> Einfaches `btoa(code)` korrumpiert Umlaute (ä, ö, ü). Richtige Methode:
> ```javascript
> // Lesen (Base64 → UTF-8 String):
> const binary = atob(base64content.replace(/\n/g, ''));
> const bytes = new Uint8Array(binary.length);
> for (let i = 0; i < binary.length; i++) bytes[i] = binary.charCodeAt(i);
> const code = new TextDecoder('utf-8').decode(bytes);
>
> // Schreiben (UTF-8 String → Base64):
> const encBytes = new TextEncoder().encode(code);
> let binaryStr = '';
> for (let i = 0; i < encBytes.length; i += 8192)
>   binaryStr += String.fromCharCode(...encBytes.subarray(i, i + 8192));
> const base64 = btoa(binaryStr);
> ```

---

## 🤖 KI-Mentor — Details

### Cloudflare Worker (Proxy)

- **URL:** `https://fuenftersatz-proxy.rekoproducer.workers.dev`
- **Was er macht:** Nimmt POST-Anfragen von der App entgegen, fügt den geheimen API-Key ein, leitet weiter an `api.anthropic.com/v1/messages`, gibt Antwort zurück
- **Bearbeiten:** https://dash.cloudflare.com → Workers & Pages → fuenftersatz-proxy

### Aktuelles Modell
```
claude-sonnet-4-20250514
```

### System-Prompt
Wird dynamisch aus dem Spielerprofil aufgebaut (Funktion `buildSystemPrompt()` in `index.html`). Enthält: Spielername, TTR, Spielstil, Material, aktuelles Ziel, Meilensteine.

---

## 💾 Wo werden Nutzerdaten gespeichert?

**Alles lokal im Browser** (`localStorage`) — kein Backend, keine Datenbank, kein Login.

| Key | Inhalt |
|-----|--------|
| `playerProfile` | Name, TTR, Spielstil, Material, Ziel |
| `chatHistory` | Gesprächsverlauf mit dem Mentor |
| `sessionLog` | Vergangene Sessions mit Datum |
| `milestones` | Erreichte Meilensteine |

Konsequenz: Löscht ein Nutzer den Browser-Cache, sind alle Daten weg. Kein Sync zwischen Geräten.

---

## 🎨 Design-System

| Element | Wert |
|---------|------|
| Akzentfarbe | `#3dffc0` (Mintgrün) |
| Hintergrund | `#0a0a0a` (Fast Schwarz) |
| Font | DM Sans (Google Fonts) |
| Hero | Halftone-Kugel-Animation (`<canvas>`), Ball rechts — Text links |
| Splash Screen | 2,2 Sek. Fade-out beim App-Start |

---

## 🐛 Bekannte offene Punkte

| Problem | Status | Was zu tun ist |
|---------|--------|----------------|
| Chrome-Warnung „Site hacked" | ⚠️ offen | Review bei Google Search Console beantragen: https://search.google.com/search-console (Ursache: früher geleakter, inzwischen deaktivierter API-Key) |

---

## 👥 Beta-Test-Gruppe

- Mitglieder aus Fabis Tischtennis-Verein
- Outdoor-Hobby-WhatsApp-Gruppe
- Zielgröße: 6–10 Tester

---

## 🔑 Alle wichtigen Links auf einen Blick

| Was | Link |
|-----|------|
| App (live) | https://rekoproducer.github.io/fuenftersatz/ |
| GitHub Repo | https://github.com/rekoproducer/fuenftersatz |
| Cloudflare Dashboard | https://dash.cloudflare.com |
| GitHub Tokens verwalten | https://github.com/settings/tokens |
| Google Search Console | https://search.google.com/search-console |
