# Data Encryption - Architect Deep Dive

============================================================
0. Executive Summary (Architect View)
============================================================
- **Data Encryption exists** to protect sensitive data through cryptographic transformation
- **Critical at scale** when applications handle sensitive information and regulatory compliance
- **System design impact** - transforms data handling from plaintext to encrypted storage and transmission
- **Data protection foundation** - provides essential protection for sensitive data
- **Compliance requirement** - mandatory for security standards and regulations
- **Business continuity** - maintains data confidentiality and integrity
- **Trust foundation** - builds trust with customers and stakeholders
- **Risk mitigation** - reduces data breach impact and liability

============================================================
1. Problem Framing
============================================================
**Real production problem solved:**
- Data breaches and unauthorized access to sensitive information
- Compliance violations and regulatory penalties
- Data exposure during transmission and storage
- Insider threats and data theft
- Reputation damage and loss of customer trust

**What breaks if we don't understand Data Encryption:**
- Data compromise and unauthorized access
- Compliance violations and legal consequences
- Loss of customer trust and reputation
- Financial penalties and business disruption
- Inability to protect sensitive information

**Scale importance:**
- **Small applications**: Data encryption essential
- **Medium applications**: Data encryption critical
- **Large applications**: Data encryption mandatory
- **Enterprise systems**: Data encryption fundamental

**Failure stories:**
- Healthcare platform had patient data breach due to unencrypted storage
- Financial system had credit card data exposure in transit
- E-commerce platform had customer data theft from database
- Government agency had classified data leak due to missing encryption

============================================================
2. Deep Internal Mechanics
============================================================

**Data Encryption Fundamentals:**
- **Symmetric Encryption**: Single key for encryption and decryption
- **Asymmetric Encryption**: Public/private key pair for encryption/decryption
- **Hashing**: One-way cryptographic transformation
- **Digital Signatures**: Authentication and integrity verification
- **Key Management**: Secure key generation, storage, and rotation

**Core Components:**
- **Encryption Engine**: Core encryption/decryption operations
- **Key Management Service**: Secure key generation and management
- **Certificate Management**: Digital certificate lifecycle management
- **Data Classification**: Data sensitivity classification
- **Encryption Policy**: Encryption policy and configuration management
- **Audit Logging**: Encryption operation auditing

**Key Concepts:**
- **AES**: Advanced Encryption Standard
- **RSA**: Rivest-Shamir-Adleman encryption
- **TLS**: Transport Layer Security
- **PKI**: Public Key Infrastructure
- **HSM**: Hardware Security Module
- **Key Rotation**: Periodic key replacement
- **Data-at-Rest**: Encrypted data storage
- **Data-in-Transit**: Encrypted data transmission

**Encryption Operations:**
- **Key Generation**: Cryptographic key creation
- **Encryption**: Plaintext to ciphertext transformation
- **Decryption**: Ciphertext to plaintext transformation
- **Key Distribution**: Secure key sharing
- **Key Rotation**: Periodic key replacement
- **Key Revocation**: Key invalidation and replacement

**Security Layers:**
- **Application Layer**: Application-level encryption
- **Database Layer**: Database encryption
- **File System Layer**: File system encryption
- **Network Layer**: Network encryption
- **Storage Layer**: Storage device encryption

**Data Structures:**
- **Key Registry**: Key metadata and status
- **Certificate Registry**: Certificate metadata and status
- **Encryption Registry**: Encryption operation metadata
- **Data Classification Registry**: Data classification metadata
- **Audit Registry**: Encryption audit logs

**Performance Characteristics:**
- **Symmetric Encryption**: 100-1000 MB/s
- **Asymmetric Encryption**: 1-10 MB/s
- **Hashing**: 1000-10000 MB/s
- **Key Generation**: 10-1000ms
- **Key Rotation**: 100ms-10s

============================================================
3. .NET Production Implementation
============================================================

**Naive Implementation (No Encryption):**
```csharp
[ApiController]
[Route("api/[controller]")]
public class InsecureDataController : ControllerBase
{
    private readonly IDataRepository _dataRepository;
    
    public InsecureDataController(IDataRepository dataRepository)
    {
        _dataRepository = dataRepository;
    }
    
    [HttpPost("data")]
    public async Task<ActionResult> StoreData([FromBody] DataRequest request)
    {
        // Store data without encryption
        await _dataRepository.StoreDataAsync(request.Data);
        return Ok();
    }
    
    [HttpGet("data/{id}")]
    public async Task<ActionResult<string>> GetData(int id)
    {
        // Retrieve data without decryption
        var data = await _dataRepository.GetDataAsync(id);
        return Ok(data);
    }
}
```

