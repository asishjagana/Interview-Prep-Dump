# API Security - Architect Deep Dive

**0. Executive Summary (Architect View)**
- **API Security exists** to protect application programming interfaces from unauthorized access and attacks
- **Critical at scale** when APIs are the primary interface for applications and services
- **System design impact** - transforms API development from functional to security-first approach
- **Interface protection** - provides essential protection for API endpoints and data
- **Authentication foundation** - enables secure API authentication and authorization
- **Data protection** - protects sensitive data in transit and at rest
- **Compliance requirement** - mandatory for security standards and regulations
- **Business continuity** - maintains API availability and integrity

**1. Problem Framing**
**Real production problem solved:**
- Unauthorized API access and data breaches
- API abuse and rate limiting violations
- Data exposure through insecure endpoints
- Authentication and authorization bypass
- API denial of service attacks

**What breaks if we don't understand API Security:**
- API compromise and data breaches
- Authentication and authorization bypass
- Service abuse and resource exhaustion
- Compliance violations and legal consequences
- Business disruption and reputational damage

**Scale importance:**
- **Small applications**: API security essential
- **Medium applications**: API security critical
- **Large applications**: API security mandatory
- **Enterprise systems**: API security fundamental

**Failure stories:**
- E-commerce platform had customer data breach through API
- Banking system had unauthorized transactions via API
- Healthcare platform had patient data exposure through API
- Social media platform had API abuse and data scraping

**2. Deep Internal Mechanics**

**API Security Fundamentals:**
- **Authentication**: Verifying API client identity
- **Authorization**: Controlling API access permissions
- **Input Validation**: Validating API input data
- **Output Encoding**: Secure API response encoding
- **Rate Limiting**: Controlling API request rates

**Core Components:**
- **API Gateway**: Central API management and security
- **Authentication Service**: API authentication and token management
- **Authorization Service**: API authorization and permission management
- **Rate Limiting Service**: API rate limiting and throttling
- **Security Monitoring**: API security monitoring and alerting
- **WAF Integration**: Web Application Firewall integration

**Key Concepts:**
- **OAuth 2.0**: Authorization framework
- **JWT Tokens**: JSON Web Tokens for authentication
- **API Keys**: API key authentication
- **Mutual TLS**: Certificate-based authentication
- **Rate Limiting**: Request rate control
- **Input Validation**: Input data validation
- **Output Encoding**: Response data encoding

**Security Layers:**
- **Network Security**: Network-level protection
- **Transport Security**: TLS/SSL encryption
- **Application Security**: Application-level security
- **Data Security**: Data protection and encryption
- **Monitoring Security**: Security monitoring and detection

**Attack Vectors:**
- **Injection Attacks**: SQL injection, NoSQL injection
- **Authentication Bypass**: Token manipulation, session hijacking
- **Authorization Bypass**: Privilege escalation, access control bypass
- **Data Exposure**: Sensitive data leakage
- **Denial of Service**: API abuse and resource exhaustion

**Data Structures:**
- **API Registry**: API metadata and configuration
- **Token Registry**: Token metadata and state
- **Security Event Registry**: Security event logging
- **Rate Limit Registry**: Rate limit state and configuration
- **Permission Registry**: Permission and role management

**Performance Characteristics:**
- **Authentication**: 5-50ms
- **Authorization**: 1-10ms
- **Input Validation**: 1-5ms
- **Rate Limiting**: 1-3ms
- **Security Monitoring**: 1-10ms

**3. .NET Production Implementation**

**Naive Implementation (Insecure API):**
```csharp
[ApiController]
[Route("api/[controller]")]
public class InsecureUserController : ControllerBase
{
    private readonly IUserService _userService;
    
    public InsecureUserController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("users/{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        // No authentication or authorization
        var user = await _userService.GetUserAsync(id);
        return Ok(user);
    }
    
    [HttpPost("users")]
    public async Task<ActionResult<User>> CreateUser([FromBody] User user)
    {
        // No input validation
        var createdUser = await _userService.CreateUserAsync(user);
        return Ok(createdUser);
    }
}
```

