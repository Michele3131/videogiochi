# 10. Componenti e Pagine Frontend

## 10.1 Tema e Styling

### 10.1.1 theme/index.ts

```typescript
import { createTheme, ThemeOptions } from '@mui/material/styles';
import { PaletteMode } from '@mui/material';

const getDesignTokens = (mode: PaletteMode): ThemeOptions => ({
  palette: {
    mode,
    ...(mode === 'light'
      ? {
          // Palette per tema chiaro
          primary: {
            main: '#1976d2',
            light: '#42a5f5',
            dark: '#1565c0',
            contrastText: '#fff',
          },
          secondary: {
            main: '#dc004e',
            light: '#ff5983',
            dark: '#9a0036',
            contrastText: '#fff',
          },
          background: {
            default: '#fafafa',
            paper: '#fff',
          },
          text: {
            primary: '#212121',
            secondary: '#757575',
          },
        }
      : {
          // Palette per tema scuro
          primary: {
            main: '#90caf9',
            light: '#e3f2fd',
            dark: '#42a5f5',
            contrastText: '#000',
          },
          secondary: {
            main: '#f48fb1',
            light: '#ffc1e3',
            dark: '#bf5f82',
            contrastText: '#000',
          },
          background: {
            default: '#121212',
            paper: '#1e1e1e',
          },
          text: {
            primary: '#fff',
            secondary: '#aaa',
          },
        }),
  },
  typography: {
    fontFamily: '"Roboto", "Helvetica", "Arial", sans-serif',
    h1: {
      fontSize: '2.5rem',
      fontWeight: 600,
      lineHeight: 1.2,
    },
    h2: {
      fontSize: '2rem',
      fontWeight: 600,
      lineHeight: 1.3,
    },
    h3: {
      fontSize: '1.75rem',
      fontWeight: 600,
      lineHeight: 1.4,
    },
    h4: {
      fontSize: '1.5rem',
      fontWeight: 600,
      lineHeight: 1.4,
    },
    h5: {
      fontSize: '1.25rem',
      fontWeight: 600,
      lineHeight: 1.5,
    },
    h6: {
      fontSize: '1rem',
      fontWeight: 600,
      lineHeight: 1.6,
    },
    body1: {
      fontSize: '1rem',
      lineHeight: 1.6,
    },
    body2: {
      fontSize: '0.875rem',
      lineHeight: 1.6,
    },
  },
  shape: {
    borderRadius: 8,
  },
  spacing: 8,
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none',
          borderRadius: 8,
          fontWeight: 600,
        },
      },
    },
    MuiCard: {
      styleOverrides: {
        root: {
          boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
          borderRadius: 12,
        },
      },
    },
    MuiTextField: {
      styleOverrides: {
        root: {
          '& .MuiOutlinedInput-root': {
            borderRadius: 8,
          },
        },
      },
    },
  },
});

export const createAppTheme = (mode: PaletteMode) => createTheme(getDesignTokens(mode));
```

## 10.2 Componenti Common

### 10.2.1 components/common/LoadingSpinner.tsx

```typescript
import React from 'react';
import { Box, CircularProgress, Typography } from '@mui/material';

interface LoadingSpinnerProps {
  message?: string;
  size?: number;
  fullScreen?: boolean;
}

const LoadingSpinner: React.FC<LoadingSpinnerProps> = ({
  message = 'Caricamento...',
  size = 40,
  fullScreen = false,
}) => {
  const content = (
    <Box
      display="flex"
      flexDirection="column"
      alignItems="center"
      justifyContent="center"
      gap={2}
    >
      <CircularProgress size={size} />
      {message && (
        <Typography variant="body2" color="text.secondary">
          {message}
        </Typography>
      )}
    </Box>
  );

  if (fullScreen) {
    return (
      <Box
        position="fixed"
        top={0}
        left={0}
        right={0}
        bottom={0}
        display="flex"
        alignItems="center"
        justifyContent="center"
        bgcolor="background.default"
        zIndex={9999}
      >
        {content}
      </Box>
    );
  }

  return (
    <Box
      display="flex"
      alignItems="center"
      justifyContent="center"
      minHeight={200}
    >
      {content}
    </Box>
  );
};

export default LoadingSpinner;
```

### 10.2.2 components/common/ErrorMessage.tsx

