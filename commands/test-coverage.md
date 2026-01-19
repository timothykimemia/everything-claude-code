# Test Coverage

Analyze test coverage and generate missing tests to ensure comprehensive code quality across all supported languages and frameworks.

---

## Workflow

1. **Run coverage** - Execute language-specific coverage tools
2. **Analyze report** - Identify under-covered files and functions
3. **Generate tests** - Create tests for uncovered code paths
4. **Verify** - Ensure new tests pass
5. **Report** - Show before/after coverage metrics
6. **Target**: Achieve 80%+ overall coverage (100% for critical code)

---

## Coverage Commands by Language

### JavaScript/TypeScript

```bash
# Jest with coverage
npm test -- --coverage
npm test -- --coverage --watchAll=false

# Coverage thresholds in package.json
"jest": {
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  }
}

# View HTML report
open coverage/lcov-report/index.html

# Coverage for specific files
npm test -- --coverage --collectCoverageFrom='src/**/*.{ts,tsx}'

# Exclude files from coverage
npm test -- --coverage --coveragePathIgnorePatterns='node_modules|test'
```

**Coverage Report Location:**
- `coverage/lcov-report/index.html` - HTML report
- `coverage/coverage-summary.json` - JSON summary
- `coverage/lcov.info` - LCOV format

**Tools:**
- Jest (built-in coverage with Istanbul/nyc)
- Vitest (`vitest --coverage`)
- Playwright (E2E coverage)

### PHP/Laravel

```bash
# PHPUnit with coverage
php artisan test --coverage
./vendor/bin/phpunit --coverage-html coverage

# Pest with coverage
./vendor/bin/pest --coverage
./vendor/bin/pest --coverage --min=80

# Coverage with filters
php artisan test --coverage --path=app/Services

# Coverage thresholds in phpunit.xml
<coverage>
    <include>
        <directory suffix=".php">./app</directory>
    </include>
    <report>
        <html outputDirectory="coverage/html"/>
        <text outputFile="php://stdout" showOnlySummary="true"/>
    </report>
</coverage>

# View HTML report
open coverage/html/index.html

# Xdebug coverage (more accurate)
XDEBUG_MODE=coverage php artisan test --coverage
```

**Coverage Report Location:**
- `coverage/html/index.html` - HTML report
- `coverage/clover.xml` - Clover XML
- Terminal output with Pest `--coverage` flag

**Requirements:**
- Xdebug or PCOV extension
- PHPUnit 9.3+ or Pest 1.0+

**Tools:**
- PHPUnit (with Xdebug/PCOV)
- Pest (built on PHPUnit)
- PHPStan/Psalm (static analysis complements coverage)

### Python/Django

```bash
# pytest with coverage
pytest --cov=. --cov-report=html
pytest --cov=app --cov-report=term-missing

# Django test runner with coverage
coverage run manage.py test
coverage report
coverage html

# Coverage with minimum threshold
pytest --cov=. --cov-report=html --cov-fail-under=80

# Coverage for specific modules
pytest --cov=markets --cov=users --cov-report=html

# View HTML report
open htmlcov/index.html

# Coverage configuration in pyproject.toml
[tool.coverage.run]
source = ["."]
omit = [
    "*/migrations/*",
    "*/tests/*",
    "*/venv/*",
    "manage.py",
]

[tool.coverage.report]
fail_under = 80
show_missing = true
```

**Coverage Report Location:**
- `htmlcov/index.html` - HTML report
- `.coverage` - Coverage data file
- Terminal output with `--cov-report=term`

**Tools:**
- pytest-cov (pytest plugin)
- coverage.py (standalone tool)
- Django's built-in test coverage support

### Flutter/Dart

```bash
# Run tests with coverage
flutter test --coverage

# Generate HTML report from LCOV
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html

# Coverage for specific tests
flutter test test/services/ --coverage

# Filter coverage (in test/test.dart)
# Exclude generated files
flutter test --coverage --exclude-from-coverage='*.g.dart|*.freezed.dart'

# Check coverage percentage
lcov --summary coverage/lcov.info

# Set minimum coverage threshold (custom script)
# Add to pubspec.yaml scripts
# Then: dart tool/check_coverage.dart

# Integration test coverage
flutter test integration_test/ --coverage
```