**Production-Ready API Security Implementation:**
```csharp
// API Security Configuration
public class APISecurityConfiguration
{
    public bool EnableAuthentication { get; set; } = true;
    public bool EnableAuthorization { get; set; } = true;
    public bool EnableRateLimiting { get; set; } = true;
    public bool EnableInputValidation { get; set; } = true;
    public bool EnableOutputEncoding { get; set; } = true;
    public bool EnableSecurityMonitoring { get; set; } = true;
    public int MaxRequestsPerMinute { get; set; } = 100;
    public int MaxRequestsPerHour { get; set; } = 1000;
    public List<string> AllowedOrigins { get; set; } = new();
    public List<string> BlockedIPs { get; set; } = new();
    public TokenValidationParameters TokenValidation { get; set; } = new();
}

public class TokenValidationParameters
{
    public string Issuer { get; set; }
    public string Audience { get; set; }
    public string SecretKey { get; set; }
    public int TokenExpirationMinutes { get; set; } = 60;
    public int RefreshTokenExpirationDays { get; set; } = 7;
}

// API Security Service
public interface IAPISecurityService
{
    Task<bool> AuthenticateAsync(HttpContext context);
    Task<bool> AuthorizeAsync(HttpContext context, string requiredPermission);
    Task<bool> ValidateInputAsync(HttpContext context);
    Task<bool> CheckRateLimitAsync(HttpContext context);
    Task LogSecurityEventAsync(SecurityEvent securityEvent);
    Task<bool> IsIPBlockedAsync(string ipAddress);
}

public class APISecurityService : IAPISecurityService
{
    private readonly ITokenService _tokenService;
    private readonly IAuthorizationService _authorizationService;
    private readonly IInputValidationService _inputValidationService;
    private readonly IRateLimitingService _rateLimitingService;
    private readonly ISecurityMonitoringService _securityMonitoringService;
    private readonly ILogger<APISecurityService> _logger;
    private readonly IMetrics _metrics;
    private readonly APISecurityConfiguration _configuration;

    public APISecurityService(
        ITokenService tokenService,
        IAuthorizationService authorizationService,
        IInputValidationService inputValidationService,
        IRateLimitingService rateLimitingService,
        ISecurityMonitoringService securityMonitoringService,
        ILogger<APISecurityService> logger,
        IMetrics metrics,
        IOptions<APISecurityConfiguration> configuration)
    {
        _tokenService = tokenService ?? throw new ArgumentNullException(nameof(tokenService));
        _authorizationService = authorizationService ?? throw new ArgumentNullException(nameof(authorizationService));
        _inputValidationService = inputValidationService ?? throw new ArgumentNullException(nameof(inputValidationService));
        _rateLimitingService = rateLimitingService ?? throw new ArgumentNullException(nameof(rateLimitingService));
        _securityMonitoringService = securityMonitoringService ?? throw new ArgumentNullException(nameof(securityMonitoringService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _metrics = metrics ?? throw new ArgumentNullException(nameof(metrics));
        _configuration = configuration?.Value ?? new APISecurityConfiguration();
    }

    public async Task<bool> AuthenticateAsync(HttpContext context)
    {
        using var activity = _metrics.StartTimer("api_security_authentication_duration");
        
        try
        {
            if (!_configuration.EnableAuthentication)
            {
                return true;
            }
            
            var ipAddress = GetClientIpAddress(context);
            
            // Check if IP is blocked
            if (await IsIPBlockedAsync(ipAddress))
            {
                await LogSecurityEventAsync(new SecurityEvent
                {
                    Type = "BlockedIPAccess",
                    IPAddress = ipAddress,
                    UserAgent = context.Request.Headers["User-Agent"].ToString(),
                    Timestamp = DateTime.UtcNow,
                    Endpoint = context.Request.Path,
                    Severity = "High"
                });
                
                return false;
            }
            
            // Extract token from Authorization header
            var authHeader = context.Request.Headers["Authorization"].FirstOrDefault();
            if (string.IsNullOrEmpty(authHeader) || !authHeader.StartsWith("Bearer "))
            {
                await LogSecurityEventAsync(new SecurityEvent
                {
                    Type = "MissingToken",
                    IPAddress = ipAddress,
                    UserAgent = context.Request.Headers["User-Agent"].ToString(),
                    Timestamp = DateTime.UtcNow,
                    Endpoint = context.Request.Path,
                    Severity = "Medium"
                });
                
                return false;
            }
            
            var token = authHeader.Substring("Bearer ".Length);
            
            // Validate token
            var validationResult = await _tokenService.ValidateTokenAsync(token);
            
            if (!validationResult.IsValid)
            {
                await LogSecurityEventAsync(new SecurityEvent
                {
                    Type = "InvalidToken",
                    IPAddress = ipAddress,
                    UserAgent = context.Request.Headers["User-Agent"].ToString(),
                    Timestamp = DateTime.UtcNow,
                    Endpoint = context.Request.Path,
                    Severity = "High",
                    Details = validationResult.ErrorMessage
                });
                
                return false;
            }
            
            // Add user context to request
            context.Items["UserId"] = validationResult.UserId;
            context.Items["UserRoles"] = validationResult.Roles;
            
            _metrics.Increment("api_security_authentication_success");
            return true;
        }
        catch (Exception ex)
        {
            _metrics.Increment("api_security_authentication_error");
            _logger.LogError(ex, "Error during API authentication");
            
            await LogSecurityEventAsync(new SecurityEvent
            {
                Type = "AuthenticationError",
                IPAddress = GetClientIpAddress(context),
                UserAgent = context.Request.Headers["User-Agent"].ToString(),
                Timestamp = DateTime.UtcNow,
                Endpoint = context.Request.Path,
                Severity = "High",
                Details = ex.Message
            });
            
            return false;
        }
    }

    public async Task<bool> AuthorizeAsync(HttpContext context, string requiredPermission)
    {
        using var activity = _metrics.StartTimer("api_security_authorization_duration");
        
        try
        {
            if (!_configuration.EnableAuthorization)
            {
                return true;
            }
            
            if (!context.Items.ContainsKey("UserId"))
            {
                return false;
            }
            
            var userId = context.Items["UserId"].ToString();
            var userRoles = context.Items["UserRoles"] as List<string> ?? new List<string>();
            
            // Check authorization
            var isAuthorized = await _authorizationService.HasPermissionAsync(userId, userRoles, requiredPermission);
            
            if (!isAuthorized)
            {
                await LogSecurityEventAsync(new SecurityEvent
                {
                    Type = "AuthorizationDenied",
                    IPAddress = GetClientIpAddress(context),
                    UserAgent = context.Request.Headers["User-Agent"].ToString(),
                    Timestamp = DateTime.UtcNow,
                    Endpoint = context.Request.Path,
                    Severity = "Medium",
                    UserId = userId,
                    RequiredPermission = requiredPermission
                });
            }
            
            _metrics.Increment(isAuthorized ? "api_security_authorization_success" : "api_security_authorization_denied");
            return isAuthorized;
        }
        catch (Exception ex)
        {
            _metrics.Increment("api_security_authorization_error");
            _logger.LogError(ex, "Error during API authorization");
            return false;
        }
    }

    public async Task<bool> ValidateInputAsync(HttpContext context)
    {
        using var activity = _metrics.StartTimer("api_security_input_validation_duration");
        
        try
        {
            if (!_configuration.EnableInputValidation)
            {
                return true;
            }
            
            // Validate request size
            if (context.Request.ContentLength > 10 * 1024 * 1024) // 10MB limit
            {
                await LogSecurityEventAsync(new SecurityEvent
                {
                    Type = "RequestSizeExceeded",
                    IPAddress = GetClientIpAddress(context),
                    UserAgent = context.Request.Headers["User-Agent"].ToString(),
                    Timestamp = DateTime.UtcNow,
                    Endpoint = context.Request.Path,
                    Severity = "Medium",
                    Details = $"Request size: {context.Request.ContentLength}"
                });
                
                return false;
            }
            
            // Validate headers
            var suspiciousHeaders = new[] { "X-Forwarded-For", "X-Real-IP", "X-Originating-IP" };
            foreach (var header in suspiciousHeaders)
            {
                if (context.Request.Headers.ContainsKey(header))
                {
                    var headerValue = context.Request.Headers[header].ToString();
                    if (IsValidIPAddress(headerValue))
                    {
                        await LogSecurityEventAsync(new SecurityEvent
                        {
                            Type = "SuspiciousHeader",
                            IPAddress = GetClientIpAddress(context),
                            UserAgent = context.Request.Headers["User-Agent"].ToString(),
                            Timestamp = DateTime.UtcNow,
                            Endpoint = context.Request.Path,
                            Severity = "Low",
                            Details = $"Header: {header}={headerValue}"
                        });
                    }
                }
            }
            
            // Validate query parameters
            foreach (var queryParam in context.Request.Query)
            {
                if (ContainsSuspiciousContent(queryParam.Value))
                {
                    await LogSecurityEventAsync(new SecurityEvent
                    {
                        Type = "SuspiciousQueryParameter",
                        IPAddress = GetClientIpAddress(context),
                        UserAgent = context.Request.Headers["User-Agent"].ToString(),
                        Timestamp = DateTime.UtcNow,
                        Endpoint = context.Request.Path,
                        Severity = "Medium",
                        Details = $"Parameter: {queryParam.Key}={queryParam.Value}"
                    });
                    
                    return false;
                }
            }
            
            _metrics.Increment("api_security_input_validation_success");
            return true;
        }
        catch (Exception ex)
        {
            _metrics.Increment("api_security_input_validation_error");
            _logger.LogError(ex, "Error during API input validation");
            return false;
        }
    }

    public async Task<bool> CheckRateLimitAsync(HttpContext context)
    {
        using var activity = _metrics.StartTimer("api_security_rate_limit_duration");
        
        try
        {
            if (!_configuration.EnableRateLimiting)
            {
                return true;
            }
            
            var ipAddress = GetClientIpAddress(context);
            var endpoint = context.Request.Path;
            
            // Check rate limits
            var rateLimitResult = await _rateLimitingService.CheckRateLimitAsync(ipAddress, endpoint);
            
            if (!rateLimitResult.IsAllowed)
            {
                await LogSecurityEventAsync(new SecurityEvent
                {
                    Type = "RateLimitExceeded",
                    IPAddress = ipAddress,
                    UserAgent = context.Request.Headers["User-Agent"].ToString(),
                    Timestamp = DateTime.UtcNow,
                    Endpoint = endpoint,
                    Severity = "Medium",
                    Details = $"Requests: {rateLimitResult.CurrentRequests}, Limit: {rateLimitResult.Limit}"
                });
                
                context.Response.Headers["Retry-After"] = rateLimitResult.RetryAfter?.ToString();
            }
            
            _metrics.Increment(rateLimitResult.IsAllowed ? "api_security_rate_limit_success" : "api_security_rate_limit_exceeded");
            return rateLimitResult.IsAllowed;
        }
        catch (Exception ex)
        {
            _metrics.Increment("api_security_rate_limit_error");
            _logger.LogError(ex, "Error during API rate limit check");
            return false;
        }
    }

    public async Task LogSecurityEventAsync(SecurityEvent securityEvent)
    {
        try
        {
            await _securityMonitoringService.LogEventAsync(securityEvent);
            _metrics.Increment("api_security_events_logged");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error logging security event");
        }
    }

    public async Task<bool> IsIPBlockedAsync(string ipAddress)
    {
        try
        {
            return _configuration.BlockedIPs.Contains(ipAddress) || 
                   await _securityMonitoringService.IsIPBlockedAsync(ipAddress);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking if IP is blocked: {IPAddress}", ipAddress);
            return false;
        }
    }

    private string GetClientIpAddress(HttpContext context)
    {
        var ipAddress = context.Connection.RemoteIpAddress?.ToString();
        
        // Check for forwarded IP addresses
        if (context.Request.Headers.ContainsKey("X-Forwarded-For"))
        {
            var forwardedIps = context.Request.Headers["X-Forwarded-For"].ToString().Split(',');
            if (forwardedIps.Length > 0)
            {
                ipAddress = forwardedIps[0].Trim();
            }
        }
        
        return ipAddress ?? "unknown";
    }

    private bool IsValidIPAddress(string ipAddress)
    {
        return System.Net.IPAddress.TryParse(ipAddress, out _);
    }

    private bool ContainsSuspiciousContent(string content)
    {
        var suspiciousPatterns = new[]
        {
            "(?i)(select|insert|update|delete|drop|create|alter|exec|execute)",
            "(?i)(script|javascript|vbscript|onload|onerror)",
            "(?i)(<|>|&lt;|&gt;|&amp;)",
            "(?i)(union|or|and)\\s+\\d+\\s*=\\s*\\d+",
            "(?i)(--|#|/\\*|\\*/)"
        };

        return suspiciousPatterns.Any(pattern => Regex.IsMatch(content, pattern));
    }
}

// Enhanced Secure API Controller
[ApiController]
[Route("api/[controller]")]
public class SecureUserController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IAPISecurityService _apiSecurityService;
    private readonly ILogger<SecureUserController> _logger;
    private readonly IMetrics _metrics;

    public SecureUserController(
        IUserService userService,
        IAPISecurityService apiSecurityService,
        ILogger<SecureUserController> logger,
        IMetrics metrics)
    {
        _userService = userService ?? throw new ArgumentNullException(nameof(userService));
        _apiSecurityService = apiSecurityService ?? throw new ArgumentNullException(nameof(apiSecurityService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _metrics = metrics ?? throw new ArgumentNullException(nameof(metrics));
    }

    [HttpGet("users/{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        using var activity = _metrics.StartTimer("api_user_get_duration");
        
        try
        {
            // Authenticate
            if (!await _apiSecurityService.AuthenticateAsync(HttpContext))
            {
                return Unauthorized(new { Error = "Authentication failed" });
            }
            
            // Authorize
            if (!await _apiSecurityService.AuthorizeAsync(HttpContext, "users:read"))
            {
                return Forbid();
            }
            
            // Validate input
            if (!await _apiSecurityService.ValidateInputAsync(HttpContext))
            {
                return BadRequest(new { Error = "Invalid input" });
            }
            
            // Check rate limit
            if (!await _apiSecurityService.CheckRateLimitAsync(HttpContext))
            {
                return StatusCode(429, new { Error = "Rate limit exceeded" });
            }
            
            var user = await _userService.GetUserAsync(id);
            
            if (user == null)
            {
                return NotFound();
            }
            
            // Sanitize output
            var sanitizedUser = SanitizeUserOutput(user);
            
            _metrics.Increment("api_user_get_success");
            _logger.LogInformation("User retrieved successfully: {UserId}", id);
            
            return Ok(sanitizedUser);
        }
        catch (Exception ex)
        {
            _metrics.Increment("api_user_get_error");
            _logger.LogError(ex, "Error getting user: {UserId}", id);
            throw;
        }
    }

    [HttpPost("users")]
    public async Task<ActionResult<User>> CreateUser([FromBody] CreateUserRequest request)
    {
        using var activity = _metrics.StartTimer("api_user_create_duration");
        
        try
        {
            // Authenticate
            if (!await _apiSecurityService.AuthenticateAsync(HttpContext))
            {
                return Unauthorized(new { Error = "Authentication failed" });
            }
            
            // Authorize
            if (!await _apiSecurityService.AuthorizeAsync(HttpContext, "users:create"))
            {
                return Forbid();
            }
            
            // Validate input
            if (!await _apiSecurityService.ValidateInputAsync(HttpContext))
            {
                return BadRequest(new { Error = "Invalid input" });
            }
            
            // Check rate limit
            if (!await _apiSecurityService.CheckRateLimitAsync(HttpContext))
            {
                return StatusCode(429, new { Error = "Rate limit exceeded" });
            }
            
            // Validate request data
            if (!IsValidCreateUserRequest(request))
            {
                return BadRequest(new { Error = "Invalid request data" });
            }
            
            var user = await _userService.CreateUserAsync(new User
            {
                Email = request.Email,
                Name = request.Name,
                Role = request.Role
            });
            
            // Sanitize output
            var sanitizedUser = SanitizeUserOutput(user);
            
            _metrics.Increment("api_user_create_success");
            _logger.LogInformation("User created successfully: {UserId}", user.Id);
            
            return CreatedAtAction(nameof(GetUser), new { id = user.Id }, sanitizedUser);
        }
        catch (Exception ex)
        {
            _metrics.Increment("api_user_create_error");
            _logger.LogError(ex, "Error creating user");
            throw;
        }
    }

    private bool IsValidCreateUserRequest(CreateUserRequest request)
    {
        if (string.IsNullOrEmpty(request.Email) || string.IsNullOrEmpty(request.Name))
        {
            return false;
        }
        
        if (!IsValidEmail(request.Email))
        {
            return false;
        }
        
        if (request.Name.Length > 100)
        {
            return false;
        }
        
        return true;
    }

    private bool IsValidEmail(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }

    private User SanitizeUserOutput(User user)
    {
        return new User
        {
            Id = user.Id,
            Email = WebUtility.HtmlEncode(user.Email),
            Name = WebUtility.HtmlEncode(user.Name),
            Role = user.Role,
            CreatedAt = user.CreatedAt
            // Remove sensitive fields like password hash, etc.
        };
    }
}

// API Security Middleware
public class APISecurityMiddleware
{
    private readonly IAPISecurityService _apiSecurityService;
    private readonly ILogger<APISecurityMiddleware> _logger;
    private readonly IMetrics _metrics;
    private readonly APISecurityConfiguration _configuration;

    public APISecurityMiddleware(
        IAPISecurityService apiSecurityService,
        ILogger<APISecurityMiddleware> logger,
        IMetrics metrics,
        IOptions<APISecurityConfiguration> configuration)
    {
        _apiSecurityService = apiSecurityService ?? throw new ArgumentNullException(nameof(apiSecurityService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _metrics = metrics ?? throw new ArgumentNullException(nameof(metrics));
        _configuration = configuration?.Value ?? new APISecurityConfiguration();
    }

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        using var activity = _metrics.StartTimer("api_security_middleware_duration");
        
        try
        {
            // Add security headers
            AddSecurityHeaders(context);
            
            // Check CORS
            if (!CheckCORS(context))
            {
                context.Response.StatusCode = 403;
                await context.Response.WriteAsync("CORS policy violation");
                return;
            }
            
            // Authenticate
            if (!await _apiSecurityService.AuthenticateAsync(context))
            {
                context.Response.StatusCode = 401;
                await context.Response.WriteAsync("Authentication failed");
                return;
            }
            
            // Validate input
            if (!await _apiSecurityService.ValidateInputAsync(context))
            {
                context.Response.StatusCode = 400;
                await context.Response.WriteAsync("Invalid input");
                return;
            }
            
            // Check rate limit
            if (!await _apiSecurityService.CheckRateLimitAsync(context))
            {
                context.Response.StatusCode = 429;
                await context.Response.WriteAsync("Rate limit exceeded");
                return;
            }
            
            await next(context);
        }
        catch (Exception ex)
        {
            _metrics.Increment("api_security_middleware_error");
            _logger.LogError(ex, "Error in API security middleware");
            
            context.Response.StatusCode = 500;
            await context.Response.WriteAsync("Internal server error");
        }
    }

    private void AddSecurityHeaders(HttpContext context)
    {
        context.Response.Headers["X-Content-Type-Options"] = "nosniff";
        context.Response.Headers["X-Frame-Options"] = "DENY";
        context.Response.Headers["X-XSS-Protection"] = "1; mode=block";
        context.Response.Headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains";
        context.Response.Headers["Content-Security-Policy"] = "default-src 'self'";
    }

    private bool CheckCORS(HttpContext context)
    {
        var origin = context.Request.Headers["Origin"].ToString();
        
        if (string.IsNullOrEmpty(origin))
        {
            return true; // Same origin request
        }
        
        return _configuration.AllowedOrigins.Contains(origin) || 
               _configuration.AllowedOrigins.Contains("*");
    }
}

// Request/Response Models
public class CreateUserRequest
{
    public string Email { get; set; }
    public string Name { get; set; }
    public string Role { get; set; }
}

public class SecurityEvent
{
    public string Type { get; set; }
    public string IPAddress { get; set; }
    public string UserAgent { get; set; }
    public DateTime Timestamp { get; set; }
    public string Endpoint { get; set; }
    public string Severity { get; set; }
    public string Details { get; set; }
    public string UserId { get; set; }
    public string RequiredPermission { get; set; }
}

public class TokenValidationResult
{
    public bool IsValid { get; set; }
    public string UserId { get; set; }
    public List<string> Roles { get; set; }
    public string ErrorMessage { get; set; }
}

public class RateLimitResult
{
    public bool IsAllowed { get; set; }
    public int CurrentRequests { get; set; }
    public int Limit { get; set; }
    public TimeSpan? RetryAfter { get; set; }
}

// Configuration in Startup
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Add API security configuration
        services.Configure<APISecurityConfiguration>(options =>
        {
            options.EnableAuthentication = true;
            options.EnableAuthorization = true;
            options.EnableRateLimiting = true;
            options.EnableInputValidation = true;
            options.EnableOutputEncoding = true;
            options.EnableSecurityMonitoring = true;
            options.MaxRequestsPerMinute = 100;
            options.MaxRequestsPerHour = 1000;
            options.AllowedOrigins = new List<string> { "https://example.com" };
            options.BlockedIPs = new List<string>();
            options.TokenValidation = new TokenValidationParameters
            {
                Issuer = "https://example.com",
                Audience = "api.example.com",
                SecretKey = "your-secret-key",
                TokenExpirationMinutes = 60,
                RefreshTokenExpirationDays = 7
            };
        });

        // Add API security services
        services.AddScoped<IAPISecurityService, APISecurityService>();
        services.AddScoped<ITokenService, TokenService>();
        services.AddScoped<IAuthorizationService, AuthorizationService>();
        services.AddScoped<IInputValidationService, InputValidationService>();
        services.AddScoped<IRateLimitingService, RateLimitingService>();
        services.AddScoped<ISecurityMonitoringService, SecurityMonitoringService>();
        
        // Add services
        services.AddScoped<IUserService, UserService>();
        services.AddControllers();
        
        // Add authentication
        services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.TokenValidationParameters = new Microsoft.IdentityModel.Tokens.TokenValidationParameters
                {
                    ValidateIssuer = true,
                    ValidateAudience = true,
                    ValidateLifetime = true,
                    ValidateIssuerSigningKey = true,
                    ValidIssuer = "https://example.com",
                    ValidAudience = "api.example.com",
                    IssuerSigningKey = new Microsoft.IdentityModel.Tokens.SymmetricSecurityKey(
                        System.Text.Encoding.UTF8.GetBytes("your-secret-key"))
                };
            });
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        
        // Add API security middleware
        app.UseMiddleware<APISecurityMiddleware>();
        
        app.UseAuthentication();
        app.UseAuthorization();
        
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```

