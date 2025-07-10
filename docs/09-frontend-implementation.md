# 9. Implementazione Frontend

## 9.1 Architettura Frontend

### 9.1.1 Tecnologie Scelte

- **Framework**: React 18 con TypeScript
- **Routing**: React Router v6
- **State Management**: Redux Toolkit + RTK Query
- **UI Framework**: Material-UI (MUI) v5
- **Form Management**: React Hook Form + Yup
- **HTTP Client**: Axios (integrato con RTK Query)
- **Build Tool**: Vite
- **Testing**: Jest + React Testing Library
- **Styling**: Emotion (CSS-in-JS) + MUI Theme

### 9.1.2 Struttura del Progetto

```
frontend/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── manifest.json
├── src/
│   ├── components/           # Componenti riutilizzabili
│   │   ├── common/          # Componenti comuni
│   │   ├── forms/           # Componenti per form
│   │   ├── layout/          # Layout components
│   │   └── ui/              # UI components base
│   ├── pages/               # Pagine dell'applicazione
│   │   ├── auth/           # Pagine di autenticazione
│   │   ├── games/          # Pagine dei giochi
│   │   ├── profile/        # Pagine del profilo
│   │   └── admin/          # Pagine amministrative
│   ├── hooks/               # Custom hooks
│   ├── services/            # API services
│   ├── store/               # Redux store
│   │   ├── slices/         # Redux slices
│   │   └── api/            # RTK Query APIs
│   ├── types/               # TypeScript types
│   ├── utils/               # Utility functions
│   ├── constants/           # Costanti
│   ├── theme/               # Tema MUI
│   ├── App.tsx
│   ├── index.tsx
│   └── setupTests.ts
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

## 9.2 Setup Iniziale

### 9.2.1 package.json

```json
{
  "name": "videogiochi-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint src --ext ts,tsx --fix",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.15.0",
    "@reduxjs/toolkit": "^1.9.5",
    "react-redux": "^8.1.2",
    "@mui/material": "^5.14.5",
    "@mui/icons-material": "^5.14.3",
    "@mui/x-data-grid": "^6.10.1",
    "@mui/x-date-pickers": "^6.10.1",
    "@emotion/react": "^11.11.1",
    "@emotion/styled": "^11.11.0",
    "react-hook-form": "^7.45.4",
    "@hookform/resolvers": "^3.2.0",
    "yup": "^1.2.0",
    "axios": "^1.5.0",
    "date-fns": "^2.30.0",
    "react-hot-toast": "^2.4.1",
    "react-helmet-async": "^1.3.0",
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7",
    "@types/lodash": "^4.14.195",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.3",
    "eslint": "^8.45.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.3",
    "typescript": "^5.0.2",
    "vite": "^4.4.5",
    "jest": "^29.6.2",
    "@testing-library/react": "^13.4.0",
    "@testing-library/jest-dom": "^5.17.0",
    "@testing-library/user-event": "^14.4.3",
    "jest-environment-jsdom": "^29.6.2"
  }
}
```

### 9.2.2 vite.config.ts

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@pages': path.resolve(__dirname, './src/pages'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@services': path.resolve(__dirname, './src/services'),
      '@store': path.resolve(__dirname, './src/store'),
      '@types': path.resolve(__dirname, './src/types'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@constants': path.resolve(__dirname, './src/constants'),
      '@theme': path.resolve(__dirname, './src/theme')
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        secure: false
      }
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          mui: ['@mui/material', '@mui/icons-material'],
          redux: ['@reduxjs/toolkit', 'react-redux']
        }
      }
    }
  }
})
```