**Production-Ready Data Encryption Implementation:**
```csharp
// Data Encryption Configuration
public class DataEncryptionConfiguration
{
    public bool EnableEncryption { get; set; } = true;
    public bool EnableKeyRotation { get; set; } = true;
    public bool EnableDataClassification { get; set; } = true;
    public bool EnableAuditLogging { get; set; } = true;
    public string DefaultAlgorithm { get; set; } = "AES-256-GCM";
    public int KeySize { get; set; } = 256;
    public int KeyRotationDays { get; set; } = 90;
    public string KeyManagementService { get; set; } = "AzureKeyVault";
    public List<string> EncryptedFields { get; set; } = new();
    public Dictionary<string, string> DataClassification { get; set; } = new();
}

// Data Encryption Service
public interface IDataEncryptionService
{
    Task<string> EncryptAsync(string plaintext, string keyId = null);
    Task<string> DecryptAsync(string ciphertext, string keyId = null);
    Task<byte[]> EncryptAsync(byte[] plaintext, string keyId = null);
    Task<byte[]> DecryptAsync(byte[] ciphertext, string keyId = null);
    Task<string> HashAsync(string data, string algorithm = "SHA-256");
    Task<bool> VerifyHashAsync(string data, string hash, string algorithm = "SHA-256");
    Task<string> SignAsync(string data, string privateKeyId);
    Task<bool> VerifySignatureAsync(string data, string signature, string publicKeyId);
    Task RotateKeyAsync(string keyId);
    Task<KeyMetadata> GetKeyMetadataAsync(string keyId);
}

public class DataEncryptionService : IDataEncryptionService
{
    private readonly IKeyManagementService _keyManagementService;
    private readonly IDataClassificationService _dataClassificationService;
    private readonly IAuditService _auditService;
    private readonly ILogger<DataEncryptionService> _logger;
    private readonly IMetrics _metrics;
    private readonly DataEncryptionConfiguration _configuration;

    public DataEncryptionService(
        IKeyManagementService keyManagementService,
        IDataClassificationService dataClassificationService,
        IAuditService auditService,
        ILogger<DataEncryptionService> logger,
        IMetrics metrics,
        IOptions<DataEncryptionConfiguration> configuration)
    {
        _keyManagementService = keyManagementService ?? throw new ArgumentNullException(nameof(keyManagementService));
        _dataClassificationService = dataClassificationService ?? throw new ArgumentNullException(nameof(dataClassificationService));
        _auditService = auditService ?? throw new ArgumentNullException(nameof(auditService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _metrics = metrics ?? throw new ArgumentNullException(nameof(metrics));
        _configuration = configuration?.Value ?? new DataEncryptionConfiguration();
    }

    public async Task<string> EncryptAsync(string plaintext, string keyId = null)
    {
        using var activity = _metrics.StartTimer("data_encryption_duration");
        
        try
        {
            if (!_configuration.EnableEncryption)
            {
                return plaintext;
            }
            
            if (string.IsNullOrEmpty(plaintext))
            {
                return plaintext;
            }
            
            // Classify data
            var classification = await _dataClassificationService.ClassifyDataAsync(plaintext);
            
            // Determine if encryption is required
            if (!IsEncryptionRequired(classification))
            {
                return plaintext;
            }
            
            // Get or create key
            keyId ??= await GetOrCreateKeyAsync(classification);
            
            // Get encryption key
            var encryptionKey = await _keyManagementService.GetKeyAsync(keyId);
            
            // Encrypt data
            var encryptedData = await EncryptWithAesGcmAsync(plaintext, encryptionKey);
            
            // Log encryption operation
            await LogEncryptionOperationAsync("Encrypt", keyId, classification, encryptedData.Length);
            
            _metrics.Increment("data_encryption_success");
            _logger.LogDebug("Data encrypted successfully with key: {KeyId}", keyId);
            
            return encryptedData;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_encryption_error");
            _logger.LogError(ex, "Error encrypting data");
            throw;
        }
    }

    public async Task<string> DecryptAsync(string ciphertext, string keyId = null)
    {
        using var activity = _metrics.StartTimer("data_decryption_duration");
        
        try
        {
            if (!_configuration.EnableEncryption)
            {
                return ciphertext;
            }
            
            if (string.IsNullOrEmpty(ciphertext) || !IsEncryptedData(ciphertext))
            {
                return ciphertext;
            }
            
            // Extract key ID from ciphertext
            if (string.IsNullOrEmpty(keyId))
            {
                keyId = ExtractKeyIdFromCiphertext(ciphertext);
            }
            
            if (string.IsNullOrEmpty(keyId))
            {
                throw new InvalidOperationException("Key ID not found in ciphertext");
            }
            
            // Get decryption key
            var decryptionKey = await _keyManagementService.GetKeyAsync(keyId);
            
            // Decrypt data
            var decryptedData = await DecryptWithAesGcmAsync(ciphertext, decryptionKey);
            
            // Log decryption operation
            await LogEncryptionOperationAsync("Decrypt", keyId, null, decryptedData.Length);
            
            _metrics.Increment("data_decryption_success");
            _logger.LogDebug("Data decrypted successfully with key: {KeyId}", keyId);
            
            return decryptedData;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_decryption_error");
            _logger.LogError(ex, "Error decrypting data");
            throw;
        }
    }

    public async Task<byte[]> EncryptAsync(byte[] plaintext, string keyId = null)
    {
        using var activity = _metrics.StartTimer("data_encryption_bytes_duration");
        
        try
        {
            if (!_configuration.EnableEncryption)
            {
                return plaintext;
            }
            
            if (plaintext == null || plaintext.Length == 0)
            {
                return plaintext;
            }
            
            // Get or create key
            keyId ??= await GetOrCreateKeyAsync("Confidential");
            
            // Get encryption key
            var encryptionKey = await _keyManagementService.GetKeyAsync(keyId);
            
            // Encrypt data
            var encryptedData = await EncryptBytesWithAesGcmAsync(plaintext, encryptionKey);
            
            // Log encryption operation
            await LogEncryptionOperationAsync("EncryptBytes", keyId, "Confidential", encryptedData.Length);
            
            _metrics.Increment("data_encryption_bytes_success");
            return encryptedData;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_encryption_bytes_error");
            _logger.LogError(ex, "Error encrypting bytes");
            throw;
        }
    }

    public async Task<byte[]> DecryptAsync(byte[] ciphertext, string keyId = null)
    {
        using var activity = _metrics.StartTimer("data_decryption_bytes_duration");
        
        try
        {
            if (!_configuration.EnableEncryption)
            {
                return ciphertext;
            }
            
            if (ciphertext == null || ciphertext.Length == 0)
            {
                return ciphertext;
            }
            
            // Extract key ID from ciphertext
            if (string.IsNullOrEmpty(keyId))
            {
                keyId = ExtractKeyIdFromCiphertextBytes(ciphertext);
            }
            
            if (string.IsNullOrEmpty(keyId))
            {
                throw new InvalidOperationException("Key ID not found in ciphertext");
            }
            
            // Get decryption key
            var decryptionKey = await _keyManagementService.GetKeyAsync(keyId);
            
            // Decrypt data
            var decryptedData = await DecryptBytesWithAesGcmAsync(ciphertext, decryptionKey);
            
            // Log decryption operation
            await LogEncryptionOperationAsync("DecryptBytes", keyId, null, decryptedData.Length);
            
            _metrics.Increment("data_decryption_bytes_success");
            return decryptedData;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_decryption_bytes_error");
            _logger.LogError(ex, "Error decrypting bytes");
            throw;
        }
    }

    public async Task<string> HashAsync(string data, string algorithm = "SHA-256")
    {
        using var activity = _metrics.StartTimer("data_hash_duration");
        
        try
        {
            var hashAlgorithm = HashAlgorithm.Create(algorithm);
            var hashBytes = hashAlgorithm.ComputeHash(Encoding.UTF8.GetBytes(data));
            var hashString = Convert.ToBase64String(hashBytes);
            
            await LogEncryptionOperationAsync("Hash", null, null, hashString.Length);
            
            _metrics.Increment("data_hash_success");
            return hashString;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_hash_error");
            _logger.LogError(ex, "Error hashing data");
            throw;
        }
    }

    public async Task<bool> VerifyHashAsync(string data, string hash, string algorithm = "SHA-256")
    {
        try
        {
            var computedHash = await HashAsync(data, algorithm);
            var isValid = computedHash == hash;
            
            await LogEncryptionOperationAsync("VerifyHash", null, null, hash.Length);
            
            _metrics.Increment("data_hash_verify_success");
            return isValid;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_hash_verify_error");
            _logger.LogError(ex, "Error verifying hash");
            throw;
        }
    }

    public async Task<string> SignAsync(string data, string privateKeyId)
    {
        using var activity = _metrics.StartTimer("data_sign_duration");
        
        try
        {
            var privateKey = await _keyManagementService.GetPrivateKeyAsync(privateKeyId);
            var rsa = RSA.Create();
            rsa.ImportRSAPrivateKey(privateKey, out _);
            
            var dataBytes = Encoding.UTF8.GetBytes(data);
            var signature = rsa.SignData(dataBytes, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
            
            var signatureString = Convert.ToBase64String(signature);
            
            await LogEncryptionOperationAsync("Sign", privateKeyId, null, signatureString.Length);
            
            _metrics.Increment("data_sign_success");
            return signatureString;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_sign_error");
            _logger.LogError(ex, "Error signing data");
            throw;
        }
    }

    public async Task<bool> VerifySignatureAsync(string data, string signature, string publicKeyId)
    {
        try
        {
            var publicKey = await _keyManagementService.GetPublicKeyAsync(publicKeyId);
            var rsa = RSA.Create();
            rsa.ImportRSAPublicKey(publicKey, out _);
            
            var dataBytes = Encoding.UTF8.GetBytes(data);
            var signatureBytes = Convert.FromBase64String(signature);
            
            var isValid = rsa.VerifyData(dataBytes, signatureBytes, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
            
            await LogEncryptionOperationAsync("VerifySignature", publicKeyId, null, signature.Length);
            
            _metrics.Increment("data_signature_verify_success");
            return isValid;
        }
        catch (Exception ex)
        {
            _metrics.Increment("data_signature_verify_error");
            _logger.LogError(ex, "Error verifying signature");
            throw;
        }
    }

    public async Task RotateKeyAsync(string keyId)
    {
        using var activity = _metrics.StartTimer("key_rotation_duration");
        
        try
        {
            _logger.LogInformation("Starting key rotation for key: {KeyId}", keyId);
            
            // Create new key version
            var newKeyId = await _keyManagementService.CreateKeyAsync(keyId, _configuration.DefaultAlgorithm);
            
            // Update key references
            await UpdateKeyReferencesAsync(keyId, newKeyId);
            
            // Mark old key for retirement
            await _keyManagementService.RetireKeyAsync(keyId);
            
            // Log key rotation
            await LogEncryptionOperationAsync("RotateKey", keyId, null, 0);
            
            _metrics.Increment("key_rotation_success");
            _logger.LogInformation("Key rotation completed for key: {KeyId}", keyId);
        }
        catch (Exception ex)
        {
            _metrics.Increment("key_rotation_error");
            _logger.LogError(ex, "Error rotating key: {KeyId}", keyId);
            throw;
        }
    }

    public async Task<KeyMetadata> GetKeyMetadataAsync(string keyId)
    {
        try
        {
            return await _keyManagementService.GetKeyMetadataAsync(keyId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting key metadata: {KeyId}", keyId);
            throw;
        }
    }

    private async Task<string> GetOrCreateKeyAsync(string classification)
    {
        var keyName = $"encryption-{classification.ToLower()}";
        
        try
        {
            // Try to get existing key
            var key = await _keyManagementService.GetKeyMetadataAsync(keyName);
            return key.KeyId;
        }
        catch (KeyNotFoundException)
        {
            // Create new key
            var keyId = await _keyManagementService.CreateKeyAsync(keyName, _configuration.DefaultAlgorithm);
            return keyId;
        }
    }

    private bool IsEncryptionRequired(string classification)
    {
        return classification switch
        {
            "Public" => false,
            "Internal" => false,
            "Confidential" => true,
            "Restricted" => true,
            _ => true
        };
    }

    private async Task<string> EncryptWithAesGcmAsync(string plaintext, byte[] key)
    {
        var plaintextBytes = Encoding.UTF8.GetBytes(plaintext);
        var encryptedBytes = await EncryptBytesWithAesGcmAsync(plaintextBytes, key);
        return Convert.ToBase64String(encryptedBytes);
    }

    private async Task<byte[]> EncryptBytesWithAesGcmAsync(byte[] plaintext, byte[] key)
    {
        using var aes = new AesGcm(key);
        var nonce = new byte[AesGcm.NonceByteSizes.MaxSize];
        var tag = new byte[AesGcm.TagByteSizes.MaxSize];
        var ciphertext = new byte[plaintext.Length];
        
        RandomNumberGenerator.Fill(nonce);
        
        aes.Encrypt(nonce, plaintext, ciphertext, tag);
        
        // Combine nonce, tag, and ciphertext
        var result = new byte[nonce.Length + tag.Length + ciphertext.Length];
        Buffer.BlockCopy(nonce, 0, result, 0, nonce.Length);
        Buffer.BlockCopy(tag, 0, result, nonce.Length, tag.Length);
        Buffer.BlockCopy(ciphertext, 0, result, nonce.Length + tag.Length, ciphertext.Length);
        
        return result;
    }

    private async Task<string> DecryptWithAesGcmAsync(string ciphertext, byte[] key)
    {
        var ciphertextBytes = Convert.FromBase64String(ciphertext);
        var decryptedBytes = await DecryptBytesWithAesGcmAsync(ciphertextBytes, key);
        return Encoding.UTF8.GetString(decryptedBytes);
    }

    private async Task<byte[]> DecryptBytesWithAesGcmAsync(byte[] ciphertext, byte[] key)
    {
        using var aes = new AesGcm(key);
        
        var nonce = new byte[AesGcm.NonceByteSizes.MaxSize];
        var tag = new byte[AesGcm.TagByteSizes.MaxSize];
        var ciphertextOnly = new byte[ciphertext.Length - nonce.Length - tag.Length];
        
        Buffer.BlockCopy(ciphertext, 0, nonce, 0, nonce.Length);
        Buffer.BlockCopy(ciphertext, nonce.Length, tag, 0, tag.Length);
        Buffer.BlockCopy(ciphertext, nonce.Length + tag.Length, ciphertextOnly, 0, ciphertextOnly.Length);
        
        var plaintext = new byte[ciphertextOnly.Length];
        aes.Decrypt(nonce, tag, ciphertextOnly, plaintext);
        
        return plaintext;
    }

    private bool IsEncryptedData(string data)
    {
        try
        {
            var bytes = Convert.FromBase64String(data);
            return bytes.Length >= AesGcm.NonceByteSizes.MaxSize + AesGcm.TagByteSizes.MaxSize;
        }
        catch
        {
            return false;
        }
    }

    private string ExtractKeyIdFromCiphertext(string ciphertext)
    {
        // In a real implementation, key ID would be stored with the ciphertext
        // For simplicity, we'll return a default key ID
        return "encryption-confidential";
    }

    private string ExtractKeyIdFromCiphertextBytes(byte[] ciphertext)
    {
        // In a real implementation, key ID would be stored with the ciphertext
        // For simplicity, we'll return a default key ID
        return "encryption-confidential";
    }

    private async Task UpdateKeyReferencesAsync(string oldKeyId, string newKeyId)
    {
        // Implementation to update references to old key with new key
        await Task.Delay(100); // Simulate database update
    }

    private async Task LogEncryptionOperationAsync(string operation, string keyId, string classification, int dataSize)
    {
        try
        {
            await _auditService.LogAsync(new AuditEvent
            {
                Operation = operation,
                KeyId = keyId,
                Classification = classification,
                DataSize = dataSize,
                Timestamp = DateTime.UtcNow,
                UserId = GetCurrentUserId(),
                IPAddress = GetCurrentIpAddress()
            });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error logging encryption operation");
        }
    }

    private string GetCurrentUserId()
    {
        // Implementation to get current user ID
        return "system";
    }

    private string GetCurrentIpAddress()
    {
        // Implementation to get current IP address
        return "127.0.0.1";
    }
}

// Enhanced Secure Data Controller
[ApiController]
[Route("api/[controller]")]
public class SecureDataController : ControllerBase
{
    private readonly IDataRepository _dataRepository;
    private readonly IDataEncryptionService _encryptionService;
    private readonly ILogger<SecureDataController> _logger;
    private readonly IMetrics _metrics;

    public SecureDataController(
        IDataRepository dataRepository,
        IDataEncryptionService encryptionService,
        ILogger<SecureDataController> logger,
        IMetrics metrics)
    {
        _dataRepository = dataRepository ?? throw new ArgumentNullException(nameof(dataRepository));
        _encryptionService = encryptionService ?? throw new ArgumentNullException(nameof(encryptionService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _metrics = metrics ?? throw new ArgumentNullException(nameof(metrics));
    }

    [HttpPost("data")]
    public async Task<ActionResult> StoreData([FromBody] DataRequest request)
    {
        using var activity = _metrics.StartTimer("secure_data_store_duration");
        
        try
        {
            // Encrypt sensitive data
            var encryptedData = await _encryptionService.EncryptAsync(request.Data);
            
            // Store encrypted data
            await _dataRepository.StoreDataAsync(new DataEntity
            {
                Id = request.Id,
                Data = encryptedData,
                IsEncrypted = true,
                CreatedAt = DateTime.UtcNow
            });
            
            _metrics.Increment("secure_data_store_success");
            _logger.LogInformation("Data stored securely: {DataId}", request.Id);
            
            return Ok(new { Message = "Data stored securely" });
        }
        catch (Exception ex)
        {
            _metrics.Increment("secure_data_store_error");
            _logger.LogError(ex, "Error storing secure data");
            throw;
        }
    }

    [HttpGet("data/{id}")]
    public async Task<ActionResult<string>> GetData(int id)
    {
        using var activity = _metrics.StartTimer("secure_data_retrieve_duration");
        
        try
        {
            // Retrieve encrypted data
            var dataEntity = await _dataRepository.GetDataAsync(id);
            
            if (dataEntity == null)
            {
                return NotFound();
            }
            
            // Decrypt data
            var decryptedData = dataEntity.IsEncrypted 
                ? await _encryptionService.DecryptAsync(dataEntity.Data)
                : dataEntity.Data;
            
            _metrics.Increment("secure_data_retrieve_success");
            _logger.LogInformation("Data retrieved securely: {DataId}", id);
            
            return Ok(decryptedData);
        }
        catch (Exception ex)
        {
            _metrics.Increment("secure_data_retrieve_error");
            _logger.LogError(ex, "Error retrieving secure data: {DataId}", id);
            throw;
        }
    }

    [HttpPost("hash")]
    public async Task<ActionResult<string>> HashData([FromBody] HashRequest request)
    {
        try
        {
            var hash = await _encryptionService.HashAsync(request.Data, request.Algorithm);
            return Ok(hash);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error hashing data");
            throw;
        }
    }

    [HttpPost("sign")]
    public async Task<ActionResult<string>> SignData([FromBody] SignRequest request)
    {
        try
        {
            var signature = await _encryptionService.SignAsync(request.Data, request.PrivateKeyId);
            return Ok(signature);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error signing data");
            throw;
        }
    }

    [HttpPost("verify")]
    public async Task<ActionResult<bool>> VerifySignature([FromBody] VerifyRequest request)
    {
        try
        {
            var isValid = await _encryptionService.VerifySignatureAsync(request.Data, request.Signature, request.PublicKeyId);
            return Ok(isValid);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error verifying signature");
            throw;
        }
    }

    [HttpPost("rotate-key/{keyId}")]
    public async Task<ActionResult> RotateKey(string keyId)
    {
        try
        {
            await _encryptionService.RotateKeyAsync(keyId);
            return Ok(new { Message = $"Key {keyId} rotated successfully" });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error rotating key: {KeyId}", keyId);
            throw;
        }
    }

    [HttpGet("key/{keyId}/metadata")]
    public async Task<ActionResult<KeyMetadata>> GetKeyMetadata(string keyId)
    {
        try
        {
            var metadata = await _encryptionService.GetKeyMetadataAsync(keyId);
            return Ok(metadata);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting key metadata: {KeyId}", keyId);
            throw;
        }
    }
}

// Request/Response Models
public class DataRequest
{
    public int Id { get; set; }
    public string Data { get; set; }
}

public class DataEntity
{
    public int Id { get; set; }
    public string Data { get; set; }
    public bool IsEncrypted { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class HashRequest
{
    public string Data { get; set; }
    public string Algorithm { get; set; } = "SHA-256";
}

public class SignRequest
{
    public string Data { get; set; }
    public string PrivateKeyId { get; set; }
}

public class VerifyRequest
{
    public string Data { get; set; }
    public string Signature { get; set; }
    public string PublicKeyId { get; set; }
}

public class KeyMetadata
{
    public string KeyId { get; set; }
    public string Algorithm { get; set; }
    public int KeySize { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? ExpiresAt { get; set; }
    public bool IsExpired { get; set; }
    public string Status { get; set; }
}

public class AuditEvent
{
    public string Operation { get; set; }
    public string KeyId { get; set; }
    public string Classification { get; set; }
    public int DataSize { get; set; }
    public DateTime Timestamp { get; set; }
    public string UserId { get; set; }
    public string IPAddress { get; set; }
}

// Configuration in Startup
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Add data encryption configuration
        services.Configure<DataEncryptionConfiguration>(options =>
        {
            options.EnableEncryption = true;
            options.EnableKeyRotation = true;
            options.EnableDataClassification = true;
            options.EnableAuditLogging = true;
            options.DefaultAlgorithm = "AES-256-GCM";
            options.KeySize = 256;
            options.KeyRotationDays = 90;
            options.KeyManagementService = "AzureKeyVault";
            options.EncryptedFields = new List<string> { "SSN", "CreditCard", "Email" };
            options.DataClassification = new Dictionary<string, string>
            {
                ["SSN"] = "Restricted",
                ["CreditCard"] = "Restricted",
                ["Email"] = "Confidential",
                ["Name"] = "Internal"
            };
        });

        // Add data encryption services
        services.AddScoped<IDataEncryptionService, DataEncryptionService>();
        services.AddScoped<IKeyManagementService, KeyManagementService>();
        services.AddScoped<IDataClassificationService, DataClassificationService>();
        services.AddScoped<IAuditService, AuditService>();
        
        // Add services
        services.AddScoped<IDataRepository, DataRepository>();
        services.AddControllers();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```

