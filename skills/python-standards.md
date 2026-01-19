---
name: python-standards
description: Python coding standards, best practices, and patterns for Python 3.10+, Django, FastAPI, and Flask development.
---

# Python Coding Standards

Comprehensive coding standards for Python 3.10+ development with Django, FastAPI, and Flask.

## Python Language Standards

### Type Hints (PEP 484)

```python
# ✅ GOOD: Complete type hints
from typing import Optional, List, Dict, Union
from datetime import datetime
from decimal import Decimal

def calculate_liquidity(
    volume: float,
    trader_count: int,
    last_trade: Optional[datetime] = None
) -> float:
    """Calculate market liquidity score."""
    # Implementation
    return 0.0

class Market:
    def __init__(
        self,
        id: str,
        name: str,
        status: str,
        created_at: datetime
    ) -> None:
        self.id = id
        self.name = name
        self.status = status
        self.created_at = created_at

# ❌ BAD: No type hints
def calculate_liquidity(volume, trader_count, last_trade=None):
    return 0.0
```

### Modern Type Hints (Python 3.10+)

```python
# ✅ GOOD: Use built-in generics (Python 3.10+)
from collections.abc import Sequence, Mapping

def process_markets(markets: list[dict[str, str | int]]) -> list[str]:
    """Process markets and return names."""
    return [market['name'] for market in markets]

def get_user_data(user_id: int) -> dict[str, str | int | None]:
    """Get user data by ID."""
    return {
        'id': user_id,
        'name': 'John Doe',
        'age': 30,
        'email': None
    }

# Union type with | operator
def find_market(identifier: str | int) -> Market | None:
    """Find market by ID or slug."""
    pass

# ❌ BAD: Old-style typing (avoid in Python 3.10+)
from typing import List, Dict, Optional, Union

def process_markets(markets: List[Dict[str, Union[str, int]]]) -> List[str]:
    pass
```

### Data Classes

```python
# ✅ GOOD: Use dataclasses for data structures
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum

class MarketStatus(Enum):
    ACTIVE = "active"
    RESOLVED = "resolved"
    CANCELLED = "cancelled"

@dataclass
class Market:
    id: str
    name: str
    status: MarketStatus
    created_at: datetime
    resolved_value: float | None = None
    metadata: dict[str, str] = field(default_factory=dict)

    def is_active(self) -> bool:
        """Check if market is active."""
        return self.status == MarketStatus.ACTIVE

    def __post_init__(self) -> None:
        """Validate after initialization."""
        if len(self.name) < 3:
            raise ValueError("Market name must be at least 3 characters")

# Usage
market = Market(
    id="market_123",
    name="Bitcoin $100k?",
    status=MarketStatus.ACTIVE,
    created_at=datetime.now()
)

# ❌ BAD: Manual class with boilerplate
class Market:
    def __init__(self, id, name, status, created_at):
        self.id = id
        self.name = name
        self.status = status
        self.created_at = created_at
```

### Pattern Matching (Python 3.10+)

```python
# ✅ GOOD: Use structural pattern matching
def handle_market_status(market: Market) -> str:
    match market.status:
        case MarketStatus.ACTIVE:
            return "Market is currently active"
        case MarketStatus.RESOLVED if market.resolved_value is not None:
            return f"Market resolved at {market.resolved_value}"
        case MarketStatus.CANCELLED:
            return "Market was cancelled"
        case _:
            return "Unknown status"

# Pattern matching with structures
def process_api_response(response: dict) -> str:
    match response:
        case {"success": True, "data": data}:
            return f"Success: {data}"
        case {"success": False, "error": error}:
            return f"Error: {error}"
        case _:
            return "Invalid response format"
```

### Context Managers

```python
# ✅ GOOD: Use context managers for resource management
from contextlib import contextmanager
import logging

@contextmanager
def database_transaction():
    """Context manager for database transactions."""
    try:
        # Begin transaction
        yield
        # Commit
        logging.info("Transaction committed")
    except Exception as e:
        # Rollback
        logging.error(f"Transaction rolled back: {e}")
        raise

# Usage
with database_transaction():
    # Database operations
    pass
```

