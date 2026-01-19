---
name: security-review
description: Use this skill when adding authentication, handling user input, working with secrets, creating API endpoints, or implementing payment/sensitive features. Provides comprehensive security checklist and patterns.
---

# Security Review Skill

This skill ensures all code follows security best practices and identifies potential vulnerabilities.

## When to Activate

- Implementing authentication or authorization
- Handling user input or file uploads
- Creating new API endpoints
- Working with secrets or credentials
- Implementing payment features
- Storing or transmitting sensitive data
- Integrating third-party APIs

## Security Checklist

### 1. Secrets Management

#### JavaScript/TypeScript
```typescript
// ❌ NEVER
const apiKey = "sk-proj-xxxxx"  // Hardcoded secret

// ✅ ALWAYS
const apiKey = process.env.OPENAI_API_KEY
if (!apiKey) throw new Error('OPENAI_API_KEY not configured')
```

#### PHP/Laravel
```php
// ❌ NEVER
$apiKey = 'sk-proj-xxxxx';

// ✅ ALWAYS
$apiKey = config('services.openai.key'); // or env('OPENAI_API_KEY')
if (!$apiKey) {
    throw new \Exception('OPENAI_API_KEY not configured');
}
```

#### Python/Django
```python
# ❌ NEVER
api_key = "sk-proj-xxxxx"

# ✅ ALWAYS
import os
from django.core.exceptions import ImproperlyConfigured

api_key = os.getenv('OPENAI_API_KEY')
if not api_key:
    raise ImproperlyConfigured('OPENAI_API_KEY not configured')
```

#### Flutter/Dart
```dart
// ❌ NEVER
const apiKey = 'sk-proj-xxxxx';

// ✅ ALWAYS
import 'package:flutter_dotenv/flutter_dotenv.dart';

final apiKey = dotenv.env['OPENAI_API_KEY'];
if (apiKey == null) {
  throw Exception('OPENAI_API_KEY not configured');
}
```

#### React Native
```typescript
// ❌ NEVER
const apiKey = 'sk-proj-xxxxx';

// ✅ ALWAYS
import Config from 'react-native-config';

const apiKey = Config.OPENAI_API_KEY;
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured');
}
```

#### Java/Spring Boot
```java
// ❌ NEVER
private static final String API_KEY = "sk-proj-xxxxx";

// ✅ ALWAYS
@Value("${openai.api.key}")
private String apiKey;

// Or with validation
@PostConstruct
public void init() {
    if (apiKey == null || apiKey.isEmpty()) {
        throw new IllegalStateException("OPENAI_API_KEY not configured");
    }
}
```

#### Verification Steps
- [ ] No hardcoded API keys, tokens, or passwords
- [ ] All secrets in environment variables
- [ ] `.env.local` in .gitignore
- [ ] No secrets in git history
- [ ] Production secrets in hosting platform (Vercel, Railway)

### 2. Input Validation

#### JavaScript/TypeScript (Zod)
```typescript
import { z } from 'zod'

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

export async function createUser(input: unknown) {
  const validated = CreateUserSchema.parse(input)
  return await db.users.create(validated)
}
```

#### PHP/Laravel (Form Requests)
```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class CreateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'email' => ['required', 'email', 'max:255', 'unique:users'],
            'name' => ['required', 'string', 'min:1', 'max:100'],
            'age' => ['required', 'integer', 'min:0', 'max:150'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.required' => 'Email is required',
            'email.email' => 'Must be valid email',
        ];
    }
}

// In controller
public function store(CreateUserRequest $request)
{
    $validated = $request->validated();
    return User::create($validated);
}
```

#### Python/Django (Serializers)
```python
from rest_framework import serializers
from django.contrib.auth import get_user_model

User = get_user_model()

class CreateUserSerializer(serializers.ModelSerializer):
    email = serializers.EmailField(required=True)
    name = serializers.CharField(min_length=1, max_length=100)
    age = serializers.IntegerField(min_value=0, max_value=150)

    class Meta:
        model = User
        fields = ['email', 'name', 'age']

    def validate_email(self, value):
        if User.objects.filter(email=value).exists():
            raise serializers.ValidationError("Email already exists")
        return value

# In view
def create_user(request):
    serializer = CreateUserSerializer(data=request.data)
    if serializer.is_valid():
        user = serializer.save()
        return Response(serializer.data, status=201)
    return Response(serializer.errors, status=400)
```

