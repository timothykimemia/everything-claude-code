---
name: java-springboot-standards
description: Java and Spring Boot coding standards, best practices, and patterns for building enterprise-grade applications.
---

# Java & Spring Boot Coding Standards

Comprehensive coding standards for Java 17+ and Spring Boot 3.x development.

## Java Language Standards

### Modern Java Features (Java 17+)

```java
// ✅ GOOD: Use records for DTOs (Java 16+)
public record MarketDTO(
    String id,
    String name,
    MarketStatus status,
    LocalDateTime createdAt,
    Double resolvedValue
) {
    // Compact constructor for validation
    public MarketDTO {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
    }

    // Custom methods
    public boolean isActive() {
        return status == MarketStatus.ACTIVE;
    }
}

// ❌ BAD: Old-style POJO with boilerplate
public class MarketDTO {
    private String id;
    private String name;
    // ... getters, setters, equals, hashCode, toString
}
```

### Sealed Classes (Java 17+)

```java
// ✅ GOOD: Sealed classes for restricted hierarchies
public sealed interface ApiResponse<T>
    permits Success, Error, Loading {

    record Success<T>(T data) implements ApiResponse<T> {}
    record Error<T>(String message) implements ApiResponse<T> {}
    record Loading<T>() implements ApiResponse<T> {}
}

// Pattern matching with sealed classes
public String handleResponse(ApiResponse<Market> response) {
    return switch (response) {
        case Success<Market>(Market market) -> "Loaded: " + market.getName();
        case Error<Market>(String message) -> "Error: " + message;
        case Loading<Market>() -> "Loading...";
    };
}
```

### Pattern Matching (Java 17+)

```java
// ✅ GOOD: Pattern matching for instanceof
public double calculateDiscount(Object obj) {
    if (obj instanceof Market market && market.isActive()) {
        return market.getPrice() * 0.1;
    }
    return 0.0;
}

// Pattern matching in switch
public String getStatusLabel(MarketStatus status) {
    return switch (status) {
        case ACTIVE -> "Active";
        case RESOLVED -> "Resolved";
        case CANCELLED -> "Cancelled";
        default -> "Unknown";
    };
}
```

### Text Blocks (Java 15+)

```java
// ✅ GOOD: Text blocks for multi-line strings
String query = """
    SELECT m.id, m.name, m.status, m.created_at
    FROM markets m
    WHERE m.status = :status
    AND m.end_date > :now
    ORDER BY m.created_at DESC
    """;

String json = """
    {
        "id": "%s",
        "name": "%s",
        "status": "%s"
    }
    """.formatted(market.getId(), market.getName(), market.getStatus());
```

### Optional Best Practices

```java
// ✅ GOOD: Proper Optional usage
public Optional<Market> findMarket(String id) {
    return marketRepository.findById(id);
}

// Chain operations
String marketName = findMarket(id)
    .map(Market::getName)
    .orElse("Unknown");

// Throw exception if not present
Market market = findMarket(id)
    .orElseThrow(() -> new MarketNotFoundException(id));

// ❌ BAD: Don't use Optional.get() without checking
Market market = findMarket(id).get(); // May throw NoSuchElementException

// ❌ BAD: Don't use Optional for fields
public class Market {
    private Optional<String> description; // BAD
}
```

### Stream API

```java
// ✅ GOOD: Effective stream usage
List<Market> activeMarkets = markets.stream()
    .filter(Market::isActive)
    .sorted(Comparator.comparing(Market::getCreatedAt).reversed())
    .limit(10)
    .toList(); // Java 16+

Map<MarketStatus, List<Market>> marketsByStatus = markets.stream()
    .collect(Collectors.groupingBy(Market::getStatus));

double avgLiquidity = markets.stream()
    .filter(Market::isActive)
    .mapToDouble(Market::getLiquidityScore)
    .average()
    .orElse(0.0);

// Parallel streams for large datasets
List<Market> processed = markets.parallelStream()
    .filter(Market::isActive)
    .map(this::enrichMarketData)
    .toList();
```

## Spring Boot Best Practices

### Entity Classes