### List/Dict Comprehensions

```python
# ✅ GOOD: Readable comprehensions
active_markets = [
    market for market in markets
    if market.status == MarketStatus.ACTIVE
]

market_names = {
    market.id: market.name
    for market in markets
}

# ❌ BAD: Complex comprehension (use regular loop)
# Don't do this - too complex
result = [
    x for sublist in nested_list
    for x in sublist
    if x > 0 and x % 2 == 0
    if some_complex_condition(x)
]

# ✅ GOOD: Use regular loop for clarity
result = []
for sublist in nested_list:
    for x in sublist:
        if x > 0 and x % 2 == 0:
            if some_complex_condition(x):
                result.append(x)
```

## Django Best Practices

### Models

```python
# ✅ GOOD: Comprehensive Django model
from django.db import models
from django.contrib.auth import get_user_model
from django.core.validators import MinValueValidator, MaxValueValidator
from django.utils import timezone

User = get_user_model()

class MarketStatus(models.TextChoices):
    ACTIVE = "active", "Active"
    RESOLVED = "resolved", "Resolved"
    CANCELLED = "cancelled", "Cancelled"

class Market(models.Model):
    """Market model for prediction markets."""

    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=200, db_index=True)
    description = models.TextField()
    status = models.CharField(
        max_length=20,
        choices=MarketStatus.choices,
        default=MarketStatus.ACTIVE,
        db_index=True
    )
    creator = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name="created_markets"
    )
    end_date = models.DateTimeField()
    resolved_value = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        null=True,
        blank=True,
        validators=[MinValueValidator(0), MaxValueValidator(100)]
    )
    liquidity_score = models.FloatField(default=0.0)
    metadata = models.JSONField(default=dict)

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = "markets"
        ordering = ["-created_at"]
        indexes = [
            models.Index(fields=["status", "end_date"]),
            models.Index(fields=["creator", "-created_at"]),
        ]
        verbose_name = "Market"
        verbose_name_plural = "Markets"

    def __str__(self) -> str:
        return self.name

    def is_active(self) -> bool:
        """Check if market is currently active."""
        return (
            self.status == MarketStatus.ACTIVE
            and self.end_date > timezone.now()
        )

    def save(self, *args, **kwargs) -> None:
        """Override save to add validation."""
        self.full_clean()
        super().save(*args, **kwargs)

    @property
    def trade_count(self) -> int:
        """Get number of trades."""
        return self.trades.count()
```

### Django Views (Class-Based Views)

```python
# ✅ GOOD: Class-based views with mixins
from django.views.generic import ListView, CreateView, DetailView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.db.models import Q, Count
from django.http import JsonResponse

class MarketListView(ListView):
    """List all active markets."""

    model = Market
    template_name = "markets/list.html"
    context_object_name = "markets"
    paginate_by = 20

    def get_queryset(self):
        """Filter and optimize queryset."""
        queryset = Market.objects.select_related("creator").prefetch_related(
            "categories"
        ).annotate(
            trade_count=Count("trades")
        )

        status = self.request.GET.get("status")
        if status:
            queryset = queryset.filter(status=status)

        search = self.request.GET.get("q")
        if search:
            queryset = queryset.filter(
                Q(name__icontains=search) | Q(description__icontains=search)
            )

        return queryset

    def get_context_data(self, **kwargs):
        """Add extra context."""
        context = super().get_context_data(**kwargs)
        context["total_markets"] = Market.objects.count()
        return context


class MarketCreateView(LoginRequiredMixin, CreateView):
    """Create a new market."""

    model = Market
    form_class = MarketForm
    template_name = "markets/create.html"
    success_url = "/markets/"

    def form_valid(self, form):
        """Set creator before saving."""
        form.instance.creator = self.request.user
        return super().form_valid(form)
```

