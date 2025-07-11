# Bug Report: useTheme must be used within a ThemeProvider

## Summary
The application crashes with a runtime error when trying to use the theme toggle functionality. The error occurs because the `ThemeProvider` is not properly wrapped around the application in the root layout.

## Error Details
```
Error: useTheme must be used within a ThemeProvider

src/lib/theme-context.tsx (84:11) @ useTheme

  82 |   const context = useContext(ThemeContext);
  83 |   if (context === undefined) {
> 84 |     throw new Error('useTheme must be used within a ThemeProvider');
     |           ^
  85 |   }
  86 |   return context;
  87 | }
```

## Call Stack
1. `useTheme` - src/lib/theme-context.tsx (84:11)
2. `ThemeToggle` - src/components/ui/theme-toggle.tsx (9:41)
3. `Navigation` - src/components/ui/navigation.tsx (21:13)
4. `Home` - src/app/page.tsx (9:7)

## Root Cause
The `ThemeProvider` component is defined in `src/lib/theme-context.tsx` but is not included in the root layout (`src/app/layout.tsx`). The `ThemeToggle` component attempts to use the `useTheme` hook, but since it's not wrapped within a `ThemeProvider`, the context is undefined, causing the error.

## Current Code Structure

### Theme Context (src/lib/theme-context.tsx)
```typescript
export function ThemeProvider({ children }: { children: ReactNode }) {
  // ... provider implementation
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

### Root Layout (src/app/layout.tsx)
```typescript
export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className={`${poppins.variable} ${geistMono.variable} antialiased`}>
        <StagewiseToolbar
          config={{
            plugins: [ReactPlugin],
          }}
        />
        {children}  // ❌ No ThemeProvider wrapper
      </body>
    </html>
  );
}
```

### Component Usage Chain
- `Home` page → `Navigation` component → `ThemeToggle` component → `useTheme` hook ❌

## Expected Behavior
The theme toggle should work without errors, allowing users to switch between light and dark modes.

## Actual Behavior
The application crashes with a runtime error when the theme toggle is rendered.

## Proposed Solution
Wrap the `children` in the root layout with the `ThemeProvider`:

```typescript
import { ThemeProvider } from '@/lib/theme-context';

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className={`${poppins.variable} ${geistMono.variable} antialiased`}>
        <StagewiseToolbar
          config={{
            plugins: [ReactPlugin],
          }}
        />
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

## Environment
- Next.js application
- React with TypeScript
- Client-side theme context implementation

## Files Involved
- `src/lib/theme-context.tsx` - Theme context and provider
- `src/app/layout.tsx` - Root layout (needs modification)
- `src/components/ui/theme-toggle.tsx` - Component using useTheme
- `src/components/ui/navigation.tsx` - Component containing ThemeToggle
- `src/app/page.tsx` - Home page using Navigation

## Priority
**High** - This is a blocking issue that prevents the theme toggle functionality from working at all.

## Steps to Reproduce
1. Load the application
2. The error occurs immediately when the Navigation component renders
3. The theme toggle is not functional

## Additional Context
This appears to be a setup issue where the theme context provider was implemented but not properly integrated into the application's component tree at the root level.