---
name: flutter-dart-standards
description: Flutter and Dart coding standards, best practices, and patterns for building cross-platform mobile applications.
---

# Flutter & Dart Coding Standards

Comprehensive coding standards for Flutter 3.x+ and Dart 3.x+ development.

## Dart Language Standards

### Type Safety and Null Safety

```dart
// ✅ GOOD: Null-safe Dart with proper types
class Market {
  final String id;
  final String name;
  final MarketStatus status;
  final DateTime createdAt;
  final double? resolvedValue; // Nullable
  final Map<String, dynamic> metadata;

  const Market({
    required this.id,
    required this.name,
    required this.status,
    required this.createdAt,
    this.resolvedValue,
    this.metadata = const {},
  });

  bool get isActive => status == MarketStatus.active;
}

// ❌ BAD: No null safety, dynamic types
class Market {
  var id;
  var name;
  var status;
  var createdAt;
  var resolvedValue;
}
```

### Enums and Sealed Classes

```dart
// ✅ GOOD: Use enums for fixed values
enum MarketStatus {
  active,
  resolved,
  cancelled;

  String get label {
    switch (this) {
      case MarketStatus.active:
        return 'Active';
      case MarketStatus.resolved:
        return 'Resolved';
      case MarketStatus.cancelled:
        return 'Cancelled';
    }
  }

  bool get isResolved => this == MarketStatus.resolved;
}

// Sealed classes for exhaustive pattern matching (Dart 3+)
sealed class ApiResponse<T> {
  const ApiResponse();
}

class Success<T> extends ApiResponse<T> {
  final T data;
  const Success(this.data);
}

class Error<T> extends ApiResponse<T> {
  final String message;
  const Error(this.message);
}

class Loading<T> extends ApiResponse<T> {
  const Loading();
}

// Usage with exhaustive pattern matching
String handleResponse(ApiResponse<Market> response) {
  return switch (response) {
    Success(data: final market) => 'Loaded: ${market.name}',
    Error(message: final msg) => 'Error: $msg',
    Loading() => 'Loading...',
  };
}
```

### Records (Dart 3+)

```dart
// ✅ GOOD: Use records for simple data grouping
(String, int) getUserInfo() {
  return ('John Doe', 30);
}

// Named records
({String name, int age, String email}) getUserDetails() {
  return (name: 'John Doe', age: 30, email: 'john@example.com');
}

// Usage
final (name, age) = getUserInfo();
final user = getUserDetails();
print(user.name); // Access by name
```

### Extension Methods

```dart
// ✅ GOOD: Extension methods for cleaner code
extension StringExtensions on String {
  String capitalize() {
    if (isEmpty) return this;
    return '${this[0].toUpperCase()}${substring(1)}';
  }

  bool get isValidEmail {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(this);
  }
}

extension DateTimeExtensions on DateTime {
  bool get isToday {
    final now = DateTime.now();
    return year == now.year && month == now.month && day == now.day;
  }

  String toRelativeTime() {
    final difference = DateTime.now().difference(this);
    if (difference.inDays > 0) return '${difference.inDays}d ago';
    if (difference.inHours > 0) return '${difference.inHours}h ago';
    if (difference.inMinutes > 0) return '${difference.inMinutes}m ago';
    return 'Just now';
  }
}

// Usage
final email = 'john@example.com';
print(email.isValidEmail); // true

final date = DateTime.now().subtract(Duration(hours: 2));
print(date.toRelativeTime()); // "2h ago"
```

### Async/Await and Futures

```dart
// ✅ GOOD: Proper async handling
Future<List<Market>> fetchMarkets({
  MarketStatus? status,
  int limit = 20,
}) async {
  try {
    final response = await http.get(
      Uri.parse('https://api.example.com/markets'),
      headers: {'Authorization': 'Bearer $token'},
    );

    if (response.statusCode == 200) {
      final List<dynamic> data = json.decode(response.body);
      return data.map((json) => Market.fromJson(json)).toList();
    } else {
      throw ApiException('Failed to fetch markets: ${response.statusCode}');
    }
  } on SocketException {
    throw NetworkException('No internet connection');
  } catch (e) {
    throw ApiException('Unexpected error: $e');
  }
}

// Parallel async operations
Future<(List<Market>, User, Stats)> fetchDashboardData() async {
  final results = await Future.wait([
    fetchMarkets(),
    fetchCurrentUser(),
    fetchStats(),
  ]);

  return (
    results[0] as List<Market>,
    results[1] as User,
    results[2] as Stats,
  );
}
```