### Django REST Framework

```python
# ✅ GOOD: DRF ViewSet with serializers
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from django_filters.rest_framework import DjangoFilterBackend

class MarketSerializer(serializers.ModelSerializer):
    """Market serializer."""

    creator_name = serializers.CharField(source="creator.username", read_only=True)
    is_active = serializers.BooleanField(read_only=True)
    trade_count = serializers.IntegerField(read_only=True)

    class Meta:
        model = Market
        fields = [
            "id",
            "name",
            "description",
            "status",
            "creator_name",
            "end_date",
            "is_active",
            "trade_count",
            "created_at",
        ]
        read_only_fields = ["id", "created_at"]

    def validate_end_date(self, value):
        """Validate end date is in future."""
        if value <= timezone.now():
            raise serializers.ValidationError("End date must be in the future")
        return value


class MarketViewSet(viewsets.ModelViewSet):
    """Market API endpoints."""

    queryset = Market.objects.select_related("creator").annotate(
        trade_count=Count("trades")
    )
    serializer_class = MarketSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ["status", "creator"]

    def perform_create(self, serializer):
        """Set creator when creating market."""
        serializer.save(creator=self.request.user)

    @action(detail=True, methods=["post"])
    def resolve(self, request, pk=None):
        """Resolve a market."""
        market = self.get_object()

        if market.status != MarketStatus.ACTIVE:
            return Response(
                {"error": "Market is not active"},
                status=status.HTTP_400_BAD_REQUEST
            )

        resolved_value = request.data.get("resolved_value")
        if resolved_value is None:
            return Response(
                {"error": "resolved_value is required"},
                status=status.HTTP_400_BAD_REQUEST
            )

        market.status = MarketStatus.RESOLVED
        market.resolved_value = resolved_value
        market.save()

        return Response(MarketSerializer(market).data)
```

## FastAPI Best Practices

### FastAPI Routes

```python
# ✅ GOOD: FastAPI with Pydantic models
from fastapi import FastAPI, HTTPException, Depends, status
from pydantic import BaseModel, Field, validator
from datetime import datetime
from typing import Optional

app = FastAPI(title="Market API", version="1.0.0")

# Pydantic models
class MarketBase(BaseModel):
    name: str = Field(..., min_length=3, max_length=200)
    description: str = Field(..., min_length=10, max_length=2000)
    end_date: datetime

    @validator("end_date")
    def end_date_must_be_future(cls, v):
        """Validate end date is in future."""
        if v <= datetime.now():
            raise ValueError("End date must be in the future")
        return v

class MarketCreate(MarketBase):
    """Schema for creating a market."""
    pass

class MarketResponse(MarketBase):
    """Schema for market response."""
    id: str
    status: str
    creator_id: str
    liquidity_score: float
    created_at: datetime

    class Config:
        orm_mode = True

class MarketList(BaseModel):
    """Schema for paginated market list."""
    total: int
    page: int
    size: int
    markets: list[MarketResponse]

# Dependency injection
async def get_current_user(token: str = Depends(oauth2_scheme)):
    """Get current authenticated user."""
    # Verify token and return user
    pass

# Routes
@app.post(
    "/api/markets",
    response_model=MarketResponse,
    status_code=status.HTTP_201_CREATED,
    tags=["markets"]
)
async def create_market(
    market: MarketCreate,
    current_user = Depends(get_current_user)
) -> MarketResponse:
    """Create a new market."""
    # Create market logic
    pass

@app.get(
    "/api/markets",
    response_model=MarketList,
    tags=["markets"]
)
async def list_markets(
    page: int = 1,
    size: int = 20,
    status: Optional[str] = None,
    search: Optional[str] = None
) -> MarketList:
    """List markets with pagination."""
    # Query logic
    pass

@app.get(
    "/api/markets/{market_id}",
    response_model=MarketResponse,
    tags=["markets"]
)
async def get_market(market_id: str) -> MarketResponse:
    """Get market by ID."""
    market = await find_market(market_id)
    if not market:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Market {market_id} not found"
        )
    return market
```