```typescript
import React from 'react';
import {
  Alert,
  AlertTitle,
  Box,
  Button,
  Typography,
} from '@mui/material';
import { Refresh as RefreshIcon } from '@mui/icons-material';

interface ErrorMessageProps {
  title?: string;
  message: string;
  onRetry?: () => void;
  severity?: 'error' | 'warning' | 'info';
  fullWidth?: boolean;
}

const ErrorMessage: React.FC<ErrorMessageProps> = ({
  title = 'Errore',
  message,
  onRetry,
  severity = 'error',
  fullWidth = false,
}) => {
  return (
    <Box
      display="flex"
      flexDirection="column"
      alignItems="center"
      gap={2}
      width={fullWidth ? '100%' : 'auto'}
      maxWidth={600}
      mx="auto"
      p={2}
    >
      <Alert severity={severity} sx={{ width: '100%' }}>
        <AlertTitle>{title}</AlertTitle>
        <Typography variant="body2">{message}</Typography>
      </Alert>
      
      {onRetry && (
        <Button
          variant="outlined"
          startIcon={<RefreshIcon />}
          onClick={onRetry}
        >
          Riprova
        </Button>
      )}
    </Box>
  );
};

export default ErrorMessage;
```

### 10.2.3 components/common/ConfirmDialog.tsx

```typescript
import React from 'react';
import {
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  Button,
  Typography,
} from '@mui/material';

interface ConfirmDialogProps {
  open: boolean;
  title: string;
  message: string;
  confirmText?: string;
  cancelText?: string;
  onConfirm: () => void;
  onCancel: () => void;
  severity?: 'warning' | 'error' | 'info';
}

const ConfirmDialog: React.FC<ConfirmDialogProps> = ({
  open,
  title,
  message,
  confirmText = 'Conferma',
  cancelText = 'Annulla',
  onConfirm,
  onCancel,
  severity = 'warning',
}) => {
  const getConfirmButtonColor = () => {
    switch (severity) {
      case 'error':
        return 'error';
      case 'warning':
        return 'warning';
      default:
        return 'primary';
    }
  };

  return (
    <Dialog open={open} onClose={onCancel} maxWidth="sm" fullWidth>
      <DialogTitle>{title}</DialogTitle>
      <DialogContent>
        <Typography variant="body1">{message}</Typography>
      </DialogContent>
      <DialogActions>
        <Button onClick={onCancel} color="inherit">
          {cancelText}
        </Button>
        <Button
          onClick={onConfirm}
          color={getConfirmButtonColor()}
          variant="contained"
          autoFocus
        >
          {confirmText}
        </Button>
      </DialogActions>
    </Dialog>
  );
};

export default ConfirmDialog;
```

### 10.2.4 components/common/SearchBar.tsx

```typescript
import React, { useState, useCallback } from 'react';
import {
  TextField,
  InputAdornment,
  IconButton,
  Box,
} from '@mui/material';
import {
  Search as SearchIcon,
  Clear as ClearIcon,
} from '@mui/icons-material';
import { debounce } from 'lodash';

interface SearchBarProps {
  placeholder?: string;
  onSearch: (query: string) => void;
  initialValue?: string;
  debounceMs?: number;
  fullWidth?: boolean;
}

const SearchBar: React.FC<SearchBarProps> = ({
  placeholder = 'Cerca...',
  onSearch,
  initialValue = '',
  debounceMs = 300,
  fullWidth = false,
}) => {
  const [value, setValue] = useState(initialValue);

  const debouncedSearch = useCallback(
    debounce((query: string) => {
      onSearch(query);
    }, debounceMs),
    [onSearch, debounceMs]
  );

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = event.target.value;
    setValue(newValue);
    debouncedSearch(newValue);
  };

  const handleClear = () => {
    setValue('');
    onSearch('');
  };

  const handleKeyPress = (event: React.KeyboardEvent) => {
    if (event.key === 'Enter') {
      event.preventDefault();
      onSearch(value);
    }
  };

  return (
    <Box width={fullWidth ? '100%' : 'auto'}>
      <TextField
        fullWidth={fullWidth}
        value={value}
        onChange={handleChange}
        onKeyPress={handleKeyPress}
        placeholder={placeholder}
        variant="outlined"
        size="medium"
        InputProps={{
          startAdornment: (
            <InputAdornment position="start">
              <SearchIcon color="action" />
            </InputAdornment>
          ),
          endAdornment: value && (
            <InputAdornment position="end">
              <IconButton
                size="small"
                onClick={handleClear}
                edge="end"
              >
                <ClearIcon />
              </IconButton>
            </InputAdornment>
          ),
        }}
      />
    </Box>
  );
};

export default SearchBar;
```

