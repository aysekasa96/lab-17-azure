# Azure Stream Analytics - Oefening

## Overzicht
In deze oefening ga je een **Azure Stream Analytics-job** in je **Azure-abonnement** inrichten en gebruiken om een **realtime gegevensstroom** te verwerken. De resultaten worden opgeslagen in **Azure Storage**.

Tijd nodig om deze oefening te voltooien: **ongeveer 15 minuten**.

## Vereisten
Voordat je begint, zorg ervoor dat:
- Je een **Azure-abonnement** hebt met **administratieve toegang**.

## Stappen

### 1. Azure resources inrichten
Je configureert een **Azure Event Hubs** namespace en een **Azure Storage**-account om gegevens te streamen en op te slaan.

1. Meld je aan bij de **Azure portal**: [portal.azure.com](https://portal.azure.com).
2. Start een **Cloud Shell** door op **[>_]** naast de zoekbalk te klikken.
3. Selecteer de **PowerShell**-omgeving.
4. Voer de volgende opdrachten in om de benodigde repository te klonen en de setup-script uit te voeren:

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    cd dp-203/Allfiles/labs/17
    ./setup.ps1
    ```

5. Selecteer indien nodig je **Azure-abonnement**.
6. Wacht tot het script voltooid is (ongeveer 5 minuten).

### 2. Gegevensbron bekijken
1. Ga in de **Azure portal** naar de **dp203-xxxxxxx resource group**.
2. Bekijk de resources: een **Azure Storage-account** en een **Event Hubs namespace**.
3. Open de **Cloud Shell** opnieuw en voer het volgende uit om gesimuleerde verkoopgegevens te verzenden:

    ```powershell
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

### 3. Azure Stream Analytics-job maken
1. Ga naar de **dp203-xxxxxxx resource group** en klik op **+ Create**.
2. Zoek naar **Stream Analytics job** en configureer:
    - **Naam:** `process-orders`
    - **Regio:** Zelfde regio als andere resources
    - **Hosting:** Cloud
    - **Streaming Units:** 1

### 4. Input configureren
1. Voeg een **Event Hub input** toe genaamd `orders`.
2. Selecteer het bestaande **eventhubxxxxxxx** en de **$Default consumer group**.
3. Gebruik **JSON** als serialization format.

### 5. Output configureren
1. Voeg een **Blob storage output** toe genaamd `blobstore`.
2. Gebruik **Managed Identity** als authenticatiemethode.
3. Stel het padpatroon in als `{date}` voor gestructureerde opslag.

### 6. Query maken
Voeg de volgende query toe om bestellingen per product te groeperen:

```sql
SELECT
    DateAdd(second,-10,System.TimeStamp) AS StartTime,
    System.TimeStamp AS EndTime,
    ProductID,
    SUM(Quantity) AS Orders
INTO
    [blobstore]
FROM
    [orders] TIMESTAMP BY EventProcessedUtcTime
GROUP BY ProductID, TumblingWindow(second, 10)
HAVING COUNT(*) > 1

# lab-17-azure