### FastAPI Background Tasks

```python
# ✅ GOOD: Background tasks
from fastapi import BackgroundTasks

async def send_notification(user_id: str, message: str) -> None:
    """Send notification to user."""
    # Send notification logic
    pass

@app.post("/api/markets/{market_id}/resolve")
async def resolve_market(
    market_id: str,
    resolved_value: float,
    background_tasks: BackgroundTasks
):
    """Resolve market and notify users."""
    market = await find_market(market_id)

    if not market:
        raise HTTPException(status_code=404, detail="Market not found")

    market.status = MarketStatus.RESOLVED
    market.resolved_value = resolved_value
    await market.save()

    # Add background task
    background_tasks.add_task(
        send_notification,
        market.creator_id,
        f"Market {market.name} has been resolved"
    )

    return market
```

## Async/Await Patterns

```python
# ✅ GOOD: Proper async usage
import asyncio
import aiohttp
from typing import List

async def fetch_market_data(market_id: str) -> dict:
    """Fetch market data from external API."""
    async with aiohttp.ClientSession() as session:
        async with session.get(f"https://api.example.com/markets/{market_id}") as response:
            return await response.json()

async def fetch_multiple_markets(market_ids: List[str]) -> List[dict]:
    """Fetch multiple markets concurrently."""
    tasks = [fetch_market_data(market_id) for market_id in market_ids]
    return await asyncio.gather(*tasks)

# Usage
markets = await fetch_multiple_markets(["market_1", "market_2", "market_3"])
```

## Error Handling

```python
# ✅ GOOD: Custom exceptions with proper handling
class MarketException(Exception):
    """Base exception for market operations."""
    pass

class MarketNotFoundError(MarketException):
    """Market not found exception."""
    def __init__(self, market_id: str):
        self.market_id = market_id
        super().__init__(f"Market {market_id} not found")

class MarketAlreadyResolvedException(MarketException):
    """Market already resolved exception."""
    def __init__(self, market_id: str):
        self.market_id = market_id
        super().__init__(f"Market {market_id} is already resolved")

# Usage
def resolve_market(market_id: str, value: float) -> Market:
    """Resolve a market."""
    market = find_market(market_id)

    if not market:
        raise MarketNotFoundError(market_id)

    if market.status == MarketStatus.RESOLVED:
        raise MarketAlreadyResolvedException(market_id)

    market.status = MarketStatus.RESOLVED
    market.resolved_value = value
    market.save()

    return market

# Error handling
try:
    market = resolve_market("market_123", 75.5)
except MarketNotFoundError as e:
    logger.error(f"Market not found: {e.market_id}")
except MarketAlreadyResolvedException as e:
    logger.warning(f"Market already resolved: {e.market_id}")
```

## Testing Standards

### Pytest

```python
# ✅ GOOD: Comprehensive pytest tests
import pytest
from datetime import datetime, timedelta
from unittest.mock import Mock, patch

@pytest.fixture
def sample_market():
    """Fixture for sample market."""
    return Market(
        id="market_123",
        name="Bitcoin $100k?",
        status=MarketStatus.ACTIVE,
        created_at=datetime.now(),
        end_date=datetime.now() + timedelta(days=30)
    )

def test_market_is_active(sample_market):
    """Test market is_active method."""
    assert sample_market.is_active() is True

def test_market_not_active_when_resolved(sample_market):
    """Test market is not active when resolved."""
    sample_market.status = MarketStatus.RESOLVED
    assert sample_market.is_active() is False

@pytest.mark.asyncio
async def test_fetch_market_data():
    """Test fetching market data."""
    market_data = await fetch_market_data("market_123")
    assert "id" in market_data
    assert market_data["id"] == "market_123"

@pytest.mark.parametrize("volume,trader_count,expected_score", [
    (10000, 100, 85.0),
    (1000, 10, 50.0),
    (100, 1, 15.0),
])
def test_liquidity_calculation(volume, trader_count, expected_score):
    """Test liquidity score calculation with various inputs."""
    score = calculate_liquidity(volume, trader_count)
    assert abs(score - expected_score) < 5.0
```