**Framework Integration:**
- **ASP.NET Core**: Web API framework
- **JWT Bearer Authentication**: JWT token authentication
- **System.Text.Json**: JSON serialization
- **Dependency Injection**: Service registration
- **Configuration**: Configuration management

**4. Architecture-Level Diagram (Mermaid)**

### API Security Architecture
```mermaid
graph TB
    subgraph "Client Layer"
        WEB_CLIENT[Web Client]
        MOBILE_CLIENT[Mobile Client]
        API_CLIENT[API Client]
    end
    
    subgraph "Security Layer"
        API_GATEWAY[API Gateway]
        AUTHENTICATION[Authentication Service]
        AUTHORIZATION[Authorization Service]
        RATE_LIMITING[Rate Limiting Service]
        INPUT_VALIDATION[Input Validation Service]
        SECURITY_MONITORING[Security Monitoring]
    end
    
    subgraph "Application Layer"
        API_CONTROLLER[API Controller]
        BUSINESS_LOGIC[Business Logic]
        DATA_ACCESS[Data Access]
    end
    
    subgraph "Data Layer"
        DATABASE[Database]
        CACHE[Cache]
        LOGS[Security Logs]
    end
    
    subgraph "External Services"
        TOKEN_SERVICE[Token Service]
        WAF[Web Application Firewall]
        THREAT_INTEL[Threat Intelligence]
    end
    
    WEB_CLIENT --> API_GATEWAY
    MOBILE_CLIENT --> API_GATEWAY
    API_CLIENT --> API_GATEWAY
    
    API_GATEWAY --> AUTHENTICATION
    API_GATEWAY --> AUTHORIZATION
    API_GATEWAY --> RATE_LIMITING
    API_GATEWAY --> INPUT_VALIDATION
    API_GATEWAY --> SECURITY_MONITORING
    
    AUTHENTICATION --> TOKEN_SERVICE
    AUTHORIZATION --> TOKEN_SERVICE
    SECURITY_MONITORING --> THREAT_INTEL
    API_GATEWAY --> WAF
    
    API_GATEWAY --> API_CONTROLLER
    API_CONTROLLER --> BUSINESS_LOGIC
    BUSINESS_LOGIC --> DATA_ACCESS
    
    DATA_ACCESS --> DATABASE
    DATA_ACCESS --> CACHE
    SECURITY_MONITORING --> LOGS
```

