# BladeRealms

> Ein fernöstlich inspiriertes Action-MMO mit starkem OpenPvP-System, Boni-basiertem Equipment und einem Early-Release-Modell für schnelle Markttests.

[![Unity](https://img.shields.io/badge/Unity-2022+-black.svg)](https://unity.com/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-MVP%20Development-yellow.svg)]()

## 📋 Inhaltsverzeichnis

- [Projektvision](#-projektvision)
- [Kernfeatures](#-kernfeatures-phase-1)
- [Architektur](#️-architektur)
- [Tech Stack](#-tech-stack--tools)
- [Getting Started](#-getting-started)
- [Roadmap](#-roadmap)
- [Erfolgskriterien](#-erfolgskriterien)
- [Contributing](#-contributing)

## 🎯 Projektvision

BladeRealms ist ein budgetfreundliches, modulares und skalierbares Action-MMO mit Fokus auf:
- **OpenPvP-System** mit fairem Balancing
- **Boni-basiertes Equipment-System** für tiefe Charakterprogression
- **Schnelle Iteration** durch Early-Release-Modell

### Grundprinzipien

#### ⚡ Minimaler Scope
Fokus auf Kernfeatures: PvP-System, Boni-Mechanik und Basis-PvE für Progression. Keine Feature-Bloat, sondern gezielter Aufbau eines funktionierenden Kern-Gameplays.

#### 💰 Kostenoptimierung
- **Assets**: Open-Source und lizenzfreie Asset-Packs (Unity Asset Store, itch.io, Kenney, Quaternius)
- **Engine**: Unity (kostenlos für <100k Umsatz/Jahr)
- **Hosting**: Azure Container Instances (Dev + Prod) – Production-Ready von Anfang an
- **IaC**: Bicep für konsistente Umgebungen (Dev = Prod)
- **Networking**: Open-Source-Lösungen oder Free-Tier-Services

#### 🚀 Early-Release-Strategie
Schnell eine testbare Version erstellen, Community-Feedback einholen und bei positiver Resonanz monetarisieren. Closed Alpha → Early Access → Full Release.

#### ⚖️ Rechtssicherheit
Keine Verwendung von Metin2 oder anderen geschützten Assets. Eigene Namen, eigene Artworks, eigenes Branding zur Vermeidung von rechtlichen Problemen.

## ✨ Kernfeatures (Phase 1)

### OpenPvP-System
- ⚔️ PvP aktiv ab **Level 10**
- 💎 **Kill-Reward**: Chance auf seltene Materialien
- 🛡️ **Anti-Spawnkill**: 30 Sekunden Schutz nach Respawn

### Combat-System
- ⚔️ **Standard-Schlag-Angriff**: Basisangriff mit Waffe für Grundschaden (ohne Verzögerung)
- 🎭 **3 Grundklassen**: Krieger, Magier, Assassine
- ✨ **6 Skills** pro Klasse
- 🎯 **Ausweich-Mechanik** für dynamische Kämpfe

### PvE-Grundlage
- 🗺️ **Startgebiet** zum Leveln (Level 1-20)
- 🌲 **Midlevel-Gebiet** für Progression (Level 20-35)
- 🏰 **1 Dungeon** für PvP-Gear-Materialien (Level 25+)
- 👹 **Mini-Bosse** auf allen Maps (EP + seltene Crafting-Materialien)
- 💎 **Metinstein-ähnliche Objekte** (seltene Spawns, Boni-Materialien)

### Equipment & Boni-System
- 🎲 Items mit **3–5 zufälligen Boni** (z.B. +Crit, +PvP-Damage)
- 🔄 **Re-Roll-System** mit Ingame-Währung
- ⚔️ **PvP-Gear**: Craftbar aus Dungeon-Materialien (+10–15% PvP-Damage)

### Monetarisierung (Pay2Win-Light)
- ⏰ Temporäre **PvP-Buffs** (+3–5% für 30 Tage, keine „One-Shot"-Items)
- 🎨 **Kosmetik-Shop** (Skins, Mounts)
- 🎁 **Founders Pack** (Skins, Titel, kleine PvP-Buffs)
- 🏆 **Battle-Pass Light** (nur kosmetisch, optional kleine PvP-Boni)

**Transparenz**: Nur leichte Vorteile, kein Pay-to-Win-Paywall

### Progression
- 📈 Leveln durch **PvE-Mobs + Quests**
- 📊 **EXP-Skalierung**: Mit jedem Level steigt der benötigte EXP-Betrag (+8–12% pro Level)
- 📉 **Dropraten-Skalierung**: Droprate sinkt in niedrigstufigen Gebieten (-2% pro Level über dem Gebiet)
- 🔨 **Crafting** für PvP-Gear aus Dungeon-Materialien
- ⚔️ Alle PvE-Aktivitäten geben weiterhin EXP, unabhängig vom Level

## 🏗️ Architektur

```
[Client] Unity (C#)
   |
   | Mirror/Fish-Networking oder Photon Fusion
   |
[Game Server] Docker Container (Unity Headless Linux)
   - Autoritativer Server für Combat & PvP
   - Instanz-Manager für Dungeon
   - Anti-Cheat: Movement-Checks, Skill-Cooldown-Validation
   - Deployment: Docker Compose (Dev/Test) → Azure Container Instances (Prod)
   |
[Backend Services]
   - PlayFab (Auth, Inventar, Shop)
   - Firebase (Realtime DB für Chat)
   |
[Database]
   - PlayFab integrierte Persistenz
   |
[Content Delivery]
   - Unity Addressables + CDN (Cloudflare Free Tier)
```

### Containerisierung & Skalierung

**Unified Azure-Strategie (Dev + Prod):**
- Identische Infrastruktur für Development und Production
- Azure Container Instances für alle Umgebungen
- Headless Unity Server als Docker Image in Azure Container Registry
- Infrastructure as Code mit Bicep (keine Drift zwischen Umgebungen)

**Development:**
- Azure Container Instances (Dev-Resourcegroup)
- Reduzierte Container-Specs (CPU/RAM) für Kostenoptimierung
- Bicep-Parameter für Dev-Konfiguration

**Production:**
- Azure Container Instances (Prod-Resourcegroup)
- Produktive Container-Specs
- Azure Container Apps für Auto-Scaling bei >50 CCU
- Load Balancing über Azure Application Gateway

**Vorteile:**
- Keine Environment-Drift
- Production-Ready Releases von Tag 1
- Konsistente Deployments über Bicep
- Einfaches Scaling: Bicep-Parameter anpassen

## 🛠 Tech Stack & Tools

### Engine & Networking
- **Engine**: Unity 2022+ (kostenlos für <100k Umsatz/Jahr, C#, große Community)
- **Networking**: 
  - Mirror oder Fish-Networking (Open Source)
  - Photon Fusion (kostenlos bis 20 CCU, später skalierbar) – einfacher Start für PvP

### Backend & Services
- **Backend**: PlayFab (kostenlos bis 100k MAU) oder Firebase für Auth, Inventar, Shop
- **Anti-Cheat**: Server-seitige Validation (Movement-Checks, Skill-Cooldown-Validation)

### Hosting & Infrastructure
- **Containerisierung**: Docker für Unity Headless Server
- **Development**: Azure Container Instances (Dev-Umgebung)
- **Production**: Azure Container Instances / Azure Container Apps
- **IaC**: Bicep (exklusiv) für alle Azure-Ressourcen
- **Scripting**: PowerShell 7+ mit Az-Module (KEINE Azure CLI)
- **Container Registry**: Azure Container Registry (Dev + Prod)
- **Resource Groups**: Separate RGs für Dev/Prod mit identischer Struktur
- **Orchestrierung**: Azure Container Apps für Auto-Scaling
- **Domain + SSL**: ca. 10–15 €/Jahr
- **CDN**: Cloudflare Free Tier

### Assets & Content
- **3D Assets**: Unity Asset Store (Free Packs), itch.io
- **Stilisierte Optik**: Quaternius, Kenney (einfacher zu produzieren, Performance-freundlich)
- **Premium-Packs**: Optional 2–3 Premium-Packs (je 20–50 €)

### Development Tools
- **Versionierung**: GitHub
- **CI/CD**: GitHub Actions (Build → Docker Image → Bicep Deploy → Azure)
- **Containerisierung**: Docker (lokal), Azure Container Registry
- **IaC**: Bicep (exklusiv) – keine Terraform, keine ARM-Templates
- **Scripting**: PowerShell 7+ mit Az PowerShell Module
- **Deployment**: Bicep-Modules für wiederverwendbare Infrastruktur
- **Projektmanagement**: Trello oder Notion
- **Testing**: Discord-Server für Closed Alpha

## 🚀 Getting Started

### Prerequisites
- Unity 2022.3 LTS oder höher
- Visual Studio 2022 / Rider
- Git

### Sprint 0 Setup (2 Wochen)

1. ✅ Setup Unity + Networking (Mirror oder Photon)
2. ✅ Greybox Map (Startgebiet + Dungeon)
3. ✅ Combat-Prototyp (Standard-Schlag-Angriff + 3 Klassen, Basis-Skills)
4. ✅ Inventar + Boni-System rudimentär
5. ✅ Login & Account via PlayFab
6. ✅ Closed-Test mit 10–20 Spielern (Discord)

## 🗓 Roadmap & Development Epics

### 📦 Phase 1: MVP Foundation (4–6 Wochen) | Budget: ~100–200 €

#### Epic 1: Core Combat & PvP 🎮
**Zeitaufwand**: 2–3 Wochen | **Priorität**: KRITISCH  
**Story**: Als Spieler möchte ich kämpfen können, um PvP und PvE zu erleben.

**Tasks:**
- [ ] Implementiere Movement & Kamera (Unity Character Controller)
- [ ] Implementiere Standard-Schlag-Angriff (Basisangriff mit Waffe, keine Verzögerung)
- [ ] Implementiere Basis-Skills für 3 Klassen (Krieger, Magier, Assassine)
- [ ] PvP aktivieren ab Level 10 mit Anti-Spawnkill-Mechanik
- [ ] Ausweich-Mechanik für dynamische Kämpfe

**Akzeptanzkriterien Standard-Schlag:**
- Muss ohne Verzögerung funktionieren
- Schaden basiert auf Waffe + Basiswerte
- Responsive Input-Handling

**Assets benötigt:**
- ✅ Free Character Models (Unity Asset Store, Mixamo)
- ✅ Free Animation Packs (Mixamo, Quaternius)

**Risiko**: Netzwerk-Synchronisation bei PvP → Lösung: Mirror/Photon mit autoritativem Server

---

#### Epic 2: Boni-System & Inventar 🎲
**Zeitaufwand**: 1–2 Wochen | **Priorität**: HOCH  
**Story**: Als Spieler möchte ich Items mit zufälligen Boni erhalten.

**Tasks:**
- [ ] Inventar-UI erstellen (Unity UI Toolkit)
- [ ] Boni-Generator für Items (3–5 zufällige Stats: +Crit, +PvP-Damage, +Resistenz)
- [ ] Re-Roll-Funktion mit Ingame-Währung implementieren
- [ ] Item-Tooltip-System mit Boni-Anzeige

**Assets benötigt:**
- ✅ Free UI Icons (Kenney, Game-Icons.net)

**Risiko**: Performance bei vielen Items → Lösung: Object Pooling, effiziente Datenstrukturen

---

#### Epic 4: Mini-Bosse & Metinstein-Mechanik 👹
**Zeitaufwand**: 1–2 Wochen | **Priorität**: MITTEL  
**Story**: Als Spieler möchte ich besondere Herausforderungen auf Maps haben.

**Tasks:**
- [ ] Mini-Boss-Spawn-Logik (alle 30–45 Minuten)
- [ ] Loot-Tabelle für seltene Materialien (mindestens 1 seltenes + 2–3 Standard-Materialien)
- [ ] Metinstein-ähnliche Objekte mit Boni-Drops (1% Spawn-Chance pro Stunde)
- [ ] Erhöhte Schwierigkeit, Gruppen-Content empfohlen
- [ ] Visual Feedback für seltene Spawns

**Assets benötigt:**
- ✅ Free Boss Models (Unity Asset Store)
- ✅ Free Particle Effects (Quaternius)

**Akzeptanzkriterien:**
- Mini-Bosse spawnen alle 30–45 Minuten
- Droppen mindestens 1 seltenes Material
- Metinsteine: 1% Spawn-Chance, erfordern 2+ Spieler

**Risiko**: Camping durch Spieler → Lösung: Randomisierte Spawn-Punkte, Anti-Farm-Mechanik

---

#### Epic 5: EXP- und Dropraten-Skalierung 📊
**Zeitaufwand**: 1 Woche | **Priorität**: HOCH  
**Story**: Als Spieler möchte ich eine faire Progression erleben.

**Tasks:**
- [ ] EXP-Kurve implementieren (langsameres Leveln bei höheren Levels, +8–12% pro Level)
- [ ] Dropraten-Skalierung für niedrigstufige Gebiete (-2% pro Level über dem Gebiet)
- [ ] Spieler-Level vs. Gebiets-Level Berechnung
- [ ] UI-Feedback für reduzierte Drops (Tooltip-Warnung)

**Akzeptanzkriterien:**
- EXP-Anstieg mindestens +8–12% pro Level
- Bei Level 50 in Level-10-Gebiet: <10% der ursprünglichen Droprate
- Alle PvE-Aktivitäten geben weiterhin EXP

**Risiko**: Grinding zu frustrierend → Lösung: Daily-Quests, Events mit Bonus-EXP

---

#### Epic 6: Backend & Auth 🔐
**Zeitaufwand**: 1 Woche | **Priorität**: HOCH  
**Story**: Als Spieler möchte ich mich einloggen und meine Daten speichern können.

**Tasks:**
- [ ] PlayFab-Integration für Authentication (Email/Password)
- [ ] Account-Daten speichern (Inventar, Level, Skills)
- [ ] Cloud-Save-System für Charaktere
- [ ] Leaderboard-System für PvP-Rankings

**Kosten:**
- ✅ PlayFab: Kostenlos bis 100k MAU

**Risiko**: Datenverlust → Lösung: Regelmäßige Backups, PlayFab Redundanz

---

#### Epic 7: Test & Feedback 🧪
**Zeitaufwand**: 1 Woche (parallel) | **Priorität**: MITTEL  
**Story**: Als Entwickler möchte ich früh Feedback sammeln.

**Tasks:**
- [ ] Discord-Server aufsetzen (Support, Bug-Reports, Feedback)
- [ ] Closed Alpha mit 10–20 Spielern (Friends & Family)
- [ ] Feedback-Formular (Google Forms)
- [ ] Bug-Tracking-System (GitHub Issues)

**Kosten:**
- ✅ Discord: Kostenlos

---

#### Epic 7.5: Azure Infrastructure & Deployment 🐋
**Zeitaufwand**: 3–5 Tage | **Priorität**: HOCH  
**Story**: Als Entwickler möchte ich Production-Ready Infrastruktur mit Bicep aufbauen.

**Tasks:**
- [ ] Dockerfile für Unity Headless Server erstellen
- [ ] Azure Container Registry Setup (Dev + Prod)
- [ ] Bicep-Module für Container Instances erstellen
- [ ] Bicep-Parameter für Dev/Prod Umgebungen
- [ ] GitHub Actions CI/CD Pipeline (Build → Push ACR → Bicep Deploy)
- [ ] Resource Groups strukturieren (rg-bladerealms-dev, rg-bladerealms-prod)

**Assets benötigt:**
- ✅ Docker Desktop (kostenlos)
- ✅ Azure Container Registry (kostenlos bis 10 GB)
- ✅ Azure CLI + Bicep Extension

**Akzeptanzkriterien:**
- Identische Infrastruktur in Dev und Prod (nur Parameter unterschiedlich)
- Server startet in <30 Sekunden im Container
- Automatisches Deployment bei Git Push (separate Branches für Dev/Prod)
- Rollback über Bicep-Versioning möglich

---

### ⚔️ Phase 2: PvP & Balancing (8–10 Wochen) | Budget: ~90–150 €

#### Epic 8: PvP-Gear & Crafting 🔨
**Zeitaufwand**: 2–3 Wochen | **Priorität**: HOCH  
**Story**: Als Spieler möchte ich PvP-Gear craften können, um stärker zu werden.

**Tasks:**
- [ ] Crafting-UI erstellen (Rezepte, Materialien)
- [ ] Material-Drops aus Dungeon (selten, 10–15% Drop-Rate)
- [ ] PvP-Gear mit speziellen Boni (+10–15% PvP-Damage)
- [ ] Crafting-Animation & Feedback

**Assets benötigt:**
- ✅ Free Crafting UI (Unity Asset Store)
- ⚠️ Optional: Premium Particle Effects (20–30 €)

**Risiko**: Grinding zu langweilig → Lösung: Kurzfristige Belohnungen, Daily-Quests

---

#### Epic 1 (Fortsetzung): PvP-Balancing ⚖️
**Zeitaufwand**: 2–3 Wochen | **Priorität**: KRITISCH  
**Story**: Als Spieler möchte ich faires PvP erleben.

**Tasks:**
- [ ] Damage-Balancing für 3 Klassen
- [ ] PvP-Zonen mit Kill-Rewards (Materialien, Crafting-Items)
- [ ] Anti-Cheat-Validierung (Movement, Skill-Cooldowns)
- [ ] Respawn-System mit Safe-Zone

**Risiko**: Klassen-Unbalance → Lösung: Community-Feedback, Iterationen

---

#### Epic 10: Shop & Monetarisierung 💰
**Zeitaufwand**: 2 Wochen | **Priorität**: MITTEL  
**Story**: Als Spieler möchte ich kosmetische Items und optionale Buffs kaufen können.

**Tasks:**
- [ ] Shop-UI (Kosmetik, PvP-Buffs, Founders Pack)
- [ ] PvP-Buff-Abo implementieren (+3–5% für 30 Tage)
- [ ] Payment-Integration (PlayFab Commerce)
- [ ] Founders Pack (Skins, Titel, kleine Buffs)

**Kosten:**
- ✅ PlayFab Commerce: Kostenlos (5% Transaction-Fee)

**Risiko**: Pay2Win-Kritik → Lösung: Transparente Kommunikation, nur leichte Vorteile

---

#### Epic 11: Midlevel-Content (Level 20-35) 🌲
**Zeitaufwand**: 2–3 Wochen | **Priorität**: HOCH  
**Story**: Als Spieler möchte ich nach Level 20 neue Herausforderungen haben.

**Tasks:**
- [ ] Midlevel-Gebiet erstellen (neue Map, neue Mob-Typen)
- [ ] Quest-Line für Level 20-35 (Story + Progression)
- [ ] Stärkere Mobs mit besseren Drops
- [ ] Transition-Mechanik vom Startgebiet zum Midlevel-Gebiet
- [ ] Optional: Mini-Boss für Gruppe-Content

**Assets benötigt:**
- ✅ Free Environment Assets (Quaternius, Kenney)
- ✅ Free Mob Models (Unity Asset Store)
- ⚠️ Optional: Premium Environment Pack (30–50 €)

**Risiko**: Content-Grind zu monoton → Lösung: Abwechslungsreiche Quests, OpenPvP-Anreize

---

#### Epic 12: Closed Alpha (Erweiterung) 🧪
**Zeitaufwand**: 2 Wochen | **Priorität**: HOCH  
**Story**: Als Entwickler möchte ich mit 50–100 Spielern testen.

**Tasks:**
- [ ] Discord-Community erweitern (Marketing via Reddit, TikTok)
- [ ] Alpha-Keys generieren & verteilen
- [ ] Live-Events: PvP-Turniere mit Belohnungen
- [ ] Iterationen basierend auf Feedback

**Kosten:**
- ✅ Marketing: Nullbudget (Organic Social Media)

---

### 🚀 Phase 3: Early Access | Budget: ~100–200 €

#### Epic 13: Steam-Release 🎮
**Zeitaufwand**: 2–3 Wochen | **Priorität**: KRITISCH  
**Story**: Als Entwickler möchte ich auf Steam launchen.

**Tasks:**
- [ ] Steam-Partnerschaft beantragen
- [ ] Store-Page erstellen (Screenshots, Videos, Beschreibung)
- [ ] Steam-SDK-Integration (Achievements, Cloud-Save)
- [ ] Early-Access-Launch

**Kosten:**
- ⚠️ Steam-Release: 100 € (einmalig)

---

#### Epic 14: Battle-Pass & Events 🏆
**Zeitaufwand**: 2 Wochen | **Priorität**: MITTEL  
**Story**: Als Spieler möchte ich saisonale Belohnungen freischalten.

**Tasks:**
- [ ] Battle-Pass Light (nur kosmetisch + optionale Buffs)
- [ ] Community-Events: PvP-Turniere mit Preisen
- [ ] Seasonal-Content (neue Skins, Mounts)

**Risiko**: Retention-Probleme → Lösung: Schnelle Kämpfe, Rankings, tägliche Belohnungen

---

### 🌟 Langfristig: Skalierung & Content (Post-Launch)

**Neue Features:**
- 🗺️ Neue Maps: Weitere OpenPvP-Zonen
- 🏰 Neue Dungeons: Für erweiterte Progression
- ⚔️ Neue Klassen: Balance und Vielfalt
- 🎨 Premium-Content: Kosmetik, Mounts, Battle-Passes

**Budget**: Abhängig von Early-Access-Einnahmen

## 📊 Erfolgskriterien

| Metrik | Zielwert |
|--------|----------|
| **D1-Retention** | ≥ 35% |
| **Crash-Rate** | < 1% pro Spielstunde |
| **CCU (Closed Alpha)** | ≥ 50 Spieler |
| **PvP-Engagement** | ≥ 60% der Spieler ab Level 10 |
| **Avg. Session Length** | ≥ 45 Minuten |
| **Combat Responsiveness** | Standard-Schlag < 100ms Input-Lag |

## ⚠️ Risiken & Lösungsansätze

| Risiko | Lösung |
|--------|--------|
| **Cheating** | Autoritativer Server, einfache Anti-Cheat-Checks (Movement, Skill-Cooldowns) |
| **Pay2Win-Kritik** | Transparenz, nur kleine Vorteile, kein „One-Shot"-Paywall |
| **Retention-Probleme** | PvP muss Spaß machen → schnelle Kämpfe, Belohnungen, Rankings |
| **Budget-Überschreitung** | Strenger Scope, Free/Open-Source-Assets, günstige VPS-Lösungen |

## 💰 Budget-Strategie

| Kategorie | Kosten |
|-----------|--------|
| **Assets** | Kostenlos + optional 2–3 Premium-Packs (60–150 €) |
| **Azure Dev** | Container Instances (Low-Spec): ~5–10 €/Monat |
| **Azure Prod (MVP)** | Container Instances: ~0,10 €/Stunde pro Container (~15–30 €/Monat) |
| **Azure Prod (Scale)** | Container Apps (Auto-Scale): ~0,10–0,20 €/Stunde pro Container |
| **Container Registry** | Azure: Kostenlos bis 10 GB, dann ~5 €/Monat |
| **Domain + SSL** | 10–15 €/Jahr |
| **Marketing** | Nullbudget → Discord, TikTok, Reddit, Twitch-Micro-Streamer |
| **Steam-Release** | 100 € (einmalig) |
| **Gesamt (MVP)** | ~250–450 € (erste 6 Monate, inkl. Dev + Prod) |
| **Gesamt (Scale)** | ~50–150 €/Monat bei 100+ CCU |

## 🤝 Contributing

Wir suchen engagierte Entwickler und Tester!

1. Fork das Projekt
2. Erstelle einen Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit deine Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push zum Branch (`git push origin feature/AmazingFeature`)
5. Öffne einen Pull Request

### Discord Community
Tritt unserem Discord-Server bei für:
- 🎮 Closed Alpha Testing
- 💬 Feature Diskussionen
- 🐛 Bug Reports
- 📢 Development Updates

## 📝 License

Dieses Projekt ist unter der MIT License lizenziert - siehe [LICENSE](LICENSE) Datei für Details.

## 🎮 Nächster Schritt

**Sprint 0 implementieren** und **Closed Alpha vorbereiten**!

---

**Built with ❤️ for the MMO community**
