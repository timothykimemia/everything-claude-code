# Coding Style

Universal coding style standards and language-specific quick references. For comprehensive language-specific patterns, see `skills/*-standards.md` files.

---

## Universal Principles (ALL Languages)

### 1. Immutability (CRITICAL)

**ALWAYS create new objects, NEVER mutate existing ones:**

```javascript
// ❌ WRONG: Mutation
function updateUser(user, name) {
  user.name = name;  // MUTATION!
  return user;
}

// ✅ CORRECT: Immutability
function updateUser(user, name) {
  return { ...user, name };
}
```

### 2. File Organization

**MANY SMALL FILES > FEW LARGE FILES:**
- High cohesion, low coupling
- 200-400 lines typical
- **800 lines maximum**
- Extract utilities from large components
- Organize by feature/domain, not by type

### 3. Error Handling

**ALWAYS handle errors comprehensively:**

```typescript
try {
  const result = await riskyOperation();
  return result;
} catch (error) {
  logger.error('Operation failed', { error });
  throw new Error('User-friendly message');
}
```

### 4. Code Quality Checklist

Before marking work complete:
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No debug statements (console.log, var_dump, print, etc.)
- [ ] No hardcoded secrets or config values
- [ ] Immutable patterns used where applicable
- [ ] Tests written and passing
- [ ] Documentation added for public APIs

---

## Language-Specific Style Guides

### JavaScript/TypeScript

**Key Principles:**
- Use TypeScript with strict mode
- Prefer `const` over `let`, never use `var`
- Use arrow functions for callbacks
- Immutable updates with spread operator
- Proper async/await (no floating promises)

**Type Safety:**
```typescript
// ✅ GOOD: Proper typing
interface User {
  id: string;
  name: string;
  email: string;
}

function getUser(id: string): Promise<User> {
  return fetchUser(id);
}

// ❌ BAD: Using 'any'
function getUser(id: any): any {
  return fetchUser(id);
}
```

**Naming:**
- Variables/functions: `camelCase`
- Classes/interfaces: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Files: `kebab-case.ts` or `PascalCase.tsx` (components)

**Resources:**
- See `skills/typescript-standards.md` (if using TypeScript)
- See `skills/react-native-standards.md` (if React Native)

---

### PHP/Laravel

**Key Principles:**
- Use `declare(strict_types=1)` in ALL files
- Type hint everything (parameters, return types, properties)
- Use readonly properties (PHP 8.1+)
- Follow PSR-12 coding standard
- Use DTOs for complex data structures

**Type Safety:**
```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\DTOs\MarketData;

final readonly class MarketService
{
    public function calculateScore(MarketData $market): float
    {
        return $market->volume * 0.5 + $market->trades * 0.5;
    }
}
```

**Naming:**
- Variables: `$camelCase`
- Classes: `PascalCase`
- Methods: `camelCase()`
- Constants: `UPPER_SNAKE_CASE`
- Files: `PascalCase.php`

**Resources:**
- See `skills/php-laravel-standards.md` for comprehensive patterns

---

### Python/Django

**Key Principles:**
- Follow PEP 8 style guide
- Use type hints for all function signatures
- Use dataclasses for data structures
- 4 spaces for indentation (never tabs)
- Max line length: 88 characters (Black formatter)

**Type Safety:**
```python
from dataclasses import dataclass
from typing import Protocol

@dataclass(frozen=True)
class Market:
    """Market data structure."""
    id: int
    name: str
    volume: float

def calculate_score(market: Market) -> float:
    """Calculate market quality score."""
    return market.volume * 0.5
```

**Naming:**
- Variables/functions: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Files: `snake_case.py`
- Private: `_leading_underscore`

**Resources:**
- See `skills/python-standards.md` for comprehensive patterns

---

### Flutter/Dart

**Key Principles:**
- Enable null safety
- Use `const` constructors where possible
- Follow Effective Dart guidelines
- Prefer composition over inheritance
- Extract widgets to keep files small

**Null Safety:**
```dart
// ✅ GOOD: Null safety
class Market {
  final String id;
  final String name;
  final double? volume;  // Nullable

  const Market({
    required this.id,
    required this.name,
    this.volume,
  });
}

// Handle nullable values
final volume = market.volume ?? 0.0;
```

**Naming:**
- Variables/functions: `camelCase`
- Classes: `PascalCase`
- Constants: `lowerCamelCase`
- Files: `snake_case.dart`
- Private: `_leadingUnderscore`

**Resources:**
- See `skills/flutter-dart-standards.md` for comprehensive patterns

---

### React Native

**Key Principles:**
- Use TypeScript with strict mode
- Use functional components with hooks
- Avoid inline styles (use StyleSheet)
- Proper TypeScript navigation types
- Clean up effects (remove listeners)

**Component Structure:**
```typescript
import React, { useEffect, useState } from 'react';
import { View, Text, StyleSheet } from 'react-native';

interface Props {
  market: Market;
  onPress: (id: string) => void;
}

export const MarketCard = ({ market, onPress }: Props) => {
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    // Setup
    const subscription = subscribeToMarket(market.id);

    // Cleanup
    return () => subscription.unsubscribe();
  }, [market.id]);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>{market.name}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { padding: 16 },
  title: { fontSize: 18, fontWeight: 'bold' },
});
```

**Naming:**
- Components: `PascalCase`
- Variables/functions: `camelCase`
- Files: `PascalCase.tsx` (components)
- Hooks: `use` prefix (`useMarketData`)

**Resources:**
- See `skills/react-native-standards.md` for comprehensive patterns

---

### Java/Spring Boot

**Key Principles:**
- Use Java 17+ features (records, sealed classes)
- Immutable by default (`final` fields)
- Constructor injection over field injection
- Use Optional for nullable returns
- Follow Google Java Style Guide

