# Explore an Azure AI Search index (UI)

Let’s imagine you work for Fourth Coffee, a national coffee chain. You’re asked to help build a knowledge mining solution that makes it easy to search for insights about customer experiences. You decide to build an Azure AI Search index using data extracted from customer reviews.

## In this lab you’ll:

- Create Azure resources
- Extract data from a data source
- Enrich data with AI skills
- Use Azure’s indexer in the Azure portal
- Query your search index
- Review results saved to a Knowledge Store

## Azure resources needed

The solution you’ll create for Fourth Coffee requires the following resources in your Azure subscription:

- An Azure AI Search resource, which will manage indexing and querying.
- An Azure AI services resource, which provides AI services for skills that your search solution can use to enrich the data in the data source with AI-generated insights.
  - Note: Your Azure AI Search and Azure AI services resources must be in the same location!
- A Storage account with blob containers, which will store raw documents and other collections of tables, objects, or files.

### Create an Azure AI Search resource

1. Sign into the Azure portal.
2. Click the `+ Create a resource` button, search for Azure AI Search, and create an Azure AI Search resource with the following settings:
   - Subscription: Your Azure subscription.
   - Resource group: Select or create a resource group with a unique name.
   - Service name: A unique name.
   - Location: Choose any available region.
   - Pricing tier: Basic
3. Select `Review + create`, and after you see the response `Validation Success`, select `Create`.

### Create an Azure AI services resource

1. Return to the home page of the Azure portal.
2. Click the `＋Create a resource` button and search for Azure AI services.
3. Select `Create an Azure AI services plan`.
4. Configure it with the following settings:
   - Subscription: Your Azure subscription.
   - Resource group: The same resource group as your Azure AI Search resource.
   - Region: The same location as your Azure AI Search resource.
   - Name: A unique name.
   - Pricing tier: Standard S0
5. Check the acknowledgment box and select `Review + create`.
6. After you see the response `Validation Passed`, select `Create`.

### Create a storage account

1. Return to the home page of the Azure portal, and then select the `+ Create a resource` button.
2. Search for storage account, and create a Storage account resource with the following settings:
   - Subscription: Your Azure subscription.
   - Resource group: The same resource group as your Azure AI Search and Azure AI services resources.
   - Storage account name: A unique name.
   - Location: Choose any available location.
   - Performance: Standard
   - Redundancy: Locally redundant storage (LRS)
3. Click `Review` and then click `Create`.
4. Wait for deployment to complete, and then go to the deployed resource.

### Upload Documents to Azure Storage

1. In the left-hand menu pane of the Azure Storage account, select `Containers`.
2. Select `+ Container`. Enter the following settings, and click `Create`:
   - Name: coffee-reviews
   - Public access level: Container (anonymous read access for containers and blobs)
