---
name: tdd-workflow
description: Use this skill when writing new features, fixing bugs, or refactoring code. Enforces test-driven development with 80%+ coverage including unit, integration, and E2E tests.
---

# Test-Driven Development Workflow

This skill ensures all code development follows TDD principles with comprehensive test coverage.

## When to Activate

- Writing new features or functionality
- Fixing bugs or issues
- Refactoring existing code
- Adding API endpoints
- Creating new components

## Core Principles

### 1. Tests BEFORE Code
ALWAYS write tests first, then implement code to make tests pass.

### 2. Coverage Requirements
- Minimum 80% coverage (unit + integration + E2E)
- All edge cases covered
- Error scenarios tested
- Boundary conditions verified

### 3. Test Types

#### Unit Tests
- Individual functions and utilities
- Component logic
- Pure functions
- Helpers and utilities

#### Integration Tests
- API endpoints
- Database operations
- Service interactions
- External API calls

#### E2E Tests (Playwright)
- Critical user flows
- Complete workflows
- Browser automation
- UI interactions

## TDD Workflow Steps

### Step 1: Write User Journeys
```
As a [role], I want to [action], so that [benefit]

Example:
As a user, I want to search for markets semantically,
so that I can find relevant markets even without exact keywords.
```

### Step 2: Generate Test Cases
For each user journey, create comprehensive test cases:

```typescript
describe('Semantic Search', () => {
  it('returns relevant markets for query', async () => {
    // Test implementation
  })

  it('handles empty query gracefully', async () => {
    // Test edge case
  })

  it('falls back to substring search when Redis unavailable', async () => {
    // Test fallback behavior
  })

  it('sorts results by similarity score', async () => {
    // Test sorting logic
  })
})
```

### Step 3: Run Tests (They Should Fail)
```bash
npm test
# Tests should fail - we haven't implemented yet
```

### Step 4: Implement Code
Write minimal code to make tests pass:

```typescript
// Implementation guided by tests
export async function searchMarkets(query: string) {
  // Implementation here
}
```

### Step 5: Run Tests Again
```bash
npm test
# Tests should now pass
```

### Step 6: Refactor
Improve code quality while keeping tests green:
- Remove duplication
- Improve naming
- Optimize performance
- Enhance readability

### Step 7: Verify Coverage
```bash
npm run test:coverage
# Verify 80%+ coverage achieved
```

## Testing Patterns by Language

### JavaScript/TypeScript (Jest/Vitest)
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Button Component', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### PHP/Laravel (PHPUnit/Pest)
```php
<?php

use App\Models\Market;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('creates a market successfully', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->postJson('/api/markets', [
            'name' => 'Bitcoin $100k?',
            'description' => 'Will BTC reach $100k?',
            'end_date' => now()->addDays(30)->toDateTimeString(),
            'category_ids' => [1, 2],
        ]);

    $response->assertStatus(201)
        ->assertJsonStructure([
            'success',
            'data' => ['id', 'name', 'status', 'created_at'],
        ]);

    $this->assertDatabaseHas('markets', [
        'name' => 'Bitcoin $100k?',
        'creator_id' => $user->id,
    ]);
});

it('validates required fields', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->postJson('/api/markets', []);

    $response->assertStatus(422)
        ->assertJsonValidationErrors(['name', 'description', 'end_date']);
});

test('market belongs to creator', function () {
    $market = Market::factory()->create();

    expect($market->creator)->toBeInstanceOf(User::class);
    expect($market->creator->id)->toBe($market->creator_id);
});
```

### Python/Django (pytest-django)
```python
import pytest
from django.contrib.auth import get_user_model
from rest_framework import status
from rest_framework.test import APIClient

User = get_user_model()

@pytest.fixture
def api_client():
    return APIClient()

@pytest.fixture
def user(db):
    return User.objects.create_user(
        username='testuser',
        password='testpass123'
    )

@pytest.fixture
def market(db, user):
    return Market.objects.create(
        name='Bitcoin $100k?',
        description='Will BTC reach $100k?',
        creator=user,
        status=MarketStatus.ACTIVE,
        end_date=timezone.now() + timedelta(days=30)
    )

@pytest.mark.django_db
def test_create_market(api_client, user):
    """Test user can create a market."""
    api_client.force_authenticate(user=user)

    response = api_client.post('/api/markets/', {
        'name': 'Test Market',
        'description': 'Test description for new market',
        'end_date': (timezone.now() + timedelta(days=30)).isoformat(),
        'category_ids': [1, 2],
    })

    assert response.status_code == status.HTTP_201_CREATED
    assert response.data['name'] == 'Test Market'
    assert Market.objects.filter(name='Test Market').exists()

@pytest.mark.django_db
def test_market_validation(api_client, user):
    """Test market creation validation."""
    api_client.force_authenticate(user=user)

    response = api_client.post('/api/markets/', {})

    assert response.status_code == status.HTTP_400_BAD_REQUEST
    assert 'name' in response.data
    assert 'description' in response.data

def test_market_is_active(market):
    """Test market is_active method."""
    assert market.is_active() is True

    market.status = MarketStatus.RESOLVED
    assert market.is_active() is False
```

