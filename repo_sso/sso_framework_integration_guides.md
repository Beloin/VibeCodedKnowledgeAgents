# SSO Framework Integration Guides

## Overview

This guide provides framework-specific implementation patterns for integrating SSO into popular web development frameworks. Each section includes code examples, configuration templates, and best practices.

## Python Frameworks

### Django Integration

#### 1. Django Middleware Implementation

```python
# sso/middleware.py
import logging
from django.conf import settings
from django.http import HttpResponseRedirect
from django.urls import reverse
from .saml_handler import SAMLHandler

logger = logging.getLogger(__name__)

class SSOMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.saml_handler = SAMLHandler()

    def __call__(self, request):
        # Skip SSO for static files and admin
        if self._should_skip_sso(request):
            return self.get_response(request)

        # Check if user is authenticated
        if not request.user.is_authenticated:
            return self._handle_unauthenticated(request)

        response = self.get_response(request)
        return response

    def _should_skip_sso(self, request):
        skip_paths = ['/static/', '/admin/', '/sso/']
        return any(request.path.startswith(path) for path in skip_paths)

    def _handle_unauthenticated(self, request):
        # Check for SAML response
        if 'SAMLResponse' in request.POST:
            return self._process_saml_response(request)
        
        # Redirect to IdP for authentication
        auth_url = self.saml_handler.create_authn_request(
            acs_url=request.build_absolute_uri(reverse('sso_acs'))
        )
        return HttpResponseRedirect(auth_url)

    def _process_saml_response(self, request):
        try:
            assertion = self.saml_handler.process_response(
                request.POST['SAMLResponse']
            )
            user = self._get_or_create_user(assertion)
            # Log user in
            from django.contrib.auth import login
            login(request, user)
            
            # Redirect to original destination
            next_url = request.session.pop('next', '/')
            return HttpResponseRedirect(next_url)
            
        except Exception as e:
            logger.error(f"SAML processing failed: {e}")
            return HttpResponseRedirect('/login-error/')
```

#### 2. Django Settings Configuration

```python
# settings.py
SSO_CONFIG = {
    'SAML': {
        'strict': True,
        'debug': DEBUG,
        'sp': {
            'entityId': 'https://your-app.com/saml/metadata',
            'assertionConsumerService': {
                'url': 'https://your-app.com/sso/acs/',
                'binding': 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST'
            },
            'singleLogoutService': {
                'url': 'https://your-app.com/sso/sls/',
                'binding': 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect'
            },
            'NameIDFormat': 'urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress',
            'x509cert': '',
            'privateKey': ''
        },
        'idp': {
            'entityId': 'https://idp.example.com/metadata',
            'singleSignOnService': {
                'url': 'https://idp.example.com/sso',
                'binding': 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect'
            },
            'singleLogoutService': {
                'url': 'https://idp.example.com/slo',
                'binding': 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect'
            },
            'x509cert': 'MIIC... (IdP certificate)'
        }
    }
}
```

### Flask Integration

#### 1. Flask Blueprint Implementation

```python
# sso/blueprint.py
from flask import Blueprint, request, redirect, url_for, session, current_app
import logging

sso_bp = Blueprint('sso', __name__, url_prefix='/sso')
logger = logging.getLogger(__name__)

@sso_bp.route('/login')
def login():
    """Initiate SSO login"""
    from .saml_handler import SAMLHandler
    
    # Store original destination
    next_url = request.args.get('next', '/')
    session['next'] = next_url
    
    # Create authentication request
    saml_handler = SAMLHandler()
    auth_url = saml_handler.create_authn_request(
        acs_url=url_for('sso.acs', _external=True)
    )
    
    return redirect(auth_url)

@sso_bp.route('/acs', methods=['POST'])
def acs():
    """Assertion Consumer Service endpoint"""
    from .saml_handler import SAMLHandler
    from .user_service import get_or_create_user
    
    try:
        saml_handler = SAMLHandler()
        assertion = saml_handler.process_response(request.form['SAMLResponse'])
        
        # Extract user information
        user_info = {
            'email': assertion.get_attribute('email'),
            'first_name': assertion.get_attribute('firstName'),
            'last_name': assertion.get_attribute('lastName'),
            'groups': assertion.get_attribute('groups', [])
        }
        
        # Get or create user
        user = get_or_create_user(user_info)
        
        # Log user in (implementation depends on your auth system)
        session['user_id'] = user.id
        session['user_email'] = user.email
        
        # Redirect to original destination
        next_url = session.pop('next', '/')
        return redirect(next_url)
        
    except Exception as e:
        logger.error(f"SAML ACS processing failed: {e}")
        return redirect(url_for('auth.login_error'))

@sso_bp.route('/metadata')
def metadata():
    """Service Provider metadata endpoint"""
    from .saml_handler import SAMLHandler
    
    saml_handler = SAMLHandler()
    metadata = saml_handler.get_sp_metadata()
    
    return metadata, 200, {'Content-Type': 'application/xml'}
```

