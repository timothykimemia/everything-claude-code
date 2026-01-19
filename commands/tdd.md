---
description: Enforce test-driven development workflow. Scaffold interfaces, generate tests FIRST, then implement minimal code to pass. Ensure 80%+ coverage.
---

# TDD Command

This command invokes the **tdd-guide** agent to enforce test-driven development methodology.

## What This Command Does

1. **Scaffold Interfaces** - Define types/interfaces first
2. **Generate Tests First** - Write failing tests (RED)
3. **Implement Minimal Code** - Write just enough to pass (GREEN)
4. **Refactor** - Improve code while keeping tests green (REFACTOR)
5. **Verify Coverage** - Ensure 80%+ test coverage

## When to Use

Use `/tdd` when:
- Implementing new features
- Adding new functions/components
- Fixing bugs (write test that reproduces bug first)
- Refactoring existing code
- Building critical business logic

## How It Works

The tdd-guide agent will:

1. **Define interfaces** for inputs/outputs
2. **Write tests that will FAIL** (because code doesn't exist yet)
3. **Run tests** and verify they fail for the right reason
4. **Write minimal implementation** to make tests pass
5. **Run tests** and verify they pass
6. **Refactor** code while keeping tests green
7. **Check coverage** and add more tests if below 80%

## TDD Cycle

```
RED → GREEN → REFACTOR → REPEAT

RED:      Write a failing test
GREEN:    Write minimal code to pass
REFACTOR: Improve code, keep tests passing
REPEAT:   Next feature/scenario
```

## Test Commands by Language

### JavaScript/TypeScript
```bash
npm test                    # Run all tests
npm test -- --watch         # Watch mode
npm run test:coverage       # With coverage
npx playwright test         # E2E tests
```

### PHP/Laravel
```bash
php artisan test            # Run all tests
php artisan test --filter test_name  # Specific test
./vendor/bin/pest           # Pest syntax
./vendor/bin/pest --coverage # With coverage
```

### Python/Django
```bash
python manage.py test       # Django tests
pytest                      # pytest
pytest --cov=.             # With coverage
pytest -v                   # Verbose
```

### Flutter/Dart
```bash
flutter test                # Run all tests
flutter test --coverage     # With coverage
flutter test test/widgets/  # Specific directory
```

### React Native
```bash
npm test                    # Run all tests
npm test -- --coverage      # With coverage
npm test -- --watch         # Watch mode
```

### Java/Spring Boot
```bash
mvn test                    # Maven tests
gradle test                 # Gradle tests
mvn test jacoco:report     # Maven with coverage
```

## Example Usage

### JavaScript/TypeScript Example
```
User: /tdd I need a function to calculate market liquidity score

Agent (tdd-guide):
# TDD Session: Market Liquidity Score Calculator

## Step 1: Define Interface (SCAFFOLD)

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: Implementation
  throw new Error('Not implemented')
}
```

## Step 2: Write Failing Test (RED)

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('should return high score for liquid market', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('should return low score for illiquid market', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1 day ago
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('should handle edge case: zero volume', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## Step 3: Run Tests - Verify FAIL

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ should return high score for liquid market (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // Handle zero volume edge case
  if (market.totalVolume === 0) {
    return 0
  }

  // Calculate component scores (0-100 scale)
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // Recent activity bonus
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // Weighted average
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // Clamp to 0-100
}
```

## Step 5: Run Tests - Verify PASS

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```typescript
// lib/liquidity.ts - Refactored with constants and better readability
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## Step 7: Verify Tests Still Pass

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
npm test -- --coverage lib/liquidity.test.ts

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

Coverage: 100% ✅ (Target: 80%)
```

✅ TDD session complete!
```

### PHP/Laravel Example

```
User: /tdd I need a service to validate market creation rules

Agent (tdd-guide):
# TDD Session: Market Validation Service (Laravel)

## Step 1: Define Interface (SCAFFOLD)

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\DTOs\MarketData;
use App\Exceptions\ValidationException;

interface MarketValidatorInterface
{
    /**
     * @throws ValidationException
     */
    public function validate(MarketData $data): void;
}

class MarketValidator implements MarketValidatorInterface
{
    public function validate(MarketData $data): void
    {
        throw new \RuntimeException('Not implemented');
    }
}
```

```php
<?php

namespace App\DTOs;

use Carbon\Carbon;

final readonly class MarketData
{
    public function __construct(
        public string $name,
        public float $minimumBet,
        public Carbon $endDate,
        public int $maxTraders,
    ) {}
}
```

## Step 2: Write Failing Test (RED)

```php
<?php

namespace Tests\Unit\Services;

use App\DTOs\MarketData;
use App\Services\MarketValidator;
use App\Exceptions\ValidationException;
use Carbon\Carbon;
use Tests\TestCase;

class MarketValidatorTest extends TestCase
{
    private MarketValidator $validator;

    protected function setUp(): void
    {
        parent::setUp();
        $this->validator = new MarketValidator();
    }

    /** @test */
    public function it_validates_correct_market_data(): void
    {
        $data = new MarketData(
            name: 'Will Bitcoin reach $100k?',
            minimumBet: 10.0,
            endDate: Carbon::now()->addDays(30),
            maxTraders: 100,
        );

        // Should not throw
        $this->validator->validate($data);
        $this->assertTrue(true); // Explicit assertion
    }

    /** @test */
    public function it_throws_for_past_end_date(): void
    {
        $data = new MarketData(
            name: 'Past market',
            minimumBet: 10.0,
            endDate: Carbon::now()->subDay(),
            maxTraders: 100,
        );

        $this->expectException(ValidationException::class);
        $this->expectExceptionMessage('End date must be in the future');

        $this->validator->validate($data);
    }

    /** @test */
    public function it_throws_for_negative_minimum_bet(): void
    {
        $data = new MarketData(
            name: 'Invalid bet',
            minimumBet: -5.0,
            endDate: Carbon::now()->addDays(30),
            maxTraders: 100,
        );

        $this->expectException(ValidationException::class);
        $this->expectExceptionMessage('Minimum bet must be positive');

        $this->validator->validate($data);
    }

    /** @test */
    public function it_throws_for_short_name(): void
    {
        $data = new MarketData(
            name: 'Ab',
            minimumBet: 10.0,
            endDate: Carbon::now()->addDays(30),
            maxTraders: 100,
        );

        $this->expectException(ValidationException::class);
        $this->expectExceptionMessage('Name must be at least 3 characters');

        $this->validator->validate($data);
    }
}
```

## Step 3: Run Tests - Verify FAIL

```bash
php artisan test --filter=MarketValidatorTest

FAIL  Tests\Unit\Services\MarketValidatorTest
  ✕ it validates correct market data
    RuntimeException: Not implemented

Tests:    1 failed, 3 skipped
Duration: 0.12s
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\DTOs\MarketData;
use App\Exceptions\ValidationException;
use Carbon\Carbon;

class MarketValidator implements MarketValidatorInterface
{
    public function validate(MarketData $data): void
    {
        if (mb_strlen($data->name) < 3) {
            throw new ValidationException('Name must be at least 3 characters');
        }

        if ($data->minimumBet <= 0) {
            throw new ValidationException('Minimum bet must be positive');
        }

        if ($data->endDate->isPast()) {
            throw new ValidationException('End date must be in the future');
        }

        if ($data->maxTraders < 2) {
            throw new ValidationException('Market must allow at least 2 traders');
        }
    }
}
```

## Step 5: Run Tests - Verify PASS

```bash
php artisan test --filter=MarketValidatorTest

PASS  Tests\Unit\Services\MarketValidatorTest
  ✓ it validates correct market data
  ✓ it throws for past end date
  ✓ it throws for negative minimum bet
  ✓ it throws for short name

Tests:    4 passed
Duration: 0.08s
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\DTOs\MarketData;
use App\Exceptions\ValidationException;

class MarketValidator implements MarketValidatorInterface
{
    private const MIN_NAME_LENGTH = 3;
    private const MIN_TRADERS = 2;

    public function validate(MarketData $data): void
    {
        $this->validateName($data->name);
        $this->validateMinimumBet($data->minimumBet);
        $this->validateEndDate($data->endDate);
        $this->validateMaxTraders($data->maxTraders);
    }

    private function validateName(string $name): void
    {
        if (mb_strlen($name) < self::MIN_NAME_LENGTH) {
            throw new ValidationException(
                sprintf('Name must be at least %d characters', self::MIN_NAME_LENGTH)
            );
        }
    }

    private function validateMinimumBet(float $bet): void
    {
        if ($bet <= 0) {
            throw new ValidationException('Minimum bet must be positive');
        }
    }

    private function validateEndDate(\Carbon\Carbon $endDate): void
    {
        if ($endDate->isPast()) {
            throw new ValidationException('End date must be in the future');
        }
    }

    private function validateMaxTraders(int $maxTraders): void
    {
        if ($maxTraders < self::MIN_TRADERS) {
            throw new ValidationException(
                sprintf('Market must allow at least %d traders', self::MIN_TRADERS)
            );
        }
    }
}
```

## Step 7: Verify Tests Still Pass

```bash
php artisan test --filter=MarketValidatorTest

PASS  Tests\Unit\Services\MarketValidatorTest
  ✓ it validates correct market data
  ✓ it throws for past end date
  ✓ it throws for negative minimum bet
  ✓ it throws for short name

Tests:    4 passed
Duration: 0.08s
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
./vendor/bin/pest --coverage --filter=MarketValidatorTest

PASS  Tests\Unit\Services\MarketValidatorTest
  ✓ it validates correct market data
  ✓ it throws for past end date
  ✓ it throws for negative minimum bet
  ✓ it throws for short name

Coverage:
  App\Services\MarketValidator ........................ 100.0%

Tests:    4 passed
Duration: 0.10s
```

✅ TDD session complete!
```

### Python/Django Example

```
User: /tdd I need a function to calculate market resolution score

Agent (tdd-guide):
# TDD Session: Market Resolution Score (Python/Django)

## Step 1: Define Interface (SCAFFOLD)

```python
# markets/services/resolution.py
from dataclasses import dataclass
from datetime import datetime
from typing import Protocol


@dataclass(frozen=True)
class ResolutionData:
    """Market resolution metrics."""
    total_trades: int
    winning_option_votes: int
    total_votes: int
    resolution_time: datetime
    market_end_time: datetime


class ResolutionScorer(Protocol):
    """Protocol for resolution scoring."""

    def calculate_score(self, data: ResolutionData) -> float:
        """Calculate resolution score (0-100)."""
        ...


class MarketResolutionScorer:
    """Calculate market resolution quality score."""

    def calculate_score(self, data: ResolutionData) -> float:
        """Calculate resolution score (0-100)."""
        raise NotImplementedError("Not implemented")
```

## Step 2: Write Failing Test (RED)

```python
# markets/tests/test_resolution_scorer.py
from datetime import datetime, timedelta
import pytest
from markets.services.resolution import (
    MarketResolutionScorer,
    ResolutionData,
)


@pytest.fixture
def scorer():
    return MarketResolutionScorer()


def test_high_score_for_quick_unanimous_resolution(scorer):
    """Quick resolution with high consensus should score high."""
    market_end = datetime(2024, 1, 1, 12, 0)
    resolution_time = datetime(2024, 1, 1, 12, 5)  # 5 minutes later

    data = ResolutionData(
        total_trades=500,
        winning_option_votes=95,
        total_votes=100,
        resolution_time=resolution_time,
        market_end_time=market_end,
    )

    score = scorer.calculate_score(data)

    assert score > 80
    assert score <= 100


def test_low_score_for_delayed_contested_resolution(scorer):
    """Slow resolution with low consensus should score low."""
    market_end = datetime(2024, 1, 1, 12, 0)
    resolution_time = datetime(2024, 1, 5, 12, 0)  # 4 days later

    data = ResolutionData(
        total_trades=50,
        winning_option_votes=52,
        total_votes=100,
        resolution_time=resolution_time,
        market_end_time=market_end,
    )

    score = scorer.calculate_score(data)

    assert score < 40
    assert score >= 0


def test_zero_votes_edge_case(scorer):
    """Market with no votes should return 0."""
    data = ResolutionData(
        total_trades=0,
        winning_option_votes=0,
        total_votes=0,
        resolution_time=datetime(2024, 1, 1, 12, 0),
        market_end_time=datetime(2024, 1, 1, 12, 0),
    )

    score = scorer.calculate_score(data)

    assert score == 0


def test_immediate_resolution_perfect_consensus(scorer):
    """Immediate resolution with 100% consensus."""
    time = datetime(2024, 1, 1, 12, 0)

    data = ResolutionData(
        total_trades=1000,
        winning_option_votes=100,
        total_votes=100,
        resolution_time=time,
        market_end_time=time,
    )

    score = scorer.calculate_score(data)

    assert score >= 95
```

## Step 3: Run Tests - Verify FAIL

```bash
pytest markets/tests/test_resolution_scorer.py -v

FAILED markets/tests/test_resolution_scorer.py::test_high_score_for_quick_unanimous_resolution
  NotImplementedError: Not implemented

1 failed, 3 deselected in 0.05s
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```python
# markets/services/resolution.py
from dataclasses import dataclass
from datetime import datetime


@dataclass(frozen=True)
class ResolutionData:
    """Market resolution metrics."""
    total_trades: int
    winning_option_votes: int
    total_votes: int
    resolution_time: datetime
    market_end_time: datetime


class MarketResolutionScorer:
    """Calculate market resolution quality score."""

    def calculate_score(self, data: ResolutionData) -> float:
        """Calculate resolution score (0-100)."""
        # Handle edge case: no votes
        if data.total_votes == 0:
            return 0.0

        # Calculate consensus score (0-100)
        consensus_ratio = data.winning_option_votes / data.total_votes
        consensus_score = consensus_ratio * 100

        # Calculate speed score (0-100)
        time_diff = (data.resolution_time - data.market_end_time).total_seconds()
        hours_delay = time_diff / 3600
        speed_score = max(100 - (hours_delay * 5), 0)  # -5 points per hour

        # Calculate activity score (0-100)
        activity_score = min(data.total_trades / 10, 100)

        # Weighted average
        final_score = (
            consensus_score * 0.5 +
            speed_score * 0.3 +
            activity_score * 0.2
        )

        return min(max(final_score, 0), 100)
```

## Step 5: Run Tests - Verify PASS

```bash
pytest markets/tests/test_resolution_scorer.py -v

PASSED markets/tests/test_resolution_scorer.py::test_high_score_for_quick_unanimous_resolution
PASSED markets/tests/test_resolution_scorer.py::test_low_score_for_delayed_contested_resolution
PASSED markets/tests/test_resolution_scorer.py::test_zero_votes_edge_case
PASSED markets/tests/test_resolution_scorer.py::test_immediate_resolution_perfect_consensus

4 passed in 0.03s
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```python
# markets/services/resolution.py
from dataclasses import dataclass
from datetime import datetime
from typing import Final


@dataclass(frozen=True)
class ResolutionData:
    """Market resolution metrics."""
    total_trades: int
    winning_option_votes: int
    total_votes: int
    resolution_time: datetime
    market_end_time: datetime


class MarketResolutionScorer:
    """Calculate market resolution quality score."""

    # Weights for score components
    CONSENSUS_WEIGHT: Final[float] = 0.5
    SPEED_WEIGHT: Final[float] = 0.3
    ACTIVITY_WEIGHT: Final[float] = 0.2

    # Scaling factors
    SPEED_PENALTY_PER_HOUR: Final[float] = 5.0
    ACTIVITY_SCALE_FACTOR: Final[int] = 10

    def calculate_score(self, data: ResolutionData) -> float:
        """Calculate resolution score (0-100)."""
        if data.total_votes == 0:
            return 0.0

        consensus = self._calculate_consensus_score(data)
        speed = self._calculate_speed_score(data)
        activity = self._calculate_activity_score(data)

        weighted_score = (
            consensus * self.CONSENSUS_WEIGHT +
            speed * self.SPEED_WEIGHT +
            activity * self.ACTIVITY_WEIGHT
        )

        return self._clamp(weighted_score, 0, 100)

    def _calculate_consensus_score(self, data: ResolutionData) -> float:
        """Calculate consensus score based on winning votes ratio."""
        ratio = data.winning_option_votes / data.total_votes
        return ratio * 100

    def _calculate_speed_score(self, data: ResolutionData) -> float:
        """Calculate speed score based on resolution delay."""
        time_diff = (data.resolution_time - data.market_end_time).total_seconds()
        hours_delay = time_diff / 3600
        score = 100 - (hours_delay * self.SPEED_PENALTY_PER_HOUR)
        return max(score, 0)

    def _calculate_activity_score(self, data: ResolutionData) -> float:
        """Calculate activity score based on total trades."""
        return min(data.total_trades / self.ACTIVITY_SCALE_FACTOR, 100)

    @staticmethod
    def _clamp(value: float, min_val: float, max_val: float) -> float:
        """Clamp value between min and max."""
        return min(max(value, min_val), max_val)
```

## Step 7: Verify Tests Still Pass

```bash
pytest markets/tests/test_resolution_scorer.py -v

PASSED markets/tests/test_resolution_scorer.py::test_high_score_for_quick_unanimous_resolution
PASSED markets/tests/test_resolution_scorer.py::test_low_score_for_delayed_contested_resolution
PASSED markets/tests/test_resolution_scorer.py::test_zero_votes_edge_case
PASSED markets/tests/test_resolution_scorer.py::test_immediate_resolution_perfect_consensus

4 passed in 0.03s
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
pytest markets/tests/test_resolution_scorer.py --cov=markets.services.resolution --cov-report=term

----------- coverage: platform linux, python 3.11 -----------
Name                                 Stmts   Miss  Cover
--------------------------------------------------------
markets/services/resolution.py          28      0   100%
--------------------------------------------------------
TOTAL                                   28      0   100%

4 passed in 0.08s
```

✅ TDD session complete!
```

### Flutter/Dart Example

```
User: /tdd I need a widget to display market status badge

Agent (tdd-guide):
# TDD Session: Market Status Badge Widget (Flutter)

## Step 1: Define Interface (SCAFFOLD)

```dart
// lib/widgets/market_status_badge.dart
import 'package:flutter/material.dart';

enum MarketStatus {
  open,
  closed,
  resolved,
  cancelled;
}

class MarketStatusBadge extends StatelessWidget {
  final MarketStatus status;

  const MarketStatusBadge({
    required this.status,
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    throw UnimplementedError('Not implemented');
  }
}
```

## Step 2: Write Failing Test (RED)

```dart
// test/widgets/market_status_badge_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/widgets/market_status_badge.dart';

void main() {
  group('MarketStatusBadge', () {
    testWidgets('displays green badge for open market', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: MarketStatusBadge(status: MarketStatus.open),
          ),
        ),
      );

      // Find the badge
      final badge = find.byType(Container);
      expect(badge, findsOneWidget);

      // Verify text
      expect(find.text('OPEN'), findsOneWidget);

      // Verify color (green for open)
      final container = tester.widget<Container>(badge);
      final decoration = container.decoration as BoxDecoration;
      expect(decoration.color, Colors.green);
    });

    testWidgets('displays red badge for closed market', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: MarketStatusBadge(status: MarketStatus.closed),
          ),
        ),
      );

      expect(find.text('CLOSED'), findsOneWidget);

      final container = tester.widget<Container>(find.byType(Container));
      final decoration = container.decoration as BoxDecoration;
      expect(decoration.color, Colors.red);
    });

    testWidgets('displays blue badge for resolved market', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: MarketStatusBadge(status: MarketStatus.resolved),
          ),
        ),
      );

      expect(find.text('RESOLVED'), findsOneWidget);

      final container = tester.widget<Container>(find.byType(Container));
      final decoration = container.decoration as BoxDecoration;
      expect(decoration.color, Colors.blue);
    });

    testWidgets('displays grey badge for cancelled market', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(
          home: Scaffold(
            body: MarketStatusBadge(status: MarketStatus.cancelled),
          ),
        ),
      );

      expect(find.text('CANCELLED'), findsOneWidget);

      final container = tester.widget<Container>(find.byType(Container));
      final decoration = container.decoration as BoxDecoration;
      expect(decoration.color, Colors.grey);
    });
  });
}
```

## Step 3: Run Tests - Verify FAIL

```bash
flutter test test/widgets/market_status_badge_test.dart

00:01 +0 -1: MarketStatusBadge displays green badge for open market [E]
  UnimplementedError: Not implemented

0 passed, 1 failed
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```dart
// lib/widgets/market_status_badge.dart
import 'package:flutter/material.dart';

enum MarketStatus {
  open,
  closed,
  resolved,
  cancelled;
}

class MarketStatusBadge extends StatelessWidget {
  final MarketStatus status;

  const MarketStatusBadge({
    required this.status,
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    final (color, label) = switch (status) {
      MarketStatus.open => (Colors.green, 'OPEN'),
      MarketStatus.closed => (Colors.red, 'CLOSED'),
      MarketStatus.resolved => (Colors.blue, 'RESOLVED'),
      MarketStatus.cancelled => (Colors.grey, 'CANCELLED'),
    };

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.circular(16),
      ),
      child: Text(
        label,
        style: const TextStyle(
          color: Colors.white,
          fontSize: 12,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}
```

## Step 5: Run Tests - Verify PASS

```bash
flutter test test/widgets/market_status_badge_test.dart

00:02 +4: All tests passed!

4 passed, 0 failed
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```dart
// lib/widgets/market_status_badge.dart
import 'package:flutter/material.dart';

enum MarketStatus {
  open,
  closed,
  resolved,
  cancelled;
}

class MarketStatusBadge extends StatelessWidget {
  final MarketStatus status;

  const MarketStatusBadge({
    required this.status,
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    final config = _StatusConfig.fromStatus(status);

    return Container(
      padding: const EdgeInsets.symmetric(
        horizontal: 12,
        vertical: 6,
      ),
      decoration: BoxDecoration(
        color: config.color,
        borderRadius: BorderRadius.circular(16),
      ),
      child: Text(
        config.label,
        style: const TextStyle(
          color: Colors.white,
          fontSize: 12,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}

/// Configuration for status badge appearance.
class _StatusConfig {
  final Color color;
  final String label;

  const _StatusConfig({
    required this.color,
    required this.label,
  });

  factory _StatusConfig.fromStatus(MarketStatus status) {
    return switch (status) {
      MarketStatus.open => const _StatusConfig(
        color: Colors.green,
        label: 'OPEN',
      ),
      MarketStatus.closed => const _StatusConfig(
        color: Colors.red,
        label: 'CLOSED',
      ),
      MarketStatus.resolved => const _StatusConfig(
        color: Colors.blue,
        label: 'RESOLVED',
      ),
      MarketStatus.cancelled => const _StatusConfig(
        color: Colors.grey,
        label: 'CANCELLED',
      ),
    };
  }
}
```

## Step 7: Verify Tests Still Pass

```bash
flutter test test/widgets/market_status_badge_test.dart

00:02 +4: All tests passed!

4 passed, 0 failed
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
flutter test --coverage test/widgets/market_status_badge_test.dart
genhtml coverage/lcov.info -o coverage/html

Reading data file coverage/lcov.info
Found 1 entries.
Writing directory view page.
Overall coverage rate:
  lines......: 100.0% (15 of 15 lines)
  functions..: 100.0% (4 of 4 functions)
```

✅ TDD session complete!
```

### React Native Example

```
User: /tdd I need a hook to manage market countdown timer

Agent (tdd-guide):
# TDD Session: Market Countdown Hook (React Native)

## Step 1: Define Interface (SCAFFOLD)

```typescript
// src/hooks/useMarketCountdown.ts
import { useEffect, useState } from 'react';

export interface CountdownResult {
  days: number;
  hours: number;
  minutes: number;
  seconds: number;
  isExpired: boolean;
  timeRemaining: number; // milliseconds
}

export function useMarketCountdown(endDate: Date): CountdownResult {
  throw new Error('Not implemented');
}
```

## Step 2: Write Failing Test (RED)

```typescript
// src/hooks/__tests__/useMarketCountdown.test.ts
import { renderHook, act } from '@testing-library/react-hooks';
import { useMarketCountdown } from '../useMarketCountdown';

jest.useFakeTimers();

describe('useMarketCountdown', () => {
  beforeEach(() => {
    jest.clearAllTimers();
  });

  afterEach(() => {
    jest.runOnlyPendingTimers();
  });

  it('should calculate correct countdown for future date', () => {
    const now = new Date('2024-01-01T00:00:00Z');
    const endDate = new Date('2024-01-02T12:30:00Z'); // 1 day, 12h 30m
    jest.setSystemTime(now);

    const { result } = renderHook(() => useMarketCountdown(endDate));

    expect(result.current.days).toBe(1);
    expect(result.current.hours).toBe(12);
    expect(result.current.minutes).toBe(30);
    expect(result.current.seconds).toBe(0);
    expect(result.current.isExpired).toBe(false);
  });

  it('should mark as expired for past date', () => {
    const now = new Date('2024-01-02T00:00:00Z');
    const endDate = new Date('2024-01-01T00:00:00Z'); // Yesterday
    jest.setSystemTime(now);

    const { result } = renderHook(() => useMarketCountdown(endDate));

    expect(result.current.days).toBe(0);
    expect(result.current.hours).toBe(0);
    expect(result.current.minutes).toBe(0);
    expect(result.current.seconds).toBe(0);
    expect(result.current.isExpired).toBe(true);
  });

  it('should update countdown every second', () => {
    const now = new Date('2024-01-01T00:00:00Z');
    const endDate = new Date('2024-01-01T00:01:00Z'); // 1 minute
    jest.setSystemTime(now);

    const { result } = renderHook(() => useMarketCountdown(endDate));

    expect(result.current.seconds).toBe(0);
    expect(result.current.minutes).toBe(1);

    // Advance 30 seconds
    act(() => {
      jest.advanceTimersByTime(30000);
    });

    expect(result.current.seconds).toBe(30);
    expect(result.current.minutes).toBe(0);
  });

  it('should handle transition to expired state', () => {
    const now = new Date('2024-01-01T00:00:00Z');
    const endDate = new Date('2024-01-01T00:00:05Z'); // 5 seconds
    jest.setSystemTime(now);

    const { result } = renderHook(() => useMarketCountdown(endDate));

    expect(result.current.isExpired).toBe(false);

    // Advance past expiration
    act(() => {
      jest.advanceTimersByTime(6000);
    });

    expect(result.current.isExpired).toBe(true);
    expect(result.current.seconds).toBe(0);
  });
});
```

## Step 3: Run Tests - Verify FAIL

```bash
npm test -- useMarketCountdown.test.ts

FAIL src/hooks/__tests__/useMarketCountdown.test.ts
  ✕ should calculate correct countdown for future date (5 ms)
    Error: Not implemented

Tests:    1 failed, 3 skipped
Time:     0.5s
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```typescript
// src/hooks/useMarketCountdown.ts
import { useEffect, useState } from 'react';

export interface CountdownResult {
  days: number;
  hours: number;
  minutes: number;
  seconds: number;
  isExpired: boolean;
  timeRemaining: number;
}

export function useMarketCountdown(endDate: Date): CountdownResult {
  const calculateTimeLeft = (): CountdownResult => {
    const now = Date.now();
    const end = endDate.getTime();
    const diff = end - now;

    if (diff <= 0) {
      return {
        days: 0,
        hours: 0,
        minutes: 0,
        seconds: 0,
        isExpired: true,
        timeRemaining: 0,
      };
    }

    const days = Math.floor(diff / (1000 * 60 * 60 * 24));
    const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
    const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
    const seconds = Math.floor((diff % (1000 * 60)) / 1000);

    return {
      days,
      hours,
      minutes,
      seconds,
      isExpired: false,
      timeRemaining: diff,
    };
  };

  const [countdown, setCountdown] = useState<CountdownResult>(calculateTimeLeft);

  useEffect(() => {
    const timer = setInterval(() => {
      setCountdown(calculateTimeLeft());
    }, 1000);

    return () => clearInterval(timer);
  }, [endDate]);

  return countdown;
}
```

## Step 5: Run Tests - Verify PASS

```bash
npm test -- useMarketCountdown.test.ts

PASS src/hooks/__tests__/useMarketCountdown.test.ts
  ✓ should calculate correct countdown for future date (12 ms)
  ✓ should mark as expired for past date (8 ms)
  ✓ should update countdown every second (15 ms)
  ✓ should handle transition to expired state (10 ms)

Tests:    4 passed
Time:     0.6s
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```typescript
// src/hooks/useMarketCountdown.ts
import { useEffect, useState, useCallback, useMemo } from 'react';

export interface CountdownResult {
  days: number;
  hours: number;
  minutes: number;
  seconds: number;
  isExpired: boolean;
  timeRemaining: number;
}

const MS_PER_SECOND = 1000;
const MS_PER_MINUTE = MS_PER_SECOND * 60;
const MS_PER_HOUR = MS_PER_MINUTE * 60;
const MS_PER_DAY = MS_PER_HOUR * 24;

export function useMarketCountdown(endDate: Date): CountdownResult {
  const endTime = useMemo(() => endDate.getTime(), [endDate]);

  const calculateTimeLeft = useCallback((): CountdownResult => {
    const now = Date.now();
    const diff = endTime - now;

    if (diff <= 0) {
      return {
        days: 0,
        hours: 0,
        minutes: 0,
        seconds: 0,
        isExpired: true,
        timeRemaining: 0,
      };
    }

    return {
      days: Math.floor(diff / MS_PER_DAY),
      hours: Math.floor((diff % MS_PER_DAY) / MS_PER_HOUR),
      minutes: Math.floor((diff % MS_PER_HOUR) / MS_PER_MINUTE),
      seconds: Math.floor((diff % MS_PER_MINUTE) / MS_PER_SECOND),
      isExpired: false,
      timeRemaining: diff,
    };
  }, [endTime]);

  const [countdown, setCountdown] = useState<CountdownResult>(calculateTimeLeft);

  useEffect(() => {
    // Update immediately
    setCountdown(calculateTimeLeft());

    // Set up interval
    const timer = setInterval(() => {
      const newCountdown = calculateTimeLeft();
      setCountdown(newCountdown);

      // Clear interval if expired
      if (newCountdown.isExpired) {
        clearInterval(timer);
      }
    }, MS_PER_SECOND);

    return () => clearInterval(timer);
  }, [calculateTimeLeft]);

  return countdown;
}
```

## Step 7: Verify Tests Still Pass

```bash
npm test -- useMarketCountdown.test.ts

PASS src/hooks/__tests__/useMarketCountdown.test.ts
  ✓ should calculate correct countdown for future date (12 ms)
  ✓ should mark as expired for past date (8 ms)
  ✓ should update countdown every second (15 ms)
  ✓ should handle transition to expired state (10 ms)

Tests:    4 passed
Time:     0.6s
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
npm test -- useMarketCountdown.test.ts --coverage

----------------------|---------|----------|---------|---------|
File                  | % Stmts | % Branch | % Funcs | % Lines |
----------------------|---------|----------|---------|---------|
useMarketCountdown.ts |     100 |      100 |     100 |     100 |
----------------------|---------|----------|---------|---------|

Coverage: 100% ✅ (Target: 80%)
```

✅ TDD session complete!
```

### Java/Spring Boot Example

```
User: /tdd I need a service to calculate market fee

Agent (tdd-guide):
# TDD Session: Market Fee Calculator (Java/Spring Boot)

## Step 1: Define Interface (SCAFFOLD)

```java
// src/main/java/com/example/markets/service/FeeCalculator.java
package com.example.markets.service;

import java.math.BigDecimal;

public interface FeeCalculator {
    /**
     * Calculate platform fee for a market transaction.
     *
     * @param transactionAmount the transaction amount
     * @param marketType the type of market
     * @return the calculated fee
     */
    BigDecimal calculateFee(BigDecimal transactionAmount, MarketType marketType);
}
```

```java
// src/main/java/com/example/markets/service/MarketType.java
package com.example.markets.service;

public enum MarketType {
    STANDARD,
    PREMIUM,
    SPONSORED
}
```

```java
// src/main/java/com/example/markets/service/MarketFeeCalculator.java
package com.example.markets.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;

@Service
public class MarketFeeCalculator implements FeeCalculator {

    @Override
    public BigDecimal calculateFee(BigDecimal transactionAmount, MarketType marketType) {
        throw new UnsupportedOperationException("Not implemented");
    }
}
```

## Step 2: Write Failing Test (RED)

```java
// src/test/java/com/example/markets/service/MarketFeeCalculatorTest.java
package com.example.markets.service;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import java.math.BigDecimal;
import static org.assertj.core.api.Assertions.assertThat;

@DisplayName("Market Fee Calculator")
class MarketFeeCalculatorTest {

    private MarketFeeCalculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new MarketFeeCalculator();
    }

    @Test
    @DisplayName("should calculate 5% fee for standard markets")
    void shouldCalculateStandardMarketFee() {
        // Given
        BigDecimal amount = new BigDecimal("100.00");

        // When
        BigDecimal fee = calculator.calculateFee(amount, MarketType.STANDARD);

        // Then
        assertThat(fee).isEqualByComparingTo(new BigDecimal("5.00"));
    }

    @Test
    @DisplayName("should calculate 2% fee for premium markets")
    void shouldCalculatePremiumMarketFee() {
        // Given
        BigDecimal amount = new BigDecimal("100.00");

        // When
        BigDecimal fee = calculator.calculateFee(amount, MarketType.PREMIUM);

        // Then
        assertThat(fee).isEqualByComparingTo(new BigDecimal("2.00"));
    }

    @Test
    @DisplayName("should calculate zero fee for sponsored markets")
    void shouldCalculateSponsoredMarketFee() {
        // Given
        BigDecimal amount = new BigDecimal("100.00");

        // When
        BigDecimal fee = calculator.calculateFee(amount, MarketType.SPONSORED);

        // Then
        assertThat(fee).isEqualByComparingTo(BigDecimal.ZERO);
    }

    @Test
    @DisplayName("should handle small amounts with proper rounding")
    void shouldHandleSmallAmounts() {
        // Given
        BigDecimal amount = new BigDecimal("0.99");

        // When
        BigDecimal fee = calculator.calculateFee(amount, MarketType.STANDARD);

        // Then
        assertThat(fee).isEqualByComparingTo(new BigDecimal("0.05"));
    }

    @Test
    @DisplayName("should handle zero amount")
    void shouldHandleZeroAmount() {
        // Given
        BigDecimal amount = BigDecimal.ZERO;

        // When
        BigDecimal fee = calculator.calculateFee(amount, MarketType.STANDARD);

        // Then
        assertThat(fee).isEqualByComparingTo(BigDecimal.ZERO);
    }
}
```

## Step 3: Run Tests - Verify FAIL

```bash
mvn test -Dtest=MarketFeeCalculatorTest

[ERROR] Failures:
[ERROR]   MarketFeeCalculatorTest.shouldCalculateStandardMarketFee
    UnsupportedOperationException: Not implemented

Tests run: 1, Failures: 1, Errors: 0, Skipped: 4
```

✅ Tests fail as expected. Ready to implement.

## Step 4: Implement Minimal Code (GREEN)

```java
// src/main/java/com/example/markets/service/MarketFeeCalculator.java
package com.example.markets.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.math.RoundingMode;

@Service
public class MarketFeeCalculator implements FeeCalculator {

    private static final BigDecimal STANDARD_FEE_RATE = new BigDecimal("0.05"); // 5%
    private static final BigDecimal PREMIUM_FEE_RATE = new BigDecimal("0.02");  // 2%
    private static final int SCALE = 2;

    @Override
    public BigDecimal calculateFee(BigDecimal transactionAmount, MarketType marketType) {
        if (transactionAmount.compareTo(BigDecimal.ZERO) == 0) {
            return BigDecimal.ZERO;
        }

        BigDecimal rate = switch (marketType) {
            case STANDARD -> STANDARD_FEE_RATE;
            case PREMIUM -> PREMIUM_FEE_RATE;
            case SPONSORED -> BigDecimal.ZERO;
        };

        return transactionAmount
                .multiply(rate)
                .setScale(SCALE, RoundingMode.HALF_UP);
    }
}
```

## Step 5: Run Tests - Verify PASS

```bash
mvn test -Dtest=MarketFeeCalculatorTest

[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.markets.service.MarketFeeCalculatorTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0

[INFO] BUILD SUCCESS
```

✅ All tests passing!

## Step 6: Refactor (IMPROVE)

```java
// src/main/java/com/example/markets/service/MarketFeeCalculator.java
package com.example.markets.service;

import org.springframework.stereotype.Service;
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.Map;

@Service
public class MarketFeeCalculator implements FeeCalculator {

    private static final int DECIMAL_SCALE = 2;
    private static final RoundingMode ROUNDING_MODE = RoundingMode.HALF_UP;

    private static final Map<MarketType, BigDecimal> FEE_RATES = Map.of(
        MarketType.STANDARD, new BigDecimal("0.05"),  // 5%
        MarketType.PREMIUM, new BigDecimal("0.02"),   // 2%
        MarketType.SPONSORED, BigDecimal.ZERO         // 0%
    );

    @Override
    public BigDecimal calculateFee(
            BigDecimal transactionAmount,
            MarketType marketType) {

        validateInputs(transactionAmount, marketType);

        if (isZeroAmount(transactionAmount)) {
            return BigDecimal.ZERO;
        }

        BigDecimal feeRate = getFeeRate(marketType);
        return calculateFeeAmount(transactionAmount, feeRate);
    }

    private void validateInputs(BigDecimal amount, MarketType marketType) {
        if (amount == null) {
            throw new IllegalArgumentException("Transaction amount cannot be null");
        }
        if (marketType == null) {
            throw new IllegalArgumentException("Market type cannot be null");
        }
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Transaction amount cannot be negative");
        }
    }

    private boolean isZeroAmount(BigDecimal amount) {
        return amount.compareTo(BigDecimal.ZERO) == 0;
    }

    private BigDecimal getFeeRate(MarketType marketType) {
        return FEE_RATES.get(marketType);
    }

    private BigDecimal calculateFeeAmount(BigDecimal amount, BigDecimal rate) {
        return amount
                .multiply(rate)
                .setScale(DECIMAL_SCALE, ROUNDING_MODE);
    }
}
```

## Step 7: Verify Tests Still Pass

```bash
mvn test -Dtest=MarketFeeCalculatorTest

[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.markets.service.MarketFeeCalculatorTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0

[INFO] BUILD SUCCESS
```

✅ Refactoring complete, tests still passing!

## Step 8: Check Coverage

```bash
mvn test jacoco:report

[INFO] --- jacoco-maven-plugin:0.8.10:report (default-cli) @ markets ---
[INFO] Loading execution data file target/jacoco.exec
[INFO] Analyzed bundle 'markets' with 1 classes

Coverage Summary:
  Class                          Line Coverage    Branch Coverage
  MarketFeeCalculator            100% (28/28)     100% (8/8)

[INFO] BUILD SUCCESS
```

✅ TDD session complete!
```

## TDD Best Practices

**DO:**
- ✅ Write the test FIRST, before any implementation
- ✅ Run tests and verify they FAIL before implementing
- ✅ Write minimal code to make tests pass
- ✅ Refactor only after tests are green
- ✅ Add edge cases and error scenarios
- ✅ Aim for 80%+ coverage (100% for critical code)

**DON'T:**
- ❌ Write implementation before tests
- ❌ Skip running tests after each change
- ❌ Write too much code at once
- ❌ Ignore failing tests
- ❌ Test implementation details (test behavior)
- ❌ Mock everything (prefer integration tests)

## Test Types to Include

**Unit Tests** (Function-level):
- Happy path scenarios
- Edge cases (empty, null, max values)
- Error conditions
- Boundary values

**Integration Tests** (Component-level):
- API endpoints
- Database operations
- External service calls
- React components with hooks

**E2E Tests** (use `/e2e` command):
- Critical user flows
- Multi-step processes
- Full stack integration

## Coverage Requirements

- **80% minimum** for all code
- **100% required** for:
  - Financial calculations
  - Authentication logic
  - Security-critical code
  - Core business logic

## Important Notes

**MANDATORY**: Tests must be written BEFORE implementation. The TDD cycle is:

1. **RED** - Write failing test
2. **GREEN** - Implement to pass
3. **REFACTOR** - Improve code

Never skip the RED phase. Never write code before tests.

## Integration with Other Commands

- Use `/plan` first to understand what to build
- Use `/tdd` to implement with tests
- Use `/build-and-fix` if build errors occur
- Use `/code-review` to review implementation
- Use `/test-coverage` to verify coverage

## Related Agents

This command invokes the `tdd-guide` agent located at:
`~/.claude/agents/tdd-guide.md`

And can reference the `tdd-workflow` skill at:
`~/.claude/skills/tdd-workflow/`
