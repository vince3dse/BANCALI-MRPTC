InventorConfigurator
Guida Utente Completa
Autodesk Inventor 2025  |  C# .NET Framework 4.8.1
Configurazione assiemi tramite regole JSON
senza modificare manualmente l'albero Inventor
 
1. Introduzione
InventorConfigurator è un'applicazione standalone (.NET Framework 4.8.1) che permette di applicare automaticamente configurazioni di assiemi Autodesk Inventor 2025 tramite regole JSON, senza aprire l'editor di Inventor.
1.1 Concetti chiave
Termine	Descrizione
Condizione	Una domanda mostrata all'operatore (es. Lato = DX/SX, Quantità = 3). Definita in conditions.json.
Regola	Se la condizione X vale Y → esegui queste azioni. Definita in actions.json.
Azione	Operazione Inventor: Sopprimi, Desopp., attiva LOD / Positional / Design View.
Target	Componente o rappresentazione su cui agisce l'azione.
Configurazione	Cartella sotto Config/ contenente una coppia conditions.json + actions.json.

1.2 Struttura cartelle
InventorConfigurator.exe
Config/
  BancaleDX/
    conditions.json    ← domande all'operatore
    actions.json       ← regole e azioni Inventor
  BancaleSX/
    conditions.json
    actions.json

2. Flusso di utilizzo
Passo	Azione
1	Aprire l'assieme .IAM in Autodesk Inventor 2025.
2	Avviare InventorConfigurator.exe.
3	Selezionare la configurazione nel ComboBox in alto.
4	Rispondere alle condizioni (radio, lista, numero).
5	Verificare l'anteprima delle azioni pianificate.
6	Premere Applica — le azioni vengono eseguite in una singola transazione (annullabile con Ctrl+Z).
7	Leggere il ResultsForm: verde = OK, arancione = avviso, rosso = errore.

3. Tipi di condizione
Ogni condizione genera automaticamente un controllo UI nel MainForm.
Tipo	Controllo UI	Note
Boolean	RadioButton SI/NO	I valori possono essere personalizzati (es. DX/SX). Il primo radio è selezionato di default.
Choice	ComboBox	Lista di valori fissi. L'utente sceglie uno solo.
Integer	NumericUpDown	Numero intero con Min/Max configurabili.
MultiChoice	CheckedListBox	Selezione multipla. Il valore inviato alle regole è una stringa CSV: A,B,C

3.1 DependsOn — visibilità condizionale
Una condizione può essere nascosta finché un'altra non ha un determinato valore. Formato:
DependsOn = "IDcondizione=Valore"

Esempio: "TIPO_ASSIEME=Saldato"

Esempio pratico
Condizione FORI_EXTRA (Integer) con DependsOn = TIPO_ASSIEME=Forato.
Il campo appare solo quando l'operatore seleziona Forato nel combo TIPO_ASSIEME.

