# Code Review

Comprehensive security and quality review of code changes across all supported languages and frameworks. This command performs automated code review to catch security issues, code quality problems, and best practice violations before committing.

---

## Workflow

1. **Identify changes** - Get modified/uncommitted files
2. **Analyze code** - Check for security, quality, and best practices
3. **Generate report** - Detailed findings with severity levels
4. **Suggest fixes** - Actionable recommendations
5. **Block critical issues** - Prevent commits with security vulnerabilities

---

## Get Changed Files

```bash
# Uncommitted changes
git diff --name-only HEAD

# Changes in branch vs main
git diff --name-only main...HEAD

# Staged changes only
git diff --cached --name-only

# Show actual changes
git diff HEAD
```

---

## Review Checklist by Language

### JavaScript/TypeScript

**Security Issues (CRITICAL):**
- ❌ Hardcoded API keys, tokens, passwords
- ❌ `eval()` or `Function()` constructor usage
- ❌ `dangerouslySetInnerHTML` without sanitization
- ❌ Unvalidated user input in DOM manipulation
- ❌ Missing CSRF protection on mutations
- ❌ Exposed sensitive data in client-side code

**Code Quality (HIGH):**
- ❌ Functions > 50 lines
- ❌ Files > 800 lines
- ❌ Nesting depth > 4 levels
- ❌ `console.log()` / `debugger` statements
- ❌ `any` type usage (TypeScript)
- ❌ Missing error handling (`try/catch`)
- ❌ Unhandled Promise rejections

**Best Practices (MEDIUM):**
- ❌ Direct array/object mutations
- ❌ Missing JSDoc for public APIs
- ❌ Missing tests for new code
- ❌ TODO/FIXME without tracking ticket
- ❌ Magic numbers without constants
- ❌ Deeply nested ternaries

**Example Violations:**

```typescript
// ❌ CRITICAL: Hardcoded API key
const API_KEY = 'sk-1234567890abcdef';  // NEVER do this

// ✅ FIX: Use environment variable
const API_KEY = process.env.OPENAI_API_KEY;

// ❌ HIGH: Using 'any' type
function processData(data: any): any {
  return data.map((item: any) => item.value);
}

// ✅ FIX: Proper typing
interface DataItem {
  value: number;
}
function processData(data: DataItem[]): number[] {
  return data.map((item) => item.value);
}

// ❌ HIGH: Missing error handling
async function fetchMarket(id: string) {
  const response = await fetch(`/api/markets/${id}`);
  return response.json();  // Can throw error
}

// ✅ FIX: Add error handling
async function fetchMarket(id: string): Promise<Market> {
  try {
    const response = await fetch(`/api/markets/${id}`);
    if (!response.ok) {
      throw new Error(`Failed to fetch market: ${response.statusText}`);
    }
    return await response.json();
  } catch (error) {
    logger.error('fetchMarket failed', { id, error });
    throw error;
  }
}

// ❌ MEDIUM: Direct mutation
function addItem(cart, item) {
  cart.items.push(item);  // Mutates original
  return cart;
}

// ✅ FIX: Immutable update
function addItem(cart: Cart, item: Item): Cart {
  return {
    ...cart,
    items: [...cart.items, item],
  };
}
```

### PHP/Laravel

**Security Issues (CRITICAL):**
- ❌ Raw SQL queries without parameter binding
- ❌ Missing `declare(strict_types=1)`
- ❌ Unvalidated user input
- ❌ Missing CSRF protection
- ❌ Exposed `.env` variables in code
- ❌ `eval()` or `create_function()` usage

**Code Quality (HIGH):**
- ❌ Methods > 50 lines
- ❌ Classes > 800 lines
- ❌ Missing return type declarations
- ❌ `var_dump()` or `dd()` statements
- ❌ Missing exception handling
- ❌ N+1 query problems

**Best Practices (MEDIUM):**
- ❌ Using  `DB::raw()` without sanitation
- ❌ Missing Form Request validation
- ❌ Fat controllers (business logic in controllers)
- ❌ Missing docblocks for public methods
- ❌ Missing tests for new features
- ❌ Not using DTOs for complex data