**Framework Integration:**
- **System.Security.Cryptography**: .NET cryptographic libraries
- **Azure Key Vault**: Key management service
- **System.Text.Json**: JSON serialization
- **Dependency Injection**: Service registration
- **Configuration**: Configuration management

============================================================
4. Architecture-Level Diagram (Mermaid)
============================================================

### Data Encryption Architecture
```mermaid
graph TB
    subgraph "Application Layer"
        APP_CONTROLLER[Application Controller]
        ENCRYPTION_SERVICE[Encryption Service]
        CLASSIFICATION_SERVICE[Classification Service]
    end
    
    subgraph "Key Management Layer"
        KEY_VAULT[Key Vault/HSM]
        KEY_MANAGER[Key Manager]
        CERTIFICATE_MANAGER[Certificate Manager]
    end
    
    subgraph "Encryption Engine"
        SYMMETRIC_ENCRYPTION[Symmetric Encryption]
        ASYMMETRIC_ENCRYPTION[Asymmetric Encryption]
        HASHING[Hashing Engine]
        SIGNATURE[Digital Signature]
    end
    
    subgraph "Storage Layer"
        ENCRYPTED_DATABASE[Encrypted Database]
        ENCRYPTED_FILES[Encrypted Files]
        ENCRYPTED_CACHE[Encrypted Cache]
    end
    
    subgraph "Monitoring & Audit"
        AUDIT_LOG[Audit Log]
        SECURITY_MONITORING[Security Monitoring]
        COMPLIANCE_REPORT[Compliance Report]
    end
    
    APP_CONTROLLER --> ENCRYPTION_SERVICE
    APP_CONTROLLER --> CLASSIFICATION_SERVICE
    
    ENCRYPTION_SERVICE --> KEY_VAULT
    ENCRYPTION_SERVICE --> KEY_MANAGER
    ENCRYPTION_SERVICE --> CERTIFICATE_MANAGER
    
    ENCRYPTION_SERVICE --> SYMMETRIC_ENCRYPTION
    ENCRYPTION_SERVICE --> ASYMMETRIC_ENCRYPTION
    ENCRYPTION_SERVICE --> HASHING
    ENCRYPTION_SERVICE --> SIGNATURE
    
    SYMMETRIC_ENCRYPTION --> ENCRYPTED_DATABASE
    ASYMMETRIC_ENCRYPTION --> ENCRYPTED_FILES
    HASHING --> ENCRYPTED_CACHE
    
    ENCRYPTION_SERVICE --> AUDIT_LOG
    AUDIT_LOG --> SECURITY_MONITORING
    SECURITY_MONITORING --> COMPLIANCE_REPORT
```

