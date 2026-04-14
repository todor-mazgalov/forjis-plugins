---
name: react-native
description: >
  React Native cross-platform mobile development skill. Produces clean, performant,
  type-safe TypeScript code for React Native applications targeting iOS and Android.
  Enforces proper component architecture, platform-aware patterns, performant list
  rendering, correct navigation patterns, native module integration, and mobile-specific
  security and UX best practices.
  TRIGGER when: project uses React Native (react-native in package.json, metro.config.js,
  app.json with expo, *.tsx files with React Native imports), or user asks to build
  a mobile app with React Native, implement a cross-platform feature, or fix RN bugs.
  DO NOT TRIGGER when: project is React web-only (no react-native dependency),
  Flutter, native Swift/Kotlin-only, or Ionic/Capacitor without React Native.
---

# React Native Cross-Platform Mobile Development

Write clean, performant, type-safe TypeScript for React Native. Prioritize
smooth 60fps UX, platform-aware behavior, and shared cross-platform logic.

**Related skills:** Combine with `ui` for visual design principles (spacing
scales, color palettes, typography, hierarchy) — the Refactoring UI principles
apply to mobile as well as web. Do **not** use `shadcn` here; that skill is
for web targets only.

## Before Writing Code

1. **Read existing code first.** Search the project for existing components, hooks,
   services, and shared utilities before creating anything new. Reuse what exists.
2. **Check the React Native version.** Look at `package.json` for `react-native`
   version, check if using Expo or bare workflow, and verify the New Architecture
   (Fabric/TurboModules) status.
3. **Check platform targets.** Confirm iOS minimum deployment target and Android
   `minSdkVersion` in native config files.
4. **Follow project conventions.** Match component patterns, navigation structure,
   state management approach, and styling methodology already in use.

---

## TypeScript Configuration

React Native projects must use strict TypeScript. Recommended `tsconfig.json`:

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "lib": ["ES2022"],
    "skipLibCheck": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "forceConsistentCasingInImports": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true
  },
  "exclude": ["node_modules", "babel.config.js", "metro.config.js"]
}
```

Key flags:

| Flag | Purpose |
|------|---------|
| `strict: true` | All strict type checks — non-negotiable |
| `noUncheckedIndexedAccess` | Prevents unsafe array/object index access |
| `isolatedModules` | Required by Metro bundler for correct transforms |
| `moduleResolution: "bundler"` | Matches Metro's module resolution |

---

## Component Architecture

### Use functional components exclusively

```typescript
// WRONG — class components
class UserProfile extends React.Component<Props> {
  render() {
    return <Text>{this.props.name}</Text>;
  }
}

// CORRECT — functional component with explicit typing
interface UserProfileProps {
  name: string;
  avatarUrl: string | null;
  onPress: () => void;
}

function UserProfile({ name, avatarUrl, onPress }: UserProfileProps): React.JSX.Element {
  return (
    <Pressable onPress={onPress}>
      {avatarUrl && <Image source={{ uri: avatarUrl }} style={styles.avatar} />}
      <Text style={styles.name}>{name}</Text>
    </Pressable>
  );
}
```

### Never use `React.FC` — use plain function declarations

```typescript
// WRONG — React.FC adds implicit children, poor generics support
const UserList: React.FC<Props> = ({ users }) => { ... };

// CORRECT — explicit props, explicit return type
function UserList({ users }: UserListProps): React.JSX.Element {
  return (
    <FlatList
      data={users}
      renderItem={({ item }) => <UserCard user={item} />}
      keyExtractor={(item) => item.id}
    />
  );
}
```

### Keep components small and focused

Split into three layers:

```
src/components/
├── ui/              # Pure presentational (Button, Card, Avatar)
├── features/        # Feature-specific (UserCard, OrderItem)
└── screens/         # Full screens (HomeScreen, ProfileScreen)
```

- **UI components** — no business logic, accept data via props, fully reusable
- **Feature components** — compose UI components, may use feature-specific hooks
- **Screens** — compose feature components, handle navigation, connect to state

### Extract logic into custom hooks

```typescript
// WRONG — logic mixed into component
function UserListScreen(): React.JSX.Element {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch((e) => setError(e.message))
      .finally(() => setLoading(false));
  }, []);

  // ... render
}