**Example Violations:**

```php
// ❌ CRITICAL: SQL injection vulnerability
$users = DB::select("SELECT * FROM users WHERE email = '{$email}'");

// ✅ FIX: Use parameter binding
$users = DB::table('users')->where('email', $email)->get();
// or: DB::select('SELECT * FROM users WHERE email = ?', [$email]);

// ❌ HIGH: Missing strict types
<?php
namespace App\Services;

// ✅ FIX: Add strict types declaration
<?php

declare(strict_types=1);

namespace App\Services;

// ❌ HIGH: Missing return type
public function calculateTotal($items) {
    return array_sum(array_column($items, 'price'));
}

// ✅ FIX: Add type declarations
public function calculateTotal(array $items): float
{
    return (float) array_sum(array_column($items, 'price'));
}

// ❌ MEDIUM: Controller with business logic (fat controller)
class MarketController extends Controller
{
    public function store(Request $request)
    {
        // 50 lines of validation and business logic here
        $market = new Market();
        $market->name = $request->name;
        // ... more business logic
        $market->save();
    }
}

// ✅ FIX: Use Form Request + Service Layer
class MarketController extends Controller
{
    public function __construct(
        private readonly MarketService $marketService
    ) {}

    public function store(CreateMarketRequest $request): JsonResponse
    {
        $market = $this->marketService->create($request->validated());

        return response()->json($market, 201);
    }
}

// ❌ MEDIUM: N+1 query problem
foreach ($markets as $market) {
    echo $market->user->name;  // Queries user for each market
}

// ✅ FIX: Eager loading
$markets = Market::with('user')->get();
foreach ($markets as $market) {
    echo $market->user->name;  // No additional queries
}
```

### Python/Django

**Security Issues (CRITICAL):**
- ❌ Raw SQL queries without parameters
- ❌ `eval()` or `exec()` usage
- ❌ Missing CSRF protection
- ❌ Unvalidated user input
- ❌ Pickle usage with untrusted data
- ❌ Debug mode enabled in production

**Code Quality (HIGH):**
- ❌ Functions > 50 lines
- ❌ Files > 800 lines
- ❌ Missing type hints
- ❌ `print()` statements (should use logging)
- ❌ Bare `except:` clauses
- ❌ Missing docstrings for public functions

**Best Practices (MEDIUM):**
- ❌ Not following PEP 8 style guide
- ❌ Mutable default arguments
- ❌ Missing tests for new code
- ❌ Using `django.utils.timezone.now()` instead of `timezone.now()`
- ❌ Not using Django ORM (raw SQL everywhere)
- ❌ Circular imports

**Example Violations:**

```python
# ❌ CRITICAL: SQL injection vulnerability
cursor.execute(f"SELECT * FROM markets WHERE id = {market_id}")

# ✅ FIX: Use parameterized queries
cursor.execute("SELECT * FROM markets WHERE id = %s", [market_id])
# Better: Use ORM
Market.objects.get(id=market_id)

# ❌ HIGH: Missing type hints
def calculate_score(data):
    return data['votes'] / data['total']

# ✅ FIX: Add type hints
def calculate_score(data: dict[str, int]) -> float:
    return data['votes'] / data['total']

# ❌ HIGH: Bare except clause
try:
    result = process_market(market_id)
except:  # Catches everything including KeyboardInterrupt!
    pass

# ✅ FIX: Catch specific exceptions
try:
    result = process_market(market_id)
except (ValueError, KeyError) as e:
    logger.error(f"Failed to process market {market_id}: {e}")
    raise

# ❌ HIGH: Using print() instead of logging
def process_order(order_id):
    print(f"Processing order {order_id}")  # Lost in production
    # ...

# ✅ FIX: Use logging
import logging

logger = logging.getLogger(__name__)

def process_order(order_id: int) -> Order:
    logger.info("Processing order", extra={"order_id": order_id})
    # ...

# ❌ MEDIUM: Mutable default argument
def add_item(item, cart=[]):  # Shared across calls!
    cart.append(item)
    return cart

# ✅ FIX: Use None as default
def add_item(item: Item, cart: list[Item] | None = None) -> list[Item]:
    if cart is None:
        cart = []
    cart.append(item)
    return cart
```