### Data Encryption Flow
```mermaid
sequenceDiagram
    participant App as Application
    participant Enc as Encryption Service
    participant KM as Key Manager
    participant DB as Database
    participant Audit as Audit Log
    
    App->>Enc: Encrypt Data
    Enc->>KM: Get Encryption Key
    KM-->>Enc: Encryption Key
    Enc->>Enc: Perform Encryption
    Enc->>Audit: Log Encryption
    Enc->>DB: Store Encrypted Data
    
    App->>Enc: Decrypt Data
    Enc->>KM: Get Decryption Key
    KM-->>Enc: Decryption Key
    Enc->>Enc: Perform Decryption
    Enc->>Audit: Log Decryption
    Enc-->>App: Decrypted Data
```

### Key Management Lifecycle
```mermaid
stateDiagram-v2
    [*] --> Key_Generation: Generate Key
    Key_Generation --> Key_Active: Activate Key
    Key_Active --> Key_Rotation: Rotate Key
    Key_Active --> Key_Retirement: Retire Key
    Key_Rotation --> Key_Active: New Key Active
    Key_Retirement --> Key_Archive: Archive Key
    Key_Archive --> Key_Deletion: Delete Key
    Key_Deletion --> [*]
    
    Key_Active --> Key_Compromise: Key Compromised
    Key_Compromise --> Key_Revocation: Revoke Key
    Key_Revocation --> Key_Retirement: Retire Key
```

