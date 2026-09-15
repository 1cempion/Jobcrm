# Schema-Referenz

Vollständige technische Referenz des Notion-Systems. Nützlich, wenn es
nachgebaut, erweitert oder in einen anderen Workspace übertragen werden soll.

Workspace: `Notion von Cem Turan`
Hub-Seite: `🎯 Job CRM — Mission Control`

---

## Datenbanken

| Datenbank | Database ID | Data-Source ID |
|---|---|---|
| 📮 Pipeline | `6f91de92c6f54bdd8522ad199cb8f22b` | `d1c425a2-cd05-45e1-bb8f-ab569af6f803` |
| 🏢 Firmen | `25d0ff379ded4fbdb73f321bd3c8324f` | `35bf81b5-96e7-42d8-9aa9-2a5504b57a67` |
| 👤 Kontakte | `e3102ec5f203495f86610f3290ed8538` | `ce7d3522-b7a1-4df2-bd42-755157c84c93` |
| 🎤 Gespräche | `f9efaae554b748448929318893c79bda` | `214d9228-d981-45f9-b06a-b55f64c5cab5` |
| ⭐ Story-Bank | `2863d740538945798b5a3bf320e84bd6` | `1129dbd0-3e9a-4c08-b2b2-4f72d75ee7a0` |
| ❓ Fragen-Arsenal | `4794848897aa40d6972b86497e55beb5` | `b88ff1cd-1978-4e7c-bf02-34ab64c19703` |
| 🧠 Taktik-Deck | `b014a63ecaa04bb3a587b5ad2176187f` | `04025967-44f5-4967-8d3e-e20e8f61f6eb` |
| 🏭 Branchen-Playbooks | `347c389851934411bdef2a6e4032e347` | `40273941-2b2e-4bb2-90de-c3c549de7052` |
| ⚡ Aktivitäten-Log | `998896a5de5b4b58b3dcb4500d0c7237` | `a98c869a-7eda-4b03-9a71-0aeda30366ac` |

---

## Relationen

Alle Relationen sind zweiseitig (DUAL). Die jeweils erzeugte Gegenseite steht
in Klammern.

```
Pipeline.Firma              → Firmen           (Firmen.Bewerbungen)
Pipeline.Kontakte           → Kontakte         (Kontakte.Bewerbungen)
Pipeline.Branche            → Branchen         (Branchen.Bewerbungen)
Pipeline.Stories im Einsatz → Story-Bank       (Story-Bank.Eingesetzt bei)
Pipeline.Taktiken           → Taktik-Deck      (Taktik-Deck.Eingesetzt bei)
Firmen.Branche              → Branchen         (Branchen.Firmen)
Kontakte.Firma              → Firmen           (Firmen.Menschen)
Gespräche.Bewerbung         → Pipeline         (Pipeline.Gespräche)
Gespräche.Teilnehmer        → Kontakte         (Kontakte.Gespräche)
Gespräche.Fragen die kamen  → Fragen-Arsenal   (Fragen-Arsenal.Kam vor in)
Fragen-Arsenal.Story        → Story-Bank       (Story-Bank.Passt zu Fragen)
Aktivitäten.Bewerbung       → Pipeline         (Pipeline.Aktivitäten)
```

---

## Formeln

### Pipeline

**Match** — Zehn-Segment-Balken aus dem Match-Score
```
if(empty(prop("Match-Score")), "░░░░░░░░░░",
  repeat("█", min(10, max(0, round(prop("Match-Score") / 10))))
  + repeat("░", 10 - min(10, max(0, round(prop("Match-Score") / 10)))))
```

**Tage im Rennen**
```
if(empty(prop("Beworben am")), 0, dateBetween(now(), prop("Beworben am"), "days"))
```