**Coverage Report Location:**
- `coverage/lcov.info` - LCOV format
- `coverage/html/index.html` - HTML report (after genhtml)

**Requirements:**
- `lcov` tool installed (`brew install lcov` on macOS)
- Flutter SDK with test support

**Tools:**
- Flutter test (built-in coverage)
- lcov (HTML report generation)
- Very Good Coverage (package for enforcing thresholds)

**Add to pubspec.yaml:**
```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  very_good_coverage: ^2.0.0  # For coverage enforcement
```

### React Native

```bash
# Jest with coverage (same as JavaScript/TypeScript)
npm test -- --coverage
npm test -- --coverage --collectCoverageFrom='src/**/*.{ts,tsx}'

# Coverage for native code (iOS/Android)
# iOS - Xcode coverage tools
# Android - Jacoco

# Coverage thresholds in package.json
"jest": {
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  },
  "collectCoverageFrom": [
    "src/**/*.{ts,tsx}",
    "!src/**/*.test.{ts,tsx}",
    "!src/**/__tests__/**"
  ]
}

# View HTML report
open coverage/lcov-report/index.html

# Coverage with E2E tests (Detox)
# Requires additional setup for E2E coverage
```

**Coverage Report Location:**
- `coverage/lcov-report/index.html` - HTML report
- `coverage/coverage-summary.json` - JSON summary

**Tools:**
- Jest (JavaScript/TypeScript coverage)
- Detox (E2E testing)
- Xcode Coverage (iOS native)
- Jacoco (Android native)

### Java/Spring Boot

```bash
# Maven with JaCoCo
mvn test jacoco:report
open target/site/jacoco/index.html

# Gradle with JaCoCo
gradle test jacocoTestReport
open build/reports/jacoco/test/html/index.html

# Coverage with minimum threshold
mvn test jacoco:report jacoco:check

# JaCoCo configuration in pom.xml (Maven)
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>PACKAGE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>

# JaCoCo configuration in build.gradle (Gradle)
jacoco {
    toolVersion = "0.8.10"
}

jacocoTestReport {
    reports {
        xml.required = true
        html.required = true
    }
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                minimum = 0.80
            }
        }
    }
}
```

**Coverage Report Location:**
- Maven: `target/site/jacoco/index.html`
- Gradle: `build/reports/jacoco/test/html/index.html`
- `jacoco.xml` - XML report for CI/CD

**Tools:**
- JaCoCo (Java Code Coverage)
- IntelliJ IDEA coverage runner
- SonarQube (for advanced analysis)

---

## Coverage Analysis Strategy

### 1. Identify Under-Covered Code

**Coverage Thresholds:**
- **80% minimum** for all code
- **100% required** for:
  - Financial calculations
  - Authentication/authorization logic
  - Security-critical code
  - Core business logic
  - Payment processing
  - Data validation

**Check coverage by:**
- **Lines**: Percentage of code lines executed
- **Branches**: Percentage of decision branches covered
- **Functions**: Percentage of functions called
- **Statements**: Percentage of statements executed

### 2. Prioritize Test Creation

**High Priority (Must Cover):**
1. Public APIs and endpoints
2. Business logic and calculations
3. Error handling and edge cases
4. Security-related code
5. Data transformations

**Medium Priority (Should Cover):**
1. Service layer methods
2. Repository/DAO methods
3. Utility functions
4. Validators and formatters

**Low Priority (Nice to Cover):**
1. Simple getters/setters
2. Configuration files
3. Type definitions/interfaces
4. Constants and enums

### 3. Test Types to Generate

**Unit Tests:**
- Test individual functions/methods in isolation
- Mock dependencies
- Fast execution
- Cover 70-80% of codebase

**Integration Tests:**
- Test component interactions
- Real dependencies (databases, APIs)
- Test realistic scenarios
- Cover 15-20% of codebase

**E2E Tests:**
- Test complete user flows
- Full stack integration
- Critical paths only
- Cover 5-10% of codebase

---

## Coverage Report Examples

### JavaScript/TypeScript Coverage Output