**Flow Analysis:**
- **Application Layer**: Business logic and data handling
- **Key Management Layer**: Secure key storage and management
- **Encryption Engine**: Core cryptographic operations
- **Storage Layer**: Encrypted data storage
- **Monitoring & Audit**: Security monitoring and compliance

============================================================
5. Scaling & Distributed Systems Impact
============================================================

**100 MB/day:**
- Single encryption service sufficient
- Encryption throughput 10-100 MB/s
- Key management overhead minimal
- Audit logging manageable

**10 GB/day:**
- Encryption service clustering required
- Encryption throughput 100-1000 MB/s
- Key management optimization needed
- Distributed audit logging

**1 TB+ /day:**
- Distributed encryption mandatory
- Encryption throughput 1-10 GB/s
- Advanced key management required
- High-performance audit system

============================================================
6. Production Engineering View
============================================================

**Monitoring Metrics:**
- Encryption/decryption operation counts
- Key rotation frequency and success rates
- Performance metrics (throughput, latency)
- Security events and anomalies
- Compliance status and audit completeness

**Alerting Strategy:**
- Encryption failure rate > 1% (critical)
- Key rotation failures (critical)
- Performance degradation > 20% (warning)
- Unauthorized key access attempts (critical)
- Audit log failures (warning)

**Common Production Incidents:**
- Key management service failures
- Encryption performance degradation
- Key rotation failures
- Audit log corruption