**Follow-up** — Ampel; abgeschlossene Bewerbungen fallen automatisch raus
```
if(empty(prop("Beworben am")), "⚪ noch nicht raus",
 if(prop("Status") == "🎉 Unterschrieben" or prop("Status") == "❌ Absage"
    or prop("Status") == "🧊 Eisfach" or prop("Status") == "🏆 Angebot", "⚪ erledigt",
 if(dateBetween(now(), prop("Beworben am"), "days") >= 14, "🔴 NACHHAKEN",
 if(dateBetween(now(), prop("Beworben am"), "days") >= 7, "🟡 bald dran",
    "🟢 noch ruhig"))))
```

**Countdown** — bezogen auf `Fällig am`
```
if(empty(prop("Fällig am")), "—",
 if(dateBetween(prop("Fällig am"), now(), "days") < 0, "🛑 überfällig",
 if(dateBetween(prop("Fällig am"), now(), "days") == 0, "🔥 HEUTE",
    "in " + format(dateBetween(prop("Fällig am"), now(), "days")) + " Tagen")))
```

**XP** — nach erreichtem Status
```
if(prop("Status") == "🎉 Unterschrieben", 500,
 if(prop("Status") == "🏆 Angebot", 250,
 if(prop("Status") == "💰 Verhandlung", 150,
 if(prop("Status") == "🧪 Case / Probetag", 120,
 if(prop("Status") == "🎤 Interview 2+", 100,
 if(prop("Status") == "🎤 Interview 1", 75,
 if(prop("Status") == "📞 Screening", 50,
 if(prop("Status") == "📤 Beworben", 25,
 if(prop("Status") == "❌ Absage", 10, 0)))))))))
```

**Gehalts-Delta**
```
if(empty(prop("Ihr Angebot")) or empty(prop("Wunschgehalt")), "—",
 if(prop("Ihr Angebot") >= prop("Wunschgehalt"),
   "✅ " + format(prop("Ihr Angebot") - prop("Wunschgehalt")) + " € über Wunsch",
   "🔻 " + format(prop("Wunschgehalt") - prop("Ihr Angebot")) + " € unter Wunsch"))
```

**Anzahl Gespräche** — Rollup über `Gespräche` → Titel, Funktion `count`

### Aktivitäten-Log

**XP** — nach Aktionstyp
```
if(prop("Typ") == "📤 Bewerbung raus", 25,
 if(prop("Typ") == "🔁 Follow-up", 15,
 if(prop("Typ") == "🤝 Networking", 20,
 if(prop("Typ") == "🎤 Gespräch geführt", 75,
 if(prop("Typ") == "🔍 Recherche", 10,
 if(prop("Typ") == "📝 Unterlagen gebaut", 20,
 if(prop("Typ") == "🧠 Taktik geübt", 15,
 if(prop("Typ") == "💪 Skill gelernt", 30,
 if(prop("Typ") == "❌ Absage kassiert", 10, 0)))))))))
```

**Monat** — Gruppierungsachse für die XP-Kurve
```
if(empty(prop("Datum")), "-", formatDate(prop("Datum"), "MMMM YYYY"))
```

---

## Views

| Datenbank | View | Typ | Konfiguration |
|---|---|---|---|
| Pipeline | 🎯 Board | board | `GROUP BY "Status"`, sortiert nach Priorität |
| Pipeline | ⏰ Was ansteht | table | filtert Unterschrieben/Absage/Eisfach raus, sortiert nach `Fällig am` |
| Pipeline | 📅 Kalender | calendar | `CALENDAR BY "Fällig am"` |
| Pipeline | 💰 Am Geld | table | Status in Verhandlung/Angebot/Unterschrieben |
| Pipeline | 📊 Funnel | chart | Column-Chart, `GROUP BY "Status"` |
| Taktik-Deck | 🗂 Nach Kategorie | board | `GROUP BY "Kategorie"` |
| Taktik-Deck | ⏱ Nach Gesprächsphase | board | `GROUP BY "Timing"` |
| Fragen-Arsenal | 🗂 Nach Typ | board | `GROUP BY "Typ"` |
| Fragen-Arsenal | 🚨 Hier wackelt es | table | Sicherheit ≤ Wackelig |
| Story-Bank | 💎 Nach Kompetenz | board | `GROUP BY "Kompetenz"` |
| Kontakte | 🔥 Beziehungsstatus | board | `GROUP BY "Beziehung"` |
| Aktivitäten | 📈 XP-Kurve | chart | Column, `SUM("XP")` über `Monat` |
| Aktivitäten | 📅 Streak-Kalender | calendar | `CALENDAR BY "Datum"` |