### Flutter/Dart (flutter_test)
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

void main() {
  group('MarketRepository', () {
    late MarketRepository repository;
    late MockApiClient mockApiClient;
    late MockCacheService mockCache;

    setUp(() {
      mockApiClient = MockApiClient();
      mockCache = MockCacheService();
      repository = MarketRepositoryImpl(
        apiClient: mockApiClient,
        cache: mockCache,
      );
    });

    test('fetchMarkets returns cached data when available', () async {
      // Arrange
      final markets = [
        Market(
          id: '1',
          name: 'Test Market',
          status: MarketStatus.active,
          createdAt: DateTime.now(),
        ),
      ];
      when(mockCache.get<List<Market>>('markets'))
          .thenAnswer((_) async => markets);

      // Act
      final result = await repository.fetchMarkets();

      // Assert
      expect(result, isA<Success<List<Market>>>());
      result.when(
        success: (data) => expect(data, markets),
        error: (_) => fail('Expected success'),
      );
      verifyNever(mockApiClient.get(any));
    });

    test('fetchMarkets fetches from API when cache is empty', () async {
      // Arrange
      when(mockCache.get<List<Market>>('markets'))
          .thenAnswer((_) async => null);
      when(mockApiClient.get('/markets')).thenAnswer(
        (_) async => Response(data: [
          {'id': '1', 'name': 'Test Market', 'status': 'active'},
        ]),
      );

      // Act
      final result = await repository.fetchMarkets();

      // Assert
      expect(result, isA<Success<List<Market>>>());
      verify(mockApiClient.get('/markets')).called(1);
      verify(mockCache.set('markets', any, duration: anyNamed('duration')))
          .called(1);
    });
  });

  testWidgets('MarketCard displays market information', (tester) async {
    // Arrange
    final market = Market(
      id: '1',
      name: 'Bitcoin $100k?',
      description: 'Will BTC reach $100k?',
      status: MarketStatus.active,
      createdAt: DateTime.now(),
    );

    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          body: MarketCard(market: market),
        ),
      ),
    );

    // Assert
    expect(find.text('Bitcoin $100k?'), findsOneWidget);
    expect(find.text('Will BTC reach $100k?'), findsOneWidget);
    expect(find.text('Active'), findsOneWidget);
  });
}
```

### React Native (React Native Testing Library)
```typescript
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { MarketCard } from './MarketCard';
import { MarketStatus } from '@/types';

describe('MarketCard', () => {
  const mockMarket = {
    id: '1',
    name: 'Bitcoin $100k?',
    description: 'Will BTC reach $100k?',
    status: MarketStatus.ACTIVE,
    createdAt: new Date(),
  };

  it('renders market information correctly', () => {
    const { getByText } = render(<MarketCard market={mockMarket} />);

    expect(getByText('Bitcoin $100k?')).toBeTruthy();
    expect(getByText('Will BTC reach $100k?')).toBeTruthy();
  });

  it('calls onPress when pressed', () => {
    const onPressMock = jest.fn();
    const { getByTestId } = render(
      <MarketCard market={mockMarket} onPress={onPressMock} testID="market-card" />
    );

    fireEvent.press(getByTestId('market-card'));

    expect(onPressMock).toHaveBeenCalledTimes(1);
  });

  it('displays status badge', () => {
    const { getByText } = render(<MarketCard market={mockMarket} />);

    expect(getByText('Active')).toBeTruthy();
  });
});

describe('useMarkets hook', () => {
  it('fetches markets on mount', async () => {
    const mockMarkets = [mockMarket];
    (marketApi.fetchMarkets as jest.Mock).mockResolvedValue({
      success: true,
      data: mockMarkets,
    });

    const { result } = renderHook(() => useMarkets());

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.markets).toEqual(mockMarkets);
    expect(result.current.error).toBeNull();
  });
});
```

### Java/Spring Boot (JUnit 5)
```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
class MarketControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private MarketRepository marketRepository;

    @Test
    @WithMockUser(roles = "USER")
    void createMarket_WithValidData_ReturnsCreated() throws Exception {
        // Arrange
        MarketCreateDTO createDTO = new MarketCreateDTO(
            "Bitcoin $100k?",
            "Will BTC reach $100k?",
            LocalDateTime.now().plusDays(30),
            Set.of(UUID.randomUUID())
        );

        // Act & Assert
        mockMvc.perform(post("/api/v1/markets")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(createDTO)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.name").value(createDTO.name()))
            .andExpect(jsonPath("$.data.status").value("ACTIVE"));

        // Verify database
        assertTrue(marketRepository.existsByNameIgnoreCase(createDTO.name()));
    }

    @Test
    void getMarket_WhenNotFound_Returns404() throws Exception {
        // Arrange
        UUID nonExistentId = UUID.randomUUID();

        // Act & Assert
        mockMvc.perform(get("/api/v1/markets/{id}", nonExistentId))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.success").value(false))
            .andExpect(jsonPath("$.error").exists());
    }
}