## Java Frameworks

### Spring Security Integration

#### 1. Spring Security Configuration

```java
// SecurityConfig.java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/sso/**").permitAll()
                .anyRequest().authenticated()
            )
            .saml2Login(saml2 -> saml2
                .loginProcessingUrl("/sso/acs")
                .authenticationManager(authenticationManager())
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
            );
        
        return http.build();
    }
    
    @Bean
    public RelyingPartyRegistrationRepository relyingPartyRegistrations() {
        RelyingPartyRegistration registration = RelyingPartyRegistrations
            .fromMetadataLocation("https://idp.example.com/metadata")
            .registrationId("saml-idp")
            .entityId("https://your-app.com/saml/metadata")
            .assertionConsumerServiceLocation("/sso/acs")
            .build();
            
        return new InMemoryRelyingPartyRegistrationRepository(registration);
    }
}
```

#### 2. SAML User Details Service

```java
// SamlUserDetailsService.java
@Service
public class SamlUserDetailsService implements SAMLUserDetailsService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public Object loadUserBySAML(SAMLCredential credential) throws UsernameNotFoundException {
        String email = credential.getNameID().getValue();
        
        // Extract attributes
        String firstName = credential.getAttributeAsString("firstName");
        String lastName = credential.getAttributeAsString("lastName");
        List<String> groups = credential.getAttributeAsStringList("groups");
        
        // Find or create user
        User user = userRepository.findByEmail(email)
            .orElseGet(() -> createUser(email, firstName, lastName, groups));
            
        return new CustomUserDetails(user, groups);
    }
    
    private User createUser(String email, String firstName, String lastName, List<String> groups) {
        User user = new User();
        user.setEmail(email);
        user.setFirstName(firstName);
        user.setLastName(lastName);
        user.setGroups(groups);
        
        return userRepository.save(user);
    }
}
```

## Node.js Frameworks

### Express.js Integration

#### 1. Express Middleware Setup

```javascript
// sso/middleware.js
const saml2 = require('saml2-js');
const express = require('express');
const router = express.Router();

// SP configuration
const sp_options = {
  entity_id: "https://your-app.com/saml/metadata",
  private_key: fs.readFileSync("key-file.pem").toString(),
  certificate: fs.readFileSync("cert-file.crt").toString(),
  assert_endpoint: "https://your-app.com/sso/acs",
  allow_unencrypted_assertion: false
};

// IdP configuration
const idp_options = {
  sso_login_url: "https://idp.example.com/sso",
  sso_logout_url: "https://idp.example.com/slo",
  certificates: [fs.readFileSync("idp-cert-file.crt").toString()],
  force_authn: false,
  sign_get_request: false
};

// Create service provider
const sp = new saml2.ServiceProvider(sp_options);
const idp = new saml2.IdentityProvider(idp_options);

// Login route
router.get('/login', (req, res) => {
  sp.create_login_request_url(idp, {}, (err, login_url) => {
    if (err != null) {
      return res.status(500).send('Error creating login request');
    }
    
    // Store original destination
    req.session.returnTo = req.query.returnTo || '/';
    res.redirect(login_url);
  });
});

// ACS route
router.post('/acs', (req, res) => {
  const options = { request_body: req.body };
  
  sp.post_assert(idp, options, (err, saml_response) => {
    if (err != null) {
      return res.status(403).send('Authentication failed');
    }
    
    // Extract user information
    const user = {
      name_id: saml_response.user.name_id,
      attributes: saml_response.user.attributes,
      session_index: saml_response.user.session_index
    };
    
    // Log user in
    req.session.user = user;
    req.session.save(() => {
      const returnTo = req.session.returnTo || '/';
      delete req.session.returnTo;
      res.redirect(returnTo);
    });
  });
});

module.exports = router;
```

## .NET Frameworks

### ASP.NET Core Integration