### Flutter/Dart

**Security Issues (CRITICAL):**
- ❌ API keys hardcoded in code
- ❌ Sensitive data in logs
- ❌ Insecure HTTP instead of HTTPS
- ❌ Missing input validation
- ❌ Storing secrets in SharedPreferences (use secure_storage)
- ❌ Certificate pinning disabled in production

**Code Quality (HIGH):**
- ❌ Functions > 50 lines
- ❌ Files > 800 lines
- ❌ Missing null safety checks
- ❌ `print()` statements (should use logger)
- ❌ Deeply nested widget trees
- ❌ Missing error handling in async code

**Best Practices (MEDIUM):**
- ❌ Not using const constructors where possible
- ❌ Missing tests for new widgets
- ❌ Stateful widgets for static content
- ❌ Not extracting reusable widgets
- ❌ Magic numbers without constants
- ❌ Missing documentation for public APIs

**Example Violations:**

```dart
// ❌ CRITICAL: Hardcoded API key
const String apiKey = 'sk-1234567890';  // Visible in compiled code!

// ✅ FIX: Use environment variables + secure storage
final apiKey = dotenv.env['API_KEY'];
// Store in secure_storage for runtime values

// ❌ HIGH: Missing null safety check
Widget build(BuildContext context) {
  return Text(market.title);  // Can throw if market is null
}

// ✅ FIX: Handle null properly
Widget build(BuildContext context) {
  return Text(market?.title ?? 'No title');
}

// ❌ HIGH: No error handling in async code
Future<void> loadMarket() async {
  final response = await http.get(Uri.parse('/api/markets/1'));
  final market = Market.fromJson(jsonDecode(response.body));
  setState(() => _market = market);
}

// ✅ FIX: Add error handling
Future<void> loadMarket() async {
  try {
    final response = await http.get(Uri.parse('/api/markets/1'));

    if (response.statusCode != 200) {
      throw Exception('Failed to load market: ${response.statusCode}');
    }

    final market = Market.fromJson(jsonDecode(response.body));
    setState(() {
      _market = market;
      _error = null;
    });
  } catch (e, stackTrace) {
    logger.error('loadMarket failed', error: e, stackTrace: stackTrace);
    setState(() {
      _market = null;
      _error = e.toString();
    });
  }
}

// ❌ MEDIUM: Deeply nested widget tree
Widget build(BuildContext context) {
  return Container(
    child: Column(
      children: [
        Row(
          children: [
            Container(
              child: Column(
                children: [
                  // ... more nesting
                ],
              ),
            ),
          ],
        ),
      ],
    ),
  );
}

// ✅ FIX: Extract widgets
Widget build(BuildContext context) {
  return Container(
    child: Column(
      children: [
        _buildHeader(),
        _buildContent(),
        _buildFooter(),
      ],
    ),
  );
}

Widget _buildHeader() => _MarketHeader(market: market);
Widget _buildContent() => _MarketContent(market: market);
Widget _buildFooter() => _MarketFooter(market: market);

// ❌ MEDIUM: Not using const constructor
Widget build(BuildContext context) {
  return Padding(
    padding: EdgeInsets.all(16.0),  // Creates new instance every build
    child: Text('Hello'),
  );
}

// ✅ FIX: Use const where possible
Widget build(BuildContext context) {
  return const Padding(
    padding: EdgeInsets.all(16.0),  // Reuses same instance
    child: Text('Hello'),
  );
}
```

### React Native

**Security Issues (CRITICAL):**
- ❌ API keys/secrets in JavaScript code
- ❌ Sensitive data in AsyncStorage (use KeyChain/Keystore)
- ❌ Missing SSL pinning for production APIs
- ❌ Unvalidated deep links
- ❌ Exposing debug information in production
- ❌ Using `eval()` or dangerous WebView settings