### Streams

```dart
// ✅ GOOD: Use streams for reactive data
class MarketRepository {
  final _marketsController = StreamController<List<Market>>.broadcast();

  Stream<List<Market>> get marketsStream => _marketsController.stream;

  Future<void> fetchMarkets() async {
    try {
      final markets = await _fetchMarketsFromApi();
      _marketsController.add(markets);
    } catch (e) {
      _marketsController.addError(e);
    }
  }

  void dispose() {
    _marketsController.close();
  }
}

// Stream transformations
Stream<List<Market>> get activeMarketsStream {
  return marketsStream.map((markets) {
    return markets.where((m) => m.status == MarketStatus.active).toList();
  });
}
```

## Flutter Widget Best Practices

### Stateless vs Stateful Widgets

```dart
// ✅ GOOD: Use StatelessWidget when no state needed
class MarketCard extends StatelessWidget {
  final Market market;
  final VoidCallback? onTap;

  const MarketCard({
    super.key,
    required this.market,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: InkWell(
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                market.name,
                style: Theme.of(context).textTheme.titleLarge,
              ),
              const SizedBox(height: 8),
              Text(
                market.description,
                style: Theme.of(context).textTheme.bodyMedium,
                maxLines: 2,
                overflow: TextOverflow.ellipsis,
              ),
              const SizedBox(height: 8),
              _StatusChip(status: market.status),
            ],
          ),
        ),
      ),
    );
  }
}

// ✅ GOOD: Use StatefulWidget when managing local state
class MarketSearchBar extends StatefulWidget {
  final ValueChanged<String> onSearch;

  const MarketSearchBar({
    super.key,
    required this.onSearch,
  });

  @override
  State<MarketSearchBar> createState() => _MarketSearchBarState();
}

class _MarketSearchBarState extends State<MarketSearchBar> {
  final _controller = TextEditingController();
  Timer? _debounce;

  @override
  void dispose() {
    _controller.dispose();
    _debounce?.cancel();
    super.dispose();
  }

  void _onSearchChanged(String query) {
    _debounce?.cancel();
    _debounce = Timer(const Duration(milliseconds: 500), () {
      widget.onSearch(query);
    });
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      onChanged: _onSearchChanged,
      decoration: const InputDecoration(
        hintText: 'Search markets...',
        prefixIcon: Icon(Icons.search),
      ),
    );
  }
}
```

### State Management with Riverpod

```dart
// ✅ GOOD: Riverpod for state management
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Provider for repository
final marketRepositoryProvider = Provider<MarketRepository>((ref) {
  return MarketRepository(
    apiClient: ref.watch(apiClientProvider),
  );
});

// FutureProvider for async data
final marketsProvider = FutureProvider<List<Market>>((ref) async {
  final repository = ref.watch(marketRepositoryProvider);
  return repository.fetchMarkets();
});

// StateNotifierProvider for complex state
class MarketsNotifier extends StateNotifier<AsyncValue<List<Market>>> {
  final MarketRepository _repository;

  MarketsNotifier(this._repository) : super(const AsyncValue.loading()) {
    fetchMarkets();
  }

  Future<void> fetchMarkets() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() => _repository.fetchMarkets());
  }

  Future<void> createMarket(Market market) async {
    final result = await _repository.createMarket(market);
    result.when(
      success: (_) => fetchMarkets(),
      error: (error) => state = AsyncValue.error(error, StackTrace.current),
    );
  }
}

final marketsNotifierProvider =
    StateNotifierProvider<MarketsNotifier, AsyncValue<List<Market>>>((ref) {
  return MarketsNotifier(ref.watch(marketRepositoryProvider));
});

// Usage in widget
class MarketListScreen extends ConsumerWidget {
  const MarketListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final marketsAsync = ref.watch(marketsNotifierProvider);

    return marketsAsync.when(
      data: (markets) => ListView.builder(
        itemCount: markets.length,
        itemBuilder: (context, index) {
          return MarketCard(market: markets[index]);
        },
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, stack) => ErrorWidget(error: error.toString()),
    );
  }
}
```

### Widget Composition