@ExtendWith(MockitoExtension.class)
class MarketServiceTest {

    @Mock
    private MarketRepository marketRepository;

    @Mock
    private MarketMapper marketMapper;

    @InjectMocks
    private MarketService marketService;

    @Test
    @DisplayName("Should return market when found by ID")
    void getMarket_WhenExists_ReturnsMarket() {
        // Given
        UUID marketId = UUID.randomUUID();
        Market market = Market.builder()
            .id(marketId)
            .name("Test Market")
            .status(MarketStatus.ACTIVE)
            .build();
        MarketResponseDTO expectedResponse = new MarketResponseDTO(/* ... */);

        when(marketRepository.findById(marketId)).thenReturn(Optional.of(market));
        when(marketMapper.toResponseDTO(market)).thenReturn(expectedResponse);

        // When
        MarketResponseDTO result = marketService.getMarket(marketId);

        // Then
        assertNotNull(result);
        assertEquals(expectedResponse, result);
        verify(marketRepository).findById(marketId);
        verify(marketMapper).toResponseDTO(market);
    }
}
```

### API Integration Test Pattern
```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('returns markets successfully', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('validates query parameters', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('handles database errors gracefully', async () => {
    // Mock database failure
    const request = new NextRequest('http://localhost/api/markets')
    // Test error handling
  })
})
```

### E2E Test Pattern (Playwright)
```typescript
import { test, expect } from '@playwright/test'

test('user can search and filter markets', async ({ page }) => {
  // Navigate to markets page
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // Verify page loaded
  await expect(page.locator('h1')).toContainText('Markets')

  // Search for markets
  await page.fill('input[placeholder="Search markets"]', 'election')

  // Wait for debounce and results
  await page.waitForTimeout(600)

  // Verify search results displayed
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // Verify results contain search term
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // Filter by status
  await page.click('button:has-text("Active")')

  // Verify filtered results
  await expect(results).toHaveCount(3)
})

