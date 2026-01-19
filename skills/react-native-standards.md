---
name: react-native-standards
description: React Native coding standards, best practices, and patterns for building cross-platform mobile applications with JavaScript/TypeScript.
---

# React Native Coding Standards

Comprehensive coding standards for React Native development with TypeScript.

## TypeScript Standards for React Native

### Component Types

```typescript
// ✅ GOOD: Properly typed React Native components
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';

interface MarketCardProps {
  market: Market;
  onPress?: () => void;
  testID?: string;
}

export const MarketCard: React.FC<MarketCardProps> = ({
  market,
  onPress,
  testID,
}) => {
  return (
    <TouchableOpacity
      style={styles.container}
      onPress={onPress}
      testID={testID}
      activeOpacity={0.7}
    >
      <View style={styles.header}>
        <Text style={styles.title}>{market.name}</Text>
        <StatusBadge status={market.status} />
      </View>
      <Text style={styles.description} numberOfLines={2}>
        {market.description}
      </Text>
    </TouchableOpacity>
  );
};

// ❌ BAD: No types, any props
export const MarketCard = ({ market, onPress }) => {
  return (
    <TouchableOpacity onPress={onPress}>
      <Text>{market.name}</Text>
    </TouchableOpacity>
  );
};
```

### Hooks with TypeScript

```typescript
// ✅ GOOD: Typed custom hooks
import { useState, useEffect, useCallback } from 'react';

interface UseMarketsReturn {
  markets: Market[];
  isLoading: boolean;
  error: string | null;
  refetch: () => Promise<void>;
}

export const useMarkets = (status?: MarketStatus): UseMarketsReturn => {
  const [markets, setMarkets] = useState<Market[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchMarkets = useCallback(async () => {
    setIsLoading(true);
    setError(null);

    try {
      const data = await marketApi.fetchMarkets({ status });
      setMarkets(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setIsLoading(false);
    }
  }, [status]);

  useEffect(() => {
    fetchMarkets();
  }, [fetchMarkets]);

  return { markets, isLoading, error, refetch: fetchMarkets };
};

// Usage
const { markets, isLoading, error, refetch } = useMarkets(MarketStatus.ACTIVE);
```

### Navigation Types (React Navigation)

```typescript
// ✅ GOOD: Type-safe navigation
import { NativeStackScreenProps } from '@react-navigation/native-stack';

// Define navigation params
export type RootStackParamList = {
  Home: undefined;
  MarketList: { status?: MarketStatus };
  MarketDetails: { marketId: string };
  CreateMarket: undefined;
  Profile: { userId: string };
};

// Type for screen props
type MarketDetailsScreenProps = NativeStackScreenProps<
  RootStackParamList,
  'MarketDetails'
>;

export const MarketDetailsScreen: React.FC<MarketDetailsScreenProps> = ({
  route,
  navigation,
}) => {
  const { marketId } = route.params;

  const handleNavigateBack = () => {
    navigation.goBack();
  };

  const handleNavigateToProfile = (userId: string) => {
    navigation.navigate('Profile', { userId });
  };

  return (
    // Component implementation
  );
};

// Type-safe navigation hook
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';

type NavigationProp = NativeStackNavigationProp<RootStackParamList>;

export const useAppNavigation = () => {
  return useNavigation<NavigationProp>();
};

// Usage
const navigation = useAppNavigation();
navigation.navigate('MarketDetails', { marketId: '123' });
```

## Component Structure

### Functional Components