## 10.3 Componenti Layout

### 10.3.1 components/layout/Header.tsx

```typescript
import React from 'react';
import {
  AppBar,
  Toolbar,
  Typography,
  IconButton,
  Box,
  Avatar,
  Menu,
  MenuItem,
  Divider,
  Switch,
  FormControlLabel,
} from '@mui/material';
import {
  Menu as MenuIcon,
  AccountCircle,
  Logout,
  Settings,
  DarkMode,
  LightMode,
} from '@mui/icons-material';
import { useAppDispatch, useAppSelector } from '@hooks/redux';
import { toggleSidebar, toggleTheme } from '@store/slices/uiSlice';
import { logout } from '@store/slices/authSlice';
import { useNavigate } from 'react-router-dom';

const Header: React.FC = () => {
  const dispatch = useAppDispatch();
  const navigate = useNavigate();
  const { user, isAuthenticated } = useAppSelector((state) => state.auth);
  const { theme } = useAppSelector((state) => state.ui);
  
  const [anchorEl, setAnchorEl] = React.useState<null | HTMLElement>(null);
  const open = Boolean(anchorEl);

  const handleMenuOpen = (event: React.MouseEvent<HTMLElement>) => {
    setAnchorEl(event.currentTarget);
  };

  const handleMenuClose = () => {
    setAnchorEl(null);
  };

  const handleProfile = () => {
    handleMenuClose();
    navigate('/profile');
  };

  const handleSettings = () => {
    handleMenuClose();
    navigate('/settings');
  };

  const handleLogout = () => {
    handleMenuClose();
    dispatch(logout());
    navigate('/');
  };

  const handleThemeToggle = () => {
    dispatch(toggleTheme());
  };

  const handleSidebarToggle = () => {
    dispatch(toggleSidebar());
  };

  return (
    <AppBar position="fixed" sx={{ zIndex: (theme) => theme.zIndex.drawer + 1 }}>
      <Toolbar>
        <IconButton
          color="inherit"
          edge="start"
          onClick={handleSidebarToggle}
          sx={{ mr: 2 }}
        >
          <MenuIcon />
        </IconButton>
        
        <Typography
          variant="h6"
          component="div"
          sx={{ flexGrow: 1, cursor: 'pointer' }}
          onClick={() => navigate('/')}
        >
          VideoGiochi Library
        </Typography>

        <Box display="flex" alignItems="center" gap={1}>
          <FormControlLabel
            control={
              <Switch
                checked={theme === 'dark'}
                onChange={handleThemeToggle}
                icon={<LightMode />}
                checkedIcon={<DarkMode />}
              />
            }
            label=""
          />

          {isAuthenticated ? (
            <>
              <IconButton
                color="inherit"
                onClick={handleMenuOpen}
                size="large"
              >
                {user?.avatarUrl ? (
                  <Avatar
                    src={user.avatarUrl}
                    alt={user.username}
                    sx={{ width: 32, height: 32 }}
                  />
                ) : (
                  <AccountCircle />
                )}
              </IconButton>
              
              <Menu
                anchorEl={anchorEl}
                open={open}
                onClose={handleMenuClose}
                onClick={handleMenuClose}
                PaperProps={{
                  elevation: 0,
                  sx: {
                    overflow: 'visible',
                    filter: 'drop-shadow(0px 2px 8px rgba(0,0,0,0.32))',
                    mt: 1.5,
                    '& .MuiAvatar-root': {
                      width: 32,
                      height: 32,
                      ml: -0.5,
                      mr: 1,
                    },
                  },
                }}
                transformOrigin={{ horizontal: 'right', vertical: 'top' }}
                anchorOrigin={{ horizontal: 'right', vertical: 'bottom' }}
              >
                <MenuItem onClick={handleProfile}>
                  <Avatar /> Profilo
                </MenuItem>
                <MenuItem onClick={handleSettings}>
                  <Settings fontSize="small" sx={{ mr: 2 }} />
                  Impostazioni
                </MenuItem>
                <Divider />
                <MenuItem onClick={handleLogout}>
                  <Logout fontSize="small" sx={{ mr: 2 }} />
                  Logout
                </MenuItem>
              </Menu>
            </>
          ) : (
            <IconButton
              color="inherit"
              onClick={() => navigate('/login')}
            >
              <AccountCircle />
            </IconButton>
          )}
        </Box>
      </Toolbar>
    </AppBar>
  );
};

export default Header;
```

