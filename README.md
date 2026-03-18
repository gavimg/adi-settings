# Gadagi Settings - Configuration Micro-frontend

A dedicated micro-frontend for application settings, user preferences, and system configuration within the Gadagi platform architecture.

## Overview

Gadagi Settings is a remote micro-frontend that provides comprehensive settings management, user preferences, and administrative configuration features. It's designed to be loaded dynamically by the Gadagi Host application using Webpack Module Federation.

## Features

- ⚙️ **Application Settings** - System-wide configuration
- 👤 **User Preferences** - Personalized user settings
- 🎨 **Theme Configuration** - Visual customization options
- 🔔 **Notification Settings** - Alert and email preferences
- 🛡️ **Security Settings** - Password and authentication options
- 📱 **Responsive Design** - Mobile-friendly interface

## Architecture

```
┌─────────────────────────────────────┐
│            Gadagi Settings             │
│  ┌─────────────┬─────────────────┐  │
│  │ User Profile │ System Settings │  │
│  │ - Personal   │ - Configuration │  │
│  │ - Preferences│ - Features      │  │
│  │ - Security   │ - Integrations  │  │
│  └─────────────┴─────────────────┘  │
│  ┌─────────────────────────────────┐  │
│  │      Theme Settings             │  │
│  │  - Color Scheme                 │  │
│  │  - Typography                   │  │
│  │  - Layout Options               │  │
│  └─────────────────────────────────┘  │
└─────────────────────────────────────┘
```

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm start

# Build for production
npm run build
```

## Development Server

- **Development URL**: http://localhost:3003
- **Remote Entry**: http://localhost:3003/remoteEntry.js

## Module Federation Configuration

### Remote Configuration

```javascript
// webpack.config.js
const ModuleFederationPlugin = require('@module-federation/enhanced');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'adiSettings',
      filename: 'remoteEntry.js',
      exposes: {
        './SettingsApp': './src/SettingsApp.tsx',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
        '@gadagi/types': { singleton: true },
        '@gadagi/design-system': { singleton: true },
      },
    }),
  ],
};
```

### Exposed Module

```typescript
// src/SettingsApp.tsx
import React from 'react';
import { UserProfile, SystemSettings, ThemeSettings } from './components';

const SettingsApp: React.FC = () => {
  return (
    <div style={{ padding: '2rem', background: '#faf5ff', minHeight: '100vh' }}>
      <h1>Settings Module</h1>
      <p>This is the adi-settings micro-frontend.</p>
      
      {/* Settings Components */}
      <UserProfile />
      <SystemSettings />
      <ThemeSettings />
    </div>
  );
};

export default SettingsApp;
```

## Components

### UserProfile

User profile and personal settings.

```tsx
import { UserProfile } from './components/UserProfile';

<UserProfile
  user={currentUser}
  onUpdate={handleProfileUpdate}
  onPasswordChange={handlePasswordChange}
/>
```

**Props:**
- `user: User` - Current user object
- `onUpdate: (updates: Partial<User>) => void` - Profile update handler
- `onPasswordChange: (oldPassword: string, newPassword: string) => void` - Password change handler
- `loading?: boolean` - Loading state

### SystemSettings

System-wide configuration options.

```tsx
import { SystemSettings } from './components/SystemSettings';

<SystemSettings
  settings={systemSettings}
  onUpdate={handleSettingsUpdate}
  permissions={userPermissions}
/>
```

**Props:**
- `settings: SystemSettings` - Current system settings
- `onUpdate: (updates: Partial<SystemSettings>) => void` - Update handler
- `permissions: UserPermissions` - User permission levels

### ThemeSettings

Theme and appearance configuration.

```tsx
import { ThemeSettings } from './components/ThemeSettings';

<ThemeSettings
  currentTheme={currentTheme}
  availableThemes={themes}
  onThemeChange={handleThemeChange}
/>
```

**Props:**
- `currentTheme: Theme` - Current active theme
- `availableThemes: Theme[]` - Available theme options
- `onThemeChange: (theme: Theme) => void` - Theme change handler

## Data Management

### Settings Service

```typescript
// src/services/settingsService.ts
import { UserPreferences, SystemSettings } from '@gadagi/types';