// CORRECT — logic extracted to hook
function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch((e: Error) => setError(e.message))
      .finally(() => setLoading(false));
  }, []);

  return { users, loading, error } as const;
}

function UserListScreen(): React.JSX.Element {
  const { users, loading, error } = useUsers();
  // ... render only
}
```

---

## Performance

### Memoize expensive components with `React.memo`

```typescript
// Only re-renders when props actually change
const UserCard = React.memo(function UserCard({ user, onPress }: UserCardProps): React.JSX.Element {
  return (
    <Pressable onPress={() => onPress(user.id)}>
      <Text>{user.name}</Text>
    </Pressable>
  );
});
```

### Use `useCallback` and `useMemo` to prevent unnecessary re-renders

```typescript
function UserListScreen(): React.JSX.Element {
  const { users } = useUsers();

  // CORRECT — stable callback reference
  const handlePress = useCallback((id: string) => {
    navigation.navigate("UserDetail", { id });
  }, [navigation]);

  // CORRECT — derived data memoized
  const activeUsers = useMemo(
    () => users.filter((u) => u.isActive),
    [users],
  );

  return (
    <FlatList
      data={activeUsers}
      renderItem={({ item }) => <UserCard user={item} onPress={handlePress} />}
      keyExtractor={(item) => item.id}
    />
  );
}
```

### FlatList optimization — always follow these rules

```typescript
<FlatList
  data={items}
  renderItem={renderItem}           // Extract outside component or useCallback
  keyExtractor={keyExtractor}       // Stable function, not inline
  getItemLayout={getItemLayout}     // Provide if items have fixed height
  removeClippedSubviews={true}      // Unmount offscreen items
  maxToRenderPerBatch={10}          // Tune based on item complexity
  windowSize={5}                    // Render 5 viewports worth of items
  initialNumToRender={10}           // Render 10 items initially
/>
```

**Never** do these in FlatList:

```typescript
// WRONG — creates new array every render
<FlatList data={items.filter(i => i.active)} ... />

// WRONG — inline renderItem causes re-mount on every render
<FlatList renderItem={({ item }) => <Card item={item} />} ... />

// WRONG — missing keyExtractor (falls back to index)
<FlatList data={items} renderItem={renderItem} />
```

### Avoid unnecessary re-renders from StyleSheet

```typescript
// CORRECT — styles created once outside component
const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
  title: {
    fontSize: 18,
    fontWeight: "600",
  },
});

// WRONG — inline styles create new objects every render
<View style={{ flex: 1, padding: 16 }}>
```

### Use `Animated` API or Reanimated for animations

```typescript
// CORRECT — Reanimated for 60fps animations on UI thread
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from "react-native-reanimated";

function AnimatedCard(): React.JSX.Element {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePressIn = () => {
    scale.value = withSpring(0.95);
  };

  const handlePressOut = () => {
    scale.value = withSpring(1);
  };

  return (
    <Pressable onPressIn={handlePressIn} onPressOut={handlePressOut}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <Text>Card Content</Text>
      </Animated.View>
    </Pressable>
  );
}

// WRONG — JS-driven animations (janky, blocks JS thread)
// WRONG — using setTimeout/setInterval for animations
```

### Avoid bridge traffic in hot paths

```typescript
// WRONG — frequent bridge calls in scroll handler
onScroll={(event) => {
  sendAnalytics(event.nativeEvent.contentOffset.y); // bridge call every frame
}}