### API Security Flow
```mermaid
sequenceDiagram
    participant Client as Client
    participant Gateway as API Gateway
    participant Auth as Authentication
    participant Authz as Authorization
    participant RateLimit as Rate Limiting
    participant Validation as Input Validation
    participant API as API Controller
    
    Client->>Gateway: API Request
    Gateway->>Auth: Authenticate Request
    Auth-->>Gateway: Authentication Result
    Gateway->>Authz: Authorize Request
    Authz-->>Gateway: Authorization Result
    Gateway->>RateLimit: Check Rate Limit
    RateLimit-->>Gateway: Rate Limit Result
    Gateway->>Validation: Validate Input
    Validation-->>Gateway: Validation Result
    Gateway->>API: Forward Request
    API-->>Gateway: API Response
    Gateway-->>Client: Secure Response
```

### API Security Layers
```mermaid
graph TD
    A[API Request] --> B{Security Layers}
    
    B --> C[Network Security]
    B --> D[Transport Security]
    B --> E[Application Security]
    B --> F[Data Security]
    B --> G[Monitoring Security]
    
    C --> C1[Firewall Rules]
    C --> C2[IP Whitelisting]
    C --> C3[DDoS Protection]
    
    D --> D1[TLS/SSL Encryption]
    D --> D2[Certificate Validation]
    D --> D3[Secure Headers]
    
    E --> E1[Authentication]
    E --> E2[Authorization]
    E --> E3[Input Validation]
    E --> E4[Output Encoding]
    
    F --> F1[Data Encryption]
    F --> F2[Key Management]
    F --> F3[Data Masking]
    
    G --> G1[Security Logging]
    G --> G2[Threat Detection]
    G --> G3[Alerting]
    
    C1 --> H[Secure API Response]
    C2 --> H
    C3 --> H
    D1 --> H
    D2 --> H
    D3 --> H
    E1 --> H
    E2 --> H
    E3 --> H
    E4 --> H
    F1 --> H
    F2 --> H
    F3 --> H
    G1 --> H
    G2 --> H
    G3 --> H
```

