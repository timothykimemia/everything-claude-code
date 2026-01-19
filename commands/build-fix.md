# Build and Fix

Incrementally fix build errors across all supported languages and frameworks. This command automates the error detection, analysis, and resolution workflow.

---

## Workflow

1. **Run build** - Execute language-specific build command
2. **Parse errors** - Group by file, sort by severity
3. **Fix iteratively** - Fix one error at a time for safety
4. **Verify** - Re-run build after each fix
5. **Report** - Show summary of fixes applied

---

## Build Commands by Language

### JavaScript/TypeScript

```bash
# Build
npm run build
pnpm build
yarn build

# Type checking
npx tsc --noEmit

# Linting
npm run lint
```

**Common Errors:**
- `TS2339`: Property does not exist on type
- `TS2345`: Argument type not assignable
- `TS7006`: Parameter implicitly has 'any' type
- `TS2322`: Type not assignable to type
- Module resolution errors

### PHP/Laravel

```bash
# Composer validation
composer validate

# Check syntax errors
find app -name "*.php" -exec php -l {} \;

# Laravel static analysis (if installed)
./vendor/bin/phpstan analyse
./vendor/bin/psalm

# Check config cache
php artisan config:clear
php artisan config:cache
```

**Common Errors:**
- **Syntax errors**: Missing semicolons, unclosed brackets
- **Class not found**: Incorrect namespaces or missing use statements
- **Composer errors**: Version conflicts, missing extensions
- **Migration errors**: Column type mismatches, foreign key constraints
- **Type errors**: Return type mismatches (with strict types)

**Example Fixes:**

```php
// ERROR: Class 'App\Models\User' not found
// FIX: Add use statement
use App\Models\User;

// ERROR: Return type mismatch
// FIX: Add proper return type
public function getTotal(): float // was: public function getTotal()
{
    return $this->items->sum('price');
}

// ERROR: Undefined property
// FIX: Add property declaration
private float $total; // Add to class properties
```

### Python/Django

```bash
# Check for errors
python manage.py check

# Check migrations
python manage.py makemigrations --dry-run --check

# Type checking (if mypy installed)
mypy .

# Linting
flake8 .
pylint app/

# Import checking
python -m py_compile app/**/*.py
```

**Common Errors:**
- **ImportError / ModuleNotFoundError**: Missing dependencies or wrong import paths
- **IndentationError**: Incorrect spacing (mixing tabs/spaces)
- **SyntaxError**: Invalid Python syntax
- **Migration conflicts**: Multiple migration heads, conflicting operations
- **Type errors**: Type hint mismatches (with mypy)

**Example Fixes:**

```python
# ERROR: ModuleNotFoundError: No module named 'rest_framework'
# FIX: Install dependency
pip install djangorestframework
# Add to INSTALLED_APPS in settings.py

# ERROR: IndentationError: expected an indented block
# FIX: Ensure consistent indentation (4 spaces)
def calculate_total(items: list) -> float:
    total = 0.0  # Must be indented
    for item in items:
        total += item.price
    return total

# ERROR: Type mismatch - Expected 'int', got 'str'
# FIX: Add proper type conversion
user_id = int(request.GET.get('user_id', 0))  # was: user_id = request.GET.get('user_id')

# ERROR: Migration conflict detected
# FIX: Merge migrations
python manage.py makemigrations --merge
```

### Flutter/Dart

```bash
# Analyze code
flutter analyze

# Check for compile errors
flutter build apk --dry-run  # Android
flutter build ios --dry-run  # iOS

# Format check
dart format --set-exit-if-changed .

# Dependency check
flutter pub outdated
```

**Common Errors:**
- **Null safety errors**: Non-nullable variable used before initialization
- **Type errors**: Wrong type passed to function
- **Missing dependencies**: pubspec.yaml version conflicts
- **Widget errors**: Incorrect widget tree structure
- **Platform-specific errors**: iOS/Android build configuration issues

**Example Fixes:**