#### Flutter/Dart (Form Validators)
```dart
class UserFormValidator {
  static String? validateEmail(String? value) {
    if (value == null || value.isEmpty) {
      return 'Email is required';
    }
    final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    if (!emailRegex.hasMatch(value)) {
      return 'Must be valid email';
    }
    return null;
  }

  static String? validateName(String? value) {
    if (value == null || value.isEmpty) {
      return 'Name is required';
    }
    if (value.length > 100) {
      return 'Name too long (max 100)';
    }
    return null;
  }

  static String? validateAge(String? value) {
    if (value == null || value.isEmpty) {
      return 'Age is required';
    }
    final age = int.tryParse(value);
    if (age == null || age < 0 || age > 150) {
      return 'Age must be between 0 and 150';
    }
    return null;
  }
}

// Usage in Form
TextFormField(
  validator: UserFormValidator.validateEmail,
  decoration: InputDecoration(labelText: 'Email'),
)
```

#### React Native (Validation Library)
```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email('Must be valid email'),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150),
});

function validateUser(data: unknown) {
  try {
    return CreateUserSchema.parse(data);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { errors: error.flatten() };
    }
    throw error;
  }
}
```

#### Java/Spring Boot (Bean Validation)
```java
import jakarta.validation.constraints.*;

public class CreateUserRequest {
    @NotBlank(message = "Email is required")
    @Email(message = "Must be valid email")
    @Size(max = 255)
    private String email;

    @NotBlank(message = "Name is required")
    @Size(min = 1, max = 100)
    private String name;

    @NotNull(message = "Age is required")
    @Min(0)
    @Max(150)
    private Integer age;

    // Getters and setters
}

// In controller
@PostMapping("/users")
public ResponseEntity<?> createUser(
    @Valid @RequestBody CreateUserRequest request
) {
    User user = userService.createUser(request);
    return ResponseEntity.status(201).body(user);
}

// Exception handler
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<?> handleValidation(MethodArgumentNotValidException ex) {
    Map<String, String> errors = ex.getBindingResult()
        .getFieldErrors()
        .stream()
        .collect(Collectors.toMap(
            FieldError::getField,
            FieldError::getDefaultMessage
        ));
    return ResponseEntity.badRequest().body(errors);
}
```

#### File Upload Validation
```typescript
function validateFileUpload(file: File) {
  // Size check (5MB max)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // Type check
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // Extension check
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

#### Verification Steps
- [ ] All user inputs validated with schemas
- [ ] File uploads restricted (size, type, extension)
- [ ] No direct use of user input in queries
- [ ] Whitelist validation (not blacklist)
- [ ] Error messages don't leak sensitive info

### 3. SQL Injection Prevention

#### JavaScript/TypeScript
```typescript
// ❌ NEVER Concatenate
const query = `SELECT * FROM users WHERE email = '${userEmail}'`

// ✅ ALWAYS Parameterize
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// Or with raw SQL
await db.query('SELECT * FROM users WHERE email = $1', [userEmail])
```

#### PHP/Laravel
```php
// ❌ NEVER Use Raw Concatenation
$users = DB::select("SELECT * FROM users WHERE email = '$email'");

// ✅ ALWAYS Use Query Builder
$users = DB::table('users')
    ->where('email', $email)
    ->get();

// ✅ Or Eloquent
$users = User::where('email', $email)->get();

// ✅ Or Parameterized Raw Query
$users = DB::select('SELECT * FROM users WHERE email = ?', [$email]);
```

#### Python/Django
```python
# ❌ NEVER Use String Formatting
User.objects.raw(f"SELECT * FROM users WHERE email = '{email}'")

# ✅ ALWAYS Use ORM
User.objects.filter(email=email)

# ✅ Or Parameterized Raw Query
User.objects.raw('SELECT * FROM users WHERE email = %s', [email])

# ✅ Django QuerySet (automatically parameterized)
users = User.objects.filter(
    email=email,
    is_active=True
).select_related('profile')
```

#### Flutter/Dart (SQLite)
```dart
// ❌ NEVER Concatenate
final users = await db.rawQuery(
  "SELECT * FROM users WHERE email = '$email'"
);

// ✅ ALWAYS Parameterize
final users = await db.query(
  'users',
  where: 'email = ?',
  whereArgs: [email],
);