**Flow Analysis:**
- **Client Layer**: API client applications
- **Security Layer**: Comprehensive security services
- **Application Layer**: Business logic and data access
- **Data Layer**: Data storage and security logging
- **External Services**: Token service and threat intelligence

**5. Scaling & Distributed Systems Impact**

**100 RPS:**
- Single API gateway sufficient
- Authentication < 10ms
- Authorization < 5ms
- Rate limiting < 2ms
- Input validation < 3ms

**10k RPS:**
- API gateway clustering required
- Authentication < 5ms
- Authorization < 2ms
- Rate limiting < 1ms
- Input validation < 1ms

**100k+ RPS:**
- Distributed API gateway mandatory
- Authentication < 2ms
- Authorization < 1ms
- Rate limiting < 0.5ms
- Input validation < 0.5ms

**6. Production Engineering View**

**Monitoring Metrics:**
- Authentication success and failure rates
- Authorization success and denial rates
- Rate limiting violations
- Input validation failures
- Security event frequency and severity

**Alerting Strategy:**
- Authentication failure rate > 10% (warning)
- Authorization denial rate > 5% (warning)
- Rate limiting violations > 1% (warning)
- High severity security events (critical)
- Blocked IP access (warning)

**Common Production Incidents:**
- Token validation failures
- Rate limiting configuration issues
- Input validation false positives
- Security monitoring overload

