---
name: php-laravel-standards
description: PHP and Laravel coding standards, best practices, and patterns for building robust, secure Laravel applications.
---

# PHP & Laravel Coding Standards

Comprehensive coding standards for PHP 8.x and Laravel 10.x+ development.

## PHP Language Standards

### Type Declarations (PHP 8+)

```php
// ✅ GOOD: Strict types and proper type declarations
declare(strict_types=1);

class MarketService
{
    public function calculateLiquidity(
        float $volume,
        int $traderCount,
        ?DateTime $lastTrade = null
    ): float {
        // Implementation
    }

    public function getMarkets(array $filters): Collection
    {
        // Implementation
    }
}

// ❌ BAD: No type declarations
class MarketService
{
    public function calculateLiquidity($volume, $traderCount, $lastTrade)
    {
        // Implementation
    }
}
```

### Property Type Declarations

```php
// ✅ GOOD: Typed properties with visibility
class Market
{
    private string $id;
    private string $name;
    private MarketStatus $status;
    private ?float $resolvedValue = null;
    private Carbon $createdAt;

    public function __construct(
        string $id,
        string $name,
        MarketStatus $status,
        Carbon $createdAt
    ) {
        $this->id = $id;
        $this->name = $name;
        $this->status = $status;
        $this->createdAt = $createdAt;
    }
}

// ❌ BAD: No type hints
class Market
{
    public $id;
    public $name;
}
```

### Enums (PHP 8.1+)

```php
// ✅ GOOD: Use enums for fixed sets of values
enum MarketStatus: string
{
    case ACTIVE = 'active';
    case RESOLVED = 'resolved';
    case CANCELLED = 'cancelled';

    public function isResolved(): bool
    {
        return $this === self::RESOLVED;
    }

    public function label(): string
    {
        return match($this) {
            self::ACTIVE => 'Active',
            self::RESOLVED => 'Resolved',
            self::CANCELLED => 'Cancelled',
        };
    }
}

// Usage
$market->status = MarketStatus::ACTIVE;

// ❌ BAD: Magic strings
$market->status = 'active';
```

### Null Coalescing and Safe Navigation

```php
// ✅ GOOD: Use null coalescing and null safe operator
$userName = $user?->profile?->name ?? 'Guest';
$count = $request->input('count') ?? 10;

// ❌ BAD: Nested isset checks
$userName = 'Guest';
if (isset($user)) {
    if (isset($user->profile)) {
        if (isset($user->profile->name)) {
            $userName = $user->profile->name;
        }
    }
}
```

## Laravel Best Practices

### Eloquent Models

```php
// ✅ GOOD: Comprehensive model definition
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Carbon\Carbon;

class Market extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'name',
        'description',
        'status',
        'end_date',
        'creator_id',
    ];

    protected $casts = [
        'status' => MarketStatus::class,
        'end_date' => 'datetime',
        'metadata' => 'array',
        'is_featured' => 'boolean',
    ];

    protected $hidden = [
        'creator_ip',
        'internal_notes',
    ];

    // Relationships
    public function creator(): BelongsTo
    {
        return $this->belongsTo(User::class, 'creator_id');
    }

    public function trades(): HasMany
    {
        return $this->hasMany(Trade::class);
    }

    // Scopes
    public function scopeActive($query)
    {
        return $query->where('status', MarketStatus::ACTIVE);
    }

    // Accessors
    public function getIsActiveAttribute(): bool
    {
        return $this->status === MarketStatus::ACTIVE;
    }

    // Mutators
    public function setNameAttribute(string $value): void
    {
        $this->attributes['name'] = ucfirst(trim($value));
    }
}
```

### Service Classes