**Modern Java:**
```java
// ✅ GOOD: Java 17 record (immutable)
public record MarketDTO(
    String id,
    String name,
    BigDecimal volume
) {
    // Compact constructor for validation
    public MarketDTO {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
    }
}

// ✅ GOOD: Constructor injection
@RestController
public class MarketController {
    private final MarketService marketService;

    @Autowired
    public MarketController(MarketService marketService) {
        this.marketService = marketService;
    }

    @GetMapping("/markets/{id}")
    public ResponseEntity<MarketDTO> getMarket(@PathVariable Long id) {
        return marketService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
}
```

**Naming:**
- Variables: `camelCase`
- Classes/interfaces: `PascalCase`
- Methods: `camelCase()`
- Constants: `UPPER_SNAKE_CASE`
- Files: `PascalCase.java`

**Resources:**
- See `skills/java-springboot-standards.md` for comprehensive patterns

---

## Common Code Smells (ALL Languages)

### ❌ Avoid These Patterns:

1. **Long Functions** - Functions > 50 lines
2. **Deep Nesting** - Nesting > 4 levels
3. **Magic Numbers** - Unexplained hardcoded values
4. **Debug Statements** - console.log, var_dump, print(), etc.
5. **God Classes** - Classes that do too much
6. **Code Duplication** - Copy-paste programming
7. **Premature Optimization** - Optimize only when needed
8. **Missing Error Handling** - Bare try/catch without logging
9. **Mutable State** - Direct mutations instead of immutable updates
10. **Poor Naming** - `x`, `data`, `temp`, `foo`

---

## Formatting Standards

### Indentation
- **JavaScript/TypeScript**: 2 spaces
- **PHP**: 4 spaces
- **Python**: 4 spaces (PEP 8)
- **Dart**: 2 spaces
- **Java**: 4 spaces (or 2, be consistent)

### Line Length
- **JavaScript/TypeScript**: 100 characters
- **PHP**: 120 characters
- **Python**: 88 characters (Black formatter)
- **Dart**: 80 characters
- **Java**: 100 characters

### Auto-Formatters (Use These!)
- **JavaScript/TypeScript**: Prettier
- **PHP**: php-cs-fixer, Laravel Pint
- **Python**: Black, isort
- **Dart**: `dart format`
- **Java**: google-java-format

---

## Input Validation by Language

### JavaScript/TypeScript (Zod)
```typescript
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150),
});

const validated = schema.parse(input);
```

### PHP/Laravel (Form Requests)
```php
class CreateMarketRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'min:3', 'max:200'],
            'end_date' => ['required', 'date', 'after:now'],
        ];
    }
}
```

### Python/Django (Serializers)
```python
from rest_framework import serializers

class MarketSerializer(serializers.Serializer):
    name = serializers.CharField(max_length=200, min_length=3)
    end_date = serializers.DateTimeField()

    def validate_end_date(self, value):
        if value < timezone.now():
            raise serializers.ValidationError("End date must be in future")
        return value
```

### Flutter/Dart (Form Validators)
```dart
TextFormField(
  validator: (value) {
    if (value == null || value.isEmpty) {
      return 'Please enter a value';
    }
    if (value.length < 3) {
      return 'Must be at least 3 characters';
    }
    return null;
  },
)
```

### Java/Spring Boot (Bean Validation)
```java
import jakarta.validation.constraints.*;

public record MarketRequest(
    @NotBlank(message = "Name cannot be blank")
    @Size(min = 3, max = 200)
    String name,

    @NotNull
    @Future(message = "End date must be in future")
    LocalDateTime endDate
) {}
```

---

## Documentation Standards

### Function/Method Documentation

**JavaScript/TypeScript (JSDoc):**
```typescript
/**
 * Calculates the market liquidity score.
 *
 * @param market - The market data
 * @returns Score between 0-100
 * @throws {ValidationError} If market data is invalid
 */
function calculateScore(market: Market): number {
  // ...
}
```

**PHP (PHPDoc):**
```php
/**
 * Calculate market liquidity score.
 *
 * @param MarketData $market The market data
 * @return float Score between 0-100
 * @throws ValidationException If market data is invalid
 */
public function calculateScore(MarketData $market): float
{
    // ...
}
```

**Python (Docstring):**
```python
def calculate_score(market: Market) -> float:
    """Calculate market liquidity score.

    Args:
        market: The market data

    Returns:
        Score between 0-100

    Raises:
        ValidationError: If market data is invalid
    """
    pass
```

**Dart (DartDoc):**
```dart
/// Calculates the market liquidity score.
///
/// Returns a score between 0-100.
///
/// Throws [ArgumentError] if [market] is invalid.
double calculateScore(Market market) {
  // ...
}
```

**Java (Javadoc):**
```java
/**
 * Calculates the market liquidity score.
 *
 * @param market the market data
 * @return score between 0-100
 * @throws IllegalArgumentException if market data is invalid
 */
public double calculateScore(Market market) {
    // ...
}
```

---

## Resources

For comprehensive language-specific coding standards:

- **JavaScript/TypeScript**: ESLint, Prettier, TypeScript strict mode
- **PHP/Laravel**: `skills/php-laravel-standards.md`, PSR-12, Laravel Pint
- **Python/Django**: `skills/python-standards.md`, PEP 8, Black
- **Flutter/Dart**: `skills/flutter-dart-standards.md`, Effective Dart
- **React Native**: `skills/react-native-standards.md`, ESLint + Prettier
- **Java/Spring Boot**: `skills/java-springboot-standards.md`, Google Java Style

---

**Remember**: Consistent style improves readability, reduces bugs, and makes collaboration easier. Use auto-formatters, follow language conventions, and prioritize clarity over cleverness.
