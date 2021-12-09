# Security

## KeyVault
### code


```c#
static void Main(string[] args)
{
    string keyVaultUrl = "https://demovault2090.vault.azure.net/";
    var client = new KeyClient(vaultUri: new Uri(keyVaultUrl), credential: new DefaultAzureCredential());

    // Getting the Encryption key from the key vault
    KeyVaultKey key = client.GetKey("newkey");

    var cryptoClient = new CryptographyClient(keyId: key.Id, credential: new DefaultAzureCredential());
    byte[] plaintext = Encoding.UTF8.GetBytes("This is sensitive data");

    // Encrypting data
    EncryptResult encryptResult = cryptoClient.Encrypt(EncryptionAlgorithm.RsaOaep256, plaintext);

    //Decrypting data
    DecryptResult decryptResult = cryptoClient.Decrypt(EncryptionAlgorithm.RsaOaep256, encryptResult.Ciphertext);
    Console.WriteLine("Plain text");

    string txt=Encoding.UTF8.GetString(decryptResult.Plaintext);
    Console.WriteLine(txt);
    Console.ReadLine();
}
```
## Acessing keyvault
```c#
# reading value of a secret
string secret_name = "dbpassword";
ClientSecretCredential _client_secret = new ClientSecretCredential(tenantid,clientid,clientsecret);
SecretClient _secret_client = new SecretClient(new Uri(keyvault_url), _client_secret);
var secret= _secret_client.GetSecret(secret_name);
Console.WriteLine($"The value of the secret is {secret.Value.Value}");

```
### Powershell
```shell
New-AzKeyVault -Name "<your-unique-keyvault-name>" -ResourceGroupName "myResourceGroup" -Location "East US"

```

## Custom Roles
```bash
Get-AzRoleDefinition <role_name> | ConvertTo-Json

```
## Azure built-in roles
- **Contributor**	Create and manage all of types of Azure resources Create a new tenant in Azure Active Directory Cannot grant access to others.
- **Owner**	Grants full access to manage all resources, including the ability to assign roles in Azure RBAC.
- **Reader**	View all resources, but does not allow you to make any changes.
- **User Access Administrator**	Lets you manage user access to Azure resources.