```typescript
// ✅ GOOD: Clean functional component structure
import React, { useState, useCallback } from 'react';
import { View, Text, FlatList, RefreshControl } from 'react-native';

interface MarketListScreenProps {
  initialStatus?: MarketStatus;
}

export const MarketListScreen: React.FC<MarketListScreenProps> = ({
  initialStatus,
}) => {
  const { markets, isLoading, error, refetch } = useMarkets(initialStatus);
  const [refreshing, setRefreshing] = useState(false);

  const handleRefresh = useCallback(async () => {
    setRefreshing(true);
    await refetch();
    setRefreshing(false);
  }, [refetch]);

  const renderItem = useCallback(
    ({ item }: { item: Market }) => (
      <MarketCard
        market={item}
        onPress={() => navigation.navigate('MarketDetails', { marketId: item.id })}
      />
    ),
    [navigation]
  );

  const keyExtractor = useCallback((item: Market) => item.id, []);

  if (error) {
    return <ErrorView message={error} onRetry={refetch} />;
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={markets}
        renderItem={renderItem}
        keyExtractor={keyExtractor}
        refreshControl={
          <RefreshControl refreshing={refreshing} onRefresh={handleRefresh} />
        }
        ListEmptyComponent={
          isLoading ? <LoadingView /> : <EmptyView message="No markets found" />
        }
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
  },
});
```

### Memoization

```typescript
// ✅ GOOD: Use React.memo for expensive components
import React, { memo } from 'react';

interface MarketCardProps {
  market: Market;
  onPress: () => void;
}

export const MarketCard = memo<MarketCardProps>(
  ({ market, onPress }) => {
    return (
      <TouchableOpacity onPress={onPress}>
        <Text>{market.name}</Text>
        <Text>{market.description}</Text>
      </TouchableOpacity>
    );
  },
  (prevProps, nextProps) => {
    // Custom comparison
    return (
      prevProps.market.id === nextProps.market.id &&
      prevProps.market.status === nextProps.market.status
    );
  }
);

MarketCard.displayName = 'MarketCard';

// ✅ GOOD: useMemo for expensive calculations
const sortedMarkets = useMemo(() => {
  return markets
    .filter((m) => m.status === MarketStatus.ACTIVE)
    .sort((a, b) => b.liquidityScore - a.liquidityScore);
}, [markets]);

// ✅ GOOD: useCallback for callbacks passed to children
const handleMarketPress = useCallback(
  (marketId: string) => {
    navigation.navigate('MarketDetails', { marketId });
  },
  [navigation]
);
```

## StyleSheet Best Practices

```typescript
// ✅ GOOD: Organized styles with TypeScript
import { StyleSheet, ViewStyle, TextStyle } from 'react-native';

interface Styles {
  container: ViewStyle;
  header: ViewStyle;
  title: TextStyle;
  description: TextStyle;
  button: ViewStyle;
  buttonText: TextStyle;
}

const styles = StyleSheet.create<Styles>({
  container: {
    flex: 1,
    padding: 16,
    backgroundColor: '#FFFFFF',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: '700',
    color: '#000000',
  },
  description: {
    fontSize: 14,
    color: '#666666',
    lineHeight: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
});

// ✅ GOOD: Theme-based styles
interface Theme {
  colors: {
    primary: string;
    background: string;
    text: string;
    border: string;
  };
  spacing: {
    xs: number;
    sm: number;
    md: number;
    lg: number;
    xl: number;
  };
  typography: {
    h1: TextStyle;
    h2: TextStyle;
    body: TextStyle;
  };
}

export const theme: Theme = {
  colors: {
    primary: '#007AFF',
    background: '#FFFFFF',
    text: '#000000',
    border: '#E5E5E5',
  },
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
  },
  typography: {
    h1: {
      fontSize: 32,
      fontWeight: '700',
      lineHeight: 40,
    },
    h2: {
      fontSize: 24,
      fontWeight: '600',
      lineHeight: 32,
    },
    body: {
      fontSize: 16,
      fontWeight: '400',
      lineHeight: 24,
    },
  },
};

// Usage
const styles = StyleSheet.create({
  container: {
    padding: theme.spacing.md,
    backgroundColor: theme.colors.background,
  },
  title: {
    ...theme.typography.h1,
    color: theme.colors.text,
  },
});
```

## State Management with Zustand