export class SettingsService {
  // Get user preferences
  static async getUserPreferences(userId: string): Promise<UserPreferences> {
    const response = await fetch(`/api/users/${userId}/preferences`);
    return response.json();
  }

  // Update user preferences
  static async updateUserPreferences(
    userId: string, 
    preferences: Partial<UserPreferences>
  ): Promise<UserPreferences> {
    const response = await fetch(`/api/users/${userId}/preferences`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(preferences),
    });
    return response.json();
  }

  // Get system settings
  static async getSystemSettings(): Promise<SystemSettings> {
    const response = await fetch('/api/settings/system');
    return response.json();
  }

  // Update system settings
  static async updateSystemSettings(
    settings: Partial<SystemSettings>
  ): Promise<SystemSettings> {
    const response = await fetch('/api/settings/system', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(settings),
    });
    return response.json();
  }

  // Change password
  static async changePassword(
    userId: string,
    oldPassword: string,
    newPassword: string
  ): Promise<void> {
    await fetch(`/api/users/${userId}/password`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ oldPassword, newPassword }),
    });
  }

  // Upload avatar
  static async uploadAvatar(userId: string, file: File): Promise<string> {
    const formData = new FormData();
    formData.append('avatar', file);
    
    const response = await fetch(`/api/users/${userId}/avatar`, {
      method: 'POST',
      body: formData,
    });
    
    const result = await response.json();
    return result.avatarUrl;
  }
}
```

### State Management

```tsx
// src/hooks/useSettings.ts
import { useState, useEffect } from 'react';
import { UserPreferences, SystemSettings } from '@gadagi/types';
import { SettingsService } from '../services/settingsService';

export const useSettings = (userId: string) => {
  const [userPreferences, setUserPreferences] = useState<UserPreferences | null>(null);
  const [systemSettings, setSystemSettings] = useState<SystemSettings | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchUserPreferences = async () => {
    try {
      const preferences = await SettingsService.getUserPreferences(userId);
      setUserPreferences(preferences);
    } catch (err) {
      setError('Failed to fetch user preferences');
    }
  };

  const fetchSystemSettings = async () => {
    try {
      const settings = await SettingsService.getSystemSettings();
      setSystemSettings(settings);
    } catch (err) {
      setError('Failed to fetch system settings');
    }
  };

  const updateUserPreferences = async (updates: Partial<UserPreferences>) => {
    try {
      const updated = await SettingsService.updateUserPreferences(userId, updates);
      setUserPreferences(updated);
      return updated;
    } catch (err) {
      setError('Failed to update preferences');
      throw err;
    }
  };

  const updateSystemSettings = async (updates: Partial<SystemSettings>) => {
    try {
      const updated = await SettingsService.updateSystemSettings(updates);
      setSystemSettings(updated);
      return updated;
    } catch (err) {
      setError('Failed to update system settings');
      throw err;
    }
  };

  useEffect(() => {
    setLoading(true);
    Promise.all([fetchUserPreferences(), fetchSystemSettings()])
      .finally(() => setLoading(false));
  }, [userId]);

  return {
    userPreferences,
    systemSettings,
    loading,
    error,
    updateUserPreferences,
    updateSystemSettings,
    refetch: () => {
      fetchUserPreferences();
      fetchSystemSettings();
    },
  };
};
```

## Features

### Theme Configuration

```tsx
// src/components/ThemeSettings.tsx
import { useTheme } from '@gadagi/design-system';