```dart
// ✅ GOOD: Break down complex widgets into smaller components
class MarketDetailsScreen extends StatelessWidget {
  final Market market;

  const MarketDetailsScreen({
    super.key,
    required this.market,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(market.name),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            _MarketHeader(market: market),
            const SizedBox(height: 24),
            _MarketDescription(description: market.description),
            const SizedBox(height: 24),
            _MarketStats(market: market),
            const SizedBox(height: 24),
            _TradingPanel(market: market),
          ],
        ),
      ),
    );
  }
}

class _MarketHeader extends StatelessWidget {
  final Market market;

  const _MarketHeader({required this.market});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Expanded(
          child: Text(
            market.name,
            style: Theme.of(context).textTheme.headlineMedium,
          ),
        ),
        _StatusChip(status: market.status),
      ],
    );
  }
}

// ❌ BAD: Monolithic widget with everything in build method
class MarketDetailsScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 200+ lines of nested widgets
        ],
      ),
    );
  }
}
```

### Const Constructors

```dart
// ✅ GOOD: Use const constructors for immutable widgets
class EmptyStateWidget extends StatelessWidget {
  final String message;

  const EmptyStateWidget({
    super.key,
    required this.message,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(
            Icons.inbox_outlined,
            size: 64,
            color: Colors.grey,
          ),
          const SizedBox(height: 16),
          Text(
            message,
            style: Theme.of(context).textTheme.titleMedium,
          ),
        ],
      ),
    );
  }
}

// Usage
const EmptyStateWidget(message: 'No markets found')
```

### Responsive Design

```dart
// ✅ GOOD: Responsive layouts
class ResponsiveLayout extends StatelessWidget {
  final Widget mobile;
  final Widget tablet;
  final Widget desktop;

  const ResponsiveLayout({
    super.key,
    required this.mobile,
    required this.tablet,
    required this.desktop,
  });

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth < 600) {
          return mobile;
        } else if (constraints.maxWidth < 1200) {
          return tablet;
        } else {
          return desktop;
        }
      },
    );
  }
}

// Media query helpers
class ScreenSize {
  static bool isMobile(BuildContext context) =>
      MediaQuery.of(context).size.width < 600;

  static bool isTablet(BuildContext context) {
    final width = MediaQuery.of(context).size.width;
    return width >= 600 && width < 1200;
  }

  static bool isDesktop(BuildContext context) =>
      MediaQuery.of(context).size.width >= 1200;
}
```

## Navigation

```dart
// ✅ GOOD: GoRouter for declarative routing
import 'package:go_router/go_router.dart';

final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/markets',
      builder: (context, state) => const MarketListScreen(),
      routes: [
        GoRoute(
          path: ':id',
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return MarketDetailsScreen(marketId: id);
          },
        ),
      ],
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileScreen(),
      redirect: (context, state) {
        final isAuthenticated = /* check auth */;
        if (!isAuthenticated) return '/login';
        return null;
      },
    ),
  ],
  errorBuilder: (context, state) => ErrorScreen(error: state.error),
);

// Usage
context.go('/markets/market_123');
context.push('/markets/market_123');
```

## Data Layer Architecture

```dart
// ✅ GOOD: Clean architecture with layers

// 1. Data Models
class MarketModel {
  final String id;
  final String name;
  final String status;

  const MarketModel({
    required this.id,
    required this.name,
    required this.status,
  });

  factory MarketModel.fromJson(Map<String, dynamic> json) {
    return MarketModel(
      id: json['id'] as String,
      name: json['name'] as String,
      status: json['status'] as String,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'status': status,
    };
  }
}

// 2. Repository Interface
abstract class MarketRepository {
  Future<Result<List<Market>>> fetchMarkets();
  Future<Result<Market>> getMarket(String id);
  Future<Result<Market>> createMarket(MarketCreate data);
}

// 3. Repository Implementation
class MarketRepositoryImpl implements MarketRepository {
  final ApiClient _apiClient;
  final CacheService _cache;

  const MarketRepositoryImpl({
    required ApiClient apiClient,
    required CacheService cache,
  })  : _apiClient = apiClient,
        _cache = cache;

  @override
  Future<Result<List<Market>>> fetchMarkets() async {
    try {
      // Try cache first
      final cached = await _cache.get<List<Market>>('markets');
      if (cached != null) {
        return Result.success(cached);
      }

      // Fetch from API
      final response = await _apiClient.get('/markets');
      final markets = (response.data as List)
          .map((json) => Market.fromJson(json))
          .toList();

      // Cache result
      await _cache.set('markets', markets, duration: Duration(minutes: 5));

      return Result.success(markets);
    } catch (e) {
      return Result.error(e.toString());
    }
  }
}

// 4. Result type for error handling
sealed class Result<T> {
  const Result();

  factory Result.success(T data) = Success<T>;
  factory Result.error(String message) = Error<T>;

  R when<R>({
    required R Function(T data) success,
    required R Function(String message) error,
  }) {
    return switch (this) {
      Success(data: final data) => success(data),
      Error(message: final message) => error(message),
    };
  }
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Error<T> extends Result<T> {
  final String message;
  const Error(this.message);
}
```