---

## Befüllter Inhalt

| Datenbank | Einträge |
|---|---|
| 🧠 Taktik-Deck | 27 Karten über 10 Kategorien |
| ❓ Fragen-Arsenal | 33 Fragen (7 Klassiker, 4 Stress, 4 Lücken, 2 Gehalt, 4 Kultur/Fachlich, 10 eigene Fragen an sie) |
| 🏭 Branchen-Playbooks | 9 Branchen mit vollständigem Playbook |
| ⭐ Story-Bank | 2 fertige Stories + 6 STAR-Vorlagen |

Pipeline, Firmen, Kontakte, Gespräche und Aktivitäten-Log starten leer — das
sind die Datenbanken, die mit der eigenen Suche wachsen.

---

## Personalisierungs-Ebene

Aus zwei vorbereiteten Dokumenten (Positionierung, Zielbild) eingearbeitet.
Bewusst **ohne neue Datenbanken** — nur zwei Referenzseiten plus Befüllung
bestehender Container.

**Zwei Seiten unter dem Hub**

| Seite | Page ID | Inhalt |
|---|---|---|
| 🎙 Meine Positionierung | `3dcb4b9a-b5bd-8183-a850-d291e10f004a` | 30-Sekunden-Einstieg, lange Version, drei Stärken mit Anker-Story, Zahlen-Tabelle, vorbereitete Antworten, offene Baustelle |
| 🎯 Das Zielbild | `3dcb4b9a-b5bd-819e-9ce6-fd1f9bd354ca` | 10 Kriterien aus Geschäftsführer-Sicht, je mit Beleg-Mapping (8/10 belegt), „Was er nicht braucht", abgeleitete Gegenfragen |

**In bestehende Container eingetragen**

- **Story-Bank**: zwei Template-Einträge in echte Stories überschrieben —
  *Headtube Cover* (Werkzeug über zwei Projekte geteilt) und *Der Zielkonflikt*
  (Qualität vs. Preis, Einkauf hatte recht). Beide auf `💎 Poliert`.
- **Fragen-Arsenal**: 5 bestehende Fragen mit eigenen Antworten befüllt und auf
  `🟩🟩🟩🟩⬜ Fast` gesetzt, 2 davon per Relation an die Stories gehängt.
  8 neue Fragen ergänzt (KPIs, Liefertreue, Zeit in der Rolle, Absagen,
  reaktiver Anteil + 3 Gegenfragen aus dem Zielbild).
- **Branchen-Playbooks**: *Industrie & Mittelstand* auf `🎯 Zielbranche`,
  Gehaltsband um Account-Management-Rollen ergänzt, Abschnitt
  „Account Management im Zulieferer-Umfeld" angehängt.

**Offene Lücke, die im System vermerkt ist:** die Performance-Übersicht
(Projektlast, Eskalationen, Durchlaufzeiten). Sie schließt gleichzeitig die
KPI-Frage im Interview und zwei der zehn Zielbild-Kriterien. Absichtlich *nicht*
als weitere Datenbank gebaut — sie gehört in den laufenden Job, nicht in das
Bewerbungs-CRM.

---

## Fallstricke beim Nachbau

- **Select-Optionen dürfen keine Kommas enthalten.** Die Notion-API lehnt sie ab.
- **Emojis in Select-Optionen ohne Variation Selector** (U+FE0F) verwenden, wenn
  sie in Formeln per `==` verglichen werden — sonst greift der Vergleich nicht.
  Statt `✍️` also `📝`.
- **Rollup-Funktion heißt `count`**, nicht `count_all`.
- **Relation-Ziele müssen existieren**, bevor die Relation angelegt wird — daher
  die Reihenfolge: Branchen/Taktik/Story → Firmen/Fragen → Kontakte → Pipeline →
  Gespräche.