### 10.3.2 components/layout/Sidebar.tsx

```typescript
import React from 'react';
import {
  Drawer,
  List,
  ListItem,
  ListItemButton,
  ListItemIcon,
  ListItemText,
  Toolbar,
  Divider,
  Collapse,
  Box,
} from '@mui/material';
import {
  Home,
  Gamepad,
  Star,
  PlayArrow,
  Pause,
  CheckCircle,
  Search,
  TrendingUp,
  NewReleases,
  AdminPanelSettings,
  ExpandLess,
  ExpandMore,
} from '@mui/icons-material';
import { useNavigate, useLocation } from 'react-router-dom';
import { useAppDispatch, useAppSelector } from '@hooks/redux';
import { setSidebarOpen } from '@store/slices/uiSlice';
import { UserRole } from '@types/user';

const DRAWER_WIDTH = 240;

interface MenuItem {
  text: string;
  icon: React.ReactElement;
  path?: string;
  children?: MenuItem[];
  roles?: UserRole[];
}

const menuItems: MenuItem[] = [
  {
    text: 'Home',
    icon: <Home />,
    path: '/',
  },
  {
    text: 'Esplora',
    icon: <Search />,
    children: [
      {
        text: 'Tutti i Giochi',
        icon: <Gamepad />,
        path: '/games',
      },
      {
        text: 'Popolari',
        icon: <TrendingUp />,
        path: '/games/popular',
      },
      {
        text: 'Nuove Uscite',
        icon: <NewReleases />,
        path: '/games/recent',
      },
    ],
  },
  {
    text: 'Le Mie Liste',
    icon: <Star />,
    children: [
      {
        text: 'Giocati',
        icon: <CheckCircle />,
        path: '/my-games/played',
      },
      {
        text: 'In Corso',
        icon: <PlayArrow />,
        path: '/my-games/playing',
      },
      {
        text: 'Da Giocare',
        icon: <Pause />,
        path: '/my-games/want-to-play',
      },
      {
        text: 'Preferiti',
        icon: <Star />,
        path: '/my-games/favorites',
      },
    ],
  },
  {
    text: 'Amministrazione',
    icon: <AdminPanelSettings />,
    path: '/admin',
    roles: [UserRole.ADMIN, UserRole.MODERATOR],
  },
];

const Sidebar: React.FC = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const dispatch = useAppDispatch();
  const { sidebarOpen } = useAppSelector((state) => state.ui);
  const { user, isAuthenticated } = useAppSelector((state) => state.auth);
  
  const [openItems, setOpenItems] = React.useState<string[]>([]);

  const handleItemClick = (item: MenuItem) => {
    if (item.children) {
      const isOpen = openItems.includes(item.text);
      setOpenItems(isOpen 
        ? openItems.filter(text => text !== item.text)
        : [...openItems, item.text]
      );
    } else if (item.path) {
      navigate(item.path);
      if (window.innerWidth < 900) {
        dispatch(setSidebarOpen(false));
      }
    }
  };

  const isItemVisible = (item: MenuItem): boolean => {
    if (!item.roles) return true;
    if (!isAuthenticated) return false;
    return item.roles.includes(user?.role as UserRole);
  };

  const isItemActive = (path: string): boolean => {
    return location.pathname === path;
  };

  const renderMenuItem = (item: MenuItem, level = 0) => {
    if (!isItemVisible(item)) return null;

    const hasChildren = item.children && item.children.length > 0;
    const isOpen = openItems.includes(item.text);
    const isActive = item.path ? isItemActive(item.path) : false;

    return (
      <React.Fragment key={item.text}>
        <ListItem disablePadding>
          <ListItemButton
            onClick={() => handleItemClick(item)}
            selected={isActive}
            sx={{
              pl: 2 + level * 2,
              '&.Mui-selected': {
                backgroundColor: 'primary.main',
                color: 'primary.contrastText',
                '&:hover': {
                  backgroundColor: 'primary.dark',
                },
                '& .MuiListItemIcon-root': {
                  color: 'primary.contrastText',
                },
              },
            }}
          >
            <ListItemIcon sx={{ minWidth: 40 }}>
              {item.icon}
            </ListItemIcon>
            <ListItemText primary={item.text} />
            {hasChildren && (
              isOpen ? <ExpandLess /> : <ExpandMore />
            )}
          </ListItemButton>
        </ListItem>
        
        {hasChildren && (
          <Collapse in={isOpen} timeout="auto" unmountOnExit>
            <List component="div" disablePadding>
              {item.children!.map(child => renderMenuItem(child, level + 1))}
            </List>
          </Collapse>
        )}
      </React.Fragment>
    );
  };

  const handleDrawerClose = () => {
    dispatch(setSidebarOpen(false));
  };

  return (
    <Drawer
      variant="temporary"
      open={sidebarOpen}
      onClose={handleDrawerClose}
      sx={{
        width: DRAWER_WIDTH,
        flexShrink: 0,
        '& .MuiDrawer-paper': {
          width: DRAWER_WIDTH,
          boxSizing: 'border-box',
        },
      }}
    >
      <Toolbar />
      <Box sx={{ overflow: 'auto' }}>
        <List>
          {menuItems.map(item => renderMenuItem(item))}
        </List>
      </Box>
    </Drawer>
  );
};

export default Sidebar;
```