### 9.2.3 tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@pages/*": ["./src/pages/*"],
      "@hooks/*": ["./src/hooks/*"],
      "@services/*": ["./src/services/*"],
      "@store/*": ["./src/store/*"],
      "@types/*": ["./src/types/*"],
      "@utils/*": ["./src/utils/*"],
      "@constants/*": ["./src/constants/*"],
      "@theme/*": ["./src/theme/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## 9.3 Tipi TypeScript

### 9.3.1 types/api.ts

```typescript
export interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  message?: string;
  errors?: Record<string, string[]>;
}

export interface PagedResponse<T> {
  content: T[];
  totalElements: number;
  totalPages: number;
  size: number;
  number: number;
  first: boolean;
  last: boolean;
}

export interface PaginationParams {
  page?: number;
  size?: number;
  sort?: string;
  direction?: 'asc' | 'desc';
}

export interface SearchParams extends PaginationParams {
  query?: string;
}
```

### 9.3.2 types/game.ts

```typescript
export enum EsrbRating {
  EARLY_CHILDHOOD = 'EARLY_CHILDHOOD',
  EVERYONE = 'EVERYONE',
  EVERYONE_10_PLUS = 'EVERYONE_10_PLUS',
  TEEN = 'TEEN',
  MATURE_17_PLUS = 'MATURE_17_PLUS',
  ADULTS_ONLY_18_PLUS = 'ADULTS_ONLY_18_PLUS',
  RATING_PENDING = 'RATING_PENDING'
}

export interface Developer {
  id: number;
  name: string;
  foundedYear?: number;
  country?: string;
  website?: string;
}

export interface Publisher {
  id: number;
  name: string;
  foundedYear?: number;
  country?: string;
  website?: string;
}

export interface Genre {
  id: number;
  name: string;
  description?: string;
}

export interface Platform {
  id: number;
  name: string;
  manufacturer?: string;
  releaseYear?: number;
}

export interface Game {
  id: number;
  title: string;
  description?: string;
  releaseDate: string;
  esrbRating: EsrbRating;
  price: number;
  developer: Developer;
  publisher: Publisher;
  genres: Genre[];
  platforms: Platform[];
  imageUrl?: string;
  trailerUrl?: string;
  metacriticScore?: number;
  userRating?: number;
  createdAt: string;
  updatedAt: string;
}

export interface GameSummary {
  id: number;
  title: string;
  releaseDate: string;
  esrbRating: EsrbRating;
  price: number;
  developer: string;
  publisher: string;
  imageUrl?: string;
  userRating?: number;
}

export interface GameCreateRequest {
  title: string;
  description?: string;
  releaseDate: string;
  esrbRating: EsrbRating;
  price: number;
  developerId: number;
  publisherId: number;
  genreIds: number[];
  platformIds: number[];
  imageUrl?: string;
  trailerUrl?: string;
}

export interface GameUpdateRequest extends Partial<GameCreateRequest> {
  id: number;
}

export interface GameSearchCriteria {
  title?: string;
  developerId?: number;
  publisherId?: number;
  genreIds?: number[];
  platformIds?: number[];
  esrbRating?: EsrbRating;
  minPrice?: number;
  maxPrice?: number;
  releaseDateFrom?: string;
  releaseDateTo?: string;
  minRating?: number;
}
```

### 9.3.3 types/user.ts

```typescript
export enum UserRole {
  USER = 'USER',
  ADMIN = 'ADMIN',
  MODERATOR = 'MODERATOR'
}

export enum ListType {
  PLAYED = 'PLAYED',
  WANT_TO_PLAY = 'WANT_TO_PLAY',
  CURRENTLY_PLAYING = 'CURRENTLY_PLAYING',
  FAVORITES = 'FAVORITES'
}

export interface User {
  id: number;
  username: string;
  email: string;
  firstName?: string;
  lastName?: string;
  dateOfBirth?: string;
  country?: string;
  bio?: string;
  avatarUrl?: string;
  role: UserRole;
  emailVerified: boolean;
  active: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface UserProfile extends User {
  gamesCount: number;
  listsCount: number;
  reviewsCount: number;
}

export interface UserGameList {
  id: number;
  user: User;
  game: GameSummary;
  listType: ListType;
  rating?: number;
  review?: string;
  playedHours?: number;
  completionPercentage?: number;
  addedAt: string;
  updatedAt: string;
}

export interface UserRegistrationRequest {
  username: string;
  email: string;
  password: string;
  firstName?: string;
  lastName?: string;
  dateOfBirth?: string;
  country?: string;
}

export interface UserUpdateRequest {
  firstName?: string;
  lastName?: string;
  dateOfBirth?: string;
  country?: string;
  bio?: string;
  avatarUrl?: string;
}

export interface PasswordChangeRequest {
  currentPassword: string;
  newPassword: string;
  confirmPassword: string;
}
```

### 9.3.4 types/auth.ts

```typescript
export interface LoginRequest {
  username: string;
  password: string;
  rememberMe?: boolean;
}

export interface LoginResponse {
  token: string;
  refreshToken: string;
  user: User;
  expiresIn: number;
}

export interface RefreshTokenRequest {
  refreshToken: string;
}

export interface ForgotPasswordRequest {
  email: string;
}

export interface ResetPasswordRequest {
  token: string;
  newPassword: string;
  confirmPassword: string;
}

export interface EmailVerificationRequest {
  token: string;
}

export interface AuthState {
  user: User | null;
  token: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}
```

## 9.4 Redux Store

### 9.4.1 store/index.ts

```typescript
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import authSlice from './slices/authSlice';
import uiSlice from './slices/uiSlice';
import { gameApi } from './api/gameApi';
import { userApi } from './api/userApi';
import { authApi } from './api/authApi';

export const store = configureStore({
  reducer: {
    auth: authSlice,
    ui: uiSlice,
    [gameApi.reducerPath]: gameApi.reducer,
    [userApi.reducerPath]: userApi.reducer,
    [authApi.reducerPath]: authApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: [
          'persist/PERSIST',
          'persist/REHYDRATE',
          'persist/PAUSE',
          'persist/PURGE',
          'persist/REGISTER',
        ],
      },
    })
      .concat(gameApi.middleware)
      .concat(userApi.middleware)
      .concat(authApi.middleware),
});

setupListeners(store.dispatch);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### 9.4.2 store/slices/authSlice.ts

```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { User, AuthState } from '@types/auth';

const initialState: AuthState = {
  user: null,
  token: localStorage.getItem('token'),
  refreshToken: localStorage.getItem('refreshToken'),
  isAuthenticated: !!localStorage.getItem('token'),
  isLoading: false,
  error: null,
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    loginStart: (state) => {
      state.isLoading = true;
      state.error = null;
    },
    loginSuccess: (state, action: PayloadAction<{ user: User; token: string; refreshToken: string }>) => {
      state.isLoading = false;
      state.user = action.payload.user;
      state.token = action.payload.token;
      state.refreshToken = action.payload.refreshToken;
      state.isAuthenticated = true;
      state.error = null;
      
      localStorage.setItem('token', action.payload.token);
      localStorage.setItem('refreshToken', action.payload.refreshToken);
    },
    loginFailure: (state, action: PayloadAction<string>) => {
      state.isLoading = false;
      state.error = action.payload;
      state.isAuthenticated = false;
    },
    logout: (state) => {
      state.user = null;
      state.token = null;
      state.refreshToken = null;
      state.isAuthenticated = false;
      state.error = null;
      
      localStorage.removeItem('token');
      localStorage.removeItem('refreshToken');
    },
    updateUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    },
    refreshTokenSuccess: (state, action: PayloadAction<{ token: string; refreshToken: string }>) => {
      state.token = action.payload.token;
      state.refreshToken = action.payload.refreshToken;
      
      localStorage.setItem('token', action.payload.token);
      localStorage.setItem('refreshToken', action.payload.refreshToken);
    },
  },
});