**Code Quality (HIGH):**
- ❌ Components > 300 lines
- ❌ Missing TypeScript types (`any` usage)
- ❌ `console.log()` statements
- ❌ Missing error boundaries
- ❌ Unhandled Promise rejections
- ❌ Memory leaks (listeners not cleaned up)

**Best Practices (MEDIUM):**
- ❌ Prop drilling (passing props through many levels)
- ❌ Inline styles everywhere (use StyleSheet)
- ❌ Missing accessibility props
- ❌ Not using React.memo for expensive components
- ❌ Missing tests for new components
- ❌ Mixing business logic with UI components

**Example Violations:**

```typescript
// ❌ CRITICAL: API key in code
const API_KEY = 'sk-1234567890';  // Exposed in bundle!

// ✅ FIX: Use react-native-config + secure storage
import Config from 'react-native-config';
import * as Keychain from 'react-native-keychain';

const API_KEY = Config.API_KEY;  // From .env (still visible in bundle)
// For sensitive runtime data, use Keychain

// ❌ HIGH: Missing error boundary
const App = () => {
  return <MarketScreen />;  // If MarketScreen crashes, whole app crashes
};

// ✅ FIX: Add error boundary
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    logger.error('React error boundary caught', { error, info });
  }

  render() {
    if (this.state.hasError) {
      return <ErrorScreen />;
    }
    return this.props.children;
  }
}

const App = () => (
  <ErrorBoundary>
    <MarketScreen />
  </ErrorBoundary>
);

// ❌ HIGH: Memory leak - listener not removed
useEffect(() => {
  const subscription = AppState.addEventListener('change', handleChange);
  // Missing cleanup!
}, []);

// ✅ FIX: Clean up listeners
useEffect(() => {
  const subscription = AppState.addEventListener('change', handleChange);

  return () => {
    subscription.remove();  // Cleanup
  };
}, []);

// ❌ MEDIUM: Inline styles (not optimized)
const MarketCard = ({ market }) => (
  <View style={{ padding: 16, margin: 8, backgroundColor: '#fff' }}>
    <Text style={{ fontSize: 18, fontWeight: 'bold' }}>{market.name}</Text>
  </View>
);

// ✅ FIX: Use StyleSheet
const styles = StyleSheet.create({
  container: {
    padding: 16,
    margin: 8,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
  },
});

const MarketCard = ({ market }: Props) => (
  <View style={styles.container}>
    <Text style={styles.title}>{market.name}</Text>
  </View>
);

// ❌ MEDIUM: Missing accessibility
<TouchableOpacity onPress={handlePress}>
  <Text>Submit</Text>
</TouchableOpacity>

// ✅ FIX: Add accessibility props
<TouchableOpacity
  onPress={handlePress}
  accessible={true}
  accessibilityLabel="Submit market form"
  accessibilityRole="button"
>
  <Text>Submit</Text>
</TouchableOpacity>
```

### Java/Spring Boot

**Security Issues (CRITICAL):**
- ❌ SQL injection (concatenated queries)
- ❌ Hardcoded credentials
- ❌ Missing input validation
- ❌ Deserialization vulnerabilities
- ❌ Missing CSRF protection
- ❌ Weak cryptography algorithms

**Code Quality (HIGH):**
- ❌ Methods > 50 lines
- ❌ Classes > 800 lines
- ❌ Missing JavaDoc for public APIs
- ❌ System.out.println() (should use logger)
- ❌ Catching generic Exception
- ❌ Field injection (use constructor injection)

**Best Practices (MEDIUM):**
- ❌ Not using Optional for nullable returns
- ❌ Missing tests for new code
- ❌ Using `@Autowired` on fields
- ❌ Mutable collections exposed in public APIs
- ❌ Not using Java Records (Java 17+)
- ❌ Magic numbers without constants

**Example Violations:**

