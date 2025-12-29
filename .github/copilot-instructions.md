# Grimly - Productivity Apps Hub

## Project Overview
Single-page productivity app launcher with a minimalist dark theme. The entire application is contained in `index.html` - a self-contained HTML file with embedded CSS and JavaScript. No build process, no dependencies, just open and run.

## Architecture
- **Single-file monolith**: All HTML, CSS (380+ lines), and JavaScript (~200 lines) in `index.html`
- **Data persistence**: JSONBin.io cloud storage (no backend required)
- **State management**: Vanilla JavaScript with localStorage for API keys and bin IDs
- **Styling approach**: CSS custom properties for theming, no frameworks

## Key Patterns

### State & Data Flow
```javascript
// Global state pattern used throughout
let apps = [];          // In-memory app collection
let binId = '';         // JSONBin storage ID
let apiKey = '';        // User's API key from localStorage
let editingIndex = null; // Modal state (add vs edit mode)
```

Data flows: localStorage → load → JSONBin API → render DOM → user action → save → JSONBin API → localStorage

### API Integration
JSONBin.io is the only external dependency:
- `GET /v3/b/{binId}/latest` - Load apps with `X-Master-Key` header
- `POST /v3/b` - Create new bin with `X-Bin-Name: 'productivity-apps-hub'`
- `PUT /v3/b/{binId}` - Update existing apps

All API calls in `saveToJsonBin()` and `loadApps()` use async/await with error handling.

### Styling System
Custom properties in `:root` define the entire theme:
```css
--bg: #0a0a0a;           /* Main background */
--surface: #1a1a1a;      /* Card/modal background */
--surface-hover: #2a2a2a; /* Interactive states */
--border: #333333;       /* Borders everywhere */
--text: #ffffff;         /* Primary text */
--text-dim: #888888;     /* Secondary text */
--accent: #ffffff;       /* Hover/focus states */
--radius: 12px;          /* Consistent border radius */
```

Grid layout: `minmax(300px, 1fr)` for responsive cards without media query breakpoints.

### Modal Pattern
Single modal reused for add/edit:
```javascript
// Add mode: editingIndex = null
// Edit mode: editingIndex = number (array index)
openAddModal()  // Resets form, editingIndex = null
editApp(index)  // Populates form, editingIndex = index
saveApp(event)  // Checks editingIndex to decide push vs replace
```

## Common Tasks

### Adding Features to App Cards
App objects structure:
```javascript
{
  icon: '🎨',              // Single emoji
  title: 'App Name',
  description: 'Brief desc',
  url: 'https://...',
  tags: ['tag1', 'tag2']   // Array, can be empty
}
```

Cards render in `renderApps()` using template literals with `.map()`. The three buttons (Open, Edit, Delete) use inline `onclick` handlers.

### Styling New Components
1. Use existing CSS custom properties (never hardcode colors)
2. Follow button pattern: `.btn-primary` (white bg) or default (transparent with border)
3. Animations: `fadeIn` for content, `scaleIn` for modals, `slideDown` for headers
4. Spacing scale: 0.5rem increments (0.5rem, 1rem, 1.5rem, 2rem, etc.)

### Working with the Form
Form IDs match app object keys with "app" prefix:
- `#appIcon` → `app.icon`
- `#appTitle` → `app.title`
- `#appDescription` → `app.description`
- `#appUrl` → `app.url`
- `#appTags` → `app.tags` (comma-separated string, split on save)

## Testing & Debugging
- Open `index.html` directly in browser (no server needed)
- Check browser console for JSONBin API errors
- API key stored as `localStorage.jsonbinApiKey`
- Bin ID stored as `localStorage.appsHubBinId`
- Clear localStorage to reset: `localStorage.clear()`

## Design Philosophy
- **Brutalist minimalism**: No gradients, no shadows (except backdrop-filter), grayscale aesthetic
- **Emojis as icons**: Filtered grayscale (`filter: grayscale(100%)`)
- **Micro-interactions**: Subtle hover transforms (`translateY(-4px)`) and border color changes
- **Mobile-first grid**: Single column on mobile, auto-fill grid on desktop
- **Subtle background grid**: 50px grid at 5% opacity for texture

## Constraints
- Must remain single-file (no external JS/CSS files)
- No npm, no build step, no transpilation
- JSONBin API rate limits apply (free tier: 10k requests/month)
- localStorage dependency for persistence of API keys
