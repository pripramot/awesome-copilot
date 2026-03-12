---
name: flutter-ui-expert
description: 'Build beautiful, responsive Flutter mobile and web UIs with Material 3 design, custom widgets, animations, state management patterns, and Thai-language support. Includes common UI patterns for e-commerce, booking, and business applications.'
---

# Flutter UI Expert (Flutter UI มืออาชีพ)

Create stunning Flutter UIs with best practices, reusable components, and Thai-language support.

## Project Setup

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  # State management
  riverpod: ^2.5.1
  flutter_riverpod: ^2.5.1
  # UI
  google_fonts: ^6.2.1
  flutter_animate: ^4.5.0
  cached_network_image: ^3.3.1
  # Navigation
  go_router: ^13.2.0
  # i18n
  flutter_localizations:
    sdk: flutter
  intl: ^0.19.0
```

## Thai Font Setup

```dart
// main.dart - Thai font configuration
import 'package:google_fonts/google_fonts.dart';

MaterialApp(
  theme: ThemeData(
    textTheme: GoogleFonts.sarabunTextTheme(
      ThemeData.light().textTheme,
    ),
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF1A73E8),
    ),
    useMaterial3: true,
  ),
  localizationsDelegates: const [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: const [
    Locale('th', 'TH'),
    Locale('en', 'US'),
  ],
  locale: const Locale('th', 'TH'),
)
```

## Reusable UI Components

### Thai-Optimized Card

```dart
class ThaiProductCard extends StatelessWidget {
  const ThaiProductCard({
    super.key,
    required this.name,
    required this.price,
    required this.imageUrl,
    this.onTap,
  });

  final String name;
  final double price;
  final String imageUrl;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: onTap,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            AspectRatio(
              aspectRatio: 16 / 9,
              child: CachedNetworkImage(
                imageUrl: imageUrl,
                fit: BoxFit.cover,
                placeholder: (ctx, url) => const Center(
                  child: CircularProgressIndicator(),
                ),
                errorWidget: (ctx, url, err) => const Icon(Icons.broken_image),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(12.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    name,
                    style: theme.textTheme.titleMedium?.copyWith(
                      fontWeight: FontWeight.bold,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '฿${price.toStringAsFixed(2)}',
                    style: theme.textTheme.titleLarge?.copyWith(
                      color: theme.colorScheme.primary,
                      fontWeight: FontWeight.w700,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Animated Loading State

```dart
class SkeletonLoader extends StatelessWidget {
  const SkeletonLoader({super.key, required this.child});
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return child
        .animate(onPlay: (controller) => controller.repeat())
        .shimmer(
          duration: const Duration(seconds: 1),
          color: Colors.grey.shade300,
        );
  }
}
```

### Thai Date Picker

```dart
Future<DateTime?> showThaiDatePicker(BuildContext context) async {
  return showDatePicker(
    context: context,
    initialDate: DateTime.now(),
    firstDate: DateTime(1900),
    lastDate: DateTime(2100),
    locale: const Locale('th', 'TH'),
    builder: (context, child) => Theme(
      data: Theme.of(context).copyWith(
        textTheme: GoogleFonts.sarabunTextTheme(
          Theme.of(context).textTheme,
        ),
      ),
      child: child!,
    ),
  );
}
```

## State Management with Riverpod

```dart
// providers/cart_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'cart_provider.g.dart';

@riverpod
class CartNotifier extends _$CartNotifier {
  @override
  List<CartItem> build() => [];

  void addItem(Product product, int quantity) {
    final existing = state.indexWhere((i) => i.product.id == product.id);
    if (existing >= 0) {
      state = [
        ...state.sublist(0, existing),
        state[existing].copyWith(quantity: state[existing].quantity + quantity),
        ...state.sublist(existing + 1),
      ];
    } else {
      state = [...state, CartItem(product: product, quantity: quantity)];
    }
  }

  void removeItem(String productId) {
    state = state.where((i) => i.product.id != productId).toList();
  }

  double get total =>
      state.fold(0, (sum, item) => sum + item.product.price * item.quantity);
}
```

## Navigation with GoRouter

```dart
// router.dart
final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/products/:id',
      builder: (context, state) => ProductDetailScreen(
        productId: state.pathParameters['id']!,
      ),
    ),
    GoRoute(
      path: '/cart',
      builder: (context, state) => const CartScreen(),
    ),
  ],
);
```

## Responsive Layout

```dart
class ResponsiveLayout extends StatelessWidget {
  const ResponsiveLayout({
    super.key,
    required this.mobile,
    required this.tablet,
    required this.desktop,
  });

  final Widget mobile;
  final Widget tablet;
  final Widget desktop;

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.of(context).size.width;
    if (width >= 1200) return desktop;
    if (width >= 600) return tablet;
    return mobile;
  }
}
```

## Best Practices

1. **Use const constructors** wherever possible for performance
2. **Extract widgets** - Break complex UIs into small, focused widget classes
3. **Sarabun font** - Best Google Font for Thai UI with good readability
4. **Thai locale** - Always configure `th_TH` locale for date/number formatting
5. **Riverpod** - Preferred state management for testability and scalability
6. **GoRouter** - Use for declarative navigation with deep linking support
7. **Cached images** - Use `cached_network_image` to improve performance
