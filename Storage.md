# Storage - BLOB

### Generate SAS

```c#
static Uri GenerateSAS()
{

    BlobSasBuilder sasBuilder = new BlobSasBuilder()
    {
	BlobContainerName = containerName,
	BlobName = blobname,
	Resource = "b",
	StartsOn = DateTimeOffset.UtcNow,
	ExpiresOn = DateTimeOffset.UtcNow.AddHours(5) 
    };

   sasBuilder.SetPermissions(BlobSasPermissions.Read);
    var storageSharedKeyCredential = new StorageSharedKeyCredential(accountname,accountkey);
    string sasToken = sasBuilder.ToSasQueryParameters(storageSharedKeyCredential).ToString();
    // Build the full URI to the Blob SAS
    UriBuilder w_fullUri = new UriBuilder()
    {
	Scheme = "https",
	Host = string.Format("{0}.blob.core.windows.net", accountname),
	Path = string.Format("{0}/{1}", containerName, blobname),
	Query = sasToken
    };
    return w_fullUri.Uri;
}
```

### Create BLOB
```c#
static async Task CreateBlob()
        {
            string mountpath = "/mounts/secrets";
            // Will get the directory - volumesecret
            var folders = Directory.GetDirectories(mountpath);
            foreach(var folder in folders)
            {
                Console.WriteLine($"Folder : {folder.ToString()}");
                var AllFiles = Directory.GetFiles(folder);
                foreach(var file in AllFiles)
                {
                    storageconnstring = File.ReadAllText(file);
                    Console.WriteLine(storageconnstring);
                }                   
            }
            BlobServiceClient blobServiceClient = new BlobServiceClient(storageconnstring);
            BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient(containerName);
            BlobClient blobClient = containerClient.GetBlobClient(filename);

            String str = "This is a sample file";
            using (var memoryStream = new MemoryStream(Encoding.UTF8.GetBytes(str)))
            {
                await blobClient.UploadAsync(memoryStream, overwrite: true);
            }
        }        
    }
```

### Upload BLOB
```c#
// Create a file in your local MyDocuments folder to upload to a blob.
string localPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string localFileName = "QuickStart_" + Guid.NewGuid().ToString() + ".txt";
string sourceFile = Path.Combine(localPath, localFileName);
// Write text to the file.
File.WriteAllText(sourceFile, "Hello, World!");

Console.WriteLine("Temp file = {0}", sourceFile);
Console.WriteLine("Uploading to Blob storage as blob '{0}'", localFileName);

// Get a reference to the blob address, then upload the file to the blob.
// Use the value of localFileName for the blob name.
CloudBlockBlob cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(localFileName);
await cloudBlockBlob.UploadFromFileAsync(sourceFile);
```

### Download  BLOB
```c#
private static string blob_url = "https://functionstore1000.blob.core.windows.net/data/sample.txt";
private static string local_blob = "C:\\data\\sample.txt";
private static string blob_name = "sample.txt";
private static string tenantid = "5f5f1c90-abac-4ebe-88d7-0f3d121f967e";
private static string clientid = "d8cf0b94-2d37-4902-8c79-60d426566d8e";
private static string clientsecret = "bZ-u6__-aAr~2gz.4jasbs9v2a0ahv5PSo";
static void Main(string[] args)
{
    ClientSecretCredential _client_credential = new ClientSecretCredential(tenantid, clientid, clientsecret);
    Uri blob_uri = new Uri(blob_url);
    BlobClient _blob_client = new BlobClient(blob_uri, _client_credential);
    _blob_client.DownloadTo(local_blob);            
    Console.WriteLine("Blob downloaded");
    Console.ReadKey();
}
```

## Metadata
```c#
public static void GetMetadata()
{
    BlobServiceClient _service_client = new BlobServiceClient(_connection_string);
    BlobContainerClient _container_client = _service_client.GetBlobContainerClient(_container_name);
    BlobClient _blob_client = _container_client.GetBlobClient(_blob_name);
    BlobProperties _properties = _blob_client.GetProperties();
    IDictionary<string, string> _metadata = _properties.Metadata;
    foreach (var item in _metadata)
    {
        Console.WriteLine(item.Key);
        Console.WriteLine(item.Value);
    }
}

public static void SetMetadata()
{
    BlobServiceClient _service_client = new BlobServiceClient(_connection_string);
    BlobContainerClient _container_client = _service_client.GetBlobContainerClient(_container_name);
    BlobClient _blob_client = _container_client.GetBlobClient(_blob_name);
    BlobProperties _properties = _blob_client.GetProperties();
    IDictionary<string, string> _metadata = _properties.Metadata;
    _metadata.Add("Tier", "1");
    _blob_client.SetMetadata(_metadata);
    Console.WriteLine("Metadata appended");
}
```