### Django Tests

```python
# ✅ GOOD: Django test case
from django.test import TestCase, Client
from django.contrib.auth import get_user_model
from django.urls import reverse

User = get_user_model()

class MarketViewTests(TestCase):
    """Test market views."""

    def setUp(self):
        """Set up test fixtures."""
        self.client = Client()
        self.user = User.objects.create_user(
            username="testuser",
            password="testpass123"
        )
        self.market = Market.objects.create(
            name="Test Market",
            description="Test description",
            status=MarketStatus.ACTIVE,
            creator=self.user,
            end_date=timezone.now() + timedelta(days=30)
        )

    def test_market_list_view(self):
        """Test market list view returns markets."""
        response = self.client.get(reverse("market-list"))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Test Market")

    def test_create_market_requires_authentication(self):
        """Test creating market requires login."""
        response = self.client.post(reverse("market-create"), {
            "name": "New Market",
            "description": "Description",
        })
        self.assertEqual(response.status_code, 302)  # Redirect to login

    def test_authenticated_user_can_create_market(self):
        """Test authenticated user can create market."""
        self.client.login(username="testuser", password="testpass123")
        response = self.client.post(reverse("market-create"), {
            "name": "New Market",
            "description": "Test description for new market",
            "end_date": (timezone.now() + timedelta(days=30)).isoformat(),
        })
        self.assertEqual(response.status_code, 302)
        self.assertTrue(Market.objects.filter(name="New Market").exists())
```

## Code Organization

### Project Structure (Django)

```
project/
├── manage.py
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── markets/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   ├── services.py
│   │   ├── tests/
│   │   └── migrations/
│   └── users/
├── core/
│   ├── exceptions.py
│   ├── middleware.py
│   └── utils.py
└── requirements/
    ├── base.txt
    ├── development.txt
    └── production.txt
```

### Project Structure (FastAPI)

```
project/
├── main.py
├── api/
│   ├── routes/
│   │   ├── markets.py
│   │   └── users.py
│   ├── dependencies.py
│   └── middleware.py
├── models/
│   ├── market.py
│   └── user.py
├── schemas/
│   ├── market.py
│   └── user.py
├── services/
│   ├── market_service.py
│   └── notification_service.py
├── core/
│   ├── config.py
│   ├── security.py
│   └── exceptions.py
└── tests/
    ├── test_markets.py
    └── test_users.py
```

## Performance Best Practices

### Database Query Optimization

```python
# ✅ GOOD: Optimize queries
# Django
markets = Market.objects.select_related(
    "creator"
).prefetch_related(
    "categories",
    "trades__user"
).filter(
    status=MarketStatus.ACTIVE
).only(
    "id", "name", "status", "created_at"
)

# ❌ BAD: N+1 queries
markets = Market.objects.filter(status=MarketStatus.ACTIVE)
for market in markets:
    print(market.creator.username)  # N+1 query
    print(market.trades.count())  # N+1 query
```

### Caching

```python
# ✅ GOOD: Use caching
from django.core.cache import cache
from functools import wraps

def cache_result(timeout: int = 3600):
    """Cache decorator."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            cache_key = f"{func.__name__}:{args}:{kwargs}"
            result = cache.get(cache_key)

            if result is None:
                result = func(*args, **kwargs)
                cache.set(cache_key, result, timeout)

            return result
        return wrapper
    return decorator

@cache_result(timeout=1800)
def get_active_markets() -> list[Market]:
    """Get active markets with caching."""
    return Market.objects.filter(status=MarketStatus.ACTIVE).all()
```

Remember: Write Pythonic code that follows PEP 8, use type hints, and prioritize readability over cleverness.