```java
// ❌ CRITICAL: SQL injection
public List<Market> findByName(String name) {
    String sql = "SELECT * FROM markets WHERE name = '" + name + "'";
    return jdbcTemplate.query(sql, marketRowMapper);
}

// ✅ FIX: Use parameterized queries
public List<Market> findByName(String name) {
    String sql = "SELECT * FROM markets WHERE name = ?";
    return jdbcTemplate.query(sql, marketRowMapper, name);
}
// Better: Use JPA
public List<Market> findByName(String name) {
    return marketRepository.findByName(name);
}

// ❌ HIGH: Field injection (harder to test, can be null)
@RestController
public class MarketController {
    @Autowired
    private MarketService marketService;  // Bad practice
}

// ✅ FIX: Constructor injection
@RestController
public class MarketController {
    private final MarketService marketService;

    @Autowired
    public MarketController(MarketService marketService) {
        this.marketService = marketService;
    }
}

// ❌ HIGH: Catching generic Exception
try {
    processMarket(marketId);
} catch (Exception e) {  // Too broad!
    log.error("Error", e);
}

// ✅ FIX: Catch specific exceptions
try {
    processMarket(marketId);
} catch (MarketNotFoundException e) {
    log.error("Market not found: {}", marketId, e);
    throw e;
} catch (ValidationException e) {
    log.error("Invalid market data: {}", marketId, e);
    throw e;
}

// ❌ HIGH: Using System.out.println
public void createMarket(MarketDTO dto) {
    System.out.println("Creating market: " + dto.getName());
    // ...
}

// ✅ FIX: Use proper logging
@Slf4j
public class MarketService {
    public Market createMarket(MarketDTO dto) {
        log.info("Creating market: {}", dto.getName());
        // ...
    }
}

// ❌ MEDIUM: Not using Optional
public Market findById(Long id) {
    return marketRepository.findById(id);  // Can return null
}

// ✅ FIX: Use Optional
public Optional<Market> findById(Long id) {
    return marketRepository.findById(id);
}

// Usage:
marketService.findById(1L)
    .ifPresentOrElse(
        market -> log.info("Found: {}", market),
        () -> log.warn("Market not found")
    );

// ❌ MEDIUM: Mutable collection exposed
public class Market {
    private List<Trade> trades = new ArrayList<>();

    public List<Trade> getTrades() {
        return trades;  // Callers can modify internal state!
    }
}

// ✅ FIX: Return immutable collection
public class Market {
    private final List<Trade> trades = new ArrayList<>();

    public List<Trade> getTrades() {
        return Collections.unmodifiableList(trades);
    }
}
```

---

## Review Report Template

```
Code Review Report
==================

Branch: feature/market-validation
Files Changed: 8
Severity: 🔴 CRITICAL issues found

CRITICAL Issues (Block Commit):
================================

1. [CRITICAL] app/Http/Controllers/MarketController.php:45
   Issue: SQL injection vulnerability
   Code: DB::select("SELECT * FROM markets WHERE id = {$id}")
   Fix: Use parameter binding: DB::table('markets')->where('id', $id)->get()

2. [CRITICAL] src/config/api.ts:12
   Issue: Hardcoded API key
   Code: const API_KEY = 'sk-1234567890'
   Fix: Use environment variable: process.env.API_KEY

HIGH Priority Issues:
=====================

3. [HIGH] app/Services/MarketService.php:67
   Issue: Missing return type declaration
   Code: public function calculateTotal($items)
   Fix: Add types: public function calculateTotal(array $items): float

4. [HIGH] src/services/validator.ts:34
   Issue: Using 'any' type
   Code: function validate(data: any): any
   Fix: Define proper interfaces for input/output types

5. [HIGH] markets/views.py:89
   Issue: Bare except clause
   Code: except:
   Fix: Catch specific exceptions: except (ValueError, KeyError) as e:

MEDIUM Priority Issues:
=======================

6. [MEDIUM] lib/widgets/market_card.dart:123
   Issue: Not using const constructor
   Code: EdgeInsets.all(16.0)
   Fix: Use const: const EdgeInsets.all(16.0)

7. [MEDIUM] src/components/MarketList.tsx:45
   Issue: Missing accessibility props
   Code: <TouchableOpacity onPress={...}>
   Fix: Add accessibilityLabel and accessibilityRole

Code Quality Metrics:
=====================

Functions > 50 lines: 3
  - MarketService.php:createMarket() (78 lines)
  - validator.ts:validateMarketData() (65 lines)
  - views.py:market_detail_view() (92 lines)

Files > 800 lines: 1
  - app/Services/MarketService.php (1,245 lines)
    Suggestion: Split into smaller services

Missing Tests: 5 new functions without test coverage
  - calculateMarketScore()
  - validateMarketRules()
  - processMarketResolution()
  - formatMarketData()
  - aggregateMarketStats()

Debug Statements Found: 7
  - console.log() x3
  - var_dump() x2
  - print() x2

Summary:
========

✅ Passed: 12 files
❌ Failed: 8 files
🔴 CRITICAL: 2 issues (MUST FIX BEFORE COMMIT)
🟠 HIGH: 3 issues
🟡 MEDIUM: 2 issues

Action Required:
- Fix all CRITICAL issues before committing
- Address HIGH priority issues
- Run tests after fixes
- Request re-review after changes

Estimated Fix Time: 30-45 minutes
```