```java
// ✅ GOOD: Comprehensive JPA entity
package com.example.markets.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;
import java.util.UUID;

@Entity
@Table(
    name = "markets",
    indexes = {
        @Index(name = "idx_status", columnList = "status"),
        @Index(name = "idx_creator_id", columnList = "creator_id"),
        @Index(name = "idx_status_end_date", columnList = "status,end_date")
    }
)
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Market {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(name = "id", updatable = false, nullable = false)
    private UUID id;

    @NotBlank
    @Size(min = 3, max = 200)
    @Column(name = "name", nullable = false)
    private String name;

    @NotBlank
    @Size(min = 10, max = 2000)
    @Column(name = "description", nullable = false, columnDefinition = "TEXT")
    private String description;

    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false)
    private MarketStatus status = MarketStatus.ACTIVE;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "creator_id", nullable = false)
    private User creator;

    @NotNull
    @Column(name = "end_date", nullable = false)
    private LocalDateTime endDate;

    @Column(name = "resolved_value")
    private Double resolvedValue;

    @Column(name = "liquidity_score")
    private Double liquidityScore = 0.0;

    @ManyToMany
    @JoinTable(
        name = "market_categories",
        joinColumns = @JoinColumn(name = "market_id"),
        inverseJoinColumns = @JoinColumn(name = "category_id")
    )
    @Builder.Default
    private Set<Category> categories = new HashSet<>();

    @OneToMany(mappedBy = "market", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private Set<Trade> trades = new HashSet<>();

    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @Version
    @Column(name = "version")
    private Long version;

    // Business methods
    public boolean isActive() {
        return status == MarketStatus.ACTIVE && endDate.isAfter(LocalDateTime.now());
    }

    public void addCategory(Category category) {
        categories.add(category);
        category.getMarkets().add(this);
    }

    public void removeCategory(Category category) {
        categories.remove(category);
        category.getMarkets().remove(this);
    }
}
```

### Repository Layer

```java
// ✅ GOOD: Spring Data JPA repository
package com.example.markets.repository;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

@Repository
public interface MarketRepository extends JpaRepository<Market, UUID> {

    // Query methods
    List<Market> findByStatus(MarketStatus status);

    Page<Market> findByStatusAndEndDateAfter(
        MarketStatus status,
        LocalDateTime date,
        Pageable pageable
    );

    Optional<Market> findByIdAndCreatorId(UUID marketId, UUID creatorId);

    // Custom JPQL query
    @Query("""
        SELECT m FROM Market m
        LEFT JOIN FETCH m.creator
        LEFT JOIN FETCH m.categories
        WHERE m.status = :status
        ORDER BY m.createdAt DESC
        """)
    List<Market> findActiveMarketsWithDetails(@Param("status") MarketStatus status);

    // Native query
    @Query(value = """
        SELECT * FROM markets m
        WHERE m.status = :status
        AND m.liquidity_score > :minScore
        ORDER BY m.liquidity_score DESC
        LIMIT :limit
        """, nativeQuery = true)
    List<Market> findTopMarketsByLiquidity(
        @Param("status") String status,
        @Param("minScore") Double minScore,
        @Param("limit") int limit
    );

    // Projection
    @Query("SELECT new com.example.markets.dto.MarketSummaryDTO(m.id, m.name, m.status) " +
           "FROM Market m WHERE m.status = :status")
    List<MarketSummaryDTO> findMarketSummaries(@Param("status") MarketStatus status);

    boolean existsByNameIgnoreCase(String name);

    long countByStatus(MarketStatus status);
}
```

### Service Layer