```
-------------------|---------|----------|---------|---------|-------------------
File               | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
-------------------|---------|----------|---------|---------|-------------------
All files          |   85.5  |   78.2   |   88.9  |   85.5  |
 src/              |   90.0  |   85.0   |   100   |   90.0  |
  liquidity.ts     |   100   |   100    |   100   |   100   |
  market.ts        |   80.0  |   70.0   |   100   |   80.0  | 45-48,62
 src/services/     |   75.0  |   66.7   |   75.0  |   75.0  |
  validator.ts     |   75.0  |   66.7   |   75.0  |   75.0  | 23-27,34-36
-------------------|---------|----------|---------|---------|-------------------
```

**Action Required:**
- `market.ts` lines 45-48, 62 need coverage
- `validator.ts` lines 23-27, 34-36 need coverage

### PHP/Laravel Coverage Output

```
Summary:
  Classes:  90.0% (9/10)
  Methods:  85.5% (47/55)
  Lines:    82.3% (456/554)

App\Services\MarketValidator:
  Methods: 75.0% (3/4)
  Lines:   70.0% (14/20)
  Uncovered: lines 23-27

App\Http\Controllers\MarketController:
  Methods: 100% (8/8)
  Lines:   95.0% (38/40)
  Uncovered: lines 67-68
```

**Action Required:**
- Add tests for `MarketValidator` line 23-27 (error handling)
- Add tests for `MarketController` lines 67-68 (edge case)

### Python Coverage Output

```
Name                        Stmts   Miss  Cover   Missing
---------------------------------------------------------
markets/models.py              45      3    93%   67-69
markets/services/scorer.py     28      0   100%
markets/views.py               89     15    83%   34-38, 56-60, 78
---------------------------------------------------------
TOTAL                         562     48    91%
```

**Action Required:**
- `models.py` lines 67-69 need coverage
- `views.py` lines 34-38, 56-60, 78 need coverage

---

## Generating Missing Tests

### Step-by-Step Process

For each under-covered file:

1. **Analyze uncovered code**
   - Read the function/method
   - Identify code paths (happy path, error paths, edge cases)
   - Check for conditionals (if/else, switch, try/catch)

2. **Design test cases**
   - Happy path: Normal successful execution
   - Error cases: Invalid inputs, exceptions
   - Edge cases: Null, empty, boundary values
   - Branch coverage: All if/else paths

3. **Write tests using TDD approach**
   - Follow language-specific testing patterns
   - Use appropriate assertions
   - Mock external dependencies
   - Keep tests focused and isolated

4. **Verify coverage improvement**
   - Run tests with coverage
   - Check that uncovered lines are now covered
   - Ensure new tests pass

### Language-Specific Test Examples

**JavaScript/TypeScript (Jest):**
```typescript
// Coverage target: error handling in calculateScore()
describe('calculateScore - error handling', () => {
  it('should throw error for negative values', () => {
    expect(() => calculateScore(-10)).toThrow('Value must be positive');
  });

  it('should handle null input gracefully', () => {
    expect(calculateScore(null)).toBe(0);
  });
});
```

**PHP/Laravel (Pest):**
```php
// Coverage target: validation edge cases
test('it rejects empty market name', function () {
    $this->expectException(ValidationException::class);
    $this->expectExceptionMessage('Name cannot be empty');

    $validator->validate(new MarketData(name: ''));
});

test('it handles extremely long names', function () {
    $longName = str_repeat('a', 1000);

    $this->expectException(ValidationException::class);
    $validator->validate(new MarketData(name: $longName));
});
```

**Python (pytest):**
```python
# Coverage target: error paths in scorer
def test_scorer_handles_zero_votes():
    """Should return 0 when total votes is zero."""
    data = ResolutionData(total_votes=0, ...)
    assert scorer.calculate_score(data) == 0

def test_scorer_handles_negative_values():
    """Should raise ValueError for negative values."""
    with pytest.raises(ValueError, match="votes cannot be negative"):
        ResolutionData(total_votes=-1, ...)
```