---

## Automated Review Commands

### Run Linters/Static Analysis

**JavaScript/TypeScript:**
```bash
npm run lint
npx eslint src/
npx prettier --check src/
npx tsc --noEmit  # Type checking
```

**PHP/Laravel:**
```bash
./vendor/bin/phpcs  # Code style
./vendor/bin/phpstan analyse  # Static analysis
./vendor/bin/psalm  # Type checking
php artisan test  # Run tests
```

**Python:**
```bash
flake8 .  # Style checking
pylint app/  # Code analysis
mypy .  # Type checking
black --check .  # Formatting
```

**Flutter:**
```bash
flutter analyze  # Dart analyzer
dart format --set-exit-if-changed .
flutter test  # Run tests
```

**React Native:**
```bash
npm run lint
npx tsc --noEmit
npm test
```

**Java/Spring Boot:**
```bash
mvn checkstyle:check  # Code style
mvn spotbugs:check  # Bug detection
mvn test  # Run tests
```

---

## Integration with Git Hooks

### Pre-Commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash

echo "Running code review checks..."

# Detect project type and run appropriate checks
if [ -f "package.json" ]; then
    npm run lint || exit 1
    npm test || exit 1
elif [ -f "composer.json" ]; then
    ./vendor/bin/phpcs || exit 1
    php artisan test || exit 1
elif [ -f "manage.py" ]; then
    flake8 . || exit 1
    pytest || exit 1
elif [ -f "pubspec.yaml" ]; then
    flutter analyze || exit 1
    flutter test || exit 1
elif [ -f "pom.xml" ]; then
    mvn checkstyle:check || exit 1
    mvn test || exit 1
fi

echo "✅ Code review checks passed"
```

---

## Best Practices

**DO:**
- ✅ Review security issues first (highest priority)
- ✅ Check for hardcoded secrets and credentials
- ✅ Verify input validation on all user inputs
- ✅ Ensure error handling is present
- ✅ Check test coverage for new code
- ✅ Look for performance issues (N+1 queries, memory leaks)
- ✅ Verify accessibility compliance

**DON'T:**
- ❌ Approve code with CRITICAL security issues
- ❌ Skip checking for debug statements
- ❌ Ignore code quality metrics (complexity, length)
- ❌ Overlook missing tests
- ❌ Let style issues accumulate
- ❌ Rush reviews for "quick fixes"

---

## Integration with Other Commands

- Use `/tdd` to add missing tests
- Use `/build-and-fix` to resolve linting/type errors
- Use `/refactor-clean` for code quality improvements
- Use `/test-coverage` to ensure adequate coverage

---

## Related Agents

This command can invoke the `code-reviewer` agent:
`~/.claude/agents/code-reviewer.md`

For security-specific reviews, invoke `security-reviewer` agent:
`~/.claude/agents/security-reviewer.md`