```typescript
// ✅ GOOD: Zustand store with TypeScript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface Market {
  id: string;
  name: string;
  status: MarketStatus;
}

interface MarketStore {
  markets: Market[];
  isLoading: boolean;
  error: string | null;

  // Actions
  fetchMarkets: () => Promise<void>;
  addMarket: (market: Market) => void;
  updateMarket: (id: string, updates: Partial<Market>) => void;
  removeMarket: (id: string) => void;
  clearMarkets: () => void;
}

export const useMarketStore = create<MarketStore>()(
  persist(
    (set, get) => ({
      markets: [],
      isLoading: false,
      error: null,

      fetchMarkets: async () => {
        set({ isLoading: true, error: null });
        try {
          const response = await marketApi.fetchMarkets();
          set({ markets: response.data, isLoading: false });
        } catch (error) {
          set({
            error: error instanceof Error ? error.message : 'Unknown error',
            isLoading: false,
          });
        }
      },

      addMarket: (market) => {
        set((state) => ({
          markets: [...state.markets, market],
        }));
      },

      updateMarket: (id, updates) => {
        set((state) => ({
          markets: state.markets.map((m) =>
            m.id === id ? { ...m, ...updates } : m
          ),
        }));
      },

      removeMarket: (id) => {
        set((state) => ({
          markets: state.markets.filter((m) => m.id !== id),
        }));
      },

      clearMarkets: () => {
        set({ markets: [] });
      },
    }),
    {
      name: 'market-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);

// Usage in component
const MarketListScreen: React.FC = () => {
  const { markets, isLoading, error, fetchMarkets } = useMarketStore();

  useEffect(() => {
    fetchMarkets();
  }, [fetchMarkets]);

  return (
    <View>
      {markets.map((market) => (
        <MarketCard key={market.id} market={market} />
      ))}
    </View>
  );
};
```

## API Integration

```typescript
// ✅ GOOD: Type-safe API client
import axios, { AxiosInstance, AxiosError } from 'axios';

interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

class ApiClient {
  private client: AxiosInstance;

  constructor(baseURL: string) {
    this.client = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors(): void {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        const token = getAuthToken();
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response,
      (error: AxiosError) => {
        if (error.response?.status === 401) {
          // Handle unauthorized
          handleLogout();
        }
        return Promise.reject(error);
      }
    );
  }

  async get<T>(url: string, params?: Record<string, unknown>): Promise<ApiResponse<T>> {
    try {
      const response = await this.client.get<T>(url, { params });
      return {
        success: true,
        data: response.data,
      };
    } catch (error) {
      return this.handleError(error);
    }
  }

  async post<T>(url: string, data?: unknown): Promise<ApiResponse<T>> {
    try {
      const response = await this.client.post<T>(url, data);
      return {
        success: true,
        data: response.data,
      };
    } catch (error) {
      return this.handleError(error);
    }
  }

  private handleError<T>(error: unknown): ApiResponse<T> {
    if (axios.isAxiosError(error)) {
      return {
        success: false,
        error: error.response?.data?.message || error.message,
      };
    }
    return {
      success: false,
      error: 'An unexpected error occurred',
    };
  }
}

export const apiClient = new ApiClient('https://api.example.com');

// Market API
export const marketApi = {
  fetchMarkets: async (params?: { status?: MarketStatus }) => {
    return apiClient.get<Market[]>('/markets', params);
  },

  getMarket: async (id: string) => {
    return apiClient.get<Market>(`/markets/${id}`);
  },

  createMarket: async (data: MarketCreate) => {
    return apiClient.post<Market>('/markets', data);
  },
};
```

## Platform-Specific Code

```typescript
// ✅ GOOD: Platform-specific implementations
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    padding: 16,
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
      },
      android: {
        elevation: 4,
      },
    }),
  },
  text: {
    fontSize: Platform.OS === 'ios' ? 17 : 16,
    fontFamily: Platform.select({
      ios: 'System',
      android: 'Roboto',
    }),
  },
});

// Platform-specific components
const Button = Platform.select({
  ios: () => require('./Button.ios').Button,
  android: () => require('./Button.android').Button,
})();

// Platform version check
if (Platform.Version >= 21) {
  // Android API 21+
}
```

## Performance Optimization

### FlatList Optimization

