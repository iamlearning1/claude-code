# Dashboard Package - CLAUDE.md

This file provides guidance specifically for the Next.js frontend dashboard.

## Package Commands

```bash
yarn lint:translations  # Lint translation files
```

## Dashboard Architecture

### Frontend (Next.js)

- **Routing**: App Router with internationalization support (en/es)
- **State Management**: Apollo Client for GraphQL
- **UI Framework**: Chakra UI with Tailwind CSS
- **Authentication**: Firebase Auth integration
- **Form Handling**: React Hook Form

## Directory Structure

```
packages/dashboard/
├── app/[locale]/           # Internationalized routes
│   ├── layout.tsx         # Root layout with providers
│   ├── page.tsx          # Landing page
│   ├── login/            # Public routes
│   ├── home/             # Protected routes (with sidebar)
│   │   ├── layout.tsx    # Layout with sidebar
│   │   ├── projects/     # Projects section
│   │   └── users/        # Users section
├── components/           # Reusable components
│   ├── layout/          # Layout components (Sidebar, QueryResult)
│   └── ui/              # UI components (Loader, LocaleSwitcher)
├── hooks/               # Custom React hooks
│   ├── auth/           # Authentication hooks
│   ├── project/        # Project-related hooks
│   ├── crew/           # Crew-related hooks
│   └── user/           # User-related hooks
├── libs/               # External library configurations
│   ├── apollo/         # Apollo Client setup
│   └── firebase/       # Firebase configuration
├── i18n/               # Internationalization
│   ├── en.json        # English translations
│   ├── es.json        # Spanish translations
│   └── routing.ts     # i18n routing config
└── utils/             # Utility functions
```

## Creating a New Page

1. **Create page file** in `app/[locale]/home/[section]/page.tsx`:

```tsx
"use client";

import { QueryResult } from "@/components/layout/QueryResult";
import { useGetData } from "@/hooks/[feature]/useGetData";
import { Box, Text } from "@chakra-ui/react";
import { useTranslations } from "next-intl";

export default function NewPage() {
  const t = useTranslations();
  const { data, loading } = useGetData();

  return (
    <QueryResult loading={loading}>
      <Box bg="white" px={46} py={50}>
        <Text>{t("Page Title")}</Text>
        {/* Page content */}
      </Box>
    </QueryResult>
  );
}
```

2. **Add translations** to `i18n/en.json` and `i18n/es.json`

3. **Protected routes** go under `/home/` to inherit the sidebar layout

## Creating Custom Hooks

GraphQL hooks follow a consistent pattern:

1. **Query Hook** (`hooks/[feature]/useGet[Feature].ts`):

```tsx
import { gql, useQuery } from "@apollo/client";
import { useToast } from "@chakra-ui/react";
import { FeatureType } from "./feature.types";

export const getFeatureQuery = gql`
  query getFeature($id: String!) {
    data: getFeature(id: $id) {
      id
      name
      # other fields
    }
  }
`;

export const useGetFeature = (id: string) => {
  const toast = useToast();

  const { data, loading } = useQuery<{ data: FeatureType }>(getFeatureQuery, {
    variables: { id },
    onError(error) {
      toast({ title: error.message, status: "error" });
    },
  });

  return {
    feature: data?.data,
    loading,
  };
};
```

2. **Mutation Hook** (`hooks/[feature]/useCreate[Feature].ts`):

```tsx
import { useRouter } from "@/i18n/routing";
import { gql, useMutation } from "@apollo/client";
import { useToast } from "@chakra-ui/react";

const createFeatureMutation = gql`
  mutation createFeature($input: CreateFeatureInput!) {
    data: createFeature(input: $input)
  }
`;

export const useCreateFeature = () => {
  const toast = useToast();
  const router = useRouter();

  const [createFeature, { loading }] = useMutation(createFeatureMutation, {
    onCompleted() {
      toast({ title: "Feature created", status: "success" });
      router.push("/home/features");
    },
    onError(error) {
      toast({ title: error.message, status: "error" });
    },
    refetchQueries: [
      /* queries to refetch */
    ],
  });

  return { createFeature, loading };
};
```

## UI Components and Patterns

1. **Use Chakra UI components** with custom theme colors:

   - Green: `green.500` (primary), `green.100` (hover)
   - Gray: `gray.100-600` (various shades)
   - White: `white.500` (background), `white.100` (pure white)

2. **QueryResult wrapper** handles loading and empty states:

```tsx
<QueryResult loading={loading} empty={!data} message="No data found">
  {/* Your content */}
</QueryResult>
```

3. **Form handling** with React Hook Form:

```tsx
import { useForm } from "react-hook-form";

const {
  register,
  handleSubmit,
  formState: { errors },
} = useForm();
```

4. **Internationalization** with next-intl:
   - Use `useTranslations()` hook in components
   - Access translations with `t("key")`
   - Navigation with `Link` from `@/i18n/routing`

## Common Patterns

1. **Protected routes**: Place under `/home/` directory
2. **Public routes**: Place at root level (login, forgot-password, etc.)
3. **Dynamic routes**: Use `[param]` folders (e.g., `[projectId]`)
4. **API communication**: Always use Apollo hooks, never direct fetch
5. **Error handling**: Toast notifications via Chakra UI
6. **Loading states**: Use QueryResult or Chakra Spinner/Skeleton
7. **Navigation**: Use typed router from `@/i18n/routing`
8. Never use `useMemo` or `useCallback` in the component body or render function.
9. Make sure that all the redirection and refetch logic is handled in the hooks, not in the components.
