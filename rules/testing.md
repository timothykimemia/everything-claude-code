# Testing Requirements

Comprehensive testing standards for all supported languages and frameworks. Testing is mandatory for all code changes.

---

## Universal Testing Standards

### Minimum Test Coverage: 80%

**Coverage Requirements:**
- **80% minimum** for all code
- **100% required** for:
  - Financial calculations
  - Authentication/authorization
  - Security-critical code
  - Payment processing
  - Core business logic

### Test Types (ALL Required)

1. **Unit Tests** - Individual functions, utilities, components
2. **Integration Tests** - API endpoints, database operations, service interactions
3. **E2E Tests** - Critical user flows only

---

## Test-Driven Development (TDD)

**MANDATORY workflow for all languages:**

1. **Write test first (RED)** - Test should fail
2. **Run test** - Verify it fails for the right reason
3. **Write minimal implementation (GREEN)** - Just enough to pass
4. **Run test** - Verify it passes
5. **Refactor (IMPROVE)** - Clean up while keeping tests green
6. **Verify coverage** - Ensure 80%+ coverage

---

## Testing Frameworks by Language

### JavaScript/TypeScript
- **Unit/Integration**: Jest, Vitest
- **E2E**: Playwright, Cypress
- **Component Testing**: Testing Library, React Testing Library
- **Mocking**: jest.mock(), jest.fn()

**Test file naming:**
- `*.test.ts` or `*.test.tsx`
- `*.spec.ts` or `*.spec.tsx`
- `__tests__/*.ts` directory

**Example:**
```typescript
describe('calculateMarketScore', () => {
  it('returns high score for liquid markets', () => {
    const market = { volume: 10000, spread: 0.01 };
    expect(calculateMarketScore(market)).toBeGreaterThan(80);
  });

  it('handles null input gracefully', () => {
    expect(() => calculateMarketScore(null)).toThrow('Invalid input');
  });
});
```

### PHP/Laravel
- **Unit/Integration**: PHPUnit, Pest
- **Feature Tests**: Laravel HTTP testing
- **Browser Tests**: Laravel Dusk
- **Mocking**: Mockery

**Test file naming:**
- `tests/Unit/*Test.php`
- `tests/Feature/*Test.php`
- `*_test.php` (Pest)

**Example:**
```php
<?php

// PHPUnit
class MarketValidatorTest extends TestCase
{
    /** @test */
    public function it_validates_market_data(): void
    {
        $validator = new MarketValidator();
        $data = new MarketData(name: 'Test', minimumBet: 10.0);

        $this->assertNull($validator->validate($data));
    }
}

// Pest
test('it validates market data', function () {
    $validator = new MarketValidator();
    $data = new MarketData(name: 'Test', minimumBet: 10.0);

    expect($validator->validate($data))->toBeNull();
});
```

### Python/Django
- **Unit/Integration**: pytest, unittest
- **Django Tests**: Django TestCase, APITestCase
- **Mocking**: unittest.mock, pytest-mock
- **Fixtures**: pytest fixtures

**Test file naming:**
- `test_*.py`
- `*_test.py`
- Place in `tests/` directory

**Example:**
```python
import pytest
from markets.services import MarketScorer

def test_calculates_high_score_for_active_market():
    """Should return high score for active markets."""
    scorer = MarketScorer()
    market = {'trades': 1000, 'volume': 500000}

    score = scorer.calculate_score(market)

    assert score > 80
    assert score <= 100

@pytest.mark.django_db
def test_creates_market_in_database():
    """Should persist market to database."""
    market = Market.objects.create(name='Test', slug='test')

    assert Market.objects.count() == 1
    assert market.slug == 'test'
```

### Flutter/Dart
- **Unit**: flutter_test (test package)
- **Widget**: WidgetTester
- **Integration**: integration_test package
- **Mocking**: mocktail, mockito

**Test file naming:**
- `test/*_test.dart`
- Mirror source structure

**Example:**
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/services/market_scorer.dart';