```typescript
// ✅ GOOD: Optimized FlatList
interface MarketListProps {
  markets: Market[];
}

export const MarketList: React.FC<MarketListProps> = ({ markets }) => {
  const renderItem = useCallback(
    ({ item }: { item: Market }) => <MarketCard market={item} />,
    []
  );

  const keyExtractor = useCallback((item: Market) => item.id, []);

  const getItemLayout = useCallback(
    (data: ArrayLike<Market> | null | undefined, index: number) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );

  return (
    <FlatList
      data={markets}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      maxToRenderPerBatch={10}
      updateCellsBatchingPeriod={50}
      initialNumToRender={10}
      windowSize={5}
      removeClippedSubviews={Platform.OS === 'android'}
    />
  );
};
```

### Image Optimization

```typescript
// ✅ GOOD: Optimized image loading
import FastImage from 'react-native-fast-image';

export const MarketImage: React.FC<{ uri: string }> = ({ uri }) => {
  return (
    <FastImage
      style={styles.image}
      source={{
        uri,
        priority: FastImage.priority.normal,
        cache: FastImage.cacheControl.immutable,
      }}
      resizeMode={FastImage.resizeMode.cover}
    />
  );
};
```

## Testing

### Component Testing with React Native Testing Library

```typescript
// ✅ GOOD: Component tests
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { MarketCard } from './MarketCard';

describe('MarketCard', () => {
  const mockMarket: Market = {
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
```

### Hook Testing

```typescript
// ✅ GOOD: Custom hook tests
import { renderHook, waitFor } from '@testing-library/react-native';
import { useMarkets } from './useMarkets';

jest.mock('../api/marketApi');

describe('useMarkets', () => {
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

  it('handles fetch errors', async () => {
    (marketApi.fetchMarkets as jest.Mock).mockResolvedValue({
      success: false,
      error: 'Network error',
    });

    const { result } = renderHook(() => useMarkets());

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.error).toBe('Network error');
    expect(result.current.markets).toEqual([]);
  });
});
```

## Error Boundaries

```typescript
// ✅ GOOD: Error boundary implementation
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
    };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return {
      hasError: true,
      error,
    };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to error tracking service
  }

  handleReset = (): void => {
    this.setState({
      hasError: false,
      error: null,
    });
  };

  render(): ReactNode {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <View style={styles.container}>
          <Text style={styles.title}>Something went wrong</Text>
          <Text style={styles.message}>{this.state.error?.message}</Text>
          <TouchableOpacity style={styles.button} onPress={this.handleReset}>
            <Text style={styles.buttonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}
```

## Accessibility

```typescript
// ✅ GOOD: Accessible components
export const MarketCard: React.FC<MarketCardProps> = ({ market, onPress }) => {
  return (
    <TouchableOpacity
      onPress={onPress}
      accessible={true}
      accessibilityRole="button"
      accessibilityLabel={`Market: ${market.name}`}
      accessibilityHint="Tap to view market details"
      accessibilityState={{
        disabled: market.status === MarketStatus.CANCELLED,
      }}
    >
      <Text
        accessible={true}
        accessibilityRole="header"
        style={styles.title}
      >
        {market.name}
      </Text>
      <Text
        accessible={true}
        accessibilityLabel={`Description: ${market.description}`}
        style={styles.description}
      >
        {market.description}
      </Text>
    </TouchableOpacity>
  );
};
```

## Project Structure

```
src/
├── components/
│   ├── common/
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   └── Input.tsx
│   └── markets/
│       ├── MarketCard.tsx
│       └── MarketList.tsx
├── screens/
│   ├── HomeScreen.tsx
│   ├── MarketDetailsScreen.tsx
│   └── ProfileScreen.tsx
├── navigation/
│   ├── RootNavigator.tsx
│   └── types.ts
├── hooks/
│   ├── useMarkets.ts
│   └── useAuth.ts
├── store/
│   ├── marketStore.ts
│   └── authStore.ts
├── api/
│   ├── client.ts
│   └── marketApi.ts
├── utils/
│   ├── formatters.ts
│   └── validators.ts
├── types/
│   ├── market.ts
│   └── user.ts
├── constants/
│   ├── theme.ts
│   └── config.ts
└── __tests__/
    ├── components/
    └── hooks/
```

Remember: React Native bridges JavaScript and native code. Optimize for performance, handle platform differences gracefully, and prioritize user experience across devices.