**7. Tradeoffs & Alternatives**

**Competing Approaches:**
- **API Gateway**: Centralized security management
- **Microgateway**: Distributed security
- **Service Mesh**: Service-level security
- **WAF Only**: Network-level security
- **Application Security**: Application-level security

**When to Use API Security:**
- Public-facing APIs
- Multi-tenant applications
- High-security requirements
- Regulatory compliance needs
- Complex authorization requirements

**When NOT to Use API Security:**
- Internal services with trusted networks
- Simple applications with minimal security needs
- Legacy systems with limited integration capabilities
- Performance-critical applications with minimal overhead

**Performance Comparison:**
- **API Gateway**: 5-15ms overhead
- **Microgateway**: 2-8ms overhead
- **Service Mesh**: 3-10ms overhead
- **WAF Only**: 1-5ms overhead
- **Application Security**: 1-3ms overhead

**8. Security Implications**

**Security Benefits:**
- **Comprehensive Protection**: Multi-layer security
- **Centralized Management**: Centralized security policies
- **Real-time Monitoring**: Real-time threat detection
- **Compliance**: Regulatory compliance support
- **Audit Trail**: Comprehensive security logging

**Security Challenges:**
- **Complexity**: Complex security configuration
- **Performance**: Security overhead
- **Maintenance**: Ongoing security maintenance
- **False Positives**: Legitimate requests blocked
- **Coverage Gaps**: New attack vectors