## 10.4 Componenti Game

### 10.4.1 components/game/GameCard.tsx

```typescript
import React from 'react';
import {
  Card,
  CardMedia,
  CardContent,
  CardActions,
  Typography,
  Button,
  Chip,
  Box,
  Rating,
  IconButton,
  Menu,
  MenuItem,
} from '@mui/material';
import {
  MoreVert as MoreVertIcon,
  PlayArrow,
  Pause,
  CheckCircle,
  Star,
} from '@mui/icons-material';
import { GameSummary, EsrbRating } from '@types/game';
import { ListType } from '@types/user';
import { useNavigate } from 'react-router-dom';
import { format } from 'date-fns';
import { it } from 'date-fns/locale';

interface GameCardProps {
  game: GameSummary;
  onAddToList?: (gameId: number, listType: ListType) => void;
  onRemoveFromList?: (gameId: number, listType: ListType) => void;
  showActions?: boolean;
  currentListType?: ListType;
}

const GameCard: React.FC<GameCardProps> = ({
  game,
  onAddToList,
  onRemoveFromList,
  showActions = true,
  currentListType,
}) => {
  const navigate = useNavigate();
  const [anchorEl, setAnchorEl] = React.useState<null | HTMLElement>(null);
  const open = Boolean(anchorEl);

  const handleMenuOpen = (event: React.MouseEvent<HTMLElement>) => {
    event.stopPropagation();
    setAnchorEl(event.currentTarget);
  };

  const handleMenuClose = () => {
    setAnchorEl(null);
  };

  const handleCardClick = () => {
    navigate(`/games/${game.id}`);
  };

  const handleAddToList = (listType: ListType) => {
    handleMenuClose();
    onAddToList?.(game.id, listType);
  };

  const handleRemoveFromList = () => {
    handleMenuClose();
    if (currentListType) {
      onRemoveFromList?.(game.id, currentListType);
    }
  };

  const getEsrbColor = (rating: EsrbRating) => {
    switch (rating) {
      case EsrbRating.EARLY_CHILDHOOD:
      case EsrbRating.EVERYONE:
        return 'success';
      case EsrbRating.EVERYONE_10_PLUS:
      case EsrbRating.TEEN:
        return 'warning';
      case EsrbRating.MATURE_17_PLUS:
      case EsrbRating.ADULTS_ONLY_18_PLUS:
        return 'error';
      default:
        return 'default';
    }
  };

  const getEsrbLabel = (rating: EsrbRating) => {
    switch (rating) {
      case EsrbRating.EARLY_CHILDHOOD:
        return 'EC';
      case EsrbRating.EVERYONE:
        return 'E';
      case EsrbRating.EVERYONE_10_PLUS:
        return 'E10+';
      case EsrbRating.TEEN:
        return 'T';
      case EsrbRating.MATURE_17_PLUS:
        return 'M';
      case EsrbRating.ADULTS_ONLY_18_PLUS:
        return 'AO';
      case EsrbRating.RATING_PENDING:
        return 'RP';
      default:
        return 'N/A';
    }
  };

  return (
    <Card
      sx={{
        height: '100%',
        display: 'flex',
        flexDirection: 'column',
        cursor: 'pointer',
        transition: 'transform 0.2s, box-shadow 0.2s',
        '&:hover': {
          transform: 'translateY(-4px)',
          boxShadow: 4,
        },
      }}
      onClick={handleCardClick}
    >
      <CardMedia
        component="img"
        height="200"
        image={game.imageUrl || '/placeholder-game.jpg'}
        alt={game.title}
        sx={{ objectFit: 'cover' }}
      />
      
      <CardContent sx={{ flexGrow: 1, pb: 1 }}>
        <Box display="flex" justifyContent="space-between" alignItems="flex-start" mb={1}>
          <Typography
            variant="h6"
            component="h3"
            sx={{
              fontWeight: 600,
              lineHeight: 1.2,
              overflow: 'hidden',
              textOverflow: 'ellipsis',
              display: '-webkit-box',
              WebkitLineClamp: 2,
              WebkitBoxOrient: 'vertical',
            }}
          >
            {game.title}
          </Typography>
          
          {showActions && (
            <IconButton
              size="small"
              onClick={handleMenuOpen}
              sx={{ ml: 1 }}
            >
              <MoreVertIcon />
            </IconButton>
          )}
        </Box>
        
        <Typography variant="body2" color="text.secondary" gutterBottom>
          {game.developer} • {game.publisher}
        </Typography>
        
        <Typography variant="body2" color="text.secondary" gutterBottom>
          {format(new Date(game.releaseDate), 'dd MMMM yyyy', { locale: it })}
        </Typography>
        
        <Box display="flex" justifyContent="space-between" alignItems="center" mt={1}>
          <Chip
            label={getEsrbLabel(game.esrbRating)}
            size="small"
            color={getEsrbColor(game.esrbRating) as any}
          />
          
          <Typography variant="h6" color="primary" fontWeight={600}>
            €{game.price.toFixed(2)}
          </Typography>
        </Box>
        
        {game.userRating && (
          <Box display="flex" alignItems="center" mt={1}>
            <Rating
              value={game.userRating}
              readOnly
              size="small"
              precision={0.1}
            />
            <Typography variant="body2" color="text.secondary" ml={1}>
              ({game.userRating.toFixed(1)})
            </Typography>
          </Box>
        )}
      </CardContent>
      
      <Menu
        anchorEl={anchorEl}
        open={open}
        onClose={handleMenuClose}
        onClick={(e) => e.stopPropagation()}
      >
        {currentListType ? (
          <MenuItem onClick={handleRemoveFromList}>
            Rimuovi dalla lista
          </MenuItem>
        ) : (
          [
            <MenuItem key="playing" onClick={() => handleAddToList(ListType.CURRENTLY_PLAYING)}>
              <PlayArrow sx={{ mr: 1 }} /> Sto giocando
            </MenuItem>,
            <MenuItem key="played" onClick={() => handleAddToList(ListType.PLAYED)}>
              <CheckCircle sx={{ mr: 1 }} /> Giocato
            </MenuItem>,
            <MenuItem key="want" onClick={() => handleAddToList(ListType.WANT_TO_PLAY)}>
              <Pause sx={{ mr: 1 }} /> Voglio giocare
            </MenuItem>,
            <MenuItem key="favorites" onClick={() => handleAddToList(ListType.FAVORITES)}>
              <Star sx={{ mr: 1 }} /> Preferiti
            </MenuItem>,
          ]
        )}
      </Menu>
    </Card>
  );
};

export default GameCard;
```