// ✅ Or with rawQuery
final users = await db.rawQuery(
  'SELECT * FROM users WHERE email = ?',
  [email],
);
```

#### React Native (Same as JavaScript/TypeScript)
```typescript
// Use Supabase, Firebase, or other backend with parameterized queries
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)
```

#### Java/Spring Boot
```java
// ❌ NEVER Concatenate
String query = "SELECT * FROM users WHERE email = '" + email + "'";

// ✅ ALWAYS Use JPA
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmail(@Param("email") String email);

// ✅ Or JDBC PreparedStatement
String sql = "SELECT * FROM users WHERE email = ?";
PreparedStatement stmt = connection.prepareStatement(sql);
stmt.setString(1, email);
ResultSet rs = stmt.executeQuery();

// ✅ Or JPA Criteria API (type-safe)
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<User> query = cb.createQuery(User.class);
Root<User> user = query.from(User.class);
query.select(user).where(cb.equal(user.get("email"), email));
```

#### Verification Steps
- [ ] All database queries use parameterized queries
- [ ] No string concatenation in SQL
- [ ] ORM/query builder used correctly
- [ ] All raw queries use parameter binding

### 4. Authentication & Authorization

#### JWT Token Handling
```typescript
// ❌ WRONG: localStorage (vulnerable to XSS)
localStorage.setItem('token', token)

// ✅ CORRECT: httpOnly cookies
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### Authorization Checks
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // ALWAYS verify authorization first
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // Proceed with deletion
  await db.users.delete({ where: { id: userId } })
}
```

#### Row Level Security (Supabase)
```sql
-- Enable RLS on all tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Users can only view their own data
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- Users can only update their own data
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### Verification Steps
- [ ] Tokens stored in httpOnly cookies (not localStorage)
- [ ] Authorization checks before sensitive operations
- [ ] Row Level Security enabled in Supabase
- [ ] Role-based access control implemented
- [ ] Session management secure

### 5. XSS Prevention

#### Sanitize HTML
```typescript
import DOMPurify from 'isomorphic-dompurify'

// ALWAYS sanitize user-provided HTML
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### Content Security Policy
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

#### Verification Steps
- [ ] User-provided HTML sanitized
- [ ] CSP headers configured
- [ ] No unvalidated dynamic content rendering
- [ ] React's built-in XSS protection used

### 6. CSRF Protection

#### CSRF Tokens
```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }

  // Process request
}
```

#### SameSite Cookies
```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### Verification Steps
- [ ] CSRF tokens on state-changing operations
- [ ] SameSite=Strict on all cookies
- [ ] Double-submit cookie pattern implemented

### 7. Rate Limiting

#### JavaScript/TypeScript (Express)
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: 'Too many requests'
})

app.use('/api/', limiter)
```

#### PHP/Laravel (Middleware)
```php
// In RouteServiceProvider or routes file
Route::middleware(['throttle:60,1'])->group(function () {
    Route::get('/api/markets', [MarketController::class, 'index']);
});

// Custom rate limiter
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

// Expensive operations
RateLimiter::for('searches', function (Request $request) {
    return Limit::perMinute(10)->by($request->user()?->id ?: $request->ip());
});
```

#### Python/Django (django-ratelimit)
```python
from django_ratelimit.decorators import ratelimit

@ratelimit(key='ip', rate='100/15m', method='GET')
def api_endpoint(request):
    return JsonResponse({'data': 'response'})

# Or with DRF
from rest_framework.throttling import UserRateThrottle, AnonRateThrottle

class MarketViewSet(viewsets.ModelViewSet):
    throttle_classes = [UserRateThrottle, AnonRateThrottle]

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'user': '100/hour',
        'anon': '20/hour',
    }
}
```

#### Flutter/Dart (Client-side debouncing)
```dart
import 'dart:async';

class RateLimiter {
  Timer? _timer;
  final Duration duration;

  RateLimiter({required this.duration});

  void call(VoidCallback action) {
    _timer?.cancel();
    _timer = Timer(duration, action);
  }

  void dispose() {
    _timer?.cancel();
  }
}

// Usage
final searchLimiter = RateLimiter(duration: Duration(milliseconds: 500));
searchLimiter.call(() => performSearch(query));
```

#### React Native (Same as JavaScript/TypeScript)
```typescript
// Backend rate limiting (same as Express)
// Client-side debouncing
import { useDebounce } from 'use-debounce';

const [searchQuery, setSearchQuery] = useState('');
const [debouncedQuery] = useDebounce(searchQuery, 500);