// CORRECT — debounce or use Reanimated worklets for scroll-driven logic
const scrollHandler = useAnimatedScrollHandler({
  onScroll: (event) => {
    // Runs on UI thread, no bridge
    headerOpacity.value = interpolate(
      event.contentOffset.y,
      [0, 100],
      [1, 0],
    );
  },
});
```

---

## Platform-Aware Development

### Use `Platform.select` and `Platform.OS` for platform differences

```typescript
import { Platform, StyleSheet } from "react-native";

const styles = StyleSheet.create({
  shadow: Platform.select({
    ios: {
      shadowColor: "#000",
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.1,
      shadowRadius: 4,
    },
    android: {
      elevation: 4,
    },
    default: {},
  }),
});

// For behavioral differences
if (Platform.OS === "ios") {
  // iOS-specific behavior
}
```

### Use platform-specific file extensions for major divergences

```
Button.tsx          # Shared logic
Button.ios.tsx      # iOS-specific implementation
Button.android.tsx  # Android-specific implementation
```

Metro automatically resolves the correct file per platform. Use this for
substantially different implementations, not minor style tweaks.

### Handle safe areas correctly

```typescript
import { SafeAreaView } from "react-native-safe-area-context";

// CORRECT — handles notch, home indicator, status bar
function Screen({ children }: { children: React.ReactNode }): React.JSX.Element {
  return (
    <SafeAreaView style={styles.container} edges={["top", "bottom"]}>
      {children}
    </SafeAreaView>
  );
}

// WRONG — built-in SafeAreaView (iOS only, unreliable)
import { SafeAreaView } from "react-native";
```

### Handle keyboard correctly

```typescript
import { KeyboardAvoidingView, Platform } from "react-native";

<KeyboardAvoidingView
  behavior={Platform.OS === "ios" ? "padding" : "height"}
  keyboardVerticalOffset={Platform.OS === "ios" ? 88 : 0}
>
  <TextInput ... />
  <Button ... />
</KeyboardAvoidingView>
```

---

## Navigation

### Use React Navigation with typed routes

```typescript
import type { NativeStackScreenProps } from "@react-navigation/native-stack";

// Define all routes and their params in one place
type RootStackParamList = {
  Home: undefined;
  UserDetail: { userId: string };
  Settings: { section?: string };
};

// Type-safe screen props
type UserDetailProps = NativeStackScreenProps<RootStackParamList, "UserDetail">;

function UserDetailScreen({ route, navigation }: UserDetailProps): React.JSX.Element {
  const { userId } = route.params; // type-safe
  // ...
}
```

### Use typed `useNavigation` and `useRoute`

```typescript
import { useNavigation, useRoute } from "@react-navigation/native";
import type { NativeStackNavigationProp } from "@react-navigation/native-stack";

// Create typed hook
function useAppNavigation() {
  return useNavigation<NativeStackNavigationProp<RootStackParamList>>();
}

// Usage — type-safe navigation
function UserCard({ userId }: { userId: string }): React.JSX.Element {
  const navigation = useAppNavigation();

  const handlePress = () => {
    navigation.navigate("UserDetail", { userId }); // params are type-checked
  };

  return <Pressable onPress={handlePress}>...</Pressable>;
}
```

### Deep linking configuration

```typescript
const linking = {
  prefixes: ["myapp://", "https://myapp.com"],
  config: {
    screens: {
      Home: "",
      UserDetail: "user/:userId",
      Settings: "settings",
    },
  },
};

<NavigationContainer linking={linking}>
  <Stack.Navigator>...</Stack.Navigator>
</NavigationContainer>
```

---

## State Management

### Use React's built-in state for local/component state

```typescript
const [isVisible, setIsVisible] = useState(false);
const [formData, setFormData] = useState<FormData>({ name: "", email: "" });
```

### Use React Context for small shared state (theme, auth, locale)

```typescript
interface AuthContext {
  user: User | null;
  signIn: (credentials: Credentials) => Promise<void>;
  signOut: () => Promise<void>;
}

const AuthContext = createContext<AuthContext | null>(null);