```dart
// ERROR: The property 'title' can't be unconditionally accessed
// because the receiver can be 'null'.
// FIX: Add null check or null-aware operator
final title = market?.title ?? 'Unknown';  // was: final title = market.title;

// ERROR: The argument type 'String?' can't be assigned to parameter type 'String'
// FIX: Handle nullable string
Text(market.name ?? '')  // was: Text(market.name)

// ERROR: The getter 'length' isn't defined for type 'List<dynamic>'
// FIX: Add proper type
final List<Market> markets = data as List<Market>;  // was: final markets = data;
final count = markets.length;

// ERROR: A value of type 'Future<String>' can't be assigned to variable of type 'String'
// FIX: Use await or FutureBuilder
final name = await fetchMarketName();  // was: final name = fetchMarketName();

// ERROR: Missing required parameter 'onPressed'
// FIX: Add required parameter
ElevatedButton(
  onPressed: () => _submitForm(),  // Add this
  child: Text('Submit'),
)
```

### React Native

```bash
# Type checking
npx tsc --noEmit

# Linting
npm run lint

# Metro bundler check
npx react-native doctor

# Platform-specific build
npx react-native run-android --dry-run
npx react-native run-ios --dry-run

# Check native dependencies
cd ios && pod install && cd ..
```

**Common Errors:**
- **Type errors**: TypeScript type mismatches
- **Module resolution**: Cannot find module errors
- **Native module errors**: Linking issues with native dependencies
- **Metro bundler errors**: Bundle transformation failures
- **Platform-specific errors**: iOS/Android build configuration issues

**Example Fixes:**

```typescript
// ERROR: Property 'navigation' does not exist on type
// FIX: Add proper navigation typing
import { NativeStackScreenProps } from '@react-navigation/native-stack';

type Props = NativeStackScreenProps<RootStackParamList, 'MarketDetails'>;

const MarketDetailsScreen = ({ navigation, route }: Props) => {
  // navigation is now properly typed
};

// ERROR: Cannot find module '@react-native-async-storage/async-storage'
// FIX: Install and link native module
npm install @react-native-async-storage/async-storage
cd ios && pod install && cd ..
npx react-native link @react-native-async-storage/async-storage

// ERROR: Type 'null' is not assignable to type 'Market'
// FIX: Handle nullable state properly
const [market, setMarket] = useState<Market | null>(null);
// Later use: market?.name or market ? <View>...</View> : <Loading />

// ERROR: VirtualizedList: missing keys for items
// FIX: Add keyExtractor to FlatList
<FlatList
  data={markets}
  keyExtractor={(item) => item.id}  // Add this
  renderItem={({ item }) => <MarketCard market={item} />}
/>

// ERROR: Invariant Violation: requireNativeComponent: "RNSScreen" was not found
// FIX: Rebuild native modules
rm -rf node_modules
npm install
cd ios && pod install && cd ..
npx react-native run-ios  # or run-android
```

### Java/Spring Boot

```bash
# Maven build
mvn clean compile
mvn verify

# Gradle build
gradle clean build
gradle check

# Check for compilation errors
mvn compiler:compile

# Run tests
mvn test

# Check dependency tree
mvn dependency:tree
```

**Common Errors:**
- **Compilation errors**: Syntax errors, missing imports
- **Dependency conflicts**: Version mismatches, missing dependencies
- **Bean creation errors**: Spring configuration issues
- **Annotation processor errors**: Lombok, MapStruct configuration
- **Test failures**: Failed assertions, mock issues

**Example Fixes:**

```java
// ERROR: cannot find symbol - class Market
// FIX: Add import statement
import com.example.markets.model.Market;

// ERROR: incompatible types: String cannot be converted to LocalDateTime
// FIX: Parse date properly
LocalDateTime endDate = LocalDateTime.parse(dateStr, DateTimeFormatter.ISO_DATE_TIME);
// was: LocalDateTime endDate = dateStr;

// ERROR: Field injection is not recommended
// FIX: Use constructor injection
private final MarketService marketService;

@Autowired
public MarketController(MarketService marketService) {
    this.marketService = marketService;
}
// was: @Autowired private MarketService marketService;

// ERROR: Bean creation exception - Field marketRepository required a bean of type
// that could not be found
// FIX: Add @Repository annotation to repository interface
@Repository
public interface MarketRepository extends JpaRepository<Market, Long> {}

// ERROR: Method does not override method from its superclass
// FIX: Ensure method signature matches interface
@Override
public List<Market> findAll() {  // Return type must match interface
    return marketRepository.findAll();
}

// ERROR: org.springframework.beans.factory.BeanCreationException: Error creating
// bean with name 'entityManagerFactory'
// FIX: Check database connection in application.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/markets
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
```

---

## Error Resolution Strategy

### 1. Parse Error Output