============================================================
7. Tradeoffs & Alternatives
============================================================

**Competing Approaches:**
- **Application-Level Encryption**: Application-managed encryption
- **Database Encryption**: Database-managed encryption
- **File System Encryption**: OS-level encryption
- **Cloud Encryption**: Cloud provider encryption
- **Hardware Security Module**: Hardware-based encryption

**When to Use Data Encryption:**
- Sensitive data storage and transmission
- Regulatory compliance requirements
- Multi-tenant applications
- High-security environments
- Data protection mandates

**When NOT to Use Data Encryption:**
- Public data with no sensitivity
- Performance-critical applications
- Legacy systems with limited resources
- Simple applications with minimal security needs

**Performance Comparison:**
- **Application-Level**: 10-100 MB/s
- **Database Encryption**: 100-1000 MB/s
- **File System Encryption**: 1000-10000 MB/s
- **Cloud Encryption**: 100-1000 MB/s
- **HSM Encryption**: 1-10 MB/s

============================================================
8. Security Implications
============================================================

**Security Benefits:**
- **Data Confidentiality**: Protects sensitive data
- **Data Integrity**: Ensures data integrity
- **Compliance**: Meets regulatory requirements
- **Risk Mitigation**: Reduces breach impact
- **Trust Building**: Builds customer trust