### 10.4.2 components/game/GameGrid.tsx

```typescript
import React from 'react';
import {
  Grid,
  Box,
  Typography,
  Pagination,
  FormControl,
  InputLabel,
  Select,
  MenuItem,
} from '@mui/material';
import { GameSummary } from '@types/game';
import { PagedResponse } from '@types/api';
import { ListType } from '@types/user';
import GameCard from './GameCard';
import LoadingSpinner from '@components/common/LoadingSpinner';
import ErrorMessage from '@components/common/ErrorMessage';

interface GameGridProps {
  data?: PagedResponse<GameSummary>;
  loading?: boolean;
  error?: string;
  onPageChange?: (page: number) => void;
  onSizeChange?: (size: number) => void;
  onAddToList?: (gameId: number, listType: ListType) => void;
  onRemoveFromList?: (gameId: number, listType: ListType) => void;
  currentListType?: ListType;
  showPagination?: boolean;
  emptyMessage?: string;
}

const GameGrid: React.FC<GameGridProps> = ({
  data,
  loading = false,
  error,
  onPageChange,
  onSizeChange,
  onAddToList,
  onRemoveFromList,
  currentListType,
  showPagination = true,
  emptyMessage = 'Nessun gioco trovato',
}) => {
  const handlePageChange = (event: React.ChangeEvent<unknown>, page: number) => {
    onPageChange?.(page - 1); // MUI Pagination è 1-based, API è 0-based
  };

  const handleSizeChange = (event: any) => {
    onSizeChange?.(event.target.value);
  };

  if (loading) {
    return <LoadingSpinner message="Caricamento giochi..." />;
  }

  if (error) {
    return (
      <ErrorMessage
        message={error}
        onRetry={() => window.location.reload()}
        fullWidth
      />
    );
  }

  if (!data || data.content.length === 0) {
    return (
      <Box
        display="flex"
        flexDirection="column"
        alignItems="center"
        justifyContent="center"
        minHeight={300}
        textAlign="center"
      >
        <Typography variant="h6" color="text.secondary" gutterBottom>
          {emptyMessage}
        </Typography>
        <Typography variant="body2" color="text.secondary">
          Prova a modificare i filtri di ricerca o torna più tardi.
        </Typography>
      </Box>
    );
  }

  return (
    <Box>
      <Grid container spacing={3}>
        {data.content.map((game) => (
          <Grid item xs={12} sm={6} md={4} lg={3} key={game.id}>
            <GameCard
              game={game}
              onAddToList={onAddToList}
              onRemoveFromList={onRemoveFromList}
              currentListType={currentListType}
            />
          </Grid>
        ))}
      </Grid>

      {showPagination && data.totalPages > 1 && (
        <Box
          display="flex"
          justifyContent="space-between"
          alignItems="center"
          mt={4}
          flexWrap="wrap"
          gap={2}
        >
          <FormControl size="small" sx={{ minWidth: 120 }}>
            <InputLabel>Elementi per pagina</InputLabel>
            <Select
              value={data.size}
              label="Elementi per pagina"
              onChange={handleSizeChange}
            >
              <MenuItem value={12}>12</MenuItem>
              <MenuItem value={24}>24</MenuItem>
              <MenuItem value={48}>48</MenuItem>
            </Select>
          </FormControl>

          <Box display="flex" alignItems="center" gap={2}>
            <Typography variant="body2" color="text.secondary">
              {data.totalElements} giochi totali
            </Typography>
            
            <Pagination
              count={data.totalPages}
              page={data.number + 1} // MUI Pagination è 1-based
              onChange={handlePageChange}
              color="primary"
              showFirstButton
              showLastButton
            />
          </Box>
        </Box>
      )}
    </Box>
  );
};

export default GameGrid;
```