#### 1. Startup Configuration

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(options =>
    {
        options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = "saml2";
    })
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
    })
    .AddSaml2("saml2", options =>
    {
        options.SPOptions = new SPOptions
        {
            EntityId = new EntityId("https://your-app.com/saml/metadata"),
            ReturnUrl = new Uri("https://your-app.com/sso/acs")
        };
        
        options.IdentityProviders.Add(new IdentityProvider(
            new EntityId("https://idp.example.com/metadata"),
            options.SPOptions)
        {
            LoadMetadata = true,
            MetadataLocation = "https://idp.example.com/metadata"
        });
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseAuthentication();
    app.UseAuthorization();
    
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

## Framework-Specific Considerations

### Performance Optimization

#### 1. Caching Strategies

```python
# Redis-based metadata caching
import redis
from functools import wraps

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_metadata(ttl=3600):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            cache_key = f"sso:metadata:{func.__name__}"
            
            # Try to get from cache
            cached = redis_client.get(cache_key)
            if cached:
                return pickle.loads(cached)
            
            # Execute function and cache result
            result = func(*args, **kwargs)
            redis_client.setex(cache_key, ttl, pickle.dumps(result))
            return result
        return wrapper
    return decorator

@cache_metadata(ttl=3600)  # Cache for 1 hour
def get_idp_metadata():
    # Fetch metadata from IdP
    pass
```

#### 2. Connection Pooling

```java
// Java connection pooling example
@Configuration
public class DatabaseConfig {
    
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .type(HikariDataSource.class)
            .build();
    }
}
```

### Security Considerations

#### 1. Input Validation

```python
# Python input validation
from marshmallow import Schema, fields, validate, ValidationError

class SAMLRequestSchema(Schema):
    SAMLResponse = fields.Str(required=True)
    RelayState = fields.Str(required=False)
    
    def validate_saml_response(self, value):
        if not value.startswith('PHNhbWxwOlJlc3BvbnNl'):  # Base64 encoded SAML
            raise ValidationError('Invalid SAML response format')

def validate_saml_request(request_data):
    schema = SAMLRequestSchema()
    try:
        return schema.load(request_data)
    except ValidationError as err:
        raise ValueError(f"Invalid SAML request: {err.messages}")
```

#### 2. Rate Limiting

```javascript
// Express rate limiting
const rateLimit = require('express-rate-limit');

const ssoLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Limit each IP to 5 login attempts per windowMs
  message: 'Too many authentication attempts, please try again later.',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/sso/login', ssoLimiter);
```

## Testing Strategies

### Framework-Specific Testing

#### 1. Django Testing

```python
# tests/test_sso_integration.py
from django.test import TestCase, Client
from django.urls import reverse
from unittest.mock import patch, MagicMock

class SSOIntegrationTest(TestCase):
    def setUp(self):
        self.client = Client()
    
    @patch('sso.middleware.SAMLHandler')
    def test_sso_login_redirect(self, mock_saml):
        mock_handler = MagicMock()
        mock_handler.create_authn_request.return_value = 'https://idp.example.com/sso'
        mock_saml.return_value = mock_handler
        
        response = self.client.get('/sso/login?next=/dashboard/')
        
        self.assertEqual(response.status_code, 302)
        self.assertIn('idp.example.com', response.url)
        self.assertEqual(self.client.session['next'], '/dashboard/')
    
    def test_sso_acs_with_valid_response(self):
        # Test ACS endpoint with valid SAML response
        pass
```

#### 2. Spring Boot Testing

```java
// SSOIntegrationTest.java
@SpringBootTest
@AutoConfigureTestDatabase
class SSOIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void testSSOLoginRedirect() {
        ResponseEntity<String> response = restTemplate.getForEntity("/sso/login", String.class);
        
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.FOUND);
        assertThat(response.getHeaders().getLocation())
            .hasHost("idp.example.com");
    }
}
```

## Deployment Considerations

### Environment Configuration

#### 1. Environment-Specific Settings

```yaml
# config/development.yaml
sso:
  saml:
    strict: false
    debug: true
    sp:
      entityId: "https://dev.your-app.com/saml/metadata"
      assertionConsumerService:
        url: "https://dev.your-app.com/sso/acs"

# config/production.yaml  
sso:
  saml:
    strict: true
    debug: false
    sp:
      entityId: "https://your-app.com/saml/metadata"
      assertionConsumerService:
        url: "https://your-app.com/sso/acs"
```

#### 2. Container Configuration

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser
USER appuser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000"]
```

## Monitoring and Logging

### Framework-Specific Monitoring

#### 1. Django Monitoring

```python
# monitoring/middleware.py
import time
import logging
from django.conf import settings

logger = logging.getLogger('sso.monitoring')

class SSOMonitoringMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        start_time = time.time()
        
        response = self.get_response(request)
        
        # Log SSO-related requests
        if '/sso/' in request.path:
            duration = time.time() - start_time
            logger.info(f"SSO request: {request.path} - {duration:.3f}s")
            
            # Track metrics
            if hasattr(settings, 'METRICS_CLIENT'):
                settings.METRICS_CLIENT.timing(
                    f'sso.request_duration', 
                    duration * 1000,  # Convert to milliseconds
                    tags=[f'path:{request.path}', f'method:{request.method}']
                )
        
        return response
```

#### 2. Spring Boot Actuator

```java
// Monitoring configuration
@Configuration
public class MonitoringConfig {
    
    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
        return registry -> registry.config().commonTags(
            "application", "sso-service",
            "environment", System.getenv().getOrDefault("ENVIRONMENT", "unknown")
        );
    }
}
```

## Conclusion

Framework integration requires careful consideration of each framework's architecture, security model, and best practices. By following these patterns and adapting them to your specific framework, you can build robust, secure, and maintainable SSO implementations that scale with your application's needs.

Remember to:
- Follow framework-specific security guidelines
- Implement proper error handling and logging
- Test thoroughly across different scenarios
- Monitor performance and security metrics
- Keep dependencies updated and patched