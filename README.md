# Bank Customer Churn Analysis

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/danielbolognini/bank-customer-churn/blob/main/Bank%20churn%20analysis.ipynb)

Analisi dell'abbandono dei clienti di una banca europea (Francia, Germania e Spagna), svolta in Python su Google Colab.

L'idea è mettersi nei panni del management e rispondere a due domande: quali clienti hanno più probabilità di lasciare la banca, e perché. Per la parte predittiva abbiamo usato un albero decisionale. Non è il modello più accurato in assoluto, ma ci permette di leggere le regole che portano all'abbandono (soglie di età, numero di prodotti, ecc.) e di trasformarle in azioni concrete di fidelizzazione.

## Dataset

Il file `Bank_Churn.csv` contiene 10.000 clienti e 17 colonne, senza valori mancanti né duplicati. Circa il 20% dei clienti (2.038) ha lasciato la banca.

| Colonna | Descrizione |
|---|---|
| CreditScore | Punteggio di credito (350-850) |
| Geography | Paese (France, Germany, Spain) |
| Gender | Sesso |
| Age | Età |
| Tenure | Anni da cui il cliente è con la banca |
| Balance | Saldo sul conto |
| NumOfProducts | Numero di prodotti bancari posseduti |
| HasCrCard | Possiede una carta di credito (0/1) |
| IsActiveMember | Cliente attivo (0/1) |
| EstimatedSalary | Stipendio annuo stimato |
| Satisfaction Score | Soddisfazione per la gestione dei reclami (1-5) |
| Card Type | Tipo di carta (Silver, Gold, Platinum, Diamond) |
| Point Earned | Punti accumulati con la carta |
| Exited | Variabile target: 0 = rimasto, 1 = uscito |

Sono presenti anche RowNumber, CustomerId e Surname, che abbiamo eliminato perché non servono all'analisi.

## Cosa contiene il notebook

**Pulizia dei dati**

- 3.617 clienti hanno saldo pari a zero. Non li abbiamo eliminati né sostituiti con la media, perché è normale usare il conto senza tenerci dei risparmi. Abbiamo invece aggiunto la colonna `HasBalance` (saldo positivo sì/no).
- Abbiamo tolto 59 righe con `EstimatedSalary` sotto i 1.000 €, valori poco realistici per uno stipendio annuo (il minimo era 11,58).

**Analisi descrittiva**

Distribuzioni di credit score, età, reddito, anzianità e soddisfazione, matrice di correlazione e alcuni grafici che mettono in relazione l'abbandono con età, saldo, attività del cliente e numero di prodotti.

**Analisi predittiva**

- One-hot encoding di Geography, Gender e Card Type.
- Divisione train/test 80/20.
- Scelta della profondità dell'albero con cross-validation a 10 fold, verificata poi con `GridSearchCV`. La profondità migliore sarebbe 6, ma abbiamo usato `max_depth=4` perché l'accuratezza cambia di poco e l'albero è molto più leggibile.
- Abbiamo confrontato due modelli: un albero base e un albero con `class_weight='balanced'`, per compensare il fatto che i clienti usciti sono solo il 20%.

## Risultati

| | Albero base | Albero bilanciato |
|---|:---:|:---:|
| Accuracy train | 85,3% | 77,0% |
| Accuracy test | 84,3% | 76,3% |
| Precision (Exited) | 69% | 44% |
| Recall (Exited) | 40% | 70% |
| Falsi negativi | 239 | 119 |
| Falsi positivi | 74 | 353 |

In entrambi i casi la differenza tra train e test è di circa un punto percentuale, quindi non c'è overfitting.

Il modello base ha un'accuracy più alta, ma riconosce solo 4 clienti in uscita su 10. Per una banca che vuole fare retention questo è un problema, perché la maggior parte dei clienti a rischio non viene segnalata. Con l'albero bilanciato la recall sale al 70%, in cambio di molti più falsi allarmi. Abbiamo scelto questo secondo modello: contattare un cliente che in realtà non se ne sarebbe andato costa molto meno che perderne uno senza accorgersene.

## Cosa abbiamo scoperto

- L'età è la variabile che pesa di più. L'albero divide i clienti a 42,5 anni e l'abbandono si concentra soprattutto tra i 40 e i 50 anni.
- Due prodotti è la situazione più stabile. Con tre prodotti il tasso di abbandono sale molto, con quattro se ne vanno quasi tutti.
- I clienti inattivi abbandonano molto più spesso di quelli attivi. Il gruppo più a rischio sono gli inattivi tra 50 e 67 anni: 282 su 325 hanno lasciato la banca (87%).
- In Germania il rischio è più alto anche tra i clienti giovani, cosa che non succede in Francia e Spagna.
- Chi se ne va ha in media un saldo più alto di chi resta, quindi la banca sta perdendo proprio i clienti con più capitale.
- Reddito, credit score e soddisfazione non sembrano influire sull'abbandono.

Da questi risultati abbiamo ricavato alcune proposte per la banca: non spingere la vendita di prodotti oltre il secondo, contattare i clienti intorno ai 40 anni per rivedere le loro condizioni, attivare un avviso automatico quando un cliente over 40 diventa inattivo e approfondire con un'indagine di mercato cosa succede in Germania.

## Come eseguirlo

Il dataset è nella stessa cartella del notebook. Nel notebook il file viene letto da Google Drive; per eseguirlo direttamente su Colab senza caricare nulla, basta sostituire le celle di `drive.mount` e `path` con:

```python
df = pd.read_csv("https://raw.githubusercontent.com/danielbolognini/bank-customer-churn/main/Bank_Churn.csv")
```

Per eseguirlo in locale:

```bash
git clone https://github.com/danielbolognini/bank-customer-churn.git
cd bank-customer-churn
pip install pandas matplotlib seaborn scikit-learn jupyter
jupyter notebook
```

e nel notebook usare `pd.read_csv("Bank_Churn.csv")`.

Librerie usate: pandas, matplotlib, seaborn, scikit-learn.