test('user can create a new market', async ({ page }) => {
  // Login first
  await page.goto('/creator-dashboard')

  // Fill market creation form
  await page.fill('input[name="name"]', 'Test Market')
  await page.fill('textarea[name="description"]', 'Test description')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // Submit form
  await page.click('button[type="submit"]')

  // Verify success message
  await expect(page.locator('text=Market created successfully')).toBeVisible()

  // Verify redirect to market page
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## Test Commands by Language

### JavaScript/TypeScript
```bash
# Run all tests
npm test
# or
npm run test

# Run with coverage
npm run test:coverage

# Run in watch mode
npm test -- --watch

# Run specific test file
npm test -- path/to/test.test.ts

# Run E2E tests
npm run test:e2e
# or
npx playwright test
```

### PHP/Laravel
```bash
# Run all tests (PHPUnit)
php artisan test

# Run with coverage
php artisan test --coverage

# Run specific test file
php artisan test tests/Feature/MarketTest.php

# Run specific test method
php artisan test --filter test_creates_market

# Run parallel tests
php artisan test --parallel

# Pest syntax
./vendor/bin/pest

# With coverage
./vendor/bin/pest --coverage
```

### Python/Django
```bash
# Run all tests (Django)
python manage.py test

# Run specific app tests
python manage.py test markets

# With pytest-django
pytest

# With coverage
pytest --cov=.

# With coverage report
pytest --cov=. --cov-report=html

# Run specific test
pytest markets/tests/test_models.py::test_market_creation

# Verbose mode
pytest -v

# Stop on first failure
pytest -x
```

### Flutter/Dart
```bash
# Run all tests
flutter test

# Run with coverage
flutter test --coverage

# Run specific test file
flutter test test/models/market_test.dart

# Run widget tests
flutter test test/widgets/

# Run integration tests
flutter test integration_test/

# Generate coverage report
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
```

### React Native
```bash
# Run all tests
npm test

# With coverage
npm test -- --coverage

# Run specific test
npm test -- MarketCard.test.tsx

# Watch mode
npm test -- --watch

# Update snapshots
npm test -- -u
```

### Java/Spring Boot
```bash
# Maven - Run all tests
mvn test

# Maven - Run specific test
mvn test -Dtest=MarketServiceTest

# Maven - With coverage (JaCoCo)
mvn test jacoco:report

# Maven - Skip tests
mvn install -DskipTests

# Gradle - Run all tests
gradle test

# Gradle - Run specific test
gradle test --tests MarketServiceTest

# Gradle - With coverage
gradle test jacocoTestReport
```

## Test File Organization

### JavaScript/TypeScript
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # Unit tests
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # Integration tests
└── e2e/
    ├── markets.spec.ts               # E2E tests
    ├── trading.spec.ts
    └── auth.spec.ts
```

### PHP/Laravel
```
tests/
├── Feature/
│   ├── MarketControllerTest.php
│   ├── MarketCreationTest.php
│   └── MarketValidationTest.php
├── Unit/
│   ├── MarketServiceTest.php
│   ├── MarketModelTest.php
│   └── Helpers/
│       └── MarketHelperTest.php
└── Pest.php
```

### Python/Django
```
markets/
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_views.py
│   ├── test_serializers.py
│   ├── test_services.py
│   └── conftest.py              # pytest fixtures
```

### Flutter/Dart
```
test/
├── models/
│   └── market_test.dart
├── repositories/
│   └── market_repository_test.dart
├── widgets/
│   └── market_card_test.dart
└── integration_test/
    └── app_test.dart
```

### React Native
```
src/
├── components/
│   ├── MarketCard/
│   │   ├── MarketCard.tsx
│   │   └── __tests__/
│   │       └── MarketCard.test.tsx
├── hooks/
│   ├── useMarkets.ts
│   └── __tests__/
│       └── useMarkets.test.ts
└── __tests__/
    └── integration/
        └── markets.test.tsx
```

### Java/Spring Boot
```
src/
├── main/java/com/example/markets/
└── test/java/com/example/markets/
    ├── controller/
    │   └── MarketControllerTest.java
    ├── service/
    │   └── MarketServiceTest.java
    ├── repository/
    │   └── MarketRepositoryTest.java
    └── integration/
        └── MarketIntegrationTest.java
```

## Mocking External Services

### Supabase Mock
```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: 'Test Market' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redis Mock
```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAI Mock
```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // Mock 1536-dim embedding
  ))
}))
```

## Test Coverage Verification

### Run Coverage Report
```bash
npm run test:coverage
```

### Coverage Thresholds
```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## Common Testing Mistakes to Avoid

### ❌ WRONG: Testing Implementation Details
```typescript
// Don't test internal state
expect(component.state.count).toBe(5)
```

### ✅ CORRECT: Test User-Visible Behavior
```typescript
// Test what users see
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### ❌ WRONG: Brittle Selectors
```typescript
// Breaks easily
await page.click('.css-class-xyz')
```

### ✅ CORRECT: Semantic Selectors
```typescript
// Resilient to changes
await page.click('button:has-text("Submit")')
await page.click('[data-testid="submit-button"]')
```

### ❌ WRONG: No Test Isolation
```typescript
// Tests depend on each other
test('creates user', () => { /* ... */ })
test('updates same user', () => { /* depends on previous test */ })
```

### ✅ CORRECT: Independent Tests
```typescript
// Each test sets up its own data
test('creates user', () => {
  const user = createTestUser()
  // Test logic
})

test('updates user', () => {
  const user = createTestUser()
  // Update logic
})
```

## Continuous Testing

### Watch Mode During Development
```bash
npm test -- --watch
# Tests run automatically on file changes
```

### Pre-Commit Hook
```bash
# Runs before every commit
npm test && npm run lint
```

### CI/CD Integration
```yaml
# GitHub Actions
- name: Run Tests
  run: npm test -- --coverage
- name: Upload Coverage
  uses: codecov/codecov-action@v3
```

## Best Practices

1. **Write Tests First** - Always TDD
2. **One Assert Per Test** - Focus on single behavior
3. **Descriptive Test Names** - Explain what's tested
4. **Arrange-Act-Assert** - Clear test structure
5. **Mock External Dependencies** - Isolate unit tests
6. **Test Edge Cases** - Null, undefined, empty, large
7. **Test Error Paths** - Not just happy paths
8. **Keep Tests Fast** - Unit tests < 50ms each
9. **Clean Up After Tests** - No side effects
10. **Review Coverage Reports** - Identify gaps

## Success Metrics

- 80%+ code coverage achieved
- All tests passing (green)
- No skipped or disabled tests
- Fast test execution (< 30s for unit tests)
- E2E tests cover critical user flows
- Tests catch bugs before production

---

**Remember**: Tests are not optional. They are the safety net that enables confident refactoring, rapid development, and production reliability.