## Testing Standards

### Unit Tests

```dart
// ✅ GOOD: Comprehensive unit tests
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';

@GenerateMocks([ApiClient, CacheService])
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
        Market(id: '1', name: 'Test Market', status: MarketStatus.active),
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
}
```

### Widget Tests

```dart
// ✅ GOOD: Widget tests
import 'package:flutter_test/flutter_test.dart';

void main() {
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

  testWidgets('MarketCard calls onTap when tapped', (tester) async {
    // Arrange
    bool tapped = false;
    final market = Market(
      id: '1',
      name: 'Test',
      description: 'Test',
      status: MarketStatus.active,
      createdAt: DateTime.now(),
    );

    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          body: MarketCard(
            market: market,
            onTap: () => tapped = true,
          ),
        ),
      ),
    );

    await tester.tap(find.byType(InkWell));
    await tester.pumpAndSettle();

    // Assert
    expect(tapped, true);
  });
}
```

### Integration Tests

```dart
// ✅ GOOD: Integration tests
import 'package:integration_test/integration_test.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('User can create a market', (tester) async {
    // Launch app
    await tester.pumpWidget(const MyApp());
    await tester.pumpAndSettle();

    // Navigate to create market screen
    await tester.tap(find.byIcon(Icons.add));
    await tester.pumpAndSettle();

    // Fill form
    await tester.enterText(
      find.byKey(const Key('market_name')),
      'Test Market',
    );
    await tester.enterText(
      find.byKey(const Key('market_description')),
      'This is a test market description',
    );

    // Submit
    await tester.tap(find.text('Create'));
    await tester.pumpAndSettle();

    // Verify success
    expect(find.text('Market created successfully'), findsOneWidget);
    expect(find.text('Test Market'), findsOneWidget);
  });
}
```

## Performance Optimization

```dart
// ✅ GOOD: Optimize list rendering
class MarketListView extends StatelessWidget {
  final List<Market> markets;

  const MarketListView({super.key, required this.markets});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: markets.length,
      itemBuilder: (context, index) {
        return MarketCard(
          key: ValueKey(markets[index].id),
          market: markets[index],
        );
      },
    );
  }
}

// ✅ GOOD: Use AutomaticKeepAliveClientMixin for tab views
class MarketsTab extends StatefulWidget {
  const MarketsTab({super.key});

  @override
  State<MarketsTab> createState() => _MarketsTabState();
}

class _MarketsTabState extends State<MarketsTab>
    with AutomaticKeepAliveClientMixin {
  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context); // Must call super
    return MarketListView();
  }
}

// ✅ GOOD: Image caching
CachedNetworkImage(
  imageUrl: market.imageUrl,
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(Icons.error),
  fadeInDuration: const Duration(milliseconds: 300),
)
```

## Code Organization

### Project Structure

```
lib/
├── main.dart
├── app/
│   ├── routes/
│   └── theme/
├── core/
│   ├── constants/
│   ├── extensions/
│   ├── utils/
│   └── errors/
├── features/
│   ├── markets/
│   │   ├── data/
│   │   │   ├── models/
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   └── repositories/
│   │   └── presentation/
│   │       ├── screens/
│   │       ├── widgets/
│   │       └── providers/
│   └── auth/
└── shared/
    ├── widgets/
    └── services/
```

Remember: Flutter emphasizes composition over inheritance. Build reusable widgets and leverage Dart's modern language features for clean, maintainable code.
