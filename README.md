# habit-checkin-minimal-app

App minimale per tracciare e mantenere abitudini quotidiane con check-in rapido.

## Overview

# Product Requirements Document (PRD)

## 1. Project Overview

**Project Name:** habit-checkin-minimal-app

habit-checkin-minimal-app è un'app mobile minimalista progettata per aiutare gli utenti a costruire e mantenere abitudini quotidiane o settimanali attraverso un'esperienza semplice, veloce e senza distrazioni. L'app risponde al bisogno di monitorare 3–5 abitudini alla volta, offrendo un check-in rapido, una visualizzazione chiara dei progressi e promemoria personalizzabili, evitando la complessità e il sovraccarico di funzionalità delle app concorrenti.

---

## 2. Goals & Success Metrics

### Obiettivi

- Consentire agli utenti di creare, monitorare e mantenere abitudini quotidiane e settimanali in modo semplice e intuitivo.
- Offrire una visualizzazione immediata dei progressi per aumentare la motivazione.
- Ridurre la frizione e la complessità nell'esperienza utente, mantenendo l'app leggera e focalizzata sulle funzionalità essenziali.

### Metriche di Successo

- **% utenti che fanno check-in almeno 5 giorni su 7 nella prima settimana**
- **Retention a 30 giorni**
- **Numero medio di abitudini attive per utente**

---

## 3. Target Users

- Persone che desiderano uno strumento leggero e senza distrazioni per tracciare 3–5 abitudini contemporaneamente.
- Utenti che trovano le app di habit tracking esistenti troppo complesse o ricche di funzionalità non necessarie.
- Persone interessate a costruire abitudini semplici (es. bere acqua, leggere, meditare) con una UX rapida e immediata.

---

## 4. Core Features

### 4.1 Creazione Abitudine

- **Campi richiesti:** nome, icona/emoji, frequenza (giornaliera o settimanale), target_count (per settimanale, es. 3 volte a settimana).
- **UI:** flusso semplice, massimo 2-3 tap per creare una nuova abitudine.
- **Gestione:** possibilità di archiviare abitudini (soft delete).

### 4.2 Check-in Giornaliero

- **Interazione:** tap singolo per segnare "fatto" su una abitudine per il giorno corrente.
- **Vincolo:** un solo check-in per abitudine/giorno.
- **Feedback:** animazione o feedback visivo immediato.

### 4.3 Streak Counter

- **Visualizzazione:** contatore di giorni consecutivi (per abitudini giornaliere) o settimane consecutive raggiunte (per abitudini settimanali).
- **Logica streak settimanale:** una settimana è "raggiunta" se il target_count è soddisfatto; lo streak aumenta solo in quel caso.

### 4.4 Vista Calendario

- **Mini-calendario mensile** per ogni abitudine, con i giorni completati evidenziati.
- **Per abitudini settimanali:** evidenzia i giorni di completamento e indica se la settimana è stata raggiunta.

### 4.5 Notifica Promemoria

- **Reminder di default globale** (es. 20:00) applicato a tutte le abitudini al momento della creazione.
- **Override opzionale** per singola abitudine (es. reminder alle 8:00 per "bere acqua", 22:00 per "meditare").
- **Gestione permessi:** richiesta esplicita dopo onboarding, non al primo avvio.
- **Tecnica:** utilizzo di notifiche locali tramite UserNotifications (iOS) o equivalente su Android.

### 4.6 Onboarding

- **Massimo 3 schermate** per spiegare valore, funzionamento e richiesta permessi.
- **Nessuna registrazione/account richiesto.**

### 4.7 Gestione Abitudini

- **Soft limit consigliato:** raccomandazione UI a 5 abitudini attive, nessun blocco tecnico.
- **Riordino manuale** delle abitudini tramite drag & drop.

### 4.8 Esportazione e Cancellazione Dati

- **Esporta dati:** funzione in impostazioni per esportare tutte le abitudini e check-in in formato JSON/CSV.
- **Cancella tutti i dati:** opzione per eliminare fisicamente tutti i dati locali dell’utente.

---

## 5. Technical Architecture

### 5.1 Tech Stack

- **Frontend:** React Native (Expo), TypeScript
- **State Management:** Redux Toolkit
- **Storage locale:** AsyncStorage (gestito tramite Redux persist)
- **Navigazione:** React Navigation
- **Testing:** Jest

### 5.2 Data Model

#### User