```php
// ✅ GOOD: Service class with dependency injection
namespace App\Services;

use App\Models\Market;
use App\Repositories\MarketRepository;
use App\Events\MarketCreated;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Cache;

class MarketService
{
    public function __construct(
        private MarketRepository $repository,
        private NotificationService $notificationService
    ) {}

    public function createMarket(array $data): Market
    {
        return DB::transaction(function () use ($data) {
            $market = $this->repository->create($data);

            event(new MarketCreated($market));

            $this->notificationService->notifyFollowers($market);

            Cache::tags(['markets'])->flush();

            return $market;
        });
    }

    public function calculateLiquidityScore(Market $market): float
    {
        $volume = $market->trades()->sum('volume');
        $traderCount = $market->trades()->distinct('user_id')->count();
        $hoursSinceLastTrade = $market->trades()->latest()->first()?->created_at?->diffInHours(now()) ?? 24;

        $volumeScore = min($volume / 1000, 100);
        $traderScore = min($traderCount / 10, 100);
        $recencyScore = max(100 - ($hoursSinceLastTrade * 10), 0);

        return ($volumeScore * 0.5) + ($traderScore * 0.3) + ($recencyScore * 0.2);
    }
}
```

### Controllers

```php
// ✅ GOOD: Thin controller with form requests and resources
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreMarketRequest;
use App\Http\Resources\MarketResource;
use App\Services\MarketService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

class MarketController extends Controller
{
    public function __construct(
        private MarketService $marketService
    ) {}

    public function index(): AnonymousResourceCollection
    {
        $markets = $this->marketService->getActiveMarkets();

        return MarketResource::collection($markets);
    }

    public function store(StoreMarketRequest $request): JsonResponse
    {
        $market = $this->marketService->createMarket(
            $request->validated()
        );

        return response()->json([
            'success' => true,
            'data' => new MarketResource($market),
        ], 201);
    }

    public function show(Market $market): JsonResponse
    {
        return response()->json([
            'success' => true,
            'data' => new MarketResource($market),
        ]);
    }
}

// ❌ BAD: Fat controller with business logic
class MarketController extends Controller
{
    public function store(Request $request)
    {
        // Validation in controller
        $request->validate([...]);

        // Business logic in controller
        $market = Market::create($request->all());

        // Direct model return
        return $market;
    }
}
```

### Form Requests

```php
// ✅ GOOD: Dedicated form request with validation
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;
use App\Enums\MarketStatus;

class StoreMarketRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Market::class);
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'min:3', 'max:200'],
            'description' => ['required', 'string', 'min:10', 'max:2000'],
            'end_date' => ['required', 'date', 'after:now'],
            'category_ids' => ['required', 'array', 'min:1'],
            'category_ids.*' => ['exists:categories,id'],
            'status' => ['required', Rule::enum(MarketStatus::class)],
            'is_featured' => ['boolean'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required' => 'Market name is required',
            'end_date.after' => 'End date must be in the future',
        ];
    }

    protected function prepareForValidation(): void
    {
        $this->merge([
            'slug' => str($this->name)->slug(),
        ]);
    }
}
```

### API Resources

```php
// ✅ GOOD: Proper API resource transformation
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class MarketResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'description' => $this->description,
            'status' => $this->status->value,
            'status_label' => $this->status->label(),
            'end_date' => $this->end_date->toIso8601String(),
            'liquidity_score' => $this->liquidity_score,
            'is_active' => $this->is_active,

            // Relationships
            'creator' => new UserResource($this->whenLoaded('creator')),
            'trades_count' => $this->when(
                $this->trades_count !== null,
                $this->trades_count
            ),

            // Timestamps
            'created_at' => $this->created_at->toIso8601String(),
            'updated_at' => $this->updated_at->toIso8601String(),
        ];
    }
}
```

### Database Queries - N+1 Prevention

```php
// ✅ GOOD: Eager loading to prevent N+1
$markets = Market::query()
    ->with(['creator', 'categories', 'trades' => function ($query) {
        $query->latest()->limit(10);
    }])
    ->withCount('trades')
    ->active()
    ->paginate(20);

// ❌ BAD: N+1 query problem
$markets = Market::active()->get();
foreach ($markets as $market) {
    echo $market->creator->name; // N+1 query
}
```

### Database Transactions

```php
// ✅ GOOD: Proper transaction handling
use Illuminate\Support\Facades\DB;

public function resolveMarket(Market $market, float $outcome): void
{
    DB::transaction(function () use ($market, $outcome) {
        $market->update([
            'status' => MarketStatus::RESOLVED,
            'resolved_value' => $outcome,
            'resolved_at' => now(),
        ]);

        $this->distributePayouts($market, $outcome);

        $this->notifyTraders($market);

        Cache::tags(['markets', "market:{$market->id}"])->flush();
    });
}
```