## 10.5 Pagine Principali

### 10.5.1 pages/HomePage.tsx

```typescript
import React from 'react';
import {
  Container,
  Typography,
  Box,
  Grid,
  Card,
  CardContent,
  Button,
  Chip,
} from '@mui/material';
import {
  TrendingUp,
  NewReleases,
  Star,
  ArrowForward,
} from '@mui/icons-material';
import { useNavigate } from 'react-router-dom';
import { useAppSelector } from '@hooks/redux';
import {
  useGetPopularGamesQuery,
  useGetRecentGamesQuery,
  useGetRecommendedGamesQuery,
} from '@store/api/gameApi';
import GameGrid from '@components/game/GameGrid';
import SearchBar from '@components/common/SearchBar';
import { Helmet } from 'react-helmet-async';

const HomePage: React.FC = () => {
  const navigate = useNavigate();
  const { isAuthenticated, user } = useAppSelector((state) => state.auth);
  
  const { data: popularGames, isLoading: popularLoading } = useGetPopularGamesQuery({
    page: 0,
    size: 8,
  });
  
  const { data: recentGames, isLoading: recentLoading } = useGetRecentGamesQuery({
    page: 0,
    size: 8,
  });
  
  const { data: recommendedGames, isLoading: recommendedLoading } = useGetRecommendedGamesQuery(
    { page: 0, size: 8 },
    { skip: !isAuthenticated }
  );

  const handleSearch = (query: string) => {
    if (query.trim()) {
      navigate(`/games?search=${encodeURIComponent(query)}`);
    }
  };

  const stats = [
    {
      title: 'Giochi Totali',
      value: '10,000+',
      icon: <TrendingUp />,
      color: 'primary',
    },
    {
      title: 'Nuove Uscite',
      value: '150+',
      icon: <NewReleases />,
      color: 'secondary',
    },
    {
      title: 'Utenti Attivi',
      value: '5,000+',
      icon: <Star />,
      color: 'success',
    },
  ];

  return (
    <>
      <Helmet>
        <title>VideoGiochi Library - La tua libreria di videogiochi</title>
        <meta
          name="description"
          content="Scopri, organizza e gestisci la tua collezione di videogiochi con VideoGiochi Library."
        />
      </Helmet>
      
      <Container maxWidth="lg" sx={{ py: 4 }}>
        {/* Hero Section */}
        <Box textAlign="center" mb={6}>
          <Typography
            variant="h2"
            component="h1"
            gutterBottom
            sx={{
              fontWeight: 700,
              background: 'linear-gradient(45deg, #1976d2, #dc004e)',
              backgroundClip: 'text',
              WebkitBackgroundClip: 'text',
              WebkitTextFillColor: 'transparent',
            }}
          >
            VideoGiochi Library
          </Typography>
          
          <Typography
            variant="h5"
            color="text.secondary"
            gutterBottom
            sx={{ mb: 4 }}
          >
            Scopri, organizza e gestisci la tua collezione di videogiochi
          </Typography>
          
          <Box maxWidth={600} mx="auto" mb={4}>
            <SearchBar
              placeholder="Cerca giochi, sviluppatori, generi..."
              onSearch={handleSearch}
              fullWidth
            />
          </Box>
          
          {!isAuthenticated && (
            <Box display="flex" gap={2} justifyContent="center" flexWrap="wrap">
              <Button
                variant="contained"
                size="large"
                onClick={() => navigate('/register')}
              >
                Registrati Gratis
              </Button>
              <Button
                variant="outlined"
                size="large"
                onClick={() => navigate('/login')}
              >
                Accedi
              </Button>
            </Box>
          )}
        </Box>

        {/* Stats Section */}
        <Grid container spacing={3} mb={6}>
          {stats.map((stat, index) => (
            <Grid item xs={12} md={4} key={index}>
              <Card sx={{ textAlign: 'center', height: '100%' }}>
                <CardContent>
                  <Box
                    display="flex"
                    justifyContent="center"
                    mb={2}
                    color={`${stat.color}.main`}
                  >
                    {React.cloneElement(stat.icon, { fontSize: 'large' })}
                  </Box>
                  <Typography variant="h4" component="div" gutterBottom>
                    {stat.value}
                  </Typography>
                  <Typography variant="body1" color="text.secondary">
                    {stat.title}
                  </Typography>
                </CardContent>
              </Card>
            </Grid>
          ))}
        </Grid>

        {/* Recommended Games (solo per utenti autenticati) */}
        {isAuthenticated && (
          <Box mb={6}>
            <Box
              display="flex"
              justifyContent="space-between"
              alignItems="center"
              mb={3}
            >
              <Typography variant="h4" component="h2">
                Consigliati per te
              </Typography>
              <Button
                endIcon={<ArrowForward />}
                onClick={() => navigate('/games/recommendations')}
              >
                Vedi tutti
              </Button>
            </Box>
            
            <GameGrid
              data={recommendedGames}
              loading={recommendedLoading}
              showPagination={false}
              emptyMessage="Nessun gioco consigliato al momento"
            />
          </Box>
        )}

        {/* Popular Games */}
        <Box mb={6}>
          <Box
            display="flex"
            justifyContent="space-between"
            alignItems="center"
            mb={3}
          >
            <Typography variant="h4" component="h2">
              Giochi Popolari
            </Typography>
            <Button
              endIcon={<ArrowForward />}
              onClick={() => navigate('/games/popular')}
            >
              Vedi tutti
            </Button>
          </Box>
          
          <GameGrid
            data={popularGames}
            loading={popularLoading}
            showPagination={false}
            emptyMessage="Nessun gioco popolare al momento"
          />
        </Box>

        {/* Recent Games */}
        <Box>
          <Box
            display="flex"
            justifyContent="space-between"
            alignItems="center"
            mb={3}
          >
            <Typography variant="h4" component="h2">
              Nuove Uscite
            </Typography>
            <Button
              endIcon={<ArrowForward />}
              onClick={() => navigate('/games/recent')}
            >
              Vedi tutte
            </Button>
          </Box>
          
          <GameGrid
            data={recentGames}
            loading={recentLoading}
            showPagination={false}
            emptyMessage="Nessuna nuova uscita al momento"
          />
        </Box>
      </Container>
    </>
  );
};

export default HomePage;
```

Questo documento completa l'implementazione del frontend, fornendo componenti riutilizzabili, layout responsive e pagine funzionali che seguono le best practices di React e Material-UI.