function useAuth(): AuthContext {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth must be used within AuthProvider");
  }
  return context;
}
```

### Use Zustand or Jotai for complex global state

```typescript
import { create } from "zustand";

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  total: () => number;
}

const useCartStore = create<CartStore>((set, get) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({ items: state.items.filter((i) => i.id !== id) })),
  total: () => get().items.reduce((sum, item) => sum + item.price, 0),
}));
```

### Use TanStack Query (React Query) for server state

```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

function useUser(id: string) {
  return useQuery({
    queryKey: ["user", id],
    queryFn: () => api.getUser(id),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: UpdateUserDto) => api.updateUser(data),
    onSuccess: (user) => {
      queryClient.setQueryData(["user", user.id], user);
    },
  });
}
```

---

## Styling

### Use `StyleSheet.create` — always

```typescript
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#FFFFFF",
  },
  row: {
    flexDirection: "row",
    alignItems: "center",
    paddingHorizontal: 16,
    paddingVertical: 12,
  },
});
```

### Use a theme/design token system

```typescript
const theme = {
  colors: {
    primary: "#007AFF",
    background: "#FFFFFF",
    text: "#1A1A1A",
    textSecondary: "#6B7280",
    border: "#E5E7EB",
    error: "#EF4444",
  },
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
  },
  typography: {
    heading: { fontSize: 24, fontWeight: "700" as const, lineHeight: 32 },
    body: { fontSize: 16, fontWeight: "400" as const, lineHeight: 24 },
    caption: { fontSize: 12, fontWeight: "400" as const, lineHeight: 16 },
  },
  borderRadius: {
    sm: 4,
    md: 8,
    lg: 16,
    full: 9999,
  },
} as const;
```

### Responsive sizing

```typescript
import { Dimensions, PixelRatio } from "react-native";

const { width: SCREEN_WIDTH } = Dimensions.get("window");

// Scale based on design width (e.g., 375 for iPhone design)
function scale(size: number): number {
  return PixelRatio.roundToNearestPixel((SCREEN_WIDTH / 375) * size);
}
```

---

## Security

### Never store sensitive data in AsyncStorage

```typescript
// WRONG — AsyncStorage is unencrypted
await AsyncStorage.setItem("authToken", token);

// CORRECT — use encrypted keychain/keystore
import * as Keychain from "react-native-keychain";

await Keychain.setGenericPassword("authToken", token);
const credentials = await Keychain.getGenericPassword();
```

### Validate all API responses

```typescript
import { z } from "zod";

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
});

async function fetchUser(id: string): Promise<User> {
  const response = await api.get(`/users/${id}`);
  return UserSchema.parse(response.data); // validates shape
}
```

### Pin SSL certificates for sensitive APIs

```typescript
// Use react-native-ssl-pinning or similar
import { fetch } from "react-native-ssl-pinning";

const response = await fetch(url, {
  method: "GET",
  sslPinning: {
    certs: ["server-cert"],
  },
});
```

### Disable debug features in production

```typescript
if (__DEV__) {
  // Development-only code (stripped in production builds)
}

// Never ship with:
// - console.log statements (use a logger that strips in prod)
// - React DevTools enabled
// - Flipper in production bundles
// - Hardcoded API keys or secrets
```

---

## Testing

### Component testing with React Native Testing Library

```typescript
import { render, screen, fireEvent } from "@testing-library/react-native";

describe("UserCard", () => {
  it("displays user name and handles press", () => {
    const onPress = jest.fn();
    render(<UserCard user={testUser} onPress={onPress} />);

    expect(screen.getByText("Alice")).toBeOnTheScreen();

    fireEvent.press(screen.getByText("Alice"));
    expect(onPress).toHaveBeenCalledWith(testUser.id);
  });
});
```

### Query by accessibility, not test IDs

```typescript
// CORRECT — query by role and text (what the user sees)
screen.getByRole("button", { name: "Submit" });
screen.getByText("Welcome back");
screen.getByLabelText("Email address");