**9. Advanced Edge Cases**

**Advanced API Security Features:**
- **OAuth 2.0**: Advanced authorization flows
- **Mutual TLS**: Certificate-based authentication
- **API Key Management**: Advanced key management
- **GraphQL Security**: GraphQL-specific security
- **WebSocket Security**: WebSocket connection security

**Advanced Protection:**
- **Machine Learning**: ML-based threat detection
- **Behavioral Analysis**: User behavior analysis
- **Threat Intelligence**: Global threat intelligence
- **Zero Trust**: Zero-trust architecture
- **API Discovery**: Automated API discovery

**10. Interview & Staff-Level Deep Dive**

**What a junior answers:**
"API security is about protecting APIs from unauthorized access. I use JWT tokens for authentication and check user permissions for authorization. I also validate input and implement rate limiting to prevent abuse."

**What a senior answers:**
"API security is a comprehensive approach to protecting APIs through multiple layers including authentication, authorization, input validation, rate limiting, and monitoring. I've implemented production API security with JWT authentication, OAuth 2.0 authorization, input validation and sanitization, rate limiting with Redis, security monitoring with ELK stack, and comprehensive logging. The implementation includes API gateway security, microgateway patterns, service mesh integration, and real-time threat detection."

**What an architect answers:**
"API security is a critical component of modern application security that requires a multi-layered defense strategy. At scale, API security must provide comprehensive protection through authentication, authorization, input validation, rate limiting, monitoring, and threat detection. The architecture must support multiple authentication methods, fine-grained authorization, input validation and sanitization, rate limiting and throttling, security monitoring and alerting, and compliance requirements. In enterprise environments, API security is not just a feature but a strategic capability that protects business assets, ensures regulatory compliance, and enables digital transformation. The implementation must consider scalability, performance, maintainability, and comprehensive threat protection."