Group errors by:
- **Severity**: Critical (build-breaking) → Warnings
- **File**: Group errors by affected file
- **Type**: Syntax, type, dependency, runtime

### 2. Fix Iteratively

For each error:

1. **Show context** (5 lines before/after error)
2. **Explain issue** clearly
3. **Propose fix** with rationale
4. **Apply fix** with minimal changes
5. **Re-run build** to verify
6. **Confirm resolution** or rollback

### 3. Stop Conditions

Stop and report if:
- ✋ Fix introduces new errors
- ✋ Same error persists after 3 attempts
- ✋ User requests pause
- ✋ Scope of changes becomes too large

### 4. Verification

After each fix:
```bash
# Run language-specific build
# JavaScript/TypeScript
npm run build && npm test

# PHP/Laravel
composer validate && php artisan test

# Python/Django
python manage.py check && pytest

# Flutter
flutter analyze && flutter test

# React Native
npx tsc && npm test

# Java/Spring Boot
mvn verify
```

---

## Language-Specific Error Categories

### Type Errors

**JavaScript/TypeScript:**
- Implicit `any` types
- Missing type definitions
- Incompatible union types
- Generic type mismatches

**PHP/Laravel:**
- Return type mismatches (with `declare(strict_types=1)`)
- Property type incompatibilities
- Parameter type violations

**Python:**
- Type hint mismatches (mypy)
- Union type errors
- Protocol violations

**Dart/Flutter:**
- Null safety violations
- Generic type constraints
- Widget type mismatches

**Java/Spring Boot:**
- Generic type erasure
- Autoboxing issues
- Wildcard type errors

### Dependency Errors

**JavaScript/TypeScript:**
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Version conflicts
npm ls <package-name>
npm dedupe
```

**PHP/Laravel:**
```bash
# Clear cache and reinstall
rm -rf vendor composer.lock
composer install

# Version conflicts
composer show <package-name>
composer update <package-name>
```

**Python/Django:**
```bash
# Clear cache and reinstall
pip freeze > requirements.txt
pip install -r requirements.txt --force-reinstall

# Virtual environment recreation
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**Flutter:**
```bash
# Clear cache and reinstall
flutter clean
rm pubspec.lock
flutter pub get

# Dependency conflicts
flutter pub outdated
flutter pub upgrade
```

**React Native:**
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
cd ios && pod install && cd ..

# Clear Metro bundler cache
npx react-native start --reset-cache
```

**Java/Spring Boot:**
```bash
# Maven - clear and rebuild
mvn clean install -U

# Gradle - clear and rebuild
gradle clean build --refresh-dependencies
```

### Configuration Errors

**Laravel:**
```bash
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
```

**Django:**
```bash
python manage.py migrate --run-syncdb
python manage.py collectstatic --clear
```

**Spring Boot:**
```bash
# Check application.properties/yaml
# Validate database connection
# Check port conflicts
```

---

## Summary Report Template

After completing fixes:

```
Build Fix Summary
=================

✅ Errors Fixed: X
❌ Errors Remaining: Y
⚠️  New Errors Introduced: Z

Files Modified:
- file1.ext (2 fixes)
- file2.ext (1 fix)

Fixes Applied:
1. [File:Line] Error type - Description
   Fix: Brief description of fix

2. [File:Line] Error type - Description
   Fix: Brief description of fix

Remaining Issues:
1. [File:Line] Error type - Description
   Reason: Why not fixed (scope/complexity/needs investigation)

Next Steps:
- [ ] Address remaining type errors in module X
- [ ] Investigate dependency conflict in package Y
- [ ] Run full test suite after fixes
```

---

## Best Practices

**DO:**
- ✅ Fix one error at a time
- ✅ Run build after each fix
- ✅ Keep changes minimal and focused
- ✅ Preserve existing functionality
- ✅ Add tests for fixed code
- ✅ Document complex fixes

**DON'T:**
- ❌ Fix multiple errors simultaneously
- ❌ Make sweeping refactors during error fixes
- ❌ Ignore warnings (they often become errors)
- ❌ Skip verification steps
- ❌ Apply fixes without understanding the error

---

## Integration with Other Commands

- Use `/plan` to understand large-scale fixes needed
- Use `/tdd` when fixing requires new implementation
- Use `/code-review` after completing fixes
- Use `/test-coverage` to ensure fixes are tested

---

## Related Agents

This command can invoke the `build-error-resolver` agent:
`~/.claude/agents/build-error-resolver.md`