**Security Challenges:**
- **Key Management**: Complex key lifecycle management
- **Performance**: Encryption overhead
- **Complexity**: Complex implementation and maintenance
- **Key Security**: Protecting encryption keys
- **Recovery**: Key recovery and data recovery

============================================================
9. Advanced Edge Cases
============================================================

**Advanced Encryption Features:**
- **Homomorphic Encryption**: Computation on encrypted data
- **Quantum-Resistant Encryption**: Post-quantum cryptography
- **Zero-Knowledge Proofs**: Privacy-preserving proofs
- **Secure Multi-Party Computation**: Distributed computation
- **Attribute-Based Encryption**: Fine-grained access control

**Advanced Key Management:**
- **Distributed Key Generation**: Shared key generation
- **Threshold Cryptography**: Multi-party key management
- **Key Escrow**: Secure key recovery
- **Hardware Security Modules**: Hardware-based key protection
- **Cloud Key Management**: Cloud-based key services

============================================================
10. Interview & Staff-Level Deep Dive
============================================================

**What a junior answers:**
"Data encryption is about converting plaintext to ciphertext to protect sensitive information. I use AES for symmetric encryption and RSA for asymmetric encryption. I also hash passwords and use digital signatures for authentication."

**What a senior answers:**
"Data encryption is a comprehensive approach to protecting sensitive data through cryptographic techniques. I've implemented production encryption systems with AES-256-GCM for data-at-rest, RSA for key exchange, SHA-256 for hashing, and digital signatures for integrity. The implementation includes key management with Azure Key Vault, automatic key rotation, data classification, comprehensive audit logging, and performance optimization. I've also implemented field-level encryption, database encryption, and secure key lifecycle management."