const ThemeSettings: React.FC = () => {
  const { theme, setTheme } = useTheme();

  const themes = [
    { id: 'light', name: 'Light', preview: '#ffffff' },
    { id: 'dark', name: 'Dark', preview: '#1a1a1a' },
    { id: 'auto', name: 'Auto', preview: 'linear-gradient(45deg, #ffffff 50%, #1a1a1a 50%)' },
  ];

  return (
    <div className="theme-settings">
      <h3>Theme Selection</h3>
      <div className="theme-options">
        {themes.map(th => (
          <div
            key={th.id}
            className={`theme-option ${theme === th.id ? 'active' : ''}`}
            onClick={() => setTheme(th.id as any)}
          >
            <div 
              className="theme-preview"
              style={{ background: th.preview }}
            />
            <span>{th.name}</span>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### Notification Settings

```tsx
// src/components/NotificationSettings.tsx
import { Input, Button } from '@gadagi/design-system';

interface NotificationSettingsProps {
  settings: NotificationSettings;
  onUpdate: (settings: Partial<NotificationSettings>) => void;
}

const NotificationSettings: React.FC<NotificationSettingsProps> = ({ 
  settings, 
  onUpdate 
}) => {
  const [localSettings, setLocalSettings] = useState(settings);

  const handleToggle = (key: keyof NotificationSettings) => {
    const updated = { ...localSettings, [key]: !localSettings[key] };
    setLocalSettings(updated);
    onUpdate(updated);
  };

  return (
    <div className="notification-settings">
      <h3>Notification Preferences</h3>
      
      <div className="setting-group">
        <label>
          <input
            type="checkbox"
            checked={localSettings.emailNotifications}
            onChange={() => handleToggle('emailNotifications')}
          />
          Email Notifications
        </label>
      </div>

      <div className="setting-group">
        <label>
          <input
            type="checkbox"
            checked={localSettings.pushNotifications}
            onChange={() => handleToggle('pushNotifications')}
          />
          Push Notifications
        </label>
      </div>

      <div className="setting-group">
        <label>
          <input
            type="checkbox"
            checked={localSettings.desktopNotifications}
            onChange={() => handleToggle('desktopNotifications')}
          />
          Desktop Notifications
        </label>
      </div>

      <div className="setting-group">
        <label htmlFor="notification-frequency">Frequency</label>
        <select
          id="notification-frequency"
          value={localSettings.frequency}
          onChange={(e) => onUpdate({ frequency: e.target.value as any })}
        >
          <option value="immediate">Immediate</option>
          <option value="hourly">Hourly</option>
          <option value="daily">Daily</option>
          <option value="weekly">Weekly</option>
        </select>
      </div>
    </div>
  );
};
```

### Security Settings

```tsx
// src/components/SecuritySettings.tsx
import { useState } from 'react';
import { Input, Button } from '@gadagi/design-system';

const SecuritySettings: React.FC = () => {
  const [passwordForm, setPasswordForm] = useState({
    currentPassword: '',
    newPassword: '',
    confirmPassword: '',
  });
  const [loading, setLoading] = useState(false);

  const handlePasswordChange = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (passwordForm.newPassword !== passwordForm.confirmPassword) {
      alert('Passwords do not match');
      return;
    }

    setLoading(true);
    try {
      await SettingsService.changePassword(
        userId,
        passwordForm.currentPassword,
        passwordForm.newPassword
      );
      alert('Password changed successfully');
      setPasswordForm({ currentPassword: '', newPassword: '', confirmPassword: '' });
    } catch (error) {
      alert('Failed to change password');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="security-settings">
      <h3>Security Settings</h3>
      
      <form onSubmit={handlePasswordChange} className="password-form">
        <div className="form-group">
          <label htmlFor="current-password">Current Password</label>
          <Input
            id="current-password"
            type="password"
            value={passwordForm.currentPassword}
            onChange={(e) => setPasswordForm(prev => ({
              ...prev,
              currentPassword: e.target.value
            }))}
            required
          />
        </div>

        <div className="form-group">
          <label htmlFor="new-password">New Password</label>
          <Input
            id="new-password"
            type="password"
            value={passwordForm.newPassword}
            onChange={(e) => setPasswordForm(prev => ({
              ...prev,
              newPassword: e.target.value
            }))}
            required
          />
        </div>

        <div className="form-group">
          <label htmlFor="confirm-password">Confirm New Password</label>
          <Input
            id="confirm-password"
            type="password"
            value={passwordForm.confirmPassword}
            onChange={(e) => setPasswordForm(prev => ({
              ...prev,
              confirmPassword: e.target.value
            }))}
            required
          />
        </div>

        <Button type="submit" disabled={loading}>
          {loading ? 'Changing...' : 'Change Password'}
        </Button>
      </form>
    </div>
  );
};
```

## Integration

### Host Integration

The SettingsApp component is designed to be loaded by the ADI Host:

```typescript
// In ADI Host
const SettingsApp = React.lazy(() => import('adiSettings/SettingsApp'));

// Route configuration
<Route path="/settings" element={
  <Suspense fallback={<div>Loading Settings...</div>}>
    <SettingsApp />
  </Suspense>
} />
```

### Shared Dependencies

The micro-frontend shares dependencies with the host:

- React & React DOM
- @gadagi/types
- @gadagi/design-system
- React Router DOM

## Styling

### Theme Integration

```tsx
// Use design system tokens
import { colors, spacing } from '@gadagi/design-system';

const settingsStyles = {
  container: {
    padding: spacing[4],
    backgroundColor: colors.neutral[50],
  },
  section: {
    backgroundColor: colors.neutral[100],
    borderRadius: '8px',
    padding: spacing[3],
    marginBottom: spacing[3],
  },
};
```

### Custom Styles

```css
/* src/styles/Settings.css */
.settings-container {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
  max-width: 800px;
  margin: 0 auto;
}

.settings-section {
  background: white;
  border-radius: 8px;
  padding: 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.settings-section h3 {
  margin-bottom: 1.5rem;
  color: #333;
  border-bottom: 2px solid #eee;
  padding-bottom: 0.5rem;
}

.setting-group {
  margin-bottom: 1.5rem;
}

.setting-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.theme-options {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.theme-option {
  text-align: center;
  cursor: pointer;
  padding: 1rem;
  border-radius: 8px;
  border: 2px solid transparent;
  transition: all 0.2s ease;
}

.theme-option:hover {
  border-color: #4a3fb5;
}

.theme-option.active {
  border-color: #4a3fb5;
  background: rgba(74, 63, 181, 0.1);
}

.theme-preview {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin: 0 auto 0.5rem;
  border: 2px solid #eee;
}

.password-form {
  max-width: 400px;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}
```

## Testing

### Unit Tests

```bash
# Run tests
npm test

# Run with coverage
npm run test:coverage
```

### Component Tests

```typescript
// src/components/__tests__/ThemeSettings.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ThemeSettings } from '../ThemeSettings';

describe('ThemeSettings', () => {
  it('renders theme options', () => {
    render(<ThemeSettings currentTheme="light" onThemeChange={jest.fn()} />);
    
    expect(screen.getByText('Light')).toBeInTheDocument();
    expect(screen.getByText('Dark')).toBeInTheDocument();
    expect(screen.getByText('Auto')).toBeInTheDocument();
  });

  it('calls onThemeChange when theme is selected', () => {
    const onThemeChange = jest.fn();
    render(<ThemeSettings currentTheme="light" onThemeChange={onThemeChange} />);
    
    fireEvent.click(screen.getByText('Dark'));
    expect(onThemeChange).toHaveBeenCalledWith('dark');
  });
});
```

## Performance Optimization

### Settings Caching

```tsx
// src/hooks/useSettingsCache.ts
import { useQuery } from 'react-query';
import { SettingsService } from '../services/settingsService';

export const useSettingsCache = (userId: string) => {
  return useQuery(
    ['userSettings', userId],
    () => SettingsService.getUserPreferences(userId),
    {
      staleTime: 30 * 60 * 1000, // 30 minutes
      cacheTime: 60 * 60 * 1000, // 1 hour
    }
  );
};
```

### Form Optimization

```tsx
// Memoize form components
import React, { memo } from 'react';

const OptimizedSettingsForm = memo(({ settings, onChange }) => {
  return (
    <form className="settings-form">
      {/* Form fields */}
    </form>
  );
});
```

## Deployment

### Environment Configuration

```typescript
// config/environment.ts
export const config = {
  apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:4000',
  uploadUrl: process.env.REACT_APP_UPLOAD_URL || 'http://localhost:4000/upload',
  environment: process.env.NODE_ENV || 'development',
};
```

### Build Configuration

```bash
# Production build
npm run build

# Preview build
npm run preview
```

## Troubleshooting

### Common Issues

1. **Settings Not Saving**
   - Check API endpoints
   - Verify user permissions
   - Check network connectivity

2. **Theme Not Applying**
   - Verify theme provider setup
   - Check CSS loading
   - Ensure proper token usage

3. **Profile Upload Issues**
   - Check file size limits
   - Verify upload endpoint
   - Check file format support

## Contributing

1. Follow component patterns
2. Add tests for new settings
3. Update documentation
4. Ensure accessibility compliance

## License

MIT © Gadagi Team