```java
// ✅ GOOD: Service layer with proper structure
package com.example.markets.service;

import com.example.markets.dto.MarketCreateDTO;
import com.example.markets.dto.MarketResponseDTO;
import com.example.markets.entity.Market;
import com.example.markets.exception.MarketNotFoundException;
import com.example.markets.mapper.MarketMapper;
import com.example.markets.repository.MarketRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.UUID;

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
@Slf4j
public class MarketService {

    private final MarketRepository marketRepository;
    private final MarketMapper marketMapper;
    private final NotificationService notificationService;

    @Cacheable(value = "markets", key = "#id")
    public MarketResponseDTO getMarket(UUID id) {
        log.debug("Fetching market with id: {}", id);

        Market market = marketRepository.findById(id)
            .orElseThrow(() -> new MarketNotFoundException(id));

        return marketMapper.toResponseDTO(market);
    }

    public Page<MarketResponseDTO> getMarkets(
        MarketStatus status,
        Pageable pageable
    ) {
        log.debug("Fetching markets with status: {}, page: {}", status, pageable.getPageNumber());

        return marketRepository.findByStatus(status, pageable)
            .map(marketMapper::toResponseDTO);
    }

    @Transactional
    @CacheEvict(value = "markets", allEntries = true)
    public MarketResponseDTO createMarket(MarketCreateDTO createDTO, UUID creatorId) {
        log.info("Creating market: {}", createDTO.name());

        // Validation
        if (marketRepository.existsByNameIgnoreCase(createDTO.name())) {
            throw new DuplicateMarketException(createDTO.name());
        }

        // Create entity
        Market market = marketMapper.toEntity(createDTO);
        market.setCreator(userRepository.getReferenceById(creatorId));

        // Save
        Market savedMarket = marketRepository.save(market);

        // Send notification
        notificationService.sendMarketCreatedNotification(savedMarket);

        log.info("Market created successfully with id: {}", savedMarket.getId());

        return marketMapper.toResponseDTO(savedMarket);
    }

    @Transactional
    @CacheEvict(value = "markets", key = "#id")
    public MarketResponseDTO updateMarket(
        UUID id,
        MarketUpdateDTO updateDTO
    ) {
        log.info("Updating market: {}", id);

        Market market = marketRepository.findById(id)
            .orElseThrow(() -> new MarketNotFoundException(id));

        marketMapper.updateEntity(market, updateDTO);

        Market updatedMarket = marketRepository.save(market);

        return marketMapper.toResponseDTO(updatedMarket);
    }

    @Transactional
    @CacheEvict(value = "markets", key = "#id")
    public void deleteMarket(UUID id) {
        log.info("Deleting market: {}", id);

        if (!marketRepository.existsById(id)) {
            throw new MarketNotFoundException(id);
        }

        marketRepository.deleteById(id);
    }

    public double calculateLiquidityScore(Market market) {
        long tradeCount = market.getTrades().size();
        double totalVolume = market.getTrades().stream()
            .mapToDouble(Trade::getVolume)
            .sum();

        // Calculate score (0-100)
        double volumeScore = Math.min(totalVolume / 1000.0, 100.0);
        double tradeScore = Math.min(tradeCount / 10.0, 100.0);

        return (volumeScore * 0.6) + (tradeScore * 0.4);
    }
}
```

### Controller Layer

```java
// ✅ GOOD: RESTful controller
package com.example.markets.controller;

import com.example.markets.dto.*;
import com.example.markets.service.MarketService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.web.bind.annotation.*;

import java.util.UUID;

@RestController
@RequestMapping("/api/v1/markets")
@RequiredArgsConstructor
@Tag(name = "Markets", description = "Market management API")
public class MarketController {

    private final MarketService marketService;

    @GetMapping
    @Operation(summary = "Get all markets")
    public ResponseEntity<ApiResponse<Page<MarketResponseDTO>>> getMarkets(
        @RequestParam(required = false) MarketStatus status,
        @PageableDefault(size = 20) Pageable pageable
    ) {
        Page<MarketResponseDTO> markets = marketService.getMarkets(status, pageable);

        return ResponseEntity.ok(ApiResponse.success(markets));
    }

    @GetMapping("/{id}")
    @Operation(summary = "Get market by ID")
    public ResponseEntity<ApiResponse<MarketResponseDTO>> getMarket(
        @PathVariable UUID id
    ) {
        MarketResponseDTO market = marketService.getMarket(id);

        return ResponseEntity.ok(ApiResponse.success(market));
    }

    @PostMapping
    @PreAuthorize("hasRole('USER')")
    @Operation(summary = "Create new market")
    public ResponseEntity<ApiResponse<MarketResponseDTO>> createMarket(
        @Valid @RequestBody MarketCreateDTO createDTO,
        @AuthenticationPrincipal UserDetails userDetails
    ) {
        UUID creatorId = UUID.fromString(userDetails.getUsername());
        MarketResponseDTO market = marketService.createMarket(createDTO, creatorId);

        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(ApiResponse.success(market));
    }

    @PutMapping("/{id}")
    @PreAuthorize("hasRole('USER')")
    @Operation(summary = "Update market")
    public ResponseEntity<ApiResponse<MarketResponseDTO>> updateMarket(
        @PathVariable UUID id,
        @Valid @RequestBody MarketUpdateDTO updateDTO
    ) {
        MarketResponseDTO market = marketService.updateMarket(id, updateDTO);

        return ResponseEntity.ok(ApiResponse.success(market));
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    @Operation(summary = "Delete market")
    public ResponseEntity<ApiResponse<Void>> deleteMarket(@PathVariable UUID id) {
        marketService.deleteMarket(id);

        return ResponseEntity.ok(ApiResponse.success(null));
    }
}
```

### DTO and Mapping