**Follow-up questions interviewers ask:**
- "How do you handle API versioning with security?"
- "What's the difference between authentication and authorization?"
- "How do you implement rate limiting across multiple servers?"
- "When would you use API gateway vs microgateway?"
- "How do you handle GraphQL API security?"

**Trick questions:**
- "Is JWT authentication always secure?" (No, depends on implementation)
- "Can rate limiting prevent all attacks?" (No, just one layer)
- "Do all APIs need the same security level?" (No, depends on sensitivity)
- "Is API security only about authentication?" (No, multiple layers)

**11. Real Enterprise Case Study**

**Scenario: Global Banking Platform with 10M+ API Calls/Day**

**Requirements:**
- Secure 10M+ daily API calls
- Implement multi-factor authentication
- Support multiple authorization models
- Comply with banking regulations (PCI DSS, SOX)
- Real-time threat detection and response
- High availability and performance

**Results:**
- Achieved 99.99% API security
- 95% reduction in security incidents
- 80% improvement in threat detection accuracy
- 70% reduction in false positives
- 99.95% uptime during security events

**12. Dependency Graph**

**Prerequisites:**
- **Web Development**: Understanding of web protocols and APIs
- **Authentication**: Understanding of authentication concepts
- **Authorization**: Understanding of authorization principles
- **Security**: Understanding of security fundamentals
- **Networking**: Understanding of network security

**Concepts that depend on API Security:**
- **Microservices**: Microservices security
- **API Gateway**: API management and security
- **OAuth 2.0**: Authorization framework
- **JWT**: Token-based authentication
- **Web Application Firewall**: Web application security
- **Zero Trust**: Zero-trust architecture