void main() {
  group('MarketScorer', () {
    late MarketScorer scorer;

    setUp(() {
      scorer = MarketScorer();
    });

    test('calculates high score for active market', () {
      final market = Market(trades: 1000, volume: 500000);

      final score = scorer.calculateScore(market);

      expect(score, greaterThan(80));
      expect(score, lessThanOrEqualTo(100));
    });

    test('handles null input gracefully', () {
      expect(
        () => scorer.calculateScore(null),
        throwsA(isA<ArgumentError>()),
      );
    });
  });
}
```

### React Native
- **Unit**: Jest
- **Component**: React Native Testing Library
- **E2E**: Detox, Maestro
- **Mocking**: jest.mock()

**Test file naming:**
- `*.test.ts` or `*.test.tsx`
- `__tests__/*.ts` directory

**Example:**
```typescript
import { render, fireEvent } from '@testing-library/react-native';
import { MarketCard } from './MarketCard';

describe('MarketCard', () => {
  it('renders market information', () => {
    const market = { id: '1', name: 'Test Market', status: 'open' };

    const { getByText } = render(<MarketCard market={market} />);

    expect(getByText('Test Market')).toBeTruthy();
    expect(getByText('open')).toBeTruthy();
  });

  it('calls onPress when tapped', () => {
    const onPress = jest.fn();
    const market = { id: '1', name: 'Test', status: 'open' };

    const { getByTestId } = render(
      <MarketCard market={market} onPress={onPress} />
    );

    fireEvent.press(getByTestId('market-card'));

    expect(onPress).toHaveBeenCalledWith('1');
  });
});
```

### Java/Spring Boot
- **Unit**: JUnit 5
- **Integration**: @SpringBootTest
- **Mocking**: Mockito, MockMvc
- **Test Containers**: Testcontainers (for DB)

**Test file naming:**
- `*Test.java`
- Place in `src/test/java/` directory

**Example:**
```java
@ExtendWith(MockitoExtension.class)
class MarketScorerTest {

    @Mock
    private MarketRepository repository;

    @InjectMocks
    private MarketScorer scorer;

    @Test
    @DisplayName("should calculate high score for active market")
    void shouldCalculateHighScoreForActiveMarket() {
        // Given
        Market market = new Market(1000L, 500000.0);

        // When
        double score = scorer.calculateScore(market);

        // Then
        assertTrue(score > 80);
        assertTrue(score <= 100);
    }

    @Test
    @DisplayName("should throw exception for null input")
    void shouldThrowExceptionForNullInput() {
        assertThrows(
            IllegalArgumentException.class,
            () -> scorer.calculateScore(null)
        );
    }
}
```

---

## Running Tests

### JavaScript/TypeScript
```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch

# With coverage
npm test -- --coverage

# Specific file
npm test -- path/to/test.test.ts
```

### PHP/Laravel
```bash
# Run all tests
php artisan test

# With coverage
php artisan test --coverage

# Specific test
php artisan test --filter MarketValidatorTest

# Pest tests
./vendor/bin/pest --coverage
```

### Python/Django
```bash
# Run all tests
pytest

# With coverage
pytest --cov=.

# Specific test
pytest tests/test_market_scorer.py

# Django runner
python manage.py test
```

### Flutter/Dart
```bash
# Run all tests
flutter test

# With coverage
flutter test --coverage

# Specific file
flutter test test/services/market_scorer_test.dart
```

### React Native
```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch

# With coverage
npm test -- --coverage
```

### Java/Spring Boot
```bash
# Maven
mvn test

# With coverage
mvn test jacoco:report

# Gradle
gradle test
gradle test jacocoTestReport
```

---

## Test Organization

### Test Structure

All tests should follow **AAA pattern** (Arrange-Act-Assert):

```typescript
test('calculates total price', () => {
  // Arrange
  const items = [{ price: 10 }, { price: 20 }];
  const calculator = new PriceCalculator();

  // Act
  const total = calculator.calculateTotal(items);

  // Assert
  expect(total).toBe(30);
});
```

### Test Naming

**Use descriptive names that explain what is being tested:**

✅ **Good:**
- `test_creates_market_with_valid_data()`
- `shouldThrowExceptionForNegativeAmount()`
- `it('returns high score for liquid markets')`

❌ **Bad:**
- `test1()`
- `testMarket()`
- `it('works')`

---

## What to Test

### DO Test:
- ✅ **Happy paths** - Normal successful execution
- ✅ **Error paths** - Invalid inputs, exceptions
- ✅ **Edge cases** - Null, empty, boundary values
- ✅ **Business logic** - Calculations, validations, rules
- ✅ **API contracts** - Request/response formats
- ✅ **Database operations** - CRUD, queries, transactions
- ✅ **User interactions** - Clicks, form submissions

### DON'T Test:
- ❌ Framework internals (React, Laravel, etc.)
- ❌ Third-party libraries
- ❌ Trivial getters/setters
- ❌ Implementation details
- ❌ Generated code (unless critical)

---

## Mocking Best Practices

### When to Mock:
- ✅ External APIs (HTTP requests)
- ✅ Database calls (for unit tests)
- ✅ File system operations
- ✅ Time/dates (use fixed time)
- ✅ Random number generation
- ✅ Third-party services

### When NOT to Mock:
- ❌ Integration tests (use real dependencies)
- ❌ Internal business logic
- ❌ Simple utility functions
- ❌ DTOs and data objects

---

## Test Coverage Tools

### Viewing Coverage Reports

**JavaScript/TypeScript:**
```bash
npm test -- --coverage
open coverage/lcov-report/index.html
```

**PHP/Laravel:**
```bash
php artisan test --coverage
open coverage/html/index.html
```

**Python:**
```bash
pytest --cov=. --cov-report=html
open htmlcov/index.html
```

**Flutter:**
```bash
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

**Java/Spring Boot:**
```bash
mvn test jacoco:report
open target/site/jacoco/index.html
```

---

## Continuous Integration

### Pre-commit Hooks

**Run tests before every commit:**

```bash
# .git/hooks/pre-commit (or use husky, lint-staged)

#!/bin/bash

echo "Running tests..."

# Detect project type and run appropriate tests
if [ -f "package.json" ]; then
    npm test || exit 1
elif [ -f "composer.json" ]; then
    php artisan test || exit 1
elif [ -f "manage.py" ]; then
    pytest || exit 1
elif [ -f "pubspec.yaml" ]; then
    flutter test || exit 1
elif [ -f "pom.xml" ]; then
    mvn test || exit 1
fi

echo "✅ All tests passed"
```

### CI/CD Pipeline

**GitHub Actions example:**

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run tests
        run: npm test -- --coverage

      - name: Check coverage threshold
        run: |
          COVERAGE=$(jq '.total.lines.pct' coverage/coverage-summary.json)
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80%"
            exit 1
          fi
```

---

## Troubleshooting Test Failures

### Common Issues

1. **Flaky Tests**
   - Remove time dependencies (use fixed time)
   - Avoid race conditions (use proper async/await)
   - Clear state between tests

2. **Slow Tests**
   - Mock external dependencies
   - Use test databases (in-memory)
   - Parallelize test execution

3. **Test Isolation**
   - Each test should be independent
   - Reset state in setup/teardown
   - Don't rely on execution order

4. **Assertion Failures**
   - Check actual vs expected values
   - Verify mock setup is correct
   - Ensure test data is valid

---

## Agent Support

**Use these agents for testing assistance:**

- **tdd-guide** - Enforces write-tests-first methodology (PROACTIVE)
- **e2e-runner** - Playwright E2E testing specialist
- **test-coverage** - Analyzes coverage and generates missing tests

**Commands:**
- `/tdd` - Start TDD workflow
- `/test-coverage` - Check coverage and generate tests
- `/e2e` - Run end-to-end tests

---

## Testing Checklist

Before marking code as complete:

- [ ] All public functions have unit tests
- [ ] All API endpoints have integration tests
- [ ] Critical user flows have E2E tests
- [ ] Edge cases covered (null, empty, invalid)
- [ ] Error paths tested (not just happy path)
- [ ] Mocks used for external dependencies
- [ ] Tests are independent (no shared state)
- [ ] Test names are descriptive
- [ ] Coverage is 80%+ (100% for critical code)
- [ ] All tests pass in CI/CD pipeline

---

## Resources

- **See `/tdd` command** for comprehensive TDD workflows
- **See `commands/tdd.md`** for language-specific examples
- **See `skills/tdd-workflow/SKILL.md`** for testing patterns
- **See `agents/tdd-guide.md`** for TDD agent documentation

---

**Remember**: Tests are not optional. They are the safety net that enables confident refactoring, rapid development, and production reliability. Write tests first, keep them green, and maintain high coverage across ALL supported languages.