```java
// ✅ GOOD: Request DTO with validation
package com.example.markets.dto;

import com.example.markets.entity.MarketStatus;
import jakarta.validation.constraints.*;

import java.time.LocalDateTime;
import java.util.Set;

public record MarketCreateDTO(
    @NotBlank(message = "Name is required")
    @Size(min = 3, max = 200, message = "Name must be between 3 and 200 characters")
    String name,

    @NotBlank(message = "Description is required")
    @Size(min = 10, max = 2000, message = "Description must be between 10 and 2000 characters")
    String description,

    @NotNull(message = "End date is required")
    @Future(message = "End date must be in the future")
    LocalDateTime endDate,

    @NotEmpty(message = "At least one category is required")
    Set<UUID> categoryIds
) {}

// Response DTO
public record MarketResponseDTO(
    UUID id,
    String name,
    String description,
    MarketStatus status,
    String creatorName,
    LocalDateTime endDate,
    Double resolvedValue,
    Double liquidityScore,
    Set<CategoryDTO> categories,
    LocalDateTime createdAt,
    LocalDateTime updatedAt
) {}

// MapStruct mapper
@Mapper(componentModel = "spring")
public interface MarketMapper {

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "creator", ignore = true)
    @Mapping(target = "categories", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    Market toEntity(MarketCreateDTO dto);

    @Mapping(source = "creator.username", target = "creatorName")
    MarketResponseDTO toResponseDTO(Market entity);

    void updateEntity(@MappingTarget Market entity, MarketUpdateDTO dto);
}
```

### Exception Handling

```java
// ✅ GOOD: Custom exceptions
package com.example.markets.exception;

import java.util.UUID;

public class MarketNotFoundException extends RuntimeException {
    public MarketNotFoundException(UUID id) {
        super("Market not found with id: " + id);
    }
}

public class DuplicateMarketException extends RuntimeException {
    public DuplicateMarketException(String name) {
        super("Market already exists with name: " + name);
    }
}

// Global exception handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(MarketNotFoundException.class)
    public ResponseEntity<ApiResponse<Void>> handleMarketNotFound(
        MarketNotFoundException ex
    ) {
        log.error("Market not found: {}", ex.getMessage());
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ApiResponse.error(ex.getMessage()));
    }

    @ExceptionHandler(DuplicateMarketException.class)
    public ResponseEntity<ApiResponse<Void>> handleDuplicateMarket(
        DuplicateMarketException ex
    ) {
        log.error("Duplicate market: {}", ex.getMessage());
        return ResponseEntity
            .status(HttpStatus.CONFLICT)
            .body(ApiResponse.error(ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Map<String, String>>> handleValidationErrors(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage
            ));

        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(ApiResponse.error("Validation failed", errors));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<Void>> handleGenericException(Exception ex) {
        log.error("Unexpected error: ", ex);
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ApiResponse.error("An unexpected error occurred"));
    }
}
```

### Configuration

```java
// ✅ GOOD: Configuration classes
@Configuration
@EnableCaching
@EnableAsync
public class AppConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("markets", "users");
    }

    @Bean
    public ObjectMapper objectMapper() {
        return JsonMapper.builder()
            .findAndAddModules()
            .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
            .build();
    }

    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
            .setConnectTimeout(Duration.ofSeconds(5))
            .setReadTimeout(Duration.ofSeconds(10))
            .build();
    }
}

// Security configuration
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/swagger-ui/**", "/v3/api-docs/**")
                .permitAll()
                .anyRequest()
                .authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

## Testing Standards

### Unit Tests

```java
// ✅ GOOD: JUnit 5 tests
@ExtendWith(MockitoExtension.class)
class MarketServiceTest {

    @Mock
    private MarketRepository marketRepository;

    @Mock
    private MarketMapper marketMapper;

    @Mock
    private NotificationService notificationService;

    @InjectMocks
    private MarketService marketService;

    private Market testMarket;
    private MarketCreateDTO createDTO;

    @BeforeEach
    void setUp() {
        testMarket = Market.builder()
            .id(UUID.randomUUID())
            .name("Test Market")
            .status(MarketStatus.ACTIVE)
            .build();

        createDTO = new MarketCreateDTO(
            "Test Market",
            "Test description",
            LocalDateTime.now().plusDays(30),
            Set.of(UUID.randomUUID())
        );
    }