useEffect(() => {
  if (debouncedQuery) {
    performSearch(debouncedQuery);
  }
}, [debouncedQuery]);
```

#### Java/Spring Boot (Bucket4j)
```java
@RestController
@RequestMapping("/api")
public class MarketController {

    private final Bucket bucket;

    public MarketController() {
        Bandwidth limit = Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(15)));
        this.bucket = Bucket4j.builder()
            .addLimit(limit)
            .build();
    }

    @GetMapping("/markets")
    public ResponseEntity<?> getMarkets() {
        if (bucket.tryConsume(1)) {
            return ResponseEntity.ok(marketService.getMarkets());
        }
        return ResponseEntity.status(429).body("Too many requests");
    }
}

// Or with interceptor
@Component
public class RateLimitInterceptor implements HandlerInterceptor {
    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        String key = request.getRemoteAddr();
        Bucket bucket = cache.computeIfAbsent(key, k -> createBucket());

        if (bucket.tryConsume(1)) {
            return true;
        }

        response.setStatus(429);
        response.getWriter().write("Too many requests");
        return false;
    }
}
```

#### Verification Steps
- [ ] Rate limiting on all API endpoints
- [ ] Stricter limits on expensive operations
- [ ] IP-based rate limiting
- [ ] User-based rate limiting (authenticated)

### 8. Sensitive Data Exposure

#### Logging
```typescript
// ❌ WRONG: Logging sensitive data
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// ✅ CORRECT: Redact sensitive data
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### Error Messages
```typescript
// ❌ WRONG: Exposing internal details
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ CORRECT: Generic error messages
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### Verification Steps
- [ ] No passwords, tokens, or secrets in logs
- [ ] Error messages generic for users
- [ ] Detailed errors only in server logs
- [ ] No stack traces exposed to users

### 9. Blockchain Security (Solana)

#### Wallet Verification
```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

#### Transaction Verification
```typescript
async function verifyTransaction(transaction: Transaction) {
  // Verify recipient
  if (transaction.to !== expectedRecipient) {
    throw new Error('Invalid recipient')
  }

  // Verify amount
  if (transaction.amount > maxAmount) {
    throw new Error('Amount exceeds limit')
  }

  // Verify user has sufficient balance
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('Insufficient balance')
  }

  return true
}
```

#### Verification Steps
- [ ] Wallet signatures verified
- [ ] Transaction details validated
- [ ] Balance checks before transactions
- [ ] No blind transaction signing

### 10. Dependency Security

#### Regular Updates
```bash
# Check for vulnerabilities
npm audit

# Fix automatically fixable issues
npm audit fix

# Update dependencies
npm update

# Check for outdated packages
npm outdated
```

#### Lock Files
```bash
# ALWAYS commit lock files
git add package-lock.json

# Use in CI/CD for reproducible builds
npm ci  # Instead of npm install
```

#### Verification Steps
- [ ] Dependencies up to date
- [ ] No known vulnerabilities (npm audit clean)
- [ ] Lock files committed
- [ ] Dependabot enabled on GitHub
- [ ] Regular security updates

## Security Testing

### Automated Security Tests
```typescript
// Test authentication
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// Test authorization
test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// Test input validation
test('rejects invalid input', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// Test rate limiting
test('enforces rate limits', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## Pre-Deployment Security Checklist

Before ANY production deployment:

- [ ] **Secrets**: No hardcoded secrets, all in env vars
- [ ] **Input Validation**: All user inputs validated
- [ ] **SQL Injection**: All queries parameterized
- [ ] **XSS**: User content sanitized
- [ ] **CSRF**: Protection enabled
- [ ] **Authentication**: Proper token handling
- [ ] **Authorization**: Role checks in place
- [ ] **Rate Limiting**: Enabled on all endpoints
- [ ] **HTTPS**: Enforced in production
- [ ] **Security Headers**: CSP, X-Frame-Options configured
- [ ] **Error Handling**: No sensitive data in errors
- [ ] **Logging**: No sensitive data logged
- [ ] **Dependencies**: Up to date, no vulnerabilities
- [ ] **Row Level Security**: Enabled in Supabase
- [ ] **CORS**: Properly configured
- [ ] **File Uploads**: Validated (size, type)
- [ ] **Wallet Signatures**: Verified (if blockchain)

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**Remember**: Security is not optional. One vulnerability can compromise the entire platform. When in doubt, err on the side of caution.