**Flutter (flutter_test):**
```dart
// Coverage target: widget error states
testWidgets('shows error message when loading fails', (tester) async {
  when(() => mockRepository.fetchMarket('1'))
      .thenThrow(Exception('Network error'));

  await tester.pumpWidget(createTestWidget());
  await tester.pumpAndSettle();

  expect(find.text('Failed to load market'), findsOneWidget);
  expect(find.byType(ErrorWidget), findsOneWidget);
});
```

**Java/Spring Boot (JUnit 5):**
```java
// Coverage target: exception handling
@Test
@DisplayName("should throw exception for invalid market type")
void shouldThrowExceptionForInvalidMarketType() {
    // Given
    BigDecimal amount = new BigDecimal("100.00");

    // When & Then
    assertThatThrownBy(() -> calculator.calculateFee(amount, null))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("Market type cannot be null");
}

@Test
@DisplayName("should handle negative amounts")
void shouldHandleNegativeAmounts() {
    // Given
    BigDecimal amount = new BigDecimal("-50.00");

    // When & Then
    assertThatThrownBy(() -> calculator.calculateFee(amount, MarketType.STANDARD))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("Transaction amount cannot be negative");
}
```

---

## Coverage Report Template

```
Test Coverage Report
====================

Overall Coverage:
  Statements: 85.2% (423/496)
  Branches:   78.5% (156/199)
  Functions:  88.9% (72/81)
  Lines:      85.2% (423/496)

Target: 80% ✅

Files Below Threshold (<80%):
1. src/services/validator.ts - 75.0% (Lines: 23-27, 34-36)
2. src/models/market.ts - 78.0% (Lines: 45-48, 62)

Improvements Needed:
- Add error handling tests for validator
- Cover edge cases in market model

Tests Added This Session:
✅ validator.test.ts - 5 new tests (+12% coverage)
✅ market.test.ts - 3 new tests (+8% coverage)

Before: 73.5% → After: 85.2% (+11.7%)
```

---

## Best Practices

**DO:**
- ✅ Aim for 80%+ coverage overall
- ✅ Require 100% coverage for critical code (finance, auth, security)
- ✅ Test error paths and edge cases
- ✅ Use coverage reports to find gaps
- ✅ Write tests that verify behavior, not implementation
- ✅ Keep tests fast and focused

**DON'T:**
- ❌ Chase 100% coverage on everything (diminishing returns)
- ❌ Test implementation details
- ❌ Write tests just to increase coverage percentage
- ❌ Ignore branch coverage (only checking line coverage)
- ❌ Test generated code (unless critical)
- ❌ Skip E2E tests for critical user flows

---

## Coverage Focus Areas

### Critical Code (100% Required)

1. **Financial calculations**
   - Market pricing, fees, payouts
   - Currency conversions
   - Balance updates

2. **Authentication & Authorization**
   - Login/logout flows
   - Permission checks
   - Token validation

3. **Security**
   - Input validation
   - SQL injection prevention
   - XSS protection
   - CSRF protection

4. **Data integrity**
   - Database transactions
   - Data migrations
   - Audit logging

### Standard Code (80% Target)

1. **Business logic**
   - Service layer methods
   - Domain models
   - Validators

2. **API endpoints**
   - Controllers
   - Request/response handling
   - Error responses

3. **Data access**
   - Repositories
   - Database queries
   - Data mappers

---

## Integration with Other Commands

- Use `/tdd` to write tests BEFORE implementation (best approach)
- Use `/build-and-fix` to resolve test failures
- Use `/code-review` to ensure test quality
- Use `/plan` to design comprehensive test strategy

---

## Related Skills and Agents

- Skill: `tdd-workflow` for test-driven development
- Agent: `tdd-guide` for test generation
- Command: `/e2e` for end-to-end coverage

---

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Test Coverage

on: [push, pull_request]

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests with coverage
        run: npm test -- --coverage
      - name: Check coverage threshold
        run: |
          if [ $(jq '.total.lines.pct' coverage/coverage-summary.json) -lt 80 ]; then
            echo "Coverage below 80%"
            exit 1
          fi
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
```

### Coverage Badges

Add to README.md:
```markdown
![Coverage](https://img.shields.io/codecov/c/github/username/repo)
```