| Campo                 | Tipo     | Note                                              |
|-----------------------|----------|---------------------------------------------------|
| id                    | UUID     | Generato localmente, 1 utente per device in v1    |
| created_at            | Date     |                                                   |
| default_reminder_time | Time     | Es. 20:00                                         |
| onboarding_completed  | Bool     |                                                   |

#### Habit

| Campo         | Tipo    | Note                                                      |
|---------------|---------|-----------------------------------------------------------|
| id            | UUID    |                                                           |
| user_id       | UUID    | FK → User (utile per v2 multi-profilo)                   |
| name          | String  |                                                           |
| icon          | String  | Emoji o nome asset                                        |
| frequency_type| Enum    | 'daily' \| 'weekly'                                       |
| target_count  | Int     | Default 1 per daily, N per weekly                         |
| created_at    | Date    |                                                           |
| archived      | Bool    | Soft delete                                               |
| sort_order    | Int     | Per riordino manuale nella UI                             |

#### Reminder

| Campo     | Tipo    | Note                                                        |
|-----------|---------|-------------------------------------------------------------|
| id        | UUID    |                                                             |
| habit_id  | UUID    | FK → Habit, nullable se usa default globale                 |
| time      | Time    |                                                             |
| enabled   | Bool    |                                                             |

#### CheckIn

| Campo         | Tipo      | Note                                                  |
|---------------|-----------|-------------------------------------------------------|
| id            | UUID      |                                                       |
| habit_id      | UUID      | FK → Habit                                            |
| date          | Date      | Solo giorno, no time                                  |
| completed_at  | DateTime  | Per analytics futuri                                  |
| unique        |           | Un solo check-in per abitudine/giorno                |

### 5.3 Componenti Chiave

- **HabitList:** elenco abitudini attive, con check-in rapido e streak counter.
- **HabitDetail:** dettaglio abitudine, mini-calendario, modifica reminder.
- **Onboarding:** flusso iniziale con spiegazione e richiesta permessi.
- **Settings:** esportazione/cancellazione dati, gestione reminder di default.
- **ReminderService:** gestione scheduling/cancellazione notifiche locali.

### 5.4 Persistenza e Sicurezza

- **AsyncStorage** per dati strutturati (utenti, abitudini, check-in, reminder).
- **Protezione dati:** nessun dato sensibile, nessun invio a server esterni.
- **Backup:** dati inclusi nei backup standard del device.
- **Esportazione/Cancellazione:** accessibili da impostazioni.

---

## 6. Non-Functional Requirements

- **Performance:** caricamento istantaneo (<300ms) della lista abitudini; operazioni di check-in e navigazione senza lag percepibile.
- **Sicurezza:** dati solo in locale, nessun tracciamento terze parti, esportazione/cancellazione completa dei dati.
- **Privacy:** nessuna raccolta di dati personali o di contenuto abitudini su server esterni.
- **Scalabilità:** supporto a centinaia di abitudini/check-in senza degrado prestazioni (oltre il target utente tipico).
- **Affidabilità notifiche:** reminder locali robusti, rischedulati su modifica abitudini/reminder.
- **Accessibilità:** UI leggibile, tap target sufficienti, palette colori calma.

---

## 7. Out of Scope (v1)

- **Funzionalità social o condivisione abitudini**
- **Statistiche avanzate, trend, AI coach**
- **Sincronizzazione multi-dispositivo o cloud backup**
- **Widget, integrazione con Apple Health/Google Fit**
- **Reminder avanzati (es. solo in certi giorni della settimana)**
- **Gestione multi-utente/profili**
- **Analytics di terze parti sui contenuti delle abitudini**

---

## 8. Open Questions

- **Design dettagliato delle schermate:** mockup/wireframe da definire (palette colori, layout esatto).
- **Gestione edge case reminder:** come gestire reminder saltati (es. device spento per giorni).
- **Gestione archiviazione abitudini:** UX per mostrare/recuperare abitudini archiviate.
- **Testing su Android:** anche se la v1 è iOS-first, serve validare portabilità minima su Android?
- **Gestione errori AsyncStorage:** policy su corruzione dati o spazio esaurito.
- **Eventuale supporto dark mode:** da valutare in base a feedback utenti.

---

**Nota:** Questo PRD è pensato per guidare lo sviluppo di una v1 minimalista, focalizzata su semplicità e rapidità d’uso. Tutte le funzionalità e i vincoli sono stati definiti per massimizzare la retention e la soddisfazione del target utente, senza introdurre complessità superflua.