### Caching Strategies

```php
// ✅ GOOD: Cache with tags and proper invalidation
use Illuminate\Support\Facades\Cache;

public function getActiveMarkets(): Collection
{
    return Cache::tags(['markets', 'active'])
        ->remember('markets:active', 3600, function () {
            return Market::with(['creator', 'categories'])
                ->active()
                ->get();
        });
}

public function clearMarketCache(Market $market): void
{
    Cache::tags(['markets', "market:{$market->id}"])->flush();
}
```

### Jobs and Queues

```php
// ✅ GOOD: Proper job structure
namespace App\Jobs;

use App\Models\Market;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

class CalculateMarketLiquidityJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $timeout = 120;

    public function __construct(
        private Market $market
    ) {}

    public function handle(MarketService $service): void
    {
        $score = $service->calculateLiquidityScore($this->market);

        $this->market->update(['liquidity_score' => $score]);

        Log::info('Liquidity calculated', [
            'market_id' => $this->market->id,
            'score' => $score,
        ]);
    }

    public function failed(\Throwable $exception): void
    {
        Log::error('Liquidity calculation failed', [
            'market_id' => $this->market->id,
            'error' => $exception->getMessage(),
        ]);
    }
}
```

## Security Best Practices

### SQL Injection Prevention

```php
// ✅ GOOD: Use query builder or Eloquent
$markets = DB::table('markets')
    ->where('status', $status)
    ->where('created_at', '>', $date)
    ->get();

// Parameter binding for raw queries
$markets = DB::select('SELECT * FROM markets WHERE status = ? AND created_at > ?', [$status, $date]);

// ❌ BAD: String concatenation
$markets = DB::select("SELECT * FROM markets WHERE status = '$status'");
```

### Mass Assignment Protection

```php
// ✅ GOOD: Use $fillable or $guarded
class Market extends Model
{
    protected $fillable = ['name', 'description', 'status'];

    // Or use guarded
    protected $guarded = ['id', 'created_at', 'updated_at'];
}
```

### Authorization

```php
// ✅ GOOD: Use policies and gates
namespace App\Policies;

use App\Models\Market;
use App\Models\User;

class MarketPolicy
{
    public function update(User $user, Market $market): bool
    {
        return $user->id === $market->creator_id || $user->isAdmin();
    }

    public function delete(User $user, Market $market): bool
    {
        return $user->isAdmin();
    }
}

// In controller
public function update(UpdateMarketRequest $request, Market $market)
{
    $this->authorize('update', $market);

    // Update logic
}
```

### Rate Limiting

```php
// ✅ GOOD: Apply rate limiting
use Illuminate\Support\Facades\RateLimiter;

// In RouteServiceProvider
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

// In routes
Route::middleware(['throttle:api'])->group(function () {
    Route::apiResource('markets', MarketController::class);
});
```

## Testing Standards

### Feature Tests

```php
// ✅ GOOD: Comprehensive feature test
namespace Tests\Feature;

use App\Models\Market;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class MarketControllerTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_create_market(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->postJson('/api/markets', [
                'name' => 'Will Bitcoin reach $100k?',
                'description' => 'Market description here',
                'end_date' => now()->addDays(30)->toDateTimeString(),
                'category_ids' => [1, 2],
            ]);

        $response->assertStatus(201)
            ->assertJsonStructure([
                'success',
                'data' => [
                    'id',
                    'name',
                    'status',
                    'created_at',
                ],
            ]);

        $this->assertDatabaseHas('markets', [
            'name' => 'Will Bitcoin reach $100k?',
            'creator_id' => $user->id,
        ]);
    }

    public function test_unauthorized_user_cannot_create_market(): void
    {
        $response = $this->postJson('/api/markets', [
            'name' => 'Test Market',
        ]);

        $response->assertStatus(401);
    }
}
```

### Unit Tests