    @Test
    @DisplayName("Should return market when found by ID")
    void getMarket_WhenExists_ReturnsMarket() {
        // Given
        UUID marketId = testMarket.getId();
        MarketResponseDTO expectedResponse = new MarketResponseDTO(/* ... */);

        when(marketRepository.findById(marketId)).thenReturn(Optional.of(testMarket));
        when(marketMapper.toResponseDTO(testMarket)).thenReturn(expectedResponse);

        // When
        MarketResponseDTO result = marketService.getMarket(marketId);

        // Then
        assertNotNull(result);
        assertEquals(expectedResponse, result);
        verify(marketRepository).findById(marketId);
        verify(marketMapper).toResponseDTO(testMarket);
    }

    @Test
    @DisplayName("Should throw exception when market not found")
    void getMarket_WhenNotExists_ThrowsException() {
        // Given
        UUID marketId = UUID.randomUUID();
        when(marketRepository.findById(marketId)).thenReturn(Optional.empty());

        // When & Then
        assertThrows(
            MarketNotFoundException.class,
            () -> marketService.getMarket(marketId)
        );
        verify(marketRepository).findById(marketId);
        verifyNoInteractions(marketMapper);
    }

    @Test
    @DisplayName("Should create market successfully")
    void createMarket_WithValidData_CreatesMarket() {
        // Given
        UUID creatorId = UUID.randomUUID();
        when(marketRepository.existsByNameIgnoreCase(createDTO.name())).thenReturn(false);
        when(marketMapper.toEntity(createDTO)).thenReturn(testMarket);
        when(marketRepository.save(testMarket)).thenReturn(testMarket);

        // When
        marketService.createMarket(createDTO, creatorId);

        // Then
        verify(marketRepository).save(testMarket);
        verify(notificationService).sendMarketCreatedNotification(testMarket);
    }
}
```

### Integration Tests

```java
// ✅ GOOD: Spring Boot integration tests
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
        // Given
        MarketCreateDTO createDTO = new MarketCreateDTO(
            "Integration Test Market",
            "Test description for integration test",
            LocalDateTime.now().plusDays(30),
            Set.of(UUID.randomUUID())
        );

        // When & Then
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
        // Given
        UUID nonExistentId = UUID.randomUUID();

        // When & Then
        mockMvc.perform(get("/api/v1/markets/{id}", nonExistentId))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.success").value(false))
            .andExpect(jsonPath("$.error").exists());
    }
}
```

## Performance Optimization

```java
// ✅ GOOD: Query optimization
@Query("""
    SELECT m FROM Market m
    LEFT JOIN FETCH m.creator
    LEFT JOIN FETCH m.categories
    WHERE m.status = :status
    """)
List<Market> findActiveMarketsOptimized(@Param("status") MarketStatus status);

// Batch processing
@Transactional
public void processMarkets(List<UUID> marketIds) {
    List<Market> markets = marketRepository.findAllById(marketIds);

    markets.forEach(market -> {
        market.setLiquidityScore(calculateLiquidityScore(market));
    });

    marketRepository.saveAll(markets); // Batch update
}

// Async processing
@Async
public CompletableFuture<Void> calculateLiquidityAsync(UUID marketId) {
    Market market = marketRepository.findById(marketId)
        .orElseThrow(() -> new MarketNotFoundException(marketId));

    double score = calculateLiquidityScore(market);
    market.setLiquidityScore(score);
    marketRepository.save(market);

    return CompletableFuture.completedFuture(null);
}
```

## Project Structure

```
src/
└── main/
    ├── java/
    │   └── com/example/markets/
    │       ├── MarketApplication.java
    │       ├── config/
    │       │   ├── AppConfig.java
    │       │   ├── SecurityConfig.java
    │       │   └── SwaggerConfig.java
    │       ├── controller/
    │       │   └── MarketController.java
    │       ├── dto/
    │       │   ├── MarketCreateDTO.java
    │       │   ├── MarketResponseDTO.java
    │       │   └── ApiResponse.java
    │       ├── entity/
    │       │   ├── Market.java
    │       │   └── User.java
    │       ├── repository/
    │       │   └── MarketRepository.java
    │       ├── service/
    │       │   ├── MarketService.java
    │       │   └── NotificationService.java
    │       ├── mapper/
    │       │   └── MarketMapper.java
    │       ├── exception/
    │       │   ├── MarketNotFoundException.java
    │       │   └── GlobalExceptionHandler.java
    │       └── util/
    └── resources/
        ├── application.yml
        ├── application-dev.yml
        └── application-prod.yml
```

Remember: Spring Boot embraces convention over configuration. Leverage auto-configuration, use dependency injection, and follow REST API best practices for maintainable enterprise applications.