**What an architect answers:**
"Data encryption is a fundamental security capability that protects sensitive information through cryptographic transformation. At scale, data encryption must provide comprehensive protection through symmetric and asymmetric encryption, secure key management, data classification, audit logging, and compliance support. The architecture must support multiple encryption algorithms, key lifecycle management, performance optimization, distributed key management, and comprehensive monitoring. In enterprise environments, data encryption is not just a security feature but a strategic capability that protects business assets, ensures regulatory compliance, and enables digital transformation. The implementation must consider scalability, performance, key security, compliance requirements, and operational excellence."

**Follow-up questions interviewers ask:**
- "How do you handle key rotation in production?"
- "What's the difference between symmetric and asymmetric encryption?"
- "How do you implement field-level encryption?"
- "When would you use database vs application-level encryption?"
- "How do you handle encryption key backup and recovery?"

**Trick questions:**
- "Is encryption always secure?" (No, depends on implementation)
- "Can encrypted data be searched?" (Not directly, requires special techniques)
- "Do all applications need the same encryption level?" (No, depends on data sensitivity)
- "Is key management the hardest part of encryption?" (Often yes)

============================================================
11. Real Enterprise Case Study
============================================================

**Scenario: Global Healthcare Platform with 10M+ Patient Records**

**Requirements:**
- Secure 10M+ patient records
- Comply with HIPAA and GDPR regulations
- Implement field-level encryption for PHI
- Support secure data sharing between providers
- Maintain high availability and performance
- Implement comprehensive audit logging

**Results:**
- Achieved 99.99% data security
- 100% compliance with healthcare regulations
- 95% reduction in data breach risk
- 80% improvement in audit efficiency
- 99.95% uptime during encryption operations

============================================================
12. Dependency Graph
============================================================

**Prerequisites:**
- **Cryptography**: Understanding of cryptographic principles
- **Security**: Understanding of security fundamentals
- **Database**: Understanding of database systems
- **Networking**: Understanding of network security
- **Compliance**: Understanding of regulatory requirements

**Concepts that depend on Data Encryption:**
- **API Security**: API data protection
- **Database Security**: Database encryption
- **Cloud Security**: Cloud data protection
- **Compliance**: Regulatory compliance
- **Digital Signatures**: Authentication and integrity
- **Secure Communication**: Secure data transmission