```php
// ✅ GOOD: Unit test for service
namespace Tests\Unit;

use App\Models\Market;
use App\Services\MarketService;
use Mockery;
use Tests\TestCase;

class MarketServiceTest extends TestCase
{
    public function test_calculates_liquidity_score_correctly(): void
    {
        $market = Market::factory()->make([
            'id' => 1,
        ]);

        $service = new MarketService(
            Mockery::mock(MarketRepository::class),
            Mockery::mock(NotificationService::class)
        );

        $score = $service->calculateLiquidityScore($market);

        $this->assertIsFloat($score);
        $this->assertGreaterThanOrEqual(0, $score);
        $this->assertLessThanOrEqual(100, $score);
    }
}
```

## Code Organization

### File Structure

```
app/
├── Console/
│   └── Commands/         # Artisan commands
├── Exceptions/
│   └── Handler.php
├── Http/
│   ├── Controllers/
│   │   └── Api/         # API controllers
│   ├── Middleware/
│   ├── Requests/        # Form requests
│   └── Resources/       # API resources
├── Models/
├── Policies/
├── Providers/
├── Repositories/        # Repository pattern
├── Services/            # Business logic
└── Events/
    └── Listeners/
```

### Naming Conventions

```
Controllers:    MarketController
Models:         Market
Services:       MarketService
Repositories:   MarketRepository
Requests:       StoreMarketRequest, UpdateMarketRequest
Resources:      MarketResource
Jobs:           CalculateMarketLiquidityJob
Events:         MarketCreated
Listeners:      SendMarketCreatedNotification
Migrations:     2024_01_19_create_markets_table.php
```

## Laravel Artisan Commands

```php
// ✅ GOOD: Custom artisan command
namespace App\Console\Commands;

use App\Services\MarketService;
use Illuminate\Console\Command;

class RecalculateLiquidityCommand extends Command
{
    protected $signature = 'markets:recalculate-liquidity {--market-id=}';
    protected $description = 'Recalculate liquidity scores for markets';

    public function handle(MarketService $service): int
    {
        $this->info('Starting liquidity recalculation...');

        $marketId = $this->option('market-id');

        if ($marketId) {
            $market = Market::findOrFail($marketId);
            $service->updateLiquidityScore($market);
            $this->info("Updated market {$marketId}");
        } else {
            $markets = Market::active()->get();
            $bar = $this->output->createProgressBar($markets->count());

            foreach ($markets as $market) {
                $service->updateLiquidityScore($market);
                $bar->advance();
            }

            $bar->finish();
        }

        $this->newLine();
        $this->info('Liquidity recalculation complete!');

        return Command::SUCCESS;
    }
}
```

## Error Handling

```php
// ✅ GOOD: Custom exceptions with proper handling
namespace App\Exceptions;

use Exception;

class MarketResolutionException extends Exception
{
    public static function alreadyResolved(Market $market): self
    {
        return new self("Market {$market->id} is already resolved");
    }

    public static function notEnded(Market $market): self
    {
        return new self("Market {$market->id} has not ended yet");
    }
}

// Usage
if ($market->status->isResolved()) {
    throw MarketResolutionException::alreadyResolved($market);
}
```

## Performance Optimization

### Database Indexing

```php
// In migration
Schema::create('markets', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description');
    $table->string('status');
    $table->foreignId('creator_id')->constrained('users');
    $table->timestamp('end_date');
    $table->timestamps();

    // Indexes for performance
    $table->index('status');
    $table->index('creator_id');
    $table->index(['status', 'end_date']);
});
```

### Query Optimization

```php
// ✅ GOOD: Efficient queries
$markets = Market::query()
    ->select(['id', 'name', 'status', 'created_at'])
    ->where('status', MarketStatus::ACTIVE)
    ->orderBy('created_at', 'desc')
    ->limit(50)
    ->get();

// ❌ BAD: Inefficient query
$markets = Market::all()
    ->where('status', MarketStatus::ACTIVE)
    ->sortByDesc('created_at')
    ->take(50);
```

Remember: Laravel favors convention over configuration. Follow framework conventions and use built-in features before creating custom solutions.