export const {
  loginStart,
  loginSuccess,
  loginFailure,
  logout,
  updateUser,
  clearError,
  refreshTokenSuccess,
} = authSlice.actions;

export default authSlice.reducer;
```

### 9.4.3 store/slices/uiSlice.ts

```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface UiState {
  sidebarOpen: boolean;
  theme: 'light' | 'dark';
  loading: boolean;
  notifications: Notification[];
}

interface Notification {
  id: string;
  type: 'success' | 'error' | 'warning' | 'info';
  message: string;
  autoHide?: boolean;
  duration?: number;
}

const initialState: UiState = {
  sidebarOpen: false,
  theme: (localStorage.getItem('theme') as 'light' | 'dark') || 'light',
  loading: false,
  notifications: [],
};

const uiSlice = createSlice({
  name: 'ui',
  initialState,
  reducers: {
    toggleSidebar: (state) => {
      state.sidebarOpen = !state.sidebarOpen;
    },
    setSidebarOpen: (state, action: PayloadAction<boolean>) => {
      state.sidebarOpen = action.payload;
    },
    toggleTheme: (state) => {
      state.theme = state.theme === 'light' ? 'dark' : 'light';
      localStorage.setItem('theme', state.theme);
    },
    setTheme: (state, action: PayloadAction<'light' | 'dark'>) => {
      state.theme = action.payload;
      localStorage.setItem('theme', action.payload);
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    addNotification: (state, action: PayloadAction<Omit<Notification, 'id'>>) => {
      const notification: Notification = {
        ...action.payload,
        id: Date.now().toString(),
      };
      state.notifications.push(notification);
    },
    removeNotification: (state, action: PayloadAction<string>) => {
      state.notifications = state.notifications.filter(n => n.id !== action.payload);
    },
    clearNotifications: (state) => {
      state.notifications = [];
    },
  },
});

export const {
  toggleSidebar,
  setSidebarOpen,
  toggleTheme,
  setTheme,
  setLoading,
  addNotification,
  removeNotification,
  clearNotifications,
} = uiSlice.actions;

export default uiSlice.reducer;
```

## 9.5 API Services

### 9.5.1 store/api/baseApi.ts

```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { RootState } from '@store/index';
import { logout, refreshTokenSuccess } from '@store/slices/authSlice';
import { ApiResponse } from '@types/api';

const baseQuery = fetchBaseQuery({
  baseUrl: '/api/v1',
  prepareHeaders: (headers, { getState }) => {
    const token = (getState() as RootState).auth.token;
    if (token) {
      headers.set('authorization', `Bearer ${token}`);
    }
    headers.set('content-type', 'application/json');
    return headers;
  },
});

const baseQueryWithReauth = async (args: any, api: any, extraOptions: any) => {
  let result = await baseQuery(args, api, extraOptions);
  
  if (result.error && result.error.status === 401) {
    // Tenta il refresh del token
    const refreshToken = (api.getState() as RootState).auth.refreshToken;
    
    if (refreshToken) {
      const refreshResult = await baseQuery(
        {
          url: '/auth/refresh',
          method: 'POST',
          body: { refreshToken },
        },
        api,
        extraOptions
      );
      
      if (refreshResult.data) {
        const { token, refreshToken: newRefreshToken } = (refreshResult.data as ApiResponse).data;
        
        api.dispatch(refreshTokenSuccess({ token, refreshToken: newRefreshToken }));
        
        // Riprova la richiesta originale
        result = await baseQuery(args, api, extraOptions);
      } else {
        // Refresh fallito, logout
        api.dispatch(logout());
      }
    } else {
      // Nessun refresh token, logout
      api.dispatch(logout());
    }
  }
  
  return result;
};

export const baseApi = createApi({
  reducerPath: 'api',
  baseQuery: baseQueryWithReauth,
  tagTypes: ['Game', 'User', 'Genre', 'Platform', 'Developer', 'Publisher'],
  endpoints: () => ({}),
});
```

### 9.5.2 store/api/gameApi.ts

```typescript
import { baseApi } from './baseApi';
import {
  Game,
  GameSummary,
  GameCreateRequest,
  GameUpdateRequest,
  GameSearchCriteria,
} from '@types/game';
import { ApiResponse, PagedResponse, PaginationParams } from '@types/api';

export const gameApi = baseApi.injectEndpoints({
  endpoints: (builder) => ({
    getGames: builder.query<PagedResponse<GameSummary>, PaginationParams>({
      query: (params) => ({
        url: '/games',
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getGameById: builder.query<Game, number>({
      query: (id) => `/games/${id}`,
      transformResponse: (response: ApiResponse<Game>) => response.data!,
      providesTags: (result, error, id) => [{ type: 'Game', id }],
    }),
    
    searchGames: builder.query<PagedResponse<GameSummary>, GameSearchCriteria & PaginationParams>({
      query: (params) => ({
        url: '/games/search',
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    searchGamesByTitle: builder.query<PagedResponse<GameSummary>, { title: string } & PaginationParams>({
      query: ({ title, ...params }) => ({
        url: '/games/search/title',
        params: { title, ...params },
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getGamesByDeveloper: builder.query<PagedResponse<GameSummary>, { developerId: number } & PaginationParams>({
      query: ({ developerId, ...params }) => ({
        url: `/games/search/developer/${developerId}`,
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getGamesByPublisher: builder.query<PagedResponse<GameSummary>, { publisherId: number } & PaginationParams>({
      query: ({ publisherId, ...params }) => ({
        url: `/games/search/publisher/${publisherId}`,
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getGamesByGenre: builder.query<PagedResponse<GameSummary>, { genreId: number } & PaginationParams>({
      query: ({ genreId, ...params }) => ({
        url: `/games/search/genre/${genreId}`,
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getGamesByYear: builder.query<PagedResponse<GameSummary>, { year: number } & PaginationParams>({
      query: ({ year, ...params }) => ({
        url: `/games/search/year/${year}`,
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getRecommendedGames: builder.query<PagedResponse<GameSummary>, PaginationParams>({
      query: (params) => ({
        url: '/games/recommendations',
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getPopularGames: builder.query<PagedResponse<GameSummary>, PaginationParams>({
      query: (params) => ({
        url: '/games/popular',
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    getRecentGames: builder.query<PagedResponse<GameSummary>, PaginationParams>({
      query: (params) => ({
        url: '/games/recent',
        params,
      }),
      transformResponse: (response: ApiResponse<PagedResponse<GameSummary>>) => response.data!,
      providesTags: ['Game'],
    }),
    
    createGame: builder.mutation<Game, GameCreateRequest>({
      query: (game) => ({
        url: '/games',
        method: 'POST',
        body: game,
      }),
      transformResponse: (response: ApiResponse<Game>) => response.data!,
      invalidatesTags: ['Game'],
    }),
    
    updateGame: builder.mutation<Game, GameUpdateRequest>({
      query: ({ id, ...game }) => ({
        url: `/games/${id}`,
        method: 'PUT',
        body: game,
      }),
      transformResponse: (response: ApiResponse<Game>) => response.data!,
      invalidatesTags: (result, error, { id }) => [{ type: 'Game', id }],
    }),
    
    deleteGame: builder.mutation<void, number>({
      query: (id) => ({
        url: `/games/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: (result, error, id) => [{ type: 'Game', id }],
    }),
    
    rateGame: builder.mutation<void, { gameId: number; rating: number }>({
      query: ({ gameId, rating }) => ({
        url: `/games/${gameId}/rate`,
        method: 'POST',
        body: { rating },
      }),
      invalidatesTags: (result, error, { gameId }) => [{ type: 'Game', id: gameId }],
    }),
  }),
});

export const {
  useGetGamesQuery,
  useGetGameByIdQuery,
  useSearchGamesQuery,
  useSearchGamesByTitleQuery,
  useGetGamesByDeveloperQuery,
  useGetGamesByPublisherQuery,
  useGetGamesByGenreQuery,
  useGetGamesByYearQuery,
  useGetRecommendedGamesQuery,
  useGetPopularGamesQuery,
  useGetRecentGamesQuery,
  useCreateGameMutation,
  useUpdateGameMutation,
  useDeleteGameMutation,
  useRateGameMutation,
} = gameApi;
```

Questo documento fornisce una base solida per l'implementazione del frontend, coprendo la configurazione iniziale, i tipi TypeScript, il Redux store e i servizi API. La struttura è modulare e scalabile, seguendo le best practices di React e TypeScript.