// LAST RESORT — testID for elements without accessible text
screen.getByTestId("loading-spinner");
```

### Test hooks in isolation

```typescript
import { renderHook, waitFor } from "@testing-library/react-native";

describe("useUsers", () => {
  it("fetches and returns users", async () => {
    const { result } = renderHook(() => useUsers());

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.users).toHaveLength(3);
  });
});
```

---

## Project Structure

```
src/
├── app/                    # App entry, providers, navigation
│   ├── App.tsx
│   ├── navigation/
│   │   ├── RootNavigator.tsx
│   │   └── types.ts        # Route param types
│   └── providers/
│       └── AppProviders.tsx
├── components/
│   ├── ui/                 # Reusable primitives (Button, Card, Input)
│   └── features/           # Feature-specific components
├── hooks/                  # Shared custom hooks
├── screens/                # Screen components (one per route)
│   ├── home/
│   │   ├── HomeScreen.tsx
│   │   └── components/     # Screen-specific components
│   └── profile/
├── services/               # API clients, external service wrappers
├── stores/                 # Global state (Zustand stores)
├── theme/                  # Design tokens, theme config
├── types/                  # Shared TypeScript types
└── utils/                  # Pure utility functions
```

### Naming conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Component files | PascalCase | `UserCard.tsx` |
| Hook files | camelCase with `use` prefix | `useAuth.ts` |
| Utility files | camelCase | `formatDate.ts` |
| Type files | camelCase | `user.types.ts` |
| Test files | Match source + `.test` | `UserCard.test.tsx` |
| Screen components | PascalCase + `Screen` suffix | `HomeScreen.tsx` |
| Style constants | camelCase | `theme.ts`, `colors.ts` |

---

## Common Mistakes to Avoid

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Inline styles | New objects every render, no memoization | `StyleSheet.create` |
| Inline `renderItem` in FlatList | Re-mounts all items every render | Extract or `useCallback` |
| Missing `keyExtractor` | Falls back to index, causes bugs on reorder | Always provide stable key |
| `ScrollView` for long lists | Renders all items at once, OOM | Use `FlatList` or `FlashList` |
| JS-thread animations | Janky, drops frames | Use Reanimated (UI thread) |
| Untyped navigation | Runtime crashes from wrong params | Type all route params |
| `AsyncStorage` for secrets | Unencrypted plaintext on device | Use Keychain/Keystore |
| `console.log` in production | Performance overhead, leaks data | Strip or use proper logger |
| Ignoring safe areas | Content hidden under notch/home bar | Use `SafeAreaView` from safe-area-context |
| `any` in props | Disables type checking | Use proper interfaces |
| Single massive component | Hard to test, re-renders everything | Split into smaller components |
| `useEffect` for derived data | Unnecessary re-render cycle | Use `useMemo` |
| Fetching in `useEffect` without cleanup | Race conditions, memory leaks | Use React Query or AbortController |
| Default exports for components | Harder to refactor, inconsistent | Use named exports |

---

## Quick Reference

| Aspect | Recommended | Avoid |
|--------|-------------|-------|
| Components | Functional, small, typed props | Class components, `React.FC`, mega-components |
| Styling | `StyleSheet.create`, theme tokens | Inline styles, magic numbers |
| Lists | `FlatList`/`FlashList` with optimization props | `ScrollView` for dynamic lists |
| Animation | Reanimated (UI thread) | JS-driven `Animated`, `setTimeout` |
| Navigation | Typed React Navigation | Untyped routes, string-based params |
| State | Local → Context → Zustand/Jotai → React Query | Redux for everything, prop drilling |
| Security | Keychain, SSL pinning, input validation | AsyncStorage for tokens, trust API blindly |
| Testing | RNTL, query by accessibility | Enzyme, snapshot-only tests, testIDs everywhere |
| Platform | `Platform.select`, platform extensions | Massive `if/else` blocks, iOS-only code |
| Performance | `memo`, `useCallback`, `useMemo` | Re-creating objects/functions every render |