3. Download the zipped coffee reviews from [here](https://aka.ms/mslearn-coffee-reviews), and extract the files to the `reviews` folder.
4. In the Azure portal, select your `coffee-reviews` container. In the container, select `Upload`.
5. In the Upload blob pane, select `Select a file`.
6. In the Explorer window, select all the files in the `reviews` folder, select `Open`, and then select `Upload`.
7. After the upload is complete, you can close the Upload blob pane. Your documents are now in your `coffee-reviews` storage container.

### Index the documents

1. In the Azure portal, browse to your Azure AI Search resource.
2. On the Overview page, select `Import data`.
3. On the `Connect to your data` page, in the `Data Source` list, select `Azure Blob Storage`. Complete the data store details with the following values:
   - Data Source: Azure Blob Storage
   - Data source name: coffee-customer-data
   - Data to extract: Content and metadata
   - Parsing mode: Default
   - Connection string: Select `Choose an existing connection`. Select your storage account, select the `coffee-reviews` container, and then click `Select`.
   - Managed identity authentication: None
   - Container name: this setting is auto-populated after you choose an existing connection.
   - Blob folder: Leave this blank.
   - Description: Reviews for Fourth Coffee shops.
4. In the Attach Cognitive Services section, select your Azure AI services resource.
5. In the Add enrichments section:

   - Change the Skillset name to `coffee-skillset`.
   - Select the checkbox `Enable OCR` and merge all text into `merged_content` field.
   - Ensure that the `Source data` field is set to `merged_content`.
   - Change the `Enrichment granularity level` to Pages (5000 character chunks).
   - Don’t select `Enable incremental enrichment`.
   - Select the following enriched fields:

     | Cognitive Skill               | Parameter | Field name   |
     | ----------------------------- | --------- | ------------ |
     | Extract location names        |           | locations    |
     | Extract key phrases           |           | keyphrases   |
     | Detect sentiment              |           | sentiment    |
     | Generate tags from images     |           | imageTags    |
     | Generate captions from images |           | imageCaption |

6. Under `Save enrichments to a knowledge store`, select:
   - Image projections
   - Documents
   - Pages
   - Key phrases
   - Entities
   - Image details
   - Image references
   - If a warning asking for a Storage Account Connection String appears, select `Choose an existing connection`, and choose the storage account you created earlier.
7. Click on `+ Container` to create a new container called `knowledge-store` with the privacy level set to `Private`, and select `Create`.
8. Select the `knowledge-store` container, and then click `Select` at the bottom of the screen.
9. Select `Azure blob projections: Document`. A setting for `Container name` with the `knowledge-store` container auto-populated displays. Don’t change the container name.
10. Change the `Index name` to `coffee-index`.
11. Ensure that the `Key` is set to `metadata_storage_path`. Leave `Suggester name` blank and `Search mode` auto-populated.
12. Review the index fields’ default settings. Select `filterable` for all the fields that are already selected by default.
13. Change the `Indexer name` to `coffee-indexer`.
14. Leave the `Schedule` set to `Once`.
15. Expand the `Advanced options`. Ensure that the `Base-64 Encode Keys` option is selected.
16. Select `Submit` to create the data source, skillset, index, and indexer. The indexer is run automatically and runs the indexing pipeline.
17. Return to your Azure AI Search resource page. On the left pane, under `Search Management`, select `Indexers`. Select the newly created `coffee-indexer`. Wait until the `Status` indicates success.
18. Select the indexer name to see more details.

### Query the index

Use the Search explorer to write and test queries. Search explorer is a tool built into the Azure portal that gives you an easy way to validate the quality of your search index. You can use Search explorer to write queries and review results in JSON.

1. In your Search service’s Overview page, select `Search explorer` at the top of the screen.
2. Notice how the index selected is the `coffee-index` you created. Below the index selected, change the view to `JSON view`.
3. In the JSON query editor field, copy and paste the following:
   ```json
   {
       "search": "*",
       "count": true
   }
   Select Search. The search query returns all the documents in the search index, including a count of all the documents.
   ```
4. To filter by location, copy and paste the following JSON query:
   {
   "search": "locations:'Chicago'",
   "count": true
   }
   Select Search. The query filters for reviews with a Chicago location.
5. To filter by sentiment, copy and paste the following JSON query:
   {
   "search": "sentiment:'negative'",
   "count": true
   }
   Select Search. The query filters for reviews with a negative sentiment.

Note: The results are sorted by @search.score, showing how closely the results match the given query.

Review the knowledge store
When you ran the Import data wizard, you also created a knowledge store. Inside the knowledge store, you’ll find the enriched data extracted by AI skills persists in the form of projections and tables.

In the Azure portal, navigate back to your Azure storage account.
In the left-hand menu pane, select Containers. Select the knowledge-store container.
Select any of the items, and then click the objectprojection.json file.
Select Edit to see the JSON produced for one of the documents from your Azure data store.
Select the storage blob breadcrumb at the top left of the screen to return to the Storage account Containers.
In the Containers, select the container coffee-skillset-image-projection. Select any of the items.
Select any of the .jpg files. Select Edit to see the image stored from the document.
Select the storage blob breadcrumb at the top left of the screen to return to the Storage account Containers.
Select Storage browser on the left-hand panel, and select Tables. There’s a table for each entity in the index. Select the coffeeSkillsetKeyPhrases table.
Look at the key phrases the knowledge store was able to capture from the content in the reviews. Many of the fields are keys, so you can link the tables like a relational database. The last field shows the key phrases that were extracted by the skillset.