4. Tipi di azione
Tipo azione	Comportamento
Suppress	Sopprime il/i componente/i trovato/i (nasconde dall'assieme).
Unsuppress	Toglie la soppressione (rende visibile).
ActivateLevelOfDetail	Attiva la Level of Detail Representation specificata.
ActivatePositional	Attiva la Positional Representation specificata.
ActivateDesignView	Attiva la Design View specificata.

5. Target e MatchMode
Il campo Target indica quale componente cercare. MatchMode definisce come cercarlo.
MatchMode	Cosa confronta	Esempio Target
OccurrenceName	Nome nell'albero modello	VITE M6  oppure  VITE M6:1
FileName	Nome file .ipt senza estensione	VITE_M6_ISO4762
IProperty	Valore di una proprietà custom	ACC_INOX  (con IPropertyName = Materiale)

5.1 Nome base vs nome completo (OccurrenceName)
In Inventor le istanze multiple hanno nome nella forma BaseName:N (es. VITE M6:1, VITE M6:2, …).
Target scritto	Effetto
VITE M6:1	Match esatto su una sola istanza (la prima).
VITE M6	Match su tutte le istanze: :1, :2, :3, … Necessario per FirstN / LastN / ListN.

Suggerimento — pulsante «Seleziona da Inventor»
Clicca su qualsiasi istanza del componente in Inventor.
Nel dialog "Come usare la selezione?" scegli:
  • "Nome occorrenza esatto: VITE M6:3"  →  solo quella singola istanza
  • "Base name tutte le istanze: VITE M6"  →  tutte le istanze (usa con ListN / FirstN / LastN)
La voce "Base name" appare automaticamente solo se il componente ha il suffisso :N.

6. SelectionMode — selezione tra più occorrenze
Dopo aver trovato tutte le occorrenze che corrispondono al Target, SelectionMode decide quali coinvolgere.
SelectionMode	Campo N	Comportamento
All	— (lasciare vuoto)	Tutte le occorrenze trovate.
FirstN	Numero intero (es. 3)	Le prime N occorrenze nell'ordine dell'albero.
LastN	Numero intero (es. 2)	Le ultime N occorrenze.
AtIndex	Numero intero (es. 2)	Solo quella alla posizione N (1-based). Utile per prendere la seconda istanza.
ListN	Indici separati da virgola o ; (es. 1,3,5)	Solo le occorrenze agli indici specificati.

6.1 Esempio ListN passo per passo
Scenario: assieme con 6 istanze di VITE M6. Vuoi sopprimere la 1ª, 3ª e 5ª.
Passo	Cosa fare
1	Nell'editor apri la regola → "Aggiungi azione" → tipo Suppress.
2	Premi "Seleziona da Inventor" → clicca su una qualsiasi VITE M6 nell'assieme.
3	Nel dialog scegli "Base name tutte le istanze: VITE M6". Target = VITE M6.
4	SelectionMode = ListN.
5	Campo N = 1,3,5
6	Salva. La preview mostrerà: SOPPRIMI: VITE M6 [ListN 1,3,5]

Importante
Il Target deve essere il nome BASE (VITE M6), non VITE M6:1.
Con il nome completo matched ha solo 1 elemento: gli indici 3 e 5 non troverebbero nulla.

6.2 Placeholder {VALUE} in SelectionN
Se la condizione è un Integer, puoi usare il valore scelto dall'operatore direttamente in N:
SelectionMode = FirstN
SelectionN    = "{VALUE}"

→ Se l'operatore sceglie 4, sopprime le prime 4 istanze.
Per ListN puoi comporre liste dinamiche:
SelectionMode = ListN
SelectionN    = "1,{VALUE},5"

→ Se VALUE = 3, gli indici diventano 1, 3, 5.

7. Valori condizione nelle regole
Sintassi	Significato	Esempio
SI	Valore esatto (case-insensitive)	ConditionValue = SI
*	Qualsiasi valore (catch-all)	Sempre vera per questa condizione
>=3	Numerico ≥ 3	Condizione Integer con valore 3, 4, 5…
<=2	Numerico ≤ 2	Condizione Integer con valore 1 o 2
>0	Numerico > 0	Esclude lo zero
<5	Numerico < 5	Valori 1, 2, 3, 4

7.1 Placeholder {VALUE} nel Target
Il Target può contenere {VALUE} che viene sostituito a runtime con il valore della condizione:
Condizione LATO = DX
Target = "STAFFA_{VALUE}"

→ Cerca il componente STAFFA_DX nell'assieme.

8. L'Editor (F2)
L'editor visuale permette di creare e modificare conditions.json e actions.json senza toccare i file a mano.
8.1 Tab Condizioni
Pulsante	Funzione
Aggiungi condizione	Apre il dialog per creare un nuovo ID condizione.
Modifica	Modifica l'ID, etichetta, tipo, valori, min/max, DependsOn e ordine.
Elimina	Rimuove la condizione (attenzione: verificare che nessuna regola la usi).
▲ / ▼	Modifica l'ordine di visualizzazione nel MainForm.

8.2 Tab Regole
Ogni regola ha: Condizione ID, Valore condizione, lista di azioni.
Per ogni azione si configurano: Tipo, Target, MatchMode, IPropertyName, SelectionMode, N, Ricorsivo.
8.3 Salvataggio
Pulsante	Funzione
Salva	Salva nella cartella della configurazione attiva. Crea automaticamente un .bak di backup.
Salva come…	Copia i file JSON in una cartella scelta dall'utente.
Mostra JSON	Visualizza il JSON grezzo corrente per ispezione / copia.

9. Esempi completi
9.1 Soppressione condizionale per lato DX/SX
Obiettivo: sopprimere il componente STAFFA_DX quando l'operatore sceglie Lato = SX (e viceversa).
conditions.json
{
  "ConfigName": "Assieme Base",
  "Conditions": [
    {
      "Id": "LATO",
      "Label": "Lato di montaggio",
      "Type": "Boolean",
      "Values": ["DX", "SX"],
      "Order": 0
    }
  ]
}

actions.json
{
  "Rules": [
    {
      "ConditionId": "LATO",
      "ConditionValue": "SX",
      "Actions": [{
        "Type": "Suppress",
        "Target": "STAFFA_DX",
        "MatchMode": "OccurrenceName",
        "SelectionMode": "All"
      }]
    },
    {
      "ConditionId": "LATO",
      "ConditionValue": "DX",
      "Actions": [{
        "Type": "Suppress",
        "Target": "STAFFA_SX",
        "MatchMode": "OccurrenceName",
        "SelectionMode": "All"
      }]
    }
  ]
}

9.2 ListN — sopprimere istanze specifiche
Obiettivo: assieme con 8 viti VITE M6. Quando Quantità = 4, sopprimere le viti in posizione 5, 6, 7, 8 (le ultime 4).
conditions.json
{
  "Conditions": [
    {
      "Id": "QUANTITA_VITI",
      "Label": "Quantità viti",
      "Type": "Integer",
      "Min": 4,
      "Max": 8,
      "Order": 0
    }
  ]
}

actions.json
{
  "Rules": [
    {
      "ConditionId": "QUANTITA_VITI",
      "ConditionValue": "4",
      "Actions": [{
        "Type": "Suppress",
        "Target": "VITE M6",
        "MatchMode": "OccurrenceName",
        "SelectionMode": "ListN",
        "SelectionN": "5,6,7,8"
      }]
    }
  ]
}

Alternativa con LastN
La stessa cosa si può ottenere con SelectionMode = LastN e N = 4.
LastN è più comodo quando il numero di istanze da sopprimere è fisso.
ListN è necessario quando le posizioni non sono consecutive (es. 1,3,5).

9.3 Placeholder {VALUE} — target dinamico
Obiettivo: l'operatore sceglie un colore tra ROSSO, VERDE, BLU. Attivare la Design View corrispondente.
conditions.json
{
  "Conditions": [
    {
      "Id": "COLORE",
      "Label": "Colore prodotto",
      "Type": "Choice",
      "Values": ["ROSSO", "VERDE", "BLU"],
      "Order": 0
    }
  ]
}

actions.json  (una sola regola con *)
{
  "Rules": [
    {
      "ConditionId": "COLORE",
      "ConditionValue": "*",
      "Actions": [{
        "Type": "ActivateDesignView",
        "Target": "Vista_{VALUE}"
      }]
    }
  ]
}
Risultato: se l'operatore sceglie VERDE, viene attivata la Design View chiamata Vista_VERDE.

9.4 Ricerca per iProperty
Obiettivo: sopprimere tutti i componenti con iProperty Materiale = Alluminio.
"Actions": [{
  "Type": "Suppress",
  "Target": "Alluminio",
  "MatchMode": "IProperty",
  "IPropertyName": "Materiale",
  "SelectionMode": "All",
  "Recursive": true
}]
Recursive = true estende la ricerca ai sottoassiemi.

10. Lettura del ResultsForm
Icona	Stato	Significato
✓ OK	Success	Azione eseguita correttamente. La colonna «Modificati» mostra quante occorrenze sono state toccate.
⚠ Avviso	Warning	Target non trovato nell'assieme. L'azione è stata saltata ma le altre continuano. Verificare il nome nel Target.
✗ Errore	Error	Eccezione COM durante l'esecuzione. Messaggio dettagliato nella colonna «Messaggio». L'intera transazione può essere annullata con Ctrl+Z.

11. Risoluzione problemi
Sintomo	Soluzione
Target non trovato (Warning)	Verificare il nome esatto nell'albero Inventor. Se il componente è VITE M6:1, usare VITE M6 (base name) con SelectionMode ≠ All, oppure il nome completo con All.
Preview mostra [ListN] senza indici	Il campo N è vuoto. Inserire gli indici (es. 1,3,5) nel campo N / elenco ListN dell'editor azione.
ListN trova solo 1 occorrenza	Il Target è scritto con nome completo (es. VITE M6:1) invece del base name (VITE M6). Correggere il Target.
Inventor non risponde	Verificare che l'assieme sia aperto e non in stato di modifica. Usare il pulsante Riconnetti.
Template non corrisponde	Il file .iam aperto in Inventor è diverso dal TemplateFile configurato. Aprire il file corretto o cambiare configurazione.
Design View non si attiva	Verificare il nome esatto della Design View (pulsante «Seleziona rappresentazione da Inventor»).

12. Riferimento rapido
12.1 SelectionMode — riepilogo
Modalità	Campo N	Quando usarlo	Target consigliato
All	—	Tutti i componenti col nome indicato	Base name o nome completo
FirstN	Numero (es. 3)	Le prime N istanze	Base name
LastN	Numero (es. 2)	Le ultime N istanze	Base name
AtIndex	Numero (es. 2)	Una sola istanza alla posizione N	Base name
ListN	Es. 1,3,5	Istanze a posizioni non consecutive	Base name (OBBLIGATORIO)

12.2 Sintassi ConditionValue
Valore	Corrisponde a
SI	Testo esatto «SI» (case-insensitive)
*	Qualsiasi valore della condizione
>=3	Valore numerico ≥ 3
<=2	Valore numerico ≤ 2
>0	Valore numerico > 0
<5	Valore numerico < 5

12.3 Scorciatoie tastiera
Tasto	Azione
F2	Apre l'Editor configurazione
Ctrl+Z	Annulla l'ultima operazione in Inventor (transazione)
ESC (in Inventor)	Annulla la selezione interattiva del componente

 
13. IMPORT / EXPORT REGOLE (Nuova funzionalità)
Esportazione e importazione delle regole tramite file ZIP dal MainForm.

13.1 Esporta regole
Crea uno ZIP con tutti i file JSON della configurazione selezionata.

13.2 Importa regole
Importa uno ZIP e sostituisce i file JSON esistenti.

13.3 Backup automatico
Prima dell'import viene creato automaticamente un backup ZIP nella cartella della configurazione.

14. SELEZIONE MULTIPLA COMPONENTI (Nuova funzionalità)
È possibile selezionare più componenti direttamente da Autodesk Inventor per costruire Target e regole.

14.1 Comportamento
- Selezione multipla direttamente nell'assieme
- Acquisizione di più ComponentOccurrence
- Supporto per costruzione automatica Target
- Utilizzabile nelle regole con SelectionMode

14.2 Utilizzo previsto
1. Attivare la selezione in Inventor
2. Selezionare più componenti
3. Confermare la selezione
4. Il sistema costruisce automaticamente i riferimenti

14.3 Stato attuale
La funzionalità è in fase di stabilizzazione:
- La conferma selezione potrebbe non essere visibile correttamente
- Il comportamento di cattura non è ancora completamente coerente
- È prevista una revisione completa del flusso